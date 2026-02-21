# Claude Code Agent Teams 완전 정리

[출처](https://dev.classmethod.jp/articles/claude-code-agent-teams-guide/ )
  

## **Agent Teams란:**
2026년 2월 5일, Anthropic의 Claude Opus 4.6 출시와 함께 함께 공개된 **실험적 기능(Research Preview)**이다. 기존 Claude Code에는 Subagent(서브에이전트)만 존재했으나, Agent Teams는 여러 Claude 인스턴스가 팀으로 협력하며 병렬로 작업을 수행하는 구조다.

핵심 구성요소는 다음과 같다.

- **Team Lead**: 전체를 코디네이트하는 리더 1명
- **Teammates**: 각자 독립된 컨텍스트로 작업하는 멤버들
- **공유 태스크 리스트**: `Ctrl+T`로 진행 상황 확인 (pending → in_progress → completed)
- **Mailbox**: 멤버 간 직접 메시지 교환 (SendMessage) 가능

---

## **Subagent vs Agent Teams 비교:**

| 항목 | Subagent | Agent Teams |
|------|----------|------------|
| 컨텍스트 | 독자 윈도우, 결과를 caller에 반환 | 독자 윈도우, 완전히 독립 |
| 커뮤니케이션 | 메인 에이전트에게만 보고 | 멤버 간 직접 메시지 교환 가능 |
| 코디네이션 | 메인이 집중 관리 | 공유 태스크 리스트로 자율 조정 |
| 토큰 비용 | 낮음 | 높음 (각 멤버가 독립된 Claude 인스턴스) |
| 적합한 상황 | 결과만 필요한 집중 태스크 | 논의·협조가 필요한 복잡한 작업 |

**사용 구분 기준**: 결과만 필요하거나 단일 관점으로 완결되는 작업이라면 Subagent로 충분하다. 복수 관점으로 병렬 진행하거나 멤버 간 협의가 필요한 경우(병렬 코드 리뷰, 가설 검증 디버깅, 멀티 모듈 개발 등)에는 Agent Teams가 적합하다.

---

## **셋업 방법:**
환경변수 또는 `~/.claude/settings.json`에 다음을 설정한다.

```bash
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

매번 환경변수를 붙이지 않으려면 `~/.claude/settings.json`에 아래처럼 설정한다.

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "model": "sonnet",
  "language": "Japanese",
  "teammateMode": "in-process"
}
```

표시 모드는 두 가지가 있다.

| 항목 | in-process 모드 | split panes 모드 |
|------|----------------|-----------------|
| 동작 방식 | 메인 터미널 내에서 전체 멤버 동작 | 멤버마다 독립 pane 실행 |
| 필요 환경 | 없음 (기본값) | tmux 또는 iTerm2 필수 |
| 멤버 전환 | `Shift+Up` / `Shift+Down` | pane 간 이동 |
| 지원 터미널 | 모두 | tmux / iTerm2만 (Ghostty, VS Code, Windows Terminal 비지원) |

split panes 모드에는 `send-keys` / `send-code` 관련 기존 버그가 보고되어 있다. in-process 모드에서도 공유 태스크 리스트, Mailbox, 병렬 실행이 동일하게 동작하므로, tmux 없이도 기능의 본질을 충분히 테스트할 수 있다.

**전제 조건**: Claude MAX 또는 Pro 플랜이 실질적으로 필요하다. 5인 팀 기준 토큰 비용은 약 **5~7배** 증가한다.

---

## **실험 결과 요약:**

### **실험 1 — 팀 생성 및 멤버 추가**
다음과 같은 프롬프트로 팀을 생성한다.

```
3인 팀을 만들어줘. 역할은 architect(설계), implementer(구현), reviewer(리뷰)로 나눠줘.
```

Team Lead가 팀을 생성하고 Teammates가 순서대로 spawn된다. 터미널에서는 `@architect❯`, `@implementer❯`, `@reviewer❯` 형식으로 각 멤버의 발화를 확인할 수 있다. `Shift+Down`으로 멤버 선택 패널을 열면 가동 시간, 툴 사용 횟수, 소비 토큰 수를 실시간으로 확인할 수 있다.

**중요한 발견**: `settings.json`에서 Sonnet을 지정해도 **Teammates에는 Opus 4.6이 자동 할당**된다. 아래는 실제 생성된 `config.json` 발췌이다.

```json
[
  { "name": "team-lead",    "model": "claude-sonnet-4-6" },
  { "name": "architect",    "model": "claude-opus-4-6" },
  { "name": "implementer",  "model": "claude-opus-4-6" },
  { "name": "reviewer",     "model": "claude-opus-4-6" }
]
```

토큰 비용이 5~7배 증가하는 주요 원인이 바로 이 부분이다.
  
  
### **실험 2 — 공유 태스크 리스트와 태스크 할당**

```
이 리포지토리에서 3개의 태스크를 만들고 각 멤버에게 할당해줘.
- architect: 리포지토리 구조 파악 후 개요 정리
- implementer: 주요 파일 목록화 및 의존 관계 정리
- reviewer: 코드 개선 제안 3건 작성
```

태스크 번호 체계는 다음과 같다. `#1~#3`은 팀 시작 시 자동 생성되는 내부 태스크(`_internal: true`)이며, 사용자가 생성한 태스크는 `#4`부터 채번된다. `Ctrl+T`로 태스크 리스트를 확인할 수 있으며, 상태 반영에 1~2초 지연이 있으므로 재차 눌러서 확인한다.
  
  
### **실험 3 — 병렬 실행 및 결과 취합**

```
보안·성능·테스트 관점으로 병렬로 코드 리뷰해줘.
```

각 멤버가 담당 관점에서 코드를 분석하고, 이후 리더에게 "결과를 정리해줘"라고 요청하면 우선순위가 붙은 단일 요약 결과를 얻을 수 있다. 관점별로 분리한 덕분에 지적 중복이 줄고, 병렬 실행 후 취합까지 원활하게 동작한다.
  
  
### **실험 4 — Mailbox의 자율 발화 (빈 리포지토리 실험)**
가장 흥미로운 발견이다. "서로 상의하면서"라고 명시하지 않아도, **문제를 감지한 멤버가 자율적으로 Mailbox(SendMessage)를 사용**하는 장면이 관찰되었다.

흐름은 다음과 같다.

1. `@reviewer`가 "리포지토리가 비어 있음"을 발견해 Team Lead에 보고
2. Team Lead가 직접 확인 후 공(空)임을 파악
3. Team Lead가 전 멤버에게 SendMessage로 "방침 변경" 통보
4. 각 멤버가 "빈 리포지토리 사실 보고 + 역할 범위 내 제안"으로 전환해 작업 계속 → 최종 취합본 출력

**전제가 중간에 바뀌어도, 멤버가 자율적으로 보고하고, 리더가 방침을 재수립하고, 전원이 그에 따라 움직이는 흐름이 별도 지시 없이 자동으로 발생**했다.

---

## **실험에서 발견한 예상 밖 동작들:**

| 발견 | 내용 |
|------|------|
| Teammates 모델 자동 할당 | settings.json에서 Sonnet을 지정해도 Teammates에는 Opus 4.6이 자동 적용됨. 비용 증가의 주원인. |
| 내부 태스크 채번 | 팀 시작 시 #1~#3은 내부 태스크로 자동 생성. 사용자 태스크는 #4부터 시작. |
| Mailbox 자율 발화 | "상의해"라는 지시 없이도 문제 감지 시 멤버가 자율적으로 Team Lead에 SendMessage로 보고하고 방침 변경이 전원에게 전파됨. |
| **CLAUDE.md는 Teammates도 읽는다** | 워크스페이스의 `CLAUDE.md`에 기재한 규칙(예: 태스크 완료 시 Toggl API 호출)을 **Team Lead뿐 아니라 Teammates도 실행하려고 시도**함. 실제로 Teammate 1명이 Toggl 등록에서 400 에러를 발생시키고 "Toggl 설정을 확인해주세요"라고 보고한 사례가 있었음. |

**CLAUDE.md 대응책**: "이 규칙은 Team Lead만 실행"이라고 명시하거나, Agent Teams 사용 시에는 해당 규칙을 제거하는 방식을 검토한다. 반대로, Teammates에 특정 프로토콜을 실행시키고 싶다면 CLAUDE.md에 명시하는 것으로 구현할 수 있다는 발견이기도 하다.

---

## **제한 사항 및 운용 주의점:**

| 항목 | 내용 |
|------|------|
| 토큰 비용 | 5~7배 증가. 짧은 태스크를 과도하게 몰아넣지 말 것. |
| 세션 재개 | `/resume`, `/rewind` **미지원**. 작업 중단·재개를 계획적으로 진행해야 함. |
| 태스크 상태 | 반영에 지연이 생길 수 있음. `Ctrl+T`를 다시 누르거나 잠시 기다린 후 확인. |
| 종료 시간 | 셧다운에 15~20초 걸리는 경우가 있음. |
| 컨텍스트 전달 | 멤버는 리더의 대화 이력을 이어받지 않으므로, **충분한 컨텍스트를 명시적으로 전달**해야 함. |
| 파일 충돌 방지 | 각 멤버에게 **서로 다른 파일셋**을 담당시켜 충돌을 피할 것. |
| CLAUDE.md 주의 | Teammates도 CLAUDE.md를 읽음. "Team Lead만 실행"이라고 명시하거나 Agent Teams 사용 시 해당 규칙 제거를 검토할 것. |  