# Claude Code 완전 설정 가이드 2026 — Hooks/Skills/MCP/권한 관리를 전부 담은 본격 운용 설정 파일 일식

Claude Code의 설정을 대충 감으로 쓰고 있지 않은가?

Hooks, Skills, CLAUDE.md, settings.json, MCP, 권한 관리…… 2026년의 Claude Code는 기능이 폭발적으로 늘었다. 하지만 **모든 것을 올바르게 설정하고 있는 엔지니어는 1%도 없다**.

이 글에서는 모든 기능을 망라한 「본격 운용이 가능한 설정 파일 일식」을 제공한다. 복붙으로 바로 쓸 수 있다.

---

**결론부터 말하자면:**

Claude Code의 설정은 **7개의 레이어**로 구성되어 있다:

1. **CLAUDE.md** - Claude에 대한 지시서 (팀 공유 / 개인용)
2. **Auto Memory** - Claude가 자동으로 학습하는 메모
3. **.claude/rules/** - 모듈형 룰 파일
4. **settings.json** - 권한·허가 툴·환경 설정
5. **Hooks** - 라이프사이클 이벤트 자동화 (17개 이벤트)
6. **Skills** - 커스텀 슬래시 커맨드
7. **MCP** - 외부 툴 연계 (GitHub, DB, Sentry 등)

이 모든 것을 올바르게 설정해야 비로소 「본격 운용」이라 할 수 있다.

---

**디렉터리 구성의 전체상:**

우선, 완성형 디렉터리 구성을 살펴보자.

```
your-project/
├── CLAUDE.md                    # 프로젝트 지시서 (git 관리)
├── CLAUDE.local.md              # 개인용 지시서 (gitignore)
├── .mcp.json                    # 팀 공유 MCP 설정 (git 관리)
├── .claude/
│   ├── CLAUDE.md                # 별도 배치 OK (CLAUDE.md와 동등)
│   ├── settings.json            # 공유 설정 (git 관리)
│   ├── settings.local.json      # 개인 설정 (gitignore)
│   ├── rules/                   # 모듈형 룰
│   │   ├── code-style.md
│   │   ├── testing.md
│   │   ├── security.md
│   │   └── frontend/
│   │       ├── react.md
│   │       └── styles.md
│   ├── skills/                  # 커스텀 스킬
│   │   ├── deploy/
│   │   │   └── SKILL.md
│   │   ├── review-pr/
│   │   │   └── SKILL.md
│   │   └── fix-issue/
│   │       └── SKILL.md
│   ├── agents/                  # 커스텀 서브에이전트
│   │   └── security-reviewer/
│   │       └── AGENT.md
│   └── hooks/                   # Hook 스크립트
│       ├── lint-on-save.sh
│       ├── block-secrets.sh
│       └── notify-slack.sh
│
├── ~/.claude/                   # 유저 글로벌 설정
│   ├── CLAUDE.md                # 전 프로젝트 공통 개인 지시
│   ├── settings.json            # 글로벌 설정
│   ├── settings.local.json      # 로컬 전용
│   ├── rules/                   # 개인 룰
│   │   └── preferences.md
│   └── skills/                  # 개인 스킬
│       └── explain-code/
│           └── SKILL.md
```

---

**1. CLAUDE.md - Claude에 대한 지시서:**

CLAUDE.md는 **Claude의 뇌에 설치하는 프로젝트의 DNA**다.

**메모리 계층 (우선도순):**

| 우선도 | 종류 | 경로 | 용도 |
|--------|------|------|------|
| 최고 | 관리 정책 | `/Library/Application Support/ClaudeCode/CLAUDE.md` | 조직 전체 룰 |
| 높음 | 프로젝트 | `./CLAUDE.md` | 팀 공유 지시 |
| 높음 | 룰 | `.claude/rules/*.md` | 모듈형 지시 |
| 중간 | 유저 | `~/.claude/CLAUDE.md` | 개인 글로벌 설정 |
| 낮음 | 로컬 | `./CLAUDE.local.md` | 개인 프로젝트 고유 설정 |
| 자동 | Auto Memory | `~/.claude/projects/<project>/memory/` | Claude 자동 학습 메모 |

**CLAUDE.md의 황금률: 「인간만이 알고 있는 것」을 쓴다**

**가장 흔한 실수: 소스 코드에서 읽을 수 있는 정보를 CLAUDE.md에 쓰는 것이다.**

Claude는 `package.json`을 읽으면 빌드 커맨드를 알 수 있다. `tsconfig.json`을 읽으면 TypeScript 설정을 알 수 있다. 디렉터리를 탐색하면 아키텍처를 알 수 있다. `.eslintrc`를 읽으면 코딩 규약을 알 수 있다.

**이것들을 CLAUDE.md에 쓰는 것은 토큰 낭비다.**

CLAUDE.md에는 **Claude가 소스 코드를 읽어도 절대로 알 수 없는 정보**만 쓴다.

| 써야 하는 것 (인간의 머릿속에만 있는 것) | 쓰지 말아야 하는 것 (코드에서 추론 가능한 것) |
|------------------------------------------|------------------------------------------------|
| 왜 이 아키텍처를 선택했는가 | `packages/` 하위의 디렉터리 목록 |
| 과거 인시던트에서 배운 금지 사항 | `pnpm build`로 빌드할 수 있다는 것 |
| 비즈니스상의 제약이나 우선순위 | TypeScript를 사용한다는 것 |
| 배포 대상 환경 정보 | ESLint 룰 |
| 팀 간 약속·워크플로우 | 의존 패키지 목록 |
| 「이 코드는 건드리지 말 것」의 암묵지 | 테스트 프레임워크가 Vitest라는 것 |

**본격 운용용 CLAUDE.md 템플릿:**

```
# 프로젝트의 의사결정

## 왜 이 구성인가
- 모노레포를 채택한 이유: 프론트·백 간의 타입 공유로 버그를 3건/주→0으로 줄였다
- Hono를 선택한 이유: Cloudflare Workers에서의 콜드 스타트가 50ms 이하
- PlanetScale을 사용하는 이유: 프로덕션 DB의 스키마 변경을 브랜치로 안전하게 할 수 있다

## 절대 해서는 안 되는 것
- `packages/shared/src/legacy/` 는 구 API와의 호환 레이어. 리팩터하고 싶어도 건드리지 말 것 (고객 3사가 의존 중, 2026년 Q3에 폐지 예정)
- Stripe의 webhook handler는 멱등성을 깨면 이중 청구가 발생한다. 과거에 $12,000의 사고가 있었다
- `users` 테이블의 `email` 컬럼에 UNIQUE 제약이 없다. 역사적 경위로 중복이 존재한다. 앱 레이어에서 밸리데이션하고 있다

## 배포와 환경
- staging: Cloudflare Workers (`wrangler deploy --env staging`)
- production: 프로덕션 배포는 GitHub Actions만. 수동 배포 금지
- DB migration: `pnpm db:migrate`는 반드시 staging에서 먼저 실행해 확인

## 팀 워크플로우
- PR은 반드시 1명 이상의 리뷰를 통할 것
- `feat/` 브랜치는 squash merge, `fix/` 브랜치는 일반 merge
- 금요일 15시 이후 프로덕션 배포 금지 (주말 대응을 피하기 위해)

## 비즈니스 컨텍스트
- 엔터프라이즈 고객은 응답 200ms 이내의 SLA가 있다
- GDPR 대응 필수. 유저 데이터는 반드시 논리 삭제 (물리 삭제 금지)
- 월별 리포트 생성은 매월 1일 자정에 cron으로 실행. 이 날은 DB 부하가 높다
```

**이 템플릿의 포인트:**
- `pnpm install`이나 `pnpm test`는 쓰지 않았다 → Claude는 `package.json`을 읽으면 알 수 있다
- 디렉터리 구성은 쓰지 않았다 → Claude는 `ls`하면 알 수 있다
- 대신 「왜」, 「하지 말 것」, 「과거의 사고」를 쓰고 있다 → 이것은 코드에 쓰여 있지 않다

**@import로 외부 파일을 참조:**

CLAUDE.md에서 다른 파일을 임포트할 수 있다.

```
프로젝트 개요는 @README.md 를 참조.
사용 가능한 커맨드는 @package.json 을 확인.
Git 운용 룰은 @docs/git-workflow.md 에 따른다.

# 개인 설정
- @~/.claude/my-project-instructions.md
```

**import의 주의점:**
- 상대 경로는 CLAUDE.md가 있는 디렉터리를 기점으로 해결된다
- 최대 5계층까지의 재귀 import가 가능하다
- 코드 블록 내의 `@`는 import로 해석되지 않는다
- 첫 번째 실행 시에는 임포트 허가 다이얼로그가 표시된다

**.claude/rules/ - 조건부 룰:**

특정 파일 패턴에만 적용되는 룰을 쓸 수 있다.

```
# .claude/rules/api-rules.md
---
paths:
  - "packages/backend/src/routes/**/*.ts"
