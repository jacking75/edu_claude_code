# Anthropic 공식 Claude Code Plugin이 늘어났으므로, 다시 한번 살펴본다 (2026년 3월)
[출처](https://dev.classmethod.jp/articles/claude-plugins-official-view-2026-03/ )
  
  
**Claude Code Plugins 목록 (2026년 3월 12일 기준):**

**UI · 프론트엔드**

| 폴더명 | 개요 |
|---|---|
| [frontend-design](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/frontend-design) | AI의 일반적인 미학을 피한, 독특하고 프로덕션 품질의 프론트엔드 인터페이스 생성 |
| [playground](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/playground) | 비주얼로 설정 가능한 인터랙티브 HTML 플레이그라운드 (프롬프트 출력 기능 포함) |

**개발 관련**

| 폴더명 | 개요 |
|---|---|
| [agent-sdk-dev](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/agent-sdk-dev) | Claude Agent SDK 애플리케이션 (Python/TypeScript)의 생성과 검증을 수행하는 포괄적인 플러그인 |
| [commit-commands](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/commit-commands) | Git 커밋·푸시·PR 생성을 간단한 커맨드로 자동화 |
| [feature-dev](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/feature-dev) | 7단계 워크플로우 (발견→탐색→설계→구현→확인→배포→모니터링)를 통한 구조화된 피처 개발 |
| [ralph-loop](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/ralph-loop) | Claude Code에서 자기 참조적인 AI 개발 루프를 구현하는 반복적 개발 기법 |

**코드 리뷰 · 품질 관리**

| 폴더명 | 개요 |
|---|---|
| [code-review](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-review) | 복수의 전문 에이전트를 사용한 PR 자동 코드 리뷰 (신뢰도 스코어에 의한 오탐 필터링) |
| [code-simplifier](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier) | 코드를 명확성·일관성·유지보수성을 위해 간결하게 정제하는 에이전트 |
| [pr-review-toolkit](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/pr-review-toolkit) | 코드 품질의 다양한 측면에 특화된 6개의 전문 리뷰 에이전트 (코멘트·테스트·에러 처리 등) |
| [security-guidance](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance) | 파일 편집 시 커맨드 인젝션·XSS·안전하지 않은 코드 패턴 등의 보안 문제를 경고 |

**Claude Code 플러그인 · 스킬 개발**

| 폴더명 | 개요 |
|---|---|
| [example-plugin](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/example-plugin) | 플러그인의 전체 옵션 기능을 보여주는 포괄적인 샘플 플러그인 |
| [plugin-dev](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/plugin-dev) | Hooks·MCP 통합·플러그인 구조·마켓플레이스 공개에 관한 7개의 전문 스킬을 갖춘 포괄적인 플러그인 개발 툴킷 |
| [skill-creator](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/skill-creator) | 스킬 신규 생성·기존 스킬 개선·스킬 성능 측정 및 평가 실행 |

**Claude Code 설정 · 관리**

| 폴더명 | 개요 |
|---|---|
| [claude-code-setup](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-code-setup) | 코드베이스를 분석하여 Hooks·Skills·MCP 서버 등의 Claude Code 자동화를 추천 |
| [claude-md-management](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-md-management) | CLAUDE.md 파일 유지보수 툴 (품질 감사·세션 학습 기록) |
| [hookify](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/hookify) | 대화 패턴 분석 또는 명시적 지시로부터 커스텀 훅을 간단하게 생성 |

**Claude Code 출력 스타일**

| 폴더명 | 개요 |
|---|---|
| [explanatory-output-style](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/explanatory-output-style) | 구현 선택과 코드베이스 패턴에 대해 교육적 인사이트를 제공하는 출력 스타일 |
| [learning-output-style](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/learning-output-style) | 인터랙티브 학습과 설명 기능을 결합한 출력 스타일 (사용자가 중요한 코드를 직접 작성하도록 유도) |

**LSP (Language Server Protocol)**

| 폴더명 | 개요 |
|---|---|
| [clangd-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/clangd-lsp) | C/C++ 언어 서버 (clangd)로 코드 보완·진단·포맷팅 기능 제공 |
| [csharp-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/csharp-lsp) | C# 언어 서버로 코드 보완과 진단 기능 제공 |
| [gopls-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/gopls-lsp) | Go 언어 서버로 코드 보완·리팩토링·분석 기능 제공 |
| [jdtls-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/jdtls-lsp) | Java 언어 서버 (Eclipse JDT.LS)로 코드 보완과 리팩토링 기능 제공 |
| [kotlin-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/kotlin-lsp) | Kotlin 언어 서버로 코드 보완·리팩토링·분석 기능 제공 |
| [lua-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/lua-lsp) | Lua 언어 서버로 코드 보완과 진단 기능 제공 |
| [php-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/php-lsp) | PHP 언어 서버 (Intelephense)로 코드 보완과 진단 기능 제공 |
| [pyright-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/pyright-lsp) | Python 언어 서버 (Pyright)로 정적 타입 체크와 코드 보완 기능 제공 |
| [ruby-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/ruby-lsp) | Ruby 언어 서버로 코드 보완과 분석 기능 제공 |
| [rust-analyzer-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/rust-analyzer-lsp) | Rust 언어 서버로 코드 보완과 분석 기능 제공 |
| [swift-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/swift-lsp) | Swift 언어 서버 (SourceKit-LSP)로 Swift 프로젝트의 코드 보완 기능 제공 |
| [typescript-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/typescript-lsp) | TypeScript/JavaScript 언어 서버로 go-to-definition·find-references·에러 체크 등 제공 |

---

**Plugin 도입:**

Claude Code를 시작하고, 아래 커맨드를 실행한 후 흐름에 따라 진행하면 된다.

```
/plugin
```

우선은 Marketplace에서 공식을 선택해보는 것이 좋다.

---

**개인적인 픽업:**

**Claude Code Setup**

코드베이스를 분석하여, 최적의 Claude Code 자동화를 제안해준다.

- https://claude.com/plugins/claude-code-setup
- https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-code-setup

**`/claude-automation-recommender`**

일부를 발췌하지만, 아래와 같이 여러 가지를 제안해준다. ("그건 딱히 원하지 않았는데……" 같은 제안도 있지만.) 주간, 월간 등 주기적으로 실행해도 재미있을 것 같다.

```
  Hooks

  현재 hooks가 미설정입니다. 아래 2가지가 가장 임팩트가 큽니다.

  1. Biome 자동 포맷 (PostToolUse)

  이유: Edit/Write 후에 자동으로 Biome 포맷이 실행되기 때문에, 수동으로 pnpm fix를 실행하는 수고가 없어집니다.

  .claude/settings.json에 추가:

  {
    "hooks": {
      "PostToolUse": [
        {
          "matcher": "Edit|Write",
          "hooks": [
            {
              "type": "command",
              "command": "npx @biomejs/biome check --write --unsafe $CLAUDE_FILE_PATH 2>/dev/null || true"
            }
          ]
        }
      ]
    }
  }

  2. 보호 파일 편집 차단 (PreToolUse)

  이유: .env 파일, pnpm-lock.yaml의 잘못된 편집을 방지합니다.

  {
    "hooks": {
      "PreToolUse": [
        {
          "matcher": "Edit|Write",
          "hooks": [
            {
              "type": "command",
              "command": "echo $CLAUDE_FILE_PATH | grep -qE '(\\.env|pnpm-lock\\.yaml)' && echo 'BLOCKED: This file should not be edited
  directly' && exit 1 || exit 0"
            }
          ]
        }
      ]
    }
  }
```

```
  Skills

  이미 8개의 스킬이 있습니다. 추가로 1개를 제안합니다.

  API 연결 체크리스트

  CLAUDE.md에 「API 연결 시의 체크리스트」가 있지만, 스킬로 만들어두면 확실하게 실행됩니다.

  .claude/skills/api-integration-checklist/SKILL.md:
  ---
  name: api-integration-checklist
  description: API 연결(목→실제 API) 완료 시의 체크리스트를 실행한다
  ---

  # API 연결 체크리스트

  API 연결 완료 시 아래를 확인:
  1. 생략
  2. 생략
  3. 생략
  4. 생략
```

```
  Subagents

  이미 xxx, yyy, zzz가 존재합니다.

  test-runner (테스트 실행 에이전트)

  이유: 코드 변경 후에 관련 테스트를 자동 실행하는 에이전트가 있으면, 변경의 영향 범위를 빠르게 확인할 수 있습니다. 패키지별로 pnpm --filter <pkg> test를
  실행하는 전용 에이전트입니다.

  .claude/agents/test-runner.md:
  ---
  name: test-runner
  description: 변경된 패키지의 테스트를 실행한다
  ---

  1. git diff로 변경된 파일을 특정
  2. 영향을 받는 패키지를 판정
  3. 해당 패키지의 테스트를 실행
  4. 결과를 요약하여 보고
```

---

**CLAUDE.md Management**

CLAUDE.md 파일의 유지보수 툴 (품질 감사·세션 학습 기록)이다.

- https://claude.com/plugins/claude-md-management
- https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-md-management

**`/claude-md-management:claude-md-improver`**

리포지토리의 CLAUDE.md 파일이나 룰 파일 등을 확인하고, 실제 코드와 정합성을 맞춰준다. 서브 리포지토리의 CLAUDE.md도 대상이다.

이것도 주간·월간 등 주기적으로 실행하면 좋을 것 같다. 다만, 기본적으로는 내용이 늘어나는 방향이라고 생각하기 때문에, 줄이는 기회도 필요하지 않을까 싶다.

**`/claude-md-management:revise-claude-md`**

현재 세션 내용을 되돌아보고, 누락된 컨텍스트 등을 파악하여 CLAUDE.md에 기재해준다.

Hooks로 실행시키면 재미있을 것 같다 (하지만 빈도가 너무 높을지도?).

```
⏺ 세션을 되돌아봤습니다. 아래의 교훈이 있습니다.

  제안하는 변경

  Update 1: ./CLAUDE.md

  Why: 생략

   ### 타입 기반 코딩 룰

   - 생략. TypeScript의 타입 시스템을 신뢰하고, 타입 정의에 기반하여 코드를 작성할 것.
  +- Zod의 `superRefine`으로 보장된 값을 서비스 레이어에서 사용할 경우, 생략.
  +- 생략.

  이번 세션에서 얻은 교훈은 이 2가지가 가장 중요합니다. 그 외의 배움 (테스트의 it.each화, createBody 헬퍼 설계 등)은 일반적인 프랙티스이기 때문에 CLAUDE.md에 추가할 정도는 아닙니다.

  이 변경을 적용해도 될까요?
```

---

**Skill Creator**

많은 사람들이 추천하고 있다.

- https://claude.com/plugins/skill-creator
- https://github.com/anthropics/claude-plugins-official/tree/main/plugins/skill-creator

**`/skill-creator`**

실제 실행 결과는 생략한다.

---

**Code Simplifier**

에이전트만 있다. 꽤 예전부터 도입하고 있다. 다만, 솔직히 "엄청 좋다!" 인지는 모르겠다…… (도입하지 않은 상태와 비교할 수 없기 때문에). 일단 도입해두어도 손해는 없다고 생각한다.

- https://claude.com/plugins/code-simplifier
- https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier

---

**시험해보지 않았지만 신경 쓰이는 Plugin:**

**Learning Output Style**

인간이 작성할 가치가 있는 코드라고 판단했을 때, Claude Code는 사용자에게 코드를 작성하도록 유도한다. 즉 코드를 작성하는 학습 기회를 얻을 수 있다. 처음 접하는 언어나 프레임워크, 학습 기회를 원하는 분, 개인 개발 등에서 사용해보면 좋을지도 모른다.

- https://claude.com/plugins/learning-output-style
- https://github.com/anthropics/claude-plugins-official/tree/main/plugins/learning-output-style   