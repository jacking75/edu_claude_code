# 🤖 Claude Code 치트시트

> 공식 문서 기준 최신 버전 (2026-03-17)

---

## **📦 설치 & 업데이트**

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# Homebrew
brew install --cask claude-code

# 업데이트
claude update

# 버전 확인
claude -v
```

---

## **🔐 인증 (Auth)**

```bash
claude auth login                              # 로그인
claude auth login --email you@email.com --sso # SSO 로그인
claude auth logout                            # 로그아웃
claude auth status                            # 상태 확인 (JSON)
claude auth status --text                     # 상태 확인 (읽기 편한 텍스트)
```

---

## **🚀 세션 시작**

```bash
claude                          # 인터랙티브 세션 시작
claude "프롬프트"                # 초기 프롬프트와 함께 시작
claude -p "프롬프트"             # 응답 출력 후 바로 종료 (스크립트용)
claude -c                       # 가장 최근 대화 이어서 시작
claude -c -p "프롬프트"          # 이어서 시작 + 종료 모드
claude -r "세션ID" "프롬프트"    # 특정 세션 ID/이름으로 재개
claude -n "작업이름"             # 세션 이름 붙여서 시작
```

---

## **⚙️ 주요 CLI 플래그**

| 플래그 | 설명 | 예시 |
|---|---|---|
| `--model` | 모델 지정 | `claude --model claude-sonnet-4-6` |
| `--effort` | 노력 수준 설정 (`low/medium/high/max`) | `claude --effort high` |
| `--add-dir` | 추가 작업 디렉터리 지정 | `claude --add-dir ../lib` |
| `--allowedTools` | 허가 없이 실행할 툴 지정 | `--allowedTools "Bash(git *)" "Read"` |
| `--disallowedTools` | 사용 금지 툴 지정 | `--disallowedTools "Bash(rm *)"` |
| `--tools` | 사용 가능한 툴만 제한 | `--tools "Bash,Edit,Read"` |
| `--permission-mode` | 권한 모드 설정 (`plan` 등) | `--permission-mode plan` |
| `--dangerously-skip-permissions` | 모든 권한 확인 건너뜀 ⚠️ | `claude --dangerously-skip-permissions` |
| `--max-turns` | 최대 턴 수 제한 (프린트 모드) | `claude -p --max-turns 3 "쿼리"` |
| `--max-budget-usd` | 최대 비용 제한 (프린트 모드) | `claude -p --max-budget-usd 5.00 "쿼리"` |
| `--output-format` | 출력 형식 (`text/json/stream-json`) | `claude -p --output-format json` |
| `--system-prompt` | 시스템 프롬프트 전체 교체 | `--system-prompt "You are a Python expert"` |
| `--append-system-prompt` | 시스템 프롬프트 끝에 추가 | `--append-system-prompt "Use TypeScript"` |
| `--mcp-config` | MCP 서버 설정 파일 로드 | `--mcp-config ./mcp.json` |
| `--verbose` | 상세 로그 출력 | `claude --verbose` |
| `--debug` | 디버그 모드 | `claude --debug "api,mcp"` |
| `--worktree`, `-w` | 격리된 Git worktree에서 시작 | `claude -w feature-auth` |
| `--remote` | 웹 세션으로 새 작업 시작 | `claude --remote "Fix login bug"` |
| `--fork-session` | 재개 시 새 세션 ID로 분기 | `claude -r abc123 --fork-session` |
| `--chrome` | Chrome 브라우저 통합 활성화 | `claude --chrome` |

---

## **💬 세션 내 슬래시 명령어 (Slash Commands)**

**세션 관리**

| 명령어 | 설명 |
|---|---|
| `/help` | 도움말 & 전체 명령어 목록 |
| `/clear` | 대화 기록 초기화 (= `/reset`, `/new`) |
| `/compact [지시사항]` | 대화 압축 (컨텍스트 절약) |
| `/exit` | 세션 종료 (= `/quit`) |
| `/resume [세션]` | 이전 대화 재개 |
| `/fork [이름]` | 현재 대화를 분기 |
| `/rename [이름]` | 현재 세션 이름 변경 |

**정보 & 비용**

| 명령어 | 설명 |
|---|---|
| `/cost` | 토큰 사용량 / 비용 통계 |
| `/usage` | 플랜 사용 한도 & 레이트 리밋 |
| `/context` | 현재 컨텍스트 사용량 시각화 |
| `/stats` | 일별 사용량, 세션 히스토리, 선호 모델 |
| `/status` | 버전, 모델, 계정, 연결 상태 |

**코드 & 리뷰**

| 명령어 | 설명 |
|---|---|
| `/diff` | 변경 사항 인터랙티브 diff 뷰어 |
| `/rewind` | 대화/코드를 이전 시점으로 되돌리기 (= `/checkpoint`) |
| `/security-review` | 현재 브랜치 변경사항 보안 취약점 분석 |
| `/pr-comments [PR번호]` | GitHub PR 코멘트 가져오기 |

**설정 & 구성**

| 명령어 | 설명 |
|---|---|
| `/config` | 설정 UI 열기 (= `/settings`) |
| `/model [모델명]` | 모델 변경 |
| `/effort [수준]` | 노력 수준 변경 (`low/medium/high/max`) |
| `/permissions` | 도구 권한 관리 (= `/allowed-tools`) |
| `/hooks` | 훅 설정 보기 |
| `/memory` | CLAUDE.md 메모리 파일 편집 |
| `/theme` | 색상 테마 변경 |
| `/vim` | Vim 편집 모드 토글 |
| `/terminal-setup` | 터미널 키바인딩 설정 |

**기타 유용한 명령어**

| 명령어 | 설명 |
|---|---|
| `/btw <질문>` | 현재 대화에 영향 없이 빠른 사이드 질문 |
| `/plan` | Plan 모드 진입 |
| `/init` | 프로젝트 CLAUDE.md 초기화 |
| `/export [파일명]` | 대화 내용 내보내기 |
| `/copy` | 마지막 응답 클립보드에 복사 |
| `/mcp` | MCP 서버 관리 |
| `/agents` | 서브에이전트 관리 |
| `/skills` | 사용 가능한 스킬 목록 |
| `/tasks` | 백그라운드 작업 관리 |
| `/doctor` | 설치 환경 진단 |

---

## **⌨️ 키보드 단축키**

**일반 조작**

| 단축키 | 설명 |
|---|---|
| `Ctrl+C` | 현재 입력/생성 취소 |
| `Ctrl+D` | 세션 종료 |
| `Ctrl+L` | 터미널 화면 지우기 (대화 기록 유지) |
| `Ctrl+O` | 상세 출력(verbose) 토글 |
| `Ctrl+R` | 명령어 히스토리 역방향 검색 |
| `Ctrl+G` | 기본 텍스트 에디터에서 프롬프트 편집 |
| `Ctrl+T` | 작업 목록 토글 |
| `Ctrl+B` | 현재 Bash 커맨드를 백그라운드로 전환 |
| `Ctrl+F` | 백그라운드 에이전트 전체 종료 |
| `Esc` + `Esc` | 이전 상태로 되감기 / 요약 |
| `Shift+Tab` / `Alt+M` | 권한 모드 전환 (Auto-Accept ↔ Plan ↔ 일반) |

**멀티라인 입력**

| 단축키 | 설명 |
|---|---|
| `\` + `Enter` | 줄바꿈 (모든 터미널) |
| `Option+Enter` | 줄바꿈 (macOS 기본) |
| `Shift+Enter` | 줄바꿈 (iTerm2, WezTerm, Ghostty, Kitty) |
| `Ctrl+J` | 줄바꿈 |

**모델 & 기능 전환**

| 단축키 | 설명 |
|---|---|
| `Option+P` / `Alt+P` | 모델 변경 (프롬프트 유지) |
| `Option+T` / `Alt+T` | Extended Thinking 토글 |

**이미지 붙여넣기**

| 단축키 | 설명 |
|---|---|
| `Ctrl+V` / `Cmd+V` / `Alt+V` | 클립보드 이미지 붙여넣기 |

---

## **📄 세션 내 특수 접두사**

```
@파일경로         → 파일/디렉터리 참조
!명령어           → Bash 명령 직접 실행 (대화 맥락에 출력 포함)
/명령어           → 슬래시 커맨드
```

**@참조 예시:**

```bash
@./src/Button.tsx                  # 단일 파일
@./src/api/                        # 디렉터리 전체 (재귀)
@./src/old.js @./src/new.js        # 여러 파일
@./src/**/*.test.ts                # 글로브 패턴
```

**!Bash 예시:**

```bash
! git status
! npm test
! ls -la
!                                  # 쉘 모드 토글 (다시 !로 나가기)
```

---

## **📁 설정 파일 구조**

```
~/.claude/settings.json           # 전역 설정 (모든 프로젝트 적용)
~/.claude/CLAUDE.md               # 전역 메모리 / 지시사항
~/.claude/commands/               # 개인 커스텀 커맨드

