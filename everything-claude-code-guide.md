# everything-claude-code 완전 정리

[출처](https://zenn.dev/ttks/articles/a54c7520f827be )  
  

> Anthropic x Forum Ventures 해커톤 우승자 [@affaanmustafa](https://x.com/affaanmustafa)가 10개월 이상 실제 프로덕트 개발에서 사용한 Claude Code 설정 모음집이다. X에 공개된 가이드는 **90만 뷰 이상, 1만 이상의 북마크**를 기록했다.


## **리포지터리 개요:**
해커톤 우승 당시 개발한 [zenith.chat](https://zenith.chat/)은 AI를 활용한 고객 발굴 플랫폼으로, 개발 전체가 Claude Code만으로 이루어졌다. 리포지터리는 아래 6가지 핵심 구성 요소로 이루어져 있다.

```
everything-claude-code/
├── agents/           # 전문 에이전트
├── skills/           # 워크플로 정의 및 도메인 지식
├── commands/         # 슬래시 커맨드
├── rules/            # 항상 따르는 규칙
├── hooks/            # 트리거 기반 자동화
├── mcp-configs/      # MCP 서버 설정
├── plugins/          # 플러그인 정보
└── examples/         # 설정 예시
```


## **agents/ — 전문 에이전트에 의한 태스크 위임:**
복잡한 작업을 메인 Claude에서 분리해서 위임하는 구조다.

| 파일 | 역할 |
|---|---|
| `planner.md` | 기능 구현 계획 수립 |
| `architect.md` | 시스템 설계·아키텍처 결정 |
| `tdd-guide.md` | 테스트 주도 개발 가이드 |
| `code-reviewer.md` | 코드 품질·보안 리뷰 |
| `security-reviewer.md` | 취약점 분석 (OWASP Top 10 대응) |
| `build-error-resolver.md` | 빌드 에러 해결 |
| `e2e-runner.md` | Playwright E2E 테스트 |
| `refactor-cleaner.md` | 불필요 코드 삭제 |
| `doc-updater.md` | 문서 동기화 업데이트 |

에이전트 파일은 frontmatter 형식으로 정의한다.

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior code reviewer...
```

핵심은 `tools` 지정이다. 서브에이전트에 **필요 최소한의 툴만 부여**하면 실행이 빨라지고 포커스도 유지된다. 50개 툴을 가진 에이전트보다 5개로 압축한 에이전트가 더 효율적으로 동작한다.

  
## **skills/ — 재사용 가능한 워크플로 정의:**
커맨드나 에이전트가 호출하는 워크플로 정의를 저장한다.

| 파일 | 내용 |
|---|---|
| `coding-standards.md` | 언어별 베스트 프랙티스 |
| `backend-patterns.md` | API·데이터베이스·캐시 패턴 |
| `frontend-patterns.md` | React·Next.js 패턴 |
| `project-guidelines-example.md` | 프로젝트 고유 스킬 예시 |
| `tdd-workflow/` | TDD 기법의 상세 정의 |
| `security-review/` | 보안 체크리스트 |
| `clickhouse-io.md` | ClickHouse 애널리틱스 |

특히 `tdd-workflow`는 **RED → GREEN → REFACTOR** 사이클을 정의하며, 테스트 커버리지 80% 이상을 필수로 한다.

배치 위치는 `~/.claude/skills/`(유저 공통) 또는 `.claude/skills/`(프로젝트 고유)다.

  
## **commands/ — 슬래시 커맨드로 빠르게 실행:**

| 커맨드 | 설명 |
|---|---|
| `/tdd` | 테스트 주도 개발 워크플로 |
| `/plan` | 구현 계획 작성 (유저 확인 대기) |
| `/e2e` | E2E 테스트 생성 |
| `/code-review` | 품질 리뷰 |
| `/build-fix` | 빌드 에러 수정 |
| `/refactor-clean` | 불필요 코드 삭제 |
| `/test-coverage` | 커버리지 분석 |
| `/update-codemaps` | 코드맵 업데이트 |
| `/update-docs` | 문서 동기화 |

skills와 commands의 차이는 **저장 위치와 호출 방법**이다.

- **skills**: `~/.claude/skills/` — 넓은 범위의 워크플로 정의
- **commands**: `~/.claude/commands/` — 빠르게 실행 가능한 프롬프트

여러 커맨드를 연속으로 실행하는 체인도 구성할 수 있다. 예를 들어 `/tdd` 이후 `/refactor-clean`을 실행하는 방식이다.

  
## **rules/ — 항상 따르는 가이드라인:**

| 파일 | 내용 |
|---|---|
| `security.md` | 필수 보안 체크 |
| `coding-style.md` | 불변성, 파일 구성 |
| `testing.md` | TDD, 80% 커버리지 요건 |
| `git-workflow.md` | 커밋 형식, PR 절차 |
| `agents.md` | 에이전트 사용 타이밍 |
| `performance.md` | 모델 선택, 컨텍스트 관리 |
| `patterns.md` | API 응답 형식, 훅 |
| `hooks.md` | 훅 문서 |

보안 규칙에서는 커밋 전 다음 항목을 필수 체크로 지정한다.

- 하드코딩된 시크릿이 없을 것
- 모든 유저 입력이 유효성 검사되어 있을 것
- 적절한 에러 핸들링이 있을 것
- 의존성에 알려진 취약점이 없을 것

rules 파일은 하나의 거대한 파일이 아닌 **역할별로 모듈 분할**하는 것이 권장된다.

  
## **hooks/ — 이벤트 기반 자동화:**

| 훅 종류 | 타이밍 | 예시 |
|---|---|---|
| `PreToolUse` | 툴 실행 전 | 특정 작업 차단 또는 리마인드 |
| `PostToolUse` | 툴 실행 후 | 편집 후 자동 포맷 |
| `Stop` | 세션 종료 전 | console.log 경고 |

TypeScript/JavaScript 파일 편집 시 `console.log` 잔존을 경고하는 훅 예시:

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

파일 편집 후 자동으로 `console.log`를 감지해서 삭제를 촉구한다. 프로덕션 환경에 디버그 코드가 섞이는 것을 방지하는 구조다.

  
## **mcp-configs/ — 외부 서비스 연동 + 가장 중요한 주의사항:**
GitHub, Supabase, Vercel, Railway 등 외부 서비스와의 연동 설정이 포함되어 있다.

⚠️ **MCP를 너무 많이 활성화하면 컨텍스트 윈도우가 대폭 축소된다.** 200k였던 컨텍스트가 70k까지 줄어들 가능성이 있다.

권장 운용 방법은 다음과 같다.

- 20~30개의 MCP를 설정 파일에 기술
- 프로젝트별로 **10개 이하를 활성화**
- 활성화된 툴은 **80개 이하**로 억제
- 사용하지 않는 MCP는 `disabledMcpServers`로 비활성화

  
## **퀵스타트:**
  
```bash
# 리포지터리 클론
git clone https://github.com/affaan-m/everything-claude-code.git

# agents 복사
cp everything-claude-code/agents/*.md ~/.claude/agents/

# rules 복사
cp everything-claude-code/rules/*.md ~/.claude/rules/

# commands 복사
cp everything-claude-code/commands/*.md ~/.claude/commands/

# skills 복사
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

hooks는 `~/.claude/settings.json`에, MCP 설정은 `~/.claude.json`에 추가한다. MCP 설정 내 `YOUR_*_HERE` 플레이스홀더는 실제 API 키로 교체한다.

  
## **설계 철학 요약:**

- **에이전트 퍼스트 디자인**: 복잡한 태스크는 전문 에이전트에 위임
- **TDD 중심 워크플로**: RED→GREEN→REFACTOR 사이클과 80% 커버리지
- **보안 퍼스트**: 커밋 전 필수 체크
- **컨텍스트 윈도우 관리**: MCP 활성화는 최소한으로

모든 설정을 그대로 사용할 필요는 없다. 자신의 워크플로에 맞는 것을 골라서 도입하고, 불필요한 것은 삭제하며, 독자적인 패턴을 추가해 나가는 것이 좋다.  