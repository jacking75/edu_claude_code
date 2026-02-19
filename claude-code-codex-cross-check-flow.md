# Claude Code × Codex 크로스체크 개발 플로 정리

[출처](https://zenn.dev/tokium_dev/articles/0ef3f807d67e7c )  
  

## **개요 및 배경:**
Claude Code나 Codex 같은 AI 에이전트를 단독으로 사용하면 같은 세션에서 반복해서 확인을 요청해도 동일한 시점의 답변이 돌아온다는 문제가 있다. 구체적인 과제는 다음과 같다.

- 몇 번을 물어도 항상 같은 관점의 답변이 나온다
- 국소해(local optimum)에 빠져도 알아채지 못한다
- 다른 시점의 피드백이 필요하다

인간 개발에서도 자신이 작성한 코드를 스스로 리뷰하는 것보다 다른 사람에게 리뷰를 받는 것이 놓친 부분을 더 잘 발견할 수 있다. 이 원리를 AI에게도 적용해, **다른 AI에게 세컨드 오피니언을 구함으로써 사양·구현상의 누락을 검출**한다는 것이 이 플로의 핵심 아이디어다.

  
## **전체 개발 플로:**

```
Jira (티켓 관리 툴)
    ↓ Atlassian MCP Server
Claude Code (구현·셀프 리뷰)
    ↓ tmux-sender
Codex (크로스체크)
    ↓
피드백 반영·Pull Request 작성
    ↓
GitHub Copilot 자동 리뷰
    ↓
인간 리뷰
```

개발 환경은 터미널 에뮬레이터 iTerm2 + 터미널 멀티플렉서 tmux 구성이다.


## **MCP 서버로 티켓 정보를 취득한다:**
개발의 출발점은 Jira 티켓이다. Atlassian MCP Server를 사용하면 Claude Code에서 직접 티켓 정보를 가져올 수 있다. 가능한 조작 예시는 다음과 같다.

- 티켓 번호를 지정해서 요건 취득
- 담당자로 필터링해서 자신의 태스크 목록 표시
- 상태로 필터링 (TODO 티켓에서 구현 계획 수립, 리뷰 중인 티켓 리뷰 등)

```
「PROJ-123의 요건을 확인하고 구현해줘」
「내가 어사인된 TODO 티켓을 보여줘」
「상태가 리뷰 중인 티켓을 리뷰해줘」
```

  
## **tmux-sender Skill — 다른 페인에 커맨드를 전송한다:**
Claude Code에는 커스텀 커맨드를 정의할 수 있는 **Skills** 기능이 있다. 글로벌 설정은 `~/.claude/skills/` 하위에 Markdown 파일을 두면 된다.

**`~/.claude/skills/tmux-sender/SKILL.md`**:

```markdown
---
name: tmux-sender
description: tmux의 다른 페인에 커맨드를 전송한다. 「페인에서 실행해」「tmux로 전송」 등의 요청에 사용.
allowed-tools: Bash(tmux:*)
---

# tmux 커맨드 전송 스킬

## 사용법

tmux의 페인에 커맨드를 전송하고 실행하는 경우:

tmux send-keys -t <페인 번호> '<커맨드>' Enter

## 절차

1. tmux list-panes로 페인 목록 확인
2. tmux send-keys -t <페인 번호> '<커맨드>' Enter로 전송·실행
```

`allowed-tools: Bash(tmux:*)`로 이 Skill이 tmux 커맨드만 실행할 수 있도록 제한하는 것이 중요하다. 이 제한을 설정하지 않으면 "페인1에서 npm run dev를 실행해"와 같은 지시가 그대로 통과되어버린다.


## **리뷰 관점을 Skill로 정의한다:**
별도 페인의 Codex에 리뷰를 의뢰하기 전에 먼저 Claude Code 자신이 셀프 리뷰를 수행한다. 리뷰 관점도 Skill로 정의해 품질을 안정시킨다.

**`~/.claude/skills/review/SKILL.md`**:

```markdown
---
name: review
description: 코드 리뷰를 수행한다. 「리뷰해줘」「코드 리뷰」 등의 요청에 사용.
allowed-tools: Read, Grep, Glob, Bash(git diff:*), Bash(git log:*), Bash(git show:*)
---

## 리뷰 관점

이하의 관점으로 코드를 확인한다

1. 가독성
   - 변수명·함수명은 이해하기 쉬운가
   - 코드의 의도가 전달되는가

2. 버그 가능성
   - 엣지 케이스 고려 누락이 없는가
   - null이나 undefined 처리는 괜찮은가

3. 퍼포먼스
   - 명백히 비효율적인 처리가 없는가
   - 불필요한 재계산·리렌더링이 없는가

4. 보안
   - 입력값 검증은 적절한가
   - 기밀 정보 처리에 문제가 없는가

5. 테스트
   - 테스트가 작성되어 있는가
   - 테스트 커버리지는 충분한가
```

  
## **Claude Code에서 Codex를 호출한다:**
tmux 페인 구성 예시:

```
+------------------+------------------+
|                  |                  |
|   Claude Code    |     Codex        |
|    (페인 0)       |    (페인 1)       |
|                  |                  |
+------------------+------------------+
```

Claude Code에서 구현과 셀프 리뷰를 마친 뒤, 다음과 같이 지시한다.

```
「페인1에서, 지금 변경 내용을 Codex로 리뷰해줘」
```

Claude Code가 `tmux send-keys`로 Codex에 프롬프트를 전송하고, Codex가 리뷰를 시작한다.


## **Codex 쪽에도 Skill을 설정한다:**  
Codex에도 Skills 기능이 있다. `~/.codex/skills/` 하위에 Markdown 파일을 두면 Claude Code와 동일하게 커스텀 커맨드를 정의할 수 있다.

**`~/.codex/skills/tmux-sender/SKILL.md`**:

```markdown
---
name: tmux-sender
description: tmux의 다른 페인에 커맨드를 전송한다. 「페인에서 실행해」「Claude Code에 의뢰해」 등의 요청에 사용.
metadata:
  short-description: tmux 페인 간 커맨드 전송
---
```

**`~/.codex/skills/review/SKILL.md`**:

```markdown
---
name: review
description: 코드 리뷰를 수행한다. 「리뷰해줘」「코드 리뷰」 등의 요청에 사용.
metadata:
  short-description: 코드 리뷰용 스킬
---
```

양방향 설정으로 다음과 같은 왕복 상호작용이 가능해진다.

```
Claude Code → Codex 「이 코드를 리뷰해줘」
    ↓
Codex → Claude Code 「에러 핸들링이 부족하다. 수정해줘」
    ↓
Claude Code → Codex 「수정했다. 재리뷰해줘」
```


## **현 시점의 제한 사항:**
`Codex → Claude Code` 방향의 요청은 `tmux send-keys`로 프롬프트를 전송해도 **자동으로 실행되지 않는다.** Claude Code는 인터랙티브한 TUI로 독자적인 키 입력 처리를 하고 있기 때문에, 텍스트는 입력 버퍼에 도달하지만 실행은 수동으로 해야 한다. 프롬프트를 직접 타이핑하는 수고는 덜 수 있지만, 전송 후 수신 측 페인에서 확인·실행하는 운용이 된다.

반면 `Claude Code → Codex` 방향은 자동으로 실행된다.

*(2026년 1월 시점의 검증 결과)*

---

## **실제로 얻은 효과:**

- **사양 고려 누락 검출**: Claude Code가 놓친 엣지 케이스를 Codex가 지적해주는 경우가 있다
- **구현 방침의 타당성 확인**: "이 설계로 정말 괜찮은가"라는 불안이 줄었다
- **다른 시점의 피드백**: 같은 AI에게 다시 묻는 것보다 다른 AI에게 물었을 때 더 다각적인 지적이 나온다

단, 두 AI 모두 같은 부분을 놓칠 가능성은 있다. 복잡한 비즈니스 로직의 오류 등은 잡아내지 못하는 경우도 있으므로, **최종적으로는 인간이 책임지고 리뷰해야 한다.**   