---

# API 개발 룰

- 전 엔드포인트에 zod 밸리데이션을 넣는다
- 에러 응답은 RFC 7807 형식으로 한다
- 응답에는 페이지네이션 정보를 포함한다
- 레이트 리밋 헤더를 반환한다
```

```
# .claude/rules/react-rules.md
---
paths:
  - "packages/frontend/src/**/*.{tsx,ts}"
---

# React 개발 룰

- Server Components를 기본으로 한다
- "use client"는 최소화한다
- Suspense로 로딩 상태를 관리한다
- key prop에는 index를 사용하지 않는다
```

---

**2. settings.json - 권한과 설정:**

**설정 파일의 우선순위:**

```
관리 정책 (최고)
  ↓
CLI 플래그 (--disallowedTools 등)
  ↓
.claude/settings.local.json (프로젝트 로컬)
  ↓
.claude/settings.json (프로젝트 공유)
  ↓
~/.claude/settings.local.json (유저 로컬)
  ↓
~/.claude/settings.json (유저 공유·최저)
```

**본격 운용용 .claude/settings.json (팀 공유):**

```json
{
  "permissions": {
    "defaultMode": "default",
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Bash(pnpm run *)",
      "Bash(pnpm test *)",
      "Bash(pnpm build *)",
      "Bash(pnpm lint *)",
      "Bash(pnpm typecheck)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git branch *)",
      "Bash(git checkout *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(gh pr *)",
      "Bash(gh issue *)",
      "Bash(npx prisma *)",
      "Bash(* --version)",
      "Bash(* --help)",
      "Edit(/packages/**)",
      "Write(/packages/**)"
    ],
    "deny": [
      "Bash(git push --force *)",
      "Bash(git reset --hard *)",
      "Bash(rm -rf *)",
      "Bash(curl *)",
      "Bash(wget *)",
      "Read(.env*)",
      "Read(**/*.pem)",
      "Read(**/*secret*)",
      "Edit(.env*)",
      "Edit(/.github/workflows/**)"
    ]
  },
  "env": {
    "MAX_MCP_OUTPUT_TOKENS": "50000"
  }
}
```

**개인용 .claude/settings.local.json:**

```json
{
  "permissions": {
    "allow": [
      "Bash(docker *)",
      "Bash(docker-compose *)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/lint-on-save.sh"
          }
        ]
      }
    ]
  }
}
```

**커밋·PR의 귀속 표시 (attribution)를 제어한다:**

Claude Code는 기본적으로 커밋 메시지에 `Co-Authored-By: Claude`를 추가한다. 불필요한 경우 `attribution` 설정으로 제거할 수 있다.

```json
{
  "attribution": {
    "commit": "",
    "pr": ""
  }
}
```

| 키 | 기본값 | 설명 |
|----|--------|------|
| `attribution.commit` | Co-Authored-By 트레일러 포함 | 커밋 메시지에 대한 귀속 표시 |
| `attribution.pr` | 푸터에 Claude Code 표기 | PR 설명에 대한 귀속 표시 |

빈 문자열 `""`로 완전히 비표시할 수 있다. 커스텀 메시지도 가능하다.

```json
{
  "attribution": {
    "commit": "Generated with Claude Code"
  }
}
```

`~/.claude/settings.json`에 넣으면 전 프로젝트에서 유효하다.

**권한 패턴 상세 레퍼런스:**

| 툴 | 패턴 예시 | 설명 |
|----|-----------|------|
| `Bash` | `Bash(npm run *)` | npm run으로 시작하는 모든 커맨드 |
| `Bash` | `Bash(git commit *)` | git 커밋 허가 |
| `Read` | `Read(.env*)` | .env 파일 읽기 |
| `Read` | `Read(~/.ssh/*)` | 홈의 .ssh 하위 |
| `Read` | `Read(//etc/passwd)` | 절대 경로 (//로 시작) |
| `Edit` | `Edit(/src/**)` | 프로젝트 루트로부터의 상대 경로 |
| `Edit` | `Edit(**/*.test.ts)` | 전체 테스트 파일 |
| `WebFetch` | `WebFetch(domain:github.com)` | 도메인 지정 |
| `MCP` | `mcp__github__*` | GitHub MCP의 전체 툴 |
| `Task` | `Task(Explore)` | Explore 서브에이전트 |
| `Skill` | `Skill(deploy *)` | deploy 스킬 |

**중요: `*` 앞에 스페이스가 있는지 없는지로 동작이 달라진다**
- `Bash(ls *)` → `ls -la`에 매치, `lsof`에는 매치하지 않음 (단어 경계 있음)
- `Bash(ls*)` → `ls -la`에도 `lsof`에도 매치 (단어 경계 없음)

---

**3. Hooks - 17개의 라이프사이클 이벤트:**

Hooks는 **CLAUDE.md의 「부탁」과 달리, 반드시 실행된다**.

**전체 이벤트 목록:**

| 이벤트 | 타이밍 | 블록 가능 |
|--------|--------|-----------|
| `SessionStart` | 세션 시작·재개 | - |
| `UserPromptSubmit` | 프롬프트 전송 직후 | - |
| `PreToolUse` | 툴 실행 전 | **Yes** |
| `PermissionRequest` | 권한 다이얼로그 표시 시 | - |
| `PostToolUse` | 툴 성공 후 | - |
| `PostToolUseFailure` | 툴 실패 후 | - |
| `Notification` | 알림 전송 시 | - |
| `SubagentStart` | 서브에이전트 시작 시 | - |
| `SubagentStop` | 서브에이전트 종료 시 | - |
| `Stop` | Claude 응답 완료 시 | - |
| `TeammateIdle` | 팀원이 idle 상태일 때 | - |
| `TaskCompleted` | 태스크 완료 시 | - |
| `ConfigChange` | 설정 파일 변경 시 | - |
| `WorktreeCreate` | worktree 생성 시 | - |
| `WorktreeRemove` | worktree 삭제 시 | - |
| `PreCompact` | 컨텍스트 압축 전 | - |
| `SessionEnd` | 세션 종료 시 | - |

**핸들러의 4종류:**

```
command  → 셸 스크립트 실행
http     → HTTP POST 요청 전송
prompt   → LLM에 판단을 맡긴다 (yes/no)
agent    → 서브에이전트로 검증
```

**본격 Hooks 설정 예시:**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-secrets.sh",
            "statusMessage": "보안 체크 중..."
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/lint-on-save.sh",
            "timeout": 30
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/notify-slack.sh",
            "async": true
          }
        ]
      }
    ]
  }
}
```

