# Claude Code 잘 모르는 커맨드 & 숨겨진 기술 20선【난이도별】

## **초급편（바로 쓸 수 있는 편리한 기술 5선）**

**① `/btw` — 작업을 중단하지 않고 곁다리 질문**

Claude가 작업 중이더라도, `/btw`를 사용하면 **대화 기록을 오염시키지 않고 질문**할 수 있다.

```
/btw 아까 그 설정 파일 이름이 뭐였지?
```

**포인트:**

- 대화 기록에 남지 않기 때문에 컨텍스트를 소비하지 않는다
- Claude가 작업 중이더라도 끼어들어 사용할 수 있다
- 프롬프트 캐시를 재사용하므로 추가 비용이 최소화된다
- `Space` / `Enter` / `Esc`로 답변을 닫고 작업으로 복귀

> 서브에이전트의 반대 개념으로, "툴은 사용할 수 없지만 대화 전체가 보인다"는 것이 특징이다.

---

**② `!` 프리픽스 — Claude를 거치지 않고 직접 셸 실행**

프롬프트 맨 앞에 `!`를 붙이면, **Claude를 거치지 않고 직접 Bash 커맨드를 실행**할 수 있다.

```
! npm test
! git status
! ls -la
```

**포인트:**

- 커맨드의 출력은 대화 컨텍스트에 추가된다（Claude도 볼 수 있다）
- `Ctrl+B`로 백그라운드 실행으로 전환 가능
- `Tab`으로 과거의 `!` 커맨드를 자동완성
- `Escape` · `Backspace` · `Ctrl+U`（빈 프롬프트 상태）로 빠져나감

---

**③ `Esc` × 2회 — 체크포인트로 되감기**

`Esc`를 **빠르게 2번** 누르면, **되감기 메뉴**가 열린다.

```
조작 목록：
├── Restore code and conversation（코드와 대화 모두 되감기）
├── Restore conversation（대화만 되감기, 코드는 유지）
├── Restore code（코드만 되감기, 대화는 유지）
└── Summarize from here（선택 지점 이후를 요약으로 압축）
```

**포인트:**

- Claude의 파일 편집 툴을 통한 변경은 모두 자동으로 추적된다
- 세션을 넘어도 체크포인트는 유지된다（30일간）
- "Summarize from here"는 `/compact`와 달리, **중간 지점부터** 압축할 수 있다

---

**④ `/context` — 컨텍스트 사용량을 "가시화"**

```
/context
```

컨텍스트 윈도우의 사용 상황을 **컬러 그리드**로 시각화한다.

**포인트:**

- 어떤 툴이 컨텍스트를 압박하고 있는지 한눈에 파악 가능
- 메모리 비대화 경고도 표시된다
- 남은 용량이 적어지면 최적화 제안이 나온다

---

**⑤ `/diff` — Git의 차이점을 인터랙티브하게 열람**

```
/diff
```

**포인트:**

- `←` / `→`：현재 git diff와 Claude 턴별 diff를 전환
- `↑` / `↓`：파일 간 브라우징
- 커밋 전 최종 확인에 최적

---

## **중급편（생산성이 올라가는 기술 5선）**

**⑥ `ultrathink` — 단어 하나로 최대 깊이의 추론을 발동**

프롬프트 어딘가에 **`ultrathink`**라고 쓰기만 하면, 그 턴에 한해 **effort=high**가 된다.

```
이 버그의 근본 원인을 파악해줘 ultrathink
```

**포인트（Opus 4.6 / Sonnet 4.6 한정）:**

- `/effort`로 항구적으로 변경할 필요 없이, **1회 한정**의 깊은 추론이 가능하다
- "think", "think hard", "think more"는 **효과 없음**（일반 프롬프트로 처리된다）
- Skill 파일에 `ultrathink`라고 쓰면, 해당 스킬 실행 시 항상 깊은 추론이 작동한다

---

**⑦ `/loop` — 정기 실행 × cron 스케줄링**

```
/loop 5m 배포가 완료됐는지 확인하고 결과를 알려줘
```

**포인트:**

- `5m`（5분）, `2h`（2시간）, `1d`（1일）등의 간격 지정이 가능하다
- 생략하면 기본값 10분 간격
- 다른 스킬 호출도 가능：`/loop 20m /review-pr 1234`
- 최대 3일 후 자동 만료（무한 루프 방지）
- 세션이 종료되면 모든 태스크도 소멸（지속화하려면 GitHub Actions나 Desktop scheduled tasks를 활용）