.claude/settings.json             # 프로젝트 설정 (팀 공유, git 포함)
.claude/settings.local.json       # 개인 로컬 설정 (git 제외)
.claude/CLAUDE.md                 # 프로젝트 메모리 / 지시사항
.claude/commands/                 # 프로젝트 커스텀 커맨드
.claude/agents/                   # 서브에이전트 설정
```

**settings.json 예시:**

```json
{
  "model": "claude-sonnet-4-6",
  "permissions": {
    "allowedTools": ["Read", "Write(src/**)", "Bash(git *)", "Bash(npm *)"],
    "deny": ["Read(.env*)", "Write(production.config.*)", "Bash(rm *)", "Bash(sudo *)"]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(*.py)",
        "hooks": [{ "type": "command", "command": "python -m black \"$file\"" }]
      }
    ]
  }
}
```

---

## **📝 CLAUDE.md 예시**

```markdown
# 프로젝트 컨텍스트

## 코딩 스타일
- 모든 새 코드는 TypeScript 사용
- 기존 ESLint 설정 준수
- 모든 새 함수에 Jest 테스트 작성

## 아키텍처
- Frontend: Next.js + TypeScript
- Backend: Node.js + Express
- DB: PostgreSQL + Prisma

## 파일 구조
- 컴포넌트: src/components/
- 유틸리티: src/utils/
- 테스트: 소스파일 옆 .test.ts로
```

---

## **🔩 커스텀 슬래시 커맨드 만들기**

```bash
# 프로젝트 커맨드 생성
mkdir -p .claude/commands
echo "이 코드의 성능 문제를 분석하고 최적화를 제안해줘:" > .claude/commands/optimize.md