**실용 Hook 스크립트 모음:**

**위험 커맨드 블록 (PreToolUse):**

```bash
#!/bin/bash
# .claude/hooks/block-secrets.sh
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# 비밀 정보를 포함한 커맨드를 블록
if echo "$COMMAND" | grep -qE '(password|secret|token|api.?key)'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "비밀 정보를 포함할 가능성이 있는 커맨드가 블록되었습니다"
    }
  }'
  exit 0
fi

# rm -rf를 블록
if echo "$COMMAND" | grep -qE 'rm\s+-rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "rm -rf 는 금지되어 있습니다"
    }
  }'
  exit 0
fi

exit 0
```

**파일 저장 후 자동 린트 (PostToolUse):**

```bash
#!/bin/bash
# .claude/hooks/lint-on-save.sh
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // .tool_result.filePath // empty')

if [ -z "$FILE_PATH" ]; then
  exit 0
fi

# TypeScript 파일만 린트
if [[ "$FILE_PATH" == *.ts || "$FILE_PATH" == *.tsx ]]; then
  RESULT=$(npx eslint --fix "$FILE_PATH" 2>&1)
  if [ $? -ne 0 ]; then
    jq -n --arg msg "$RESULT" '{
      hookSpecificOutput: {
        hookEventName: "PostToolUse"
      },
      transcript: ("ESLint 에러:\n" + $msg)
    }'
  fi
fi

exit 0
```

