# Claude Code 에코시스템 완전 정리 — MCP 서버 · 플러그인 · 주변 도구
  
[출처](https://qiita.com/shatolin/items/ca1810e419fee5fd963b )  
  
---

## **개요:**
2026년 2월 기준으로 9,000개 이상의 플러그인이 공개되어 있으며, 에코시스템은 날마다 확장 중이다. 이 글에서는 **MCP 서버**, **플러그인**, **주변 도구** 3가지 카테고리로 분류하여 정리한다.

**🔥 2026년 2월 최신 동향**
- **Claude-Mem**이 GitHub ★2만 돌파, 메모리 계열 플러그인이 대유행
- **Superpowers**가 ★4.3만으로 Anthropic 공식 마켓플레이스에 채택
- Anthropic이 **Cowork용 업종 특화 플러그인 11종** 발표, 법무 플러그인이 "SaaSpocalypse"를 일으킴
- **Ralph Wiggum Loop**가 Anthropic 공식 리포지토리 입성, Y Combinator 스타트업 업계에서 표준화

---

## **MCP란 무엇인가:**

**Model Context Protocol(MCP)** 은 Anthropic이 책정한 오픈 프로토콜로, AI 모델이 외부 도구나 데이터 소스와 안전하게 통신하기 위한 표준 규격이다.

MCP 서버를 연결하면 다음과 같은 것들을 자연어로 지시할 수 있다:
- "JIRA 티켓 1234의 기능을 구현하고 GitHub에 PR을 만들어줘"
- "티켓의 에러를 확인하고 고칠 수 있는 것을 수정해줘"
- "DB에서 지난달 매출을 카테고리별로 집계해줘"

**MCP 서버 추가 방법:**

```bash
# HTTP(리모트) 서버 추가 (권장)
claude mcp add --transport http github https://api.github.com/mcp

# SSE 서버 추가
claude mcp add --transport sse sentry https://mcp.sentry.dev/sse

# stdio 서버 추가 (로컬 실행)
claude mcp add --transport stdio context7 -- npx -y @upstash/context7-mcp

# 서버 목록 확인
claude mcp list

# Claude Code 내 상태 확인
/mcp
```

**스코프 구분:**

| 스코프 | 대상 | 저장 위치 | 용도 |
|---|---|---|---|
| `local` (기본값) | 현재 프로젝트 | `.claude.json` | 프로젝트 고유 도구 |
| `project` | 프로젝트 공유 | `.mcp.json` | 팀에서 공유할 도구 |
| `user` | 전체 프로젝트 공통 | `~/.claude.json` | 항상 사용할 도구 |

---

## **정번 MCP 서버:**

**Tier 1 — 거의 필수급**

**1. GitHub MCP Server** 는 GitHub 공식 MCP 서버다. 리포지토리 관리, PR 생성, Issue 조작, CI/CD 워크플로우를 Claude Code에서 직접 조작할 수 있다. OAuth 인증 지원으로 API 키 수동 관리도 불필요하다.

```bash
claude mcp add --transport sse github https://api.github.com/mcp
```
가능한 것: PR 생성·리뷰, Issue 관리, 브랜치 조작, CI/CD 상태 확인

**2. Context7 MCP** 는 라이브러리나 프레임워크의 최신 문서를 프롬프트에 자동 주입해준다. Claude Code가 오래된 정보를 기반으로 잘못된 코드를 생성하는 문제를 방지할 수 있어 효과가 크다.

```bash
claude mcp add context7 -s user -- npx -y @upstash/context7-mcp
```
가능한 것: 최신 API 문서를 참조한 코드 생성, 버전 간 차이 파악

**3. Playwright MCP** 는 브라우저 조작 및 테스트를 Claude Code에서 자동화할 수 있다. E2E 테스트 작성이나 UI 스크린샷 취득에 유용하다.

```bash
claude mcp add playwright -s project -- npx -y @playwright/mcp@latest
```
가능한 것: E2E 테스트 자동화, 스크린샷 취득, 웹 스크래핑

**4. Sentry MCP** 는 Sentry와 연계하여 프로젝트의 에러·성능 문제를 가져와 Claude Code에서 바로 수정까지 진행할 수 있다.

```bash
claude mcp add --transport sse sentry https://mcp.sentry.dev/sse
```
가능한 것: 에러 목록 취득, 스택 트레이스 분석, 수정 코드 생성

---

## **Tier 2 — 용도에 따라 추가할 것들**

**5. Firecrawl** 은 임의의 웹사이트를 깔끔한 Markdown 또는 LLM이 다루기 쉬운 형식으로 변환한다. JS 렌더링이 필요한 페이지에도 대응한다. 가능한 것: 웹 페이지 취득·변환, 사이트 크롤, 경쟁 조사

**6. PostgreSQL MCP** 는 자연어로 SQL 쿼리를 실행할 수 있다. "지난달 매출을 카테고리별로 집계해줘"라고 말하기만 해도 쿼리가 실행된다. 가능한 것: 자연어→SQL 변환, 데이터 분석, 스키마 확인

**7. Linear MCP** 는 Linear 티켓을 코딩 플로우 안에서 직접 조작한다. 티켓 취득→구현→상태 업데이트가 끊김 없이 연결된다. 가능한 것: 티켓 취득·생성·업데이트, 스프린트 관리, 서브태스크 분해

**8. Brave Search MCP** 는 Claude Code에 웹 검색 기능을 추가한다. 최신 정보 조사나 팩트체크에 활용한다.

```bash
claude mcp add brave-search -s user -- npx -y @anthropic/mcp-brave-search
```

**9. Memory MCP / Memory Bank** 는 지식 그래프 기반 영구 메모리다. 장기 프로젝트에서 과거 대화나 결정 사항을 Claude Code에 기억시켜 둘 수 있다.

---

## **기타 주목할 MCP 서버:**

| 서버명 | 용도 | 한마디 |
|---|---|---|
| Markitdown MCP | PDF/pptx 등 → Markdown 변환 | 문서 읽어들이기에 편리 |
| YouTube MCP | YouTube 자막 취득·요약 | 영상 내용 조사에 유용 |
| Figma MCP | 디자인 데이터 → 코드 변환 | Figma 디자인에서 컴포넌트 생성 |
| Slack MCP | Slack 연계 | 채널 참조·메시지 전송 |
| Backlog MCP | Backlog 연계 | 일본 팀에서 인기 |
| AWS MCP Servers | AWS 각종 서비스 연계 | CDK/Bedrock/S3 등 |
| Firebase MCP | Firebase 연계 | 모바일·웹 개발용 |
| Zapier MCP | 멀티 서비스 자동화 | 500개 이상 서비스 연계 허브 |
| Puppeteer MCP | 헤드리스 브라우저 | 스크래핑·UI 확인 |

---

## **정번 플러그인:**

2025년 10월 플러그인 시스템 공개 이후, Claude Code에서는 슬래시 커맨드, 서브에이전트, 훅, MCP 서버를 패키지로 배포·공유할 수 있게 되었다.

**메모리 계열 (현재 가장 주목받는 영역):**

Claude Code를 사용하다 많은 사람이 부딪히는 것이 **"세션 간 컨텍스트 소실"** 문제다. 어제 그렇게 설명했던 프로젝트 구성도 세션을 닫으면 전부 사라진다. 매일 아침 "DB는 MySQL이 아니라 Postgres야"라고 다시 설명해야 하는 소위 "할아버지 방금 드셨잖아요 문제"는 개발자들의 공통 고민이다. 2026년에 들어서 이 문제를 해결하는 메모리 계열 플러그인이 급격히 주목받고 있다.

**Claude-Mem (화제 폭발 중)** — GitHub ★20,000 초과, 2026년 2월 기준 가장 상승세인 플러그인이다.

세션 중 Claude가 수행한 모든 도구 사용을 자동 캡처하고, Agent SDK로 AI 압축하여 시맨틱 요약을 생성한다. 다음 세션 시작 시 관련 컨텍스트를 자동으로 주입해준다.

```bash
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

특징은 다음과 같다:
- **5가지 라이프사이클 훅**으로 자동 기록 (SessionStart → UserPromptSubmit → PostToolUse → Summary → SessionEnd)
- **3단계 점진적 검색**으로 토큰 효율 10배 향상 (수동 컨텍스트 관리 대비)
- SQLite + Chroma 벡터DB 기반 시맨틱 검색
- Web UI 뷰어 (`http://localhost:37777`)에서 과거 기록 열람·검색 가능
- **Endless Mode** (베타): 장시간 세션을 위한 바이오미메틱 메모리 아키텍처

적합한 케이스: 수 주~수 개월에 걸친 장기 프로젝트, 복잡한 코드베이스, 세션이 자주 중단되는 환경이다.

라이선스는 AGPL-3.0이다. 기업 환경에서는 카피레프트 조항을 반드시 확인해야 한다.

> 시작점은 먼저 표준 `CLAUDE.md`로 운용해보고, 컨텍스트 소실이 불편해지면 Claude-Mem을 도입하는 것이 좋다. 단순한 프로젝트라면 `CLAUDE.md`만으로도 충분한 경우가 많다. 그 외 Supermemory(클라우드형)나 Memory Bank(파일 기반형)도 선택지다.

---

## **개발 워크플로우 계열 (급성장 중):**

**Superpowers (★43,000 초과)** — 개발 방법론을 통째로 플러그인으로 만든 것이다.

2026년 1월 Anthropic 공식 마켓플레이스에 채택되었다. Superpowers는 단순한 도구 모음이 아니라 **소프트웨어 개발 워크플로우 자체**를 Claude Code에 내장하는 플러그인이다. 다음 7단계 페이즈로 구성된다:

1. **브레인스토밍** — 바로 코드를 작성하지 않고 대화로 요건을 깊이 파헤침
2. **상세 설계 사양 수립**
3. **구현 계획 작성**
4. **TDD (테스트 주도 개발)**
5. **서브에이전트 주도 개발**
6. **코드 리뷰**
7. **통합·검증**

```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

사용 예시:
```bash
# Superpowers 워크플로우 활성화
/using-superpowers

# 계획 수립
/superpowers:write-plan

# 계획 실행 (서브에이전트가 병렬로 동작)
/superpowers:execute-plan
```

일반적으로 Claude Code를 사용하면 "일단 코드를 작성 → 불완전 → 수정 반복"이 되기 쉽지만, Superpowers는 **계획→테스트→구현 순서를 강제**하기 때문에 출력 품질이 눈에 띄게 높아진다. 완전 무료 OSS (MIT 라이선스)이며, Anthropic 제품이 아닌 Jesse Vincent 씨의 커뮤니티 프로젝트다.

**Ralph Wiggum Loop (Anthropic 공식 리포지토리 입성)** — "완료할 때까지 자신의 출력을 자신에게 계속 먹이는" 자율 루프다.

심슨즈의 Ralph Wiggum에서 이름을 딴 이 기법은 Geoffrey Huntley 씨가 고안했다. 내용은 **"while true의 bash 루프로 AI에게 자신의 출력을 계속 먹이는"** 힘 기반 방식이다.

Claude Code는 "이 정도면 괜찮겠지"라고 판단한 시점에 멈추는 경향이 있다. Ralph Loop는 그 조기 종료를 훅으로 방지하여, 태스크가 진짜 완료될 때까지 몇 번이든 반복시킨다.

```bash
# 1번 실행하면 끝. 이후 Claude Code가 자율적으로 루프한다
/ralph-loop "Feature X를 구현해줘" --max-iterations 20 --completion-promise "DONE"

# Claude Code가 자동으로
# 1. 태스크를 수행
# 2. 종료하려 함
# 3. Stop 훅이 종료를 차단
# 4. 동일한 프롬프트를 재투입
# 5. 완료 조건을 만족할 때까지 반복
```

Y Combinator 스타트업에서 널리 사용되고 있으며, Claude Code의 창시자인 Boris Cherny 씨도 사용을 공언하고 있다. Huntley 씨에 따르면 AI 코딩은 **시간당 약 $10의 컴퓨트 비용**으로 돌릴 수 있다고 한다. 루프는 토큰을 대량으로 소비하므로 반드시 `--max-iterations`로 상한을 설정하고, 막혔을 때의 지시도 프롬프트에 포함시킬 것을 권장한다.

---

## **코드 리뷰·품질 관리 계열:**

**code-review / pr-review-toolkit** 은 여러 전문 에이전트(버그 검출, 테스트 분석, 타입 설계 분석, 코드 품질 등)를 병렬로 실행하여 PR 리뷰를 자동화한다. `/code-review`나 `/pr-review-toolkit:review-pr`로 기동한다.

**TDD Guard** 는 테스트 주도 개발을 강제하는 플러그인이다. 다국어 대응이며 검증 규칙 커스터마이징도 가능하다.

---

**개발 지원 계열:**

**dev-toolkit** 은 보안 스캔(`/security-scan`), 테스트 실행(`/test`), 코드 리뷰 등을 묶은 종합 툴킷이다.

**ccpm** 은 Claude Code용 프로젝트 관리 도구다. GitHub Issues + Git worktree를 조합하여, 태스크 관리와 브랜치 운용을 Claude Code 내에서 완결시킨다.

---

## **유틸리티 계열:**

**CC Usage** 는 토큰 사용량과 API 비용을 실시간으로 표시하여 비용 관리에 편리하다.

**Context Logger** 는 세션 중 Claude의 행동을 자동 기록·압축하여, 다음 세션에 관련 컨텍스트를 주입한다.

---

## **주변 도구·확장:**

| 도구명 | 카테고리 | 설명 |
|---|---|---|
| **Repomix** | 컨텍스트 준비 | 리포지토리 전체를 1개 파일로 묶어 LLM에 넘길 수 있는 형식으로 변환 |
| **claude-code.nvim** | 에디터 통합 | Neovim과의 통합 |
| **Cursor** | 에디터 통합 | VS Code 파생. 네이티브 MCP 지원 |
| **Cline** | 에디터 통합 | VS Code 확장. MCP 대응 AI 코딩 |
| **Claudia** | GUI | Claude Code용 GUI 툴킷. 에이전트 관리·사용량 분석 |
| **ccflare** | 대시보드 | 사용량 Web UI 대시보드 |
| **CC Statusline** | 터미널 표시 | 상태 표시줄 커스터마이징 (컨텍스트 사용률 등 표시) |

---

## **추천 초기 셋업:**

**최소 구성 (개인 개발):**

```bash
# 최신 문서 참조
claude mcp add context7 -s user -- npx -y @upstash/context7-mcp

# 브라우저 테스트
claude mcp add playwright -s project -- npx -y @playwright/mcp@latest

# 웹 검색
claude mcp add brave-search -s user -- npx -y @anthropic/mcp-brave-search
```

**팀 개발용 추가 구성:**

```bash
# GitHub 연계
claude mcp add --transport sse github https://api.github.com/mcp

# 에러 모니터링
claude mcp add --transport sse sentry https://mcp.sentry.dev/sse

# 프로젝트 관리 (Linear의 경우)
claude mcp add --transport sse linear https://mcp.linear.app/sse
```

**CLAUDE.md에 써두면 편리한 설정 예시:**

```markdown
# 기본 방침
- 반드시 한국어로 응대해주세요
- 조사나 디버깅에는 서브에이전트를 활용하여 컨텍스트를 절약해주세요
- 중요한 결정 사항은 정기적으로 마크다운 파일에 기록해주세요

# 코드 규약
- TypeScript 사용
- 테스트는 Vitest로 작성
- 커밋 메시지는 한국어로 간결하게
```

---

## **Tips — 컨텍스트 절약 요령:**
MCP 서버나 웹 검색을 많이 사용하면 대량의 정보가 컨텍스트를 소비한다. 다음이 효과적이다:

- **서브에이전트에게 맡기기** — 조사 태스크를 서브에이전트에 위임하고, 요약만 메인 세션에 돌려받는다
- **MCP Tool Search 활성화** — 필요할 때만 도구를 지연 로딩하여 컨텍스트 사용량을 최대 95% 절감
- **스코프를 적절히 나누기** — 전체 프로젝트에서 사용하는 것은 `user`, 특정 프로젝트만의 것은 `local` 또는 `project`로 설정

---

## **정리 — 에코시스템 3층 구조:**

| 레이어 | 역할 | 대표 예시 |
|---|---|---|
| **MCP 서버** | 외부 도구·데이터 소스와의 연결 | GitHub, Context7, Playwright, Sentry |
| **플러그인** | 워크플로우 커스터마이징·공유 | Superpowers, Claude-Mem, Ralph Loop |
| **주변 도구** | 에디터 통합·시각화 | Repomix, claude-code.nvim, Claudia |

2026년 2월 현재 트렌드는 다음과 같다:
1. **메모리 기반** — Claude-Mem의 폭발적 인기가 보여주듯 "세션 간 기억"이 가장 중요한 테마
2. **구조화 워크플로우** — Superpowers처럼 "바로 코드를 작성하지 않는" 방식이 확산
3. **자율 루프** — Ralph Loop로 대표되는 "완료까지 자율적으로 반복하는" 패턴
4. **업종 특화** — Cowork 플러그인이 개척한 코딩 이외 영역으로의 전개

우선 Context7 + Playwright + GitHub부터 시작하고, 필요에 따라 Superpowers나 Claude-Mem을 추가하는 것을 추천한다.  