# 개인 커맨드 생성
mkdir -p ~/.claude/commands
echo "보안 취약점을 검토해줘:" > ~/.claude/commands/security.md

# 인자 활용 커맨드
echo 'Fix issue #$ARGUMENTS, following our coding standards' > .claude/commands/fix-issue.md
# 사용: /fix-issue 123
```

**고급 커맨드 예시 (프론트매터 + 동적 컨텍스트):**

```markdown
---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
description: 변경사항을 보고 git 커밋 메시지 작성
---
## Context
- 현재 상태: !`git status`
- 현재 diff: !`git diff HEAD`
- 현재 브랜치: !`git branch --show-current`

위 변경사항을 바탕으로 의미있는 커밋 메시지를 작성해줘.
```

---

## **🪝 Hooks (자동화)**

Hook 이벤트 종류는 `PreToolUse` (툴 실행 전), `PostToolUse` (툴 실행 후), `UserPromptSubmit` (사용자 입력 처리 전), `SessionStart` (세션 시작 시)가 있습니다.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write(*.py)",
        "hooks": [{ "type": "command", "command": "python -m black \"$file\"" }]
      }
    ]
  }
}
```

---

## **🔌 MCP 서버 연결**

```bash
claude mcp add 서버이름 -e API_KEY=값 -- /path/to/server arg1 arg2

# 세션 내에서 MCP 명령 사용
/mcp__서버이름__프롬프트명
```

---

## **🧠 유용한 파이프 & 스크립트 패턴**

```bash
# 로그 파일 분석
cat error.log | claude -p "이 에러들을 분석해줘"

# 파일 코드 리뷰
cat src/service.ts | claude -p "이 코드를 리뷰해줘"

# JSON 출력으로 자동화
claude -p "함수 목록 추출" --output-format json

# CI/CD에서 자동 수정
claude -p "린팅 에러가 있으면 수정하고 커밋 메시지를 제안해줘"

# 비용 제한 설정
claude -p --max-budget-usd 2.00 "큰 작업 처리"

# 특정 세션 재개 후 작업 계속
claude -r "auth-refactor" "이 PR을 마무리해줘"
```

---

## **✅ 권한(Permission) 빠른 참조**

| 권한 패턴 | 설명 |
|---|---|
| `Read` | 모든 파일 읽기 허용 |
| `Write(src/**)` | src 하위 파일 쓰기 허용 |
| `Bash(git *)` | git 명령만 허용 |
| `Bash(npm *)` | npm 명령만 허용 |
| `Read(.env*)` | .env 파일 읽기 **차단** (deny) |
| `Bash(rm *)` | rm 명령 **차단** (deny) |

---

> 💡 **팁:** 세션 내에서 `/help` 를 입력하면 현재 환경에서 사용 가능한 모든 명령어를 바로 확인할 수 있습니다. 공식 문서는 [docs.anthropic.com/en/docs/claude-code](https://docs.anthropic.com/en/docs/claude-code) 에서 확인하세요.