**Slack 알림 (Stop - 비동기):**

```bash
#!/bin/bash
# .claude/hooks/notify-slack.sh
INPUT=$(cat)
STOP_REASON=$(echo "$INPUT" | jq -r '.stop_reason // "unknown"')

if [ "$STOP_REASON" = "end_turn" ]; then
  curl -s -X POST "$SLACK_WEBHOOK_URL" \
    -H 'Content-Type: application/json' \
    -d "{\"text\": \"Claude Code의 태스크가 완료되었습니다\"}" \
    > /dev/null 2>&1
fi
```

**Prompt Hook (LLM 판단형):**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "다음 Bash 커맨드가 프로덕션 환경에 영향을 줄 가능성이 있는지 판단하십시오. 커맨드: $ARGUMENTS. 프로덕션 DB 접속, 프로덕션 서버 배포, 환경 변수 변경을 포함하는 경우는 no로 답하십시오.",
            "timeout": 15
          }
        ]
      }
    ]
  }
}
```

---

**4. Skills - 커스텀 슬래시 커맨드:**

**Skill의 프론트매터 전체 필드:**

| 필드 | 필수 | 설명 |
|------|------|------|
| `name` | No | `/` 커맨드명 (생략 시 디렉터리명) |
| `description` | 권장 | 용도 설명 (Claude 자동 실행의 판단 자료) |
| `argument-hint` | No | 인수 힌트 (예: `[issue-number]`) |
| `disable-model-invocation` | No | `true`로 Claude 자동 실행을 비활성화 |
| `user-invocable` | No | `false`로 `/` 메뉴에서 비표시 |
| `allowed-tools` | No | 이 Skill 중에 허가하는 툴 |
| `model` | No | 사용 모델 지정 |
| `context` | No | `fork`로 서브에이전트 실행 |
| `agent` | No | `context: fork` 시의 에이전트 형 |
| `hooks` | No | Skill 고유의 Hooks |

**실용 스킬 예시:**

**PR 리뷰 스킬:**

```
# .claude/skills/review-pr/SKILL.md
---
name: review-pr
description: PR을 리뷰하고 개선 제안을 한다
context: fork
agent: Explore
allowed-tools: Bash(gh *)
argument-hint: "[PR번호]"
disable-model-invocation: true
---