**원샷 리마인더도 자연어로 설정 가능:**

```
45분 후에 인테그레이션 테스트가 통과했는지 확인해줘
```

---

**⑧ `opusplan` — 플래닝은 Opus, 실행은 Sonnet의 자동 전환**

```
/model opusplan
```

| 단계 | 사용 모델 | 메리트 |
|------|-----------|--------|
| Plan Mode | Opus 4.6 | 고품질 설계·추론 |
| 실행 모드 | Sonnet 4.6 | 고속·저비용 코드 생성 |

**포인트:**

- 특별한 설정 필요 없이, 모델 선택에서 `opusplan`을 선택하기만 하면 된다
- "Opus의 두뇌 × Sonnet의 속도"의 좋은 부분만 취한다

---

**⑨ `/batch` — 대규모 변경을 병렬 에이전트로 일괄 실행**

```
/batch src/ 내의 모든 컴포넌트를 Solid에서 React로 마이그레이션
```

**포인트:**

- 코드베이스를 조사 → 5~30개의 독립된 작업 단위로 분해 → 승인을 요청한다
- 승인 후, **작업 단위마다 1개의 백그라운드 에이전트**를 Git worktree로 시작한다
- 각 에이전트가 구현 → 테스트 → PR 생성까지 자동으로 진행
- Git 리포지토리가 필수

---

**⑩ `Shift+Tab` — 3가지 모드를 순식간에 전환**

`Shift+Tab`（환경에 따라서는 `Alt+M`）을 누를 때마다：

```
Normal Mode → Auto-Accept Mode（⏵⏵） → Plan Mode（⏸） → Normal Mode ...
```

**포인트:**

- **Auto-Accept Mode**：모든 편집을 자동 승인（신뢰할 수 있는 태스크 용）
- **Plan Mode**：읽기 전용으로 분석（코드 탐색·설계 시 안전）
- 상태 바에 아이콘이 표시되므로 현재 모드를 한눈에 알 수 있다

---

## **상급편（파워 유저를 위한 기술 5선）**

**⑪ `/simplify` — 3 에이전트 병렬 코드 리뷰**

```
/simplify 메모리 효율에 주목해서
```

**포인트:**

- 최근 변경한 파일을 대상으로, **3개의 리뷰 에이전트를 병렬로 시작**한다
- 코드 재사용·품질·효율성의 관점에서 리뷰
- 결과를 통합하여 **자동 수정**까지 실행
- 인수 없이 실행하면 범용 리뷰, 인수가 있으면 특정 관점에 집중

---

**⑫ `--worktree` — Git worktree로 완전히 격리된 병렬 개발**

```
# 이름 지정 worktree로 시작
claude --worktree feature-auth

# 다른 worktree에서 동시에 작업
claude --worktree bugfix-123

# 이름을 생략하면 랜덤 생성
claude --worktree
```

**포인트:**

- `.claude/worktrees/<name>/`에 worktree가 생성된다
- 브랜치는 `worktree-<name>`으로 자동 생성
- 변경 없이 종료하면 → 자동 클린업
- 변경 후 종료하면 → keep/remove를 선택
- 서브에이전트에도 `isolation: worktree`로 적용 가능

---

**⑬ `/export`와 `/copy` — 대화를 통째로 반출**

```
/export report.txt     # 파일에 출력
/export                # 클립보드에 복사 or 파일 저장을 선택
/copy                  # 마지막 답변을 복사（코드 블록이 있으면 선택 UI）
```

**포인트:**

- `/copy`는 코드 블록이 여러 개 있을 경우, **인터랙티브 피커**로 선택 가능하다
- 문서 작성이나 Slack 공유에 편리하다

---

**⑭ Chrome 연동 — 브라우저 조작을 터미널에서**

```
claude --chrome
```

```
localhost:3000을 열어서, 로그인 폼에 유효하지 않은 데이터를 전송하고,
에러 메시지가 올바르게 표시되는지 확인해줘
```

**포인트:**

- 로그인된 인증 상태를 공유한다（Google Docs, Gmail, Notion 등도 그대로 조작 가능）
- 콘솔 로그 읽기 → 버그 수정 루프가 1 세션으로 완결
- GIF 녹화도 가능：`체크아웃 플로우의 데모 GIF를 녹화해줘`
- `/chrome`으로 기본값 활성화, 연결 상태 확인, 재연결이 가능

---

**⑮ `/fast` — Opus 4.6을 2.5배 고속화**

```
/fast
```