## PR 컨텍스트
- PR diff: !`gh pr diff $ARGUMENTS`
- PR 코멘트: !`gh pr view $ARGUMENTS --comments`
- 변경 파일: !`gh pr diff $ARGUMENTS --name-only`

## 리뷰 절차
1. 변경의 의도를 이해한다
2. 코드 품질을 체크한다 (타입 안전성, 에러 핸들링)
3. 보안상 우려가 없는지 확인한다
4. 퍼포먼스에 대한 영향을 평가한다
5. 테스트 커버리지를 확인한다

리뷰 결과를 항목별로 정리하고, 중요도(Critical/Major/Minor)를 붙인다.
```

**Issue 수정 스킬:**

```
# .claude/skills/fix-issue/SKILL.md
---
name: fix-issue
description: GitHub Issue를 수정한다
disable-model-invocation: true
argument-hint: "[issue번호]"
---

GitHub issue #$ARGUMENTS 를 수정한다.

1. `gh issue view $ARGUMENTS` 로 Issue 내용을 확인
2. 관련 코드를 특정
3. 수정을 구현
4. 테스트를 추가
5. `git commit -m "fix: #$ARGUMENTS 의 수정"`
```

**배포 스킬 (서브에이전트 실행):**

```
# .claude/skills/deploy/SKILL.md
---
name: deploy
description: 스테이징 환경에 배포한다
disable-model-invocation: true
context: fork
allowed-tools: Bash(pnpm *), Bash(git *), Bash(gh *)
---

$ARGUMENTS 를 스테이징에 배포한다:

1. 테스트 스위트를 실행
2. 빌드를 실행
3. 배포 커맨드를 실행
4. 배포 성공을 확인
5. Slack에 알림
```

**동적 컨텍스트 주입:**

`` !`command` `` 문법으로 Shell 커맨드의 결과를 Skill에 주입할 수 있다.

```
---
name: status
description: 프로젝트 상태 요약
context: fork
agent: Explore
---