| 항목 | 일반 Opus 4.6 | Fast Mode |
|------|---------------|-----------|
| 속도 | 표준 | **2.5배** |
| 입력（<200K） | $15/MTok | $30/MTok |
| 출력（<200K） | $75/MTok | $150/MTok |
| 품질 | 동일 | **동일** |

**포인트:**

- 동일한 Opus 4.6의 API 설정 차이（다른 모델이 아님）
- `↯` 아이콘으로 활성 상태를 알 수 있다
- 레이트 리밋에 도달하면 자동으로 일반 속도로 폴백
- **세션 시작 시에 활성화**하는 것이 비용 최적（중간에 활성화하면 캐시가 효과가 없다）

---

## **전문가편**

**⑯ Agent Teams — 복수의 Claude에 의한 팀 개발**

```
// settings.json에 추가
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

```
PR #142를 리뷰하는 Agent Team을 만들어줘.
리뷰어를 3명 만들어：
- 보안 담당
- 퍼포먼스 담당
- 테스트 커버리지 담당
각각 리뷰하고 결과를 보고해줘.
```

**포인트:**

- Lead（리더）+ Teammates（멤버）의 구성
- 공유 태스크 리스트 × 메일박스 메시징
- `Shift+Down`으로 팀메이트 간을 순환, 직접 대화도 가능
- tmux / iTerm2로 분할 패널 표시
- "경합 가설 디버깅"：5명의 에이전트가 서로의 가설을 반증하는 기법이 강력하다
- **권장 팀 사이즈：3~5명**

---

**⑰ Plugin 에코시스템 — 설치·개발·Marketplace**

```
/plugin                            # 플러그인 관리 메뉴
/plugin marketplace add anthropics/claude-code   # 공식 Marketplace 추가
/plugin install feature-dev        # 플러그인 설치
/reload-plugins                    # 핫 리로드
```

**포인트:**

- 플러그인 = Skills + Agents + Hooks + MCP + LSP의 번들
- 활성화/비활성화를 전환하여 컨텍스트 소비를 제어한다
- 직접 만든 Marketplace는 Git 리포지토리에 `.claude-plugin/marketplace.json`을 두는 것만으로 완성
- 커뮤니티 Marketplace 예：
  - [aitmpl.com/plugins](https://www.aitmpl.com/plugins)（DevOps, 문서 생성 등）
  - [wshobson/agents](https://github.com/wshobson/agents)（80개 이상의 서브에이전트 모음）

---

**⑱ Output Styles — Claude Code를 "코딩 이외"의 에이전트로 변신**

```
/config → Output style
```

| 스타일 | 동작 |
|--------|------|
| **Default** | 일반 소프트웨어 엔지니어링 |
| **Explanatory** | 코딩하면서 "Insights"로 해설을 끼워 넣는다 |
| **Learning** | 협동 학습 모드. `TODO(human)` 마커로 인간이 코드를 작성할 부분을 제시 |
| **커스텀** | `~/.claude/output-styles/`에 Markdown 파일을 배치 |

**포인트:**

- Output Style은 시스템 프롬프트 자체를 다시 쓴다（CLAUDE.md와는 근본적으로 다르다）
- `keep-coding-instructions: true`로 코딩 지시를 유지하면서 커스텀 스타일을 적용
- 교육 용도, 리서치 에이전트, 문서 작성 에이전트 등으로의 전용에 최적

---

**⑲ 스킬의 동적 컨텍스트 주입 — 셸 출력을 사전 삽입**

Skill 파일 안에서 `` !`command` `` 구문을 사용하면, **스킬 실행 전에 셸 커맨드가 실행되고, 그 출력이 프롬프트에 삽입된다.**

```
---
name: pr-summary
description: PR의 요약
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
이 PR을 요약해줘...
```

**포인트:**

- 이것은 **전처리**이며, Claude가 실행하는 것이 아니다
- Claude는 이미 전개된 결과만 받는다
- `${CLAUDE_SESSION_ID}`나 `${CLAUDE_SKILL_DIR}` 변수 치환도 사용 가능하다
- `$ARGUMENTS[0]`, `$0`으로 인수의 위치 접근도 가능

---

**⑳ 구조화 출력 × 헤드리스 모드 — CI/CD 파이프라인 통합**

```
# JSON Schema에 따른 구조화 출력
claude -p --json-schema '{
  "type": "object",
  "properties": {
    "bugs": {"type": "array", "items": {"type": "string"}},
    "severity": {"type": "string", "enum": ["low","medium","high","critical"]}
  }
}' "이 코드의 버그를 분석해줘"

# 스트리밍 JSON（실시간 처리）
cat log.txt | claude -p --output-format stream-json "에러를 분석해줘"

# 예산 상한 지정 실행（$5를 초과하면 정지）
claude -p --max-budget-usd 5.00 "전체 테스트 파일을 리팩토링해줘"

# 턴 수 제한（3턴에서 강제 종료）
claude -p --max-turns 3 "이 함수를 설명해줘"

# 폴백 모델（Opus 과부하 시 Sonnet을 사용）
claude -p --fallback-model sonnet "코드 리뷰해줘"
```

**포인트:**

- `--json-schema`：에이전트 워크플로 완료 후 스키마에 맞는 JSON을 출력
- `--max-budget-usd`：예산 관리가 엄격한 환경에서 필수
- `--max-turns`：무한 루프 방지를 위한 안전망
- `--fallback-model`：가용성을 확보하고 싶은 프로덕션 파이프라인 용도

---

## **전체 20선 빠른 참조표**

| # | 커맨드 / 숨겨진 기술 | 용도 | 추가 버전 |
|---|----------------------|------|-----------|
| 1 | `/btw` | 작업 중단 없는 곁다리 질문 | v2.0.64+ |
| 2 | `!` 프리픽스 | 직접 셸 실행 | v1.0+ |
| 3 | `Esc` × 2 | 체크포인트 되감기 | v1.0+ |
| 4 | `/context` | 컨텍스트 사용량 가시화 | v2.1.74+ |
| 5 | `/diff` | 인터랙티브 차분 뷰 | v2.0+ |
| 6 | `ultrathink` | 단어 하나로 최대 추론 깊이 | v2.1.68+ |
| 7 | `/loop` | cron식 정기 실행 | v2.1.72+ |
| 8 | `opusplan` | Plan=Opus / 실행=Sonnet | v2.0+ |
| 9 | `/batch` | 병렬 에이전트로 대규모 변경 | v2.1.63+ |
| 10 | `Shift+Tab` | 3모드 순식간 전환 | v1.0+ |
| 11 | `/simplify` | 3에이전트 병렬 리뷰 | v2.1.63+ |
| 12 | `--worktree` | Git worktree 격리 | v2.1.49+ |
| 13 | `/export` `/copy` | 대화 내보내기 | v2.0+ |
| 14 | `--chrome` | 브라우저 조작 연동 | v2.0.73+ |
| 15 | `/fast` | Opus 4.6을 2.5배 고속화 | v2.1.36+ |
| 16 | Agent Teams | 복수 Claude 팀 개발 | v2.1.32+ |
| 17 | `/plugin` | 플러그인 에코시스템 | v2.0.12+ |
| 18 | Output Styles | 에이전트 용도 전환 | v2.0+ |
| 19 | `` !`cmd` `` in Skill | 동적 컨텍스트 주입 | v2.0+ |
| 20 | `--json-schema` 등 | CI/CD 파이프라인 통합 | v2.0+ |

---

## **단축키 목록（덤）**

| 키 | 기능 |
|----|------|
| `Ctrl+C` | 입력/생성 취소 |
| `Ctrl+B` | 백그라운드 실행으로 이동（tmux 유저는 2회） |
| `Ctrl+O` | 상세 출력（툴 실행 상세 표시）전환 |
| `Ctrl+T` | 태스크 리스트 표시/숨김 |
| `Ctrl+G` | 기본 에디터에서 입력 편집 |
| `Ctrl+L` | 터미널 클리어（대화 기록은 유지） |
| `Ctrl+R` | 커맨드 기록 역방향 검색 |
| `Ctrl+V` | 클립보드에서 이미지 붙여넣기 |
| `Shift+Down` | Agent Teams의 팀메이트 순환 |
| `Option+P` (macOS) | 모델 전환 |
| `Option+T` (macOS) | Extended Thinking 전환 |

---

  
## **참고 문헌**

- [Claude Code 공식 문서](https://code.claude.com/docs/en/overview)
- [Claude Code Changelog](https://code.claude.com/docs/en/changelog)
- [Claude Code Built-in Commands Reference](https://code.claude.com/docs/en/commands)
- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills)
- [Claude Code Agent Teams Documentation](https://code.claude.com/docs/en/agent-teams)
- [Claude Code Plugins Blog](https://claude.com/blog/claude-code-plugins)
- [Claude Code Scheduled Tasks](https://code.claude.com/docs/en/scheduled-tasks)
- [Claude Opus 4.6 Announcement](https://www.anthropic.com/news/claude-opus-4-6)  