## 현재 상태
- 브랜치: !`git branch --show-current`
- 미커밋: !`git status --short`
- 최신 커밋: !`git log --oneline -5`
- 테스트 결과: !`pnpm test --silent 2>&1 | tail -5`
- 타입 체크: !`pnpm typecheck 2>&1 | tail -3`

위의 정보를 바탕으로 프로젝트 상태를 요약해 보고한다.
```

---

**5. MCP - 외부 툴 연계:**

**MCP 서버 추가:**

```bash
# HTTP (권장)
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# SSE (비권장)
claude mcp add --transport sse sentry https://mcp.sentry.dev/sse

# stdio (로컬 실행)
claude mcp add --transport stdio --env AIRTABLE_API_KEY=$KEY airtable \
  -- npx -y airtable-mcp-server
```

**MCP의 스코프:**

| 스코프 | 저장 위치 | 공유 범위 |
|--------|-----------|-----------|
| `local` (기본) | `~/.claude.json` | 자신만·이 프로젝트 |
| `project` | `.mcp.json` | 팀 전체 (git 관리) |
| `user` | `~/.claude.json` | 자신만·전 프로젝트 |

**팀 공유 MCP 설정 (.mcp.json):**

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "sentry": {
      "type": "http",
      "url": "https://mcp.sentry.dev/mcp"
    },
    "database": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@bytebase/dbhub", "--dsn", "${DB_DSN}"],
      "env": {
        "DB_DSN": "${DB_DSN:-postgresql://readonly:pass@localhost:5432/dev}"
      }
    }
  }
}
```

**MCP Tool Search:**

MCP 툴이 너무 많아 컨텍스트를 압박하는 경우, 자동으로 Tool Search가 활성화된다.

```bash
# 임계값을 5%로 변경
ENABLE_TOOL_SEARCH=auto:5 claude

# 항상 활성화
ENABLE_TOOL_SEARCH=true claude

# 비활성화
ENABLE_TOOL_SEARCH=false claude
```

---

**6. Auto Memory - Claude의 자동 학습:**

**구조:**

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # 인덱스 (처음 200행이 로드됨)
├── debugging.md       # 디버그 패턴
├── api-conventions.md # API 설계 결정 사항
└── ...                # Claude가 자동 생성
```

**MEMORY.md는 200행까지만 로드된다.**
그 이상의 정보는 토픽별 파일로 분할하고, MEMORY.md에서 링크한다. Claude는 필요에 따라 토픽 파일을 읽어들인다.

**명시적으로 기억시키기:**

```
> pnpm을 사용하고, npm은 사용하지 않도록 기억해줘
> API 테스트에는 로컬 Redis가 필요하다는 것을 메모리에 저장해줘
```

**제어 방법:**

```json
// .claude/settings.json 으로 프로젝트 단위로 비활성화
{ "autoMemoryEnabled": false }
```

```bash
# 환경 변수로 강제 제어 (settings.json보다 우선)
CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 claude  # 강제 OFF
CLAUDE_CODE_DISABLE_AUTO_MEMORY=0 claude  # 강제 ON
```

---

**7. 본격 운용 체크리스트:**

모든 것을 설정했는지 확인하자.

**보안:**
- `.env*` 파일의 Read/Edit를 deny로 설정
- `rm -rf`를 PreToolUse 훅으로 블록
- `git push --force`를 deny로 설정
- 비밀 키 (`*.pem`)의 읽기를 deny
- `curl`/`wget`을 deny하고 WebFetch로 한정
- CI/CD 워크플로우 편집을 deny

**개발 효율:**
- 자주 사용하는 빌드/테스트 커맨드를 allow로 설정
- PostToolUse로 자동 린트를 설정
- PR 리뷰 Skill을 생성
- Issue 수정 Skill을 생성
- MCP로 GitHub 연계를 설정

**팀 표준화:**
- `.claude/settings.json`을 git 관리
- `.mcp.json`을 git 관리
- CLAUDE.md에 팀 공유 지식을 기입
- `CLAUDE.local.md`와 `settings.local.json`을 `.gitignore`에 추가  