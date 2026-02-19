# CLAUDE.md를 잘 쓰는 방법
[출처](https://zenn.dev/farstep/articles/how-to-write-a-great-claude-md)  
  

## **CLAUDE.md란 무엇인가:**
CLAUDE.md는 프로젝트 루트에 배치하는 Markdown 파일이다. Claude Code는 세션 시작 시 이 파일을 자동으로 읽어들여, 기술된 내용을 대화의 컨텍스트에 포함시킨다. LLM은 세션 간 기억을 갖지 않기 때문에, CLAUDE.md에 코딩 규약이나 빌드 커맨드를 기술해 두면 매번 "프로젝트를 이해한 상태"에서 작업을 시작할 수 있다.


## **CLAUDE.md의 내부 메커니즘:**
"짧게 써야 한다", "보편적인 내용만 써야 한다"고 말해지는 이유는 기술적인 제약에 기인한다.

**컨텍스트 윈도우의 제약**

Claude Code의 모든 동작은 **컨텍스트 윈도우**라 불리는 유한한 토큰 공간 안에서 이루어진다. 이 윈도우에는 시스템 프롬프트, CLAUDE.md의 내용, 툴 정의, 대화 이력, 파일 읽기 결과 등이 저장되며, CLAUDE.md의 기술이 길어질수록 실제 작업에 사용할 수 있는 공간이 줄어든다. 공식 문서에서는 CLAUDE.md를 **약 500행 이내**로 유지하도록 권장하고 있다.

**주입 메커니즘과 "무시되는" 리스크**

CLAUDE.md의 내용은 세션 중 **모든 리퀘스트**에 포함된다(공식 문서에서 컨텍스트 비용이 "Every request"라고 명기되어 있다). 서브에이전트 호출 시에도 CLAUDE.md는 계승되기 때문에, 기술량은 컨텍스트 비용 전체에 영향을 미친다.

나아가, 주입 시 다음과 같은 주의문이 부가된다는 것이 사용자의 API 트래픽 조사를 통해 확인되었다([GitHub Issue #22309](https://github.com/anthropics/claude-code/issues/22309), [#7571](https://github.com/anthropics/claude-code/issues/7571)).

> IMPORTANT: this context may or may not be relevant to your tasks.
> You should not respond to this context unless it is highly relevant to your task.

이 주의문으로 인해, **CLAUDE.md의 내용이 태스크와 무관하다고 Claude가 판단할 경우, 지시가 무시될 가능성이 있다.** 공식 문서에서도 "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"라고 [경고하고 있다](https://code.claude.com/docs/en/best-practices). 특정 태스크에서만 사용하는 지시를 대량으로 포함시키면, 파일 전체가 "관련성이 낮은 컨텍스트"로 간주될 리스크가 높아진다.

**명령 예산의 상한**

LLM이 일관되게 따를 수 있는 명령 수에는 상한이 있다. 서드파티 분석([HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md), [arXiv 논문](https://arxiv.org/pdf/2507.11538))에 따르면, 프론티어 모델에서 약 200개 정도로 알려져 있다. 명령 수가 늘어나면 특정 명령만 무시되는 것이 아니라, **모든 명령의 준수율이 일률적으로 저하된다**고 보고되어 있다.

동일한 원칙은 [Builder.io](https://www.builder.io/blog/claude-md-guide)에서도 소개되어 있다. Anthropic은 구체적인 상한을 공표하지 않았지만, 기술은 가능한 한 간결하게 유지하는 것이 중요하다.

  
## **메모리 계층의 전체상:**
CLAUDE.md는 Claude Code의 메모리 시스템의 일부다. 복수의 메모리 타입이 서로 다른 스코프로 존재한다.

**6가지 메모리 타입**

| 메모리 타입 | 배치 위치 | 용도 | 공유 범위 |
|---|---|---|---|
| 매니지드 폴리시 | OS 고유의 시스템 경로 | 조직 전체의 규칙 (IT/DevOps 관리) | 조직의 모든 유저 |
| 프로젝트 메모리 | `./CLAUDE.md` | 팀 공유의 프로젝트 지시 | git을 통한 팀 멤버 |
| 프로젝트 룰 | `.claude/rules/*.md` | 모듈화된 토픽별 지시 | git을 통한 팀 멤버 |
| 유저 메모리 | `~/.claude/CLAUDE.md` | 전 프로젝트 공통의 개인 설정 | 본인만 (전 프로젝트) |
| 로컬 프로젝트 메모리 | `./CLAUDE.local.md` | 프로젝트 고유의 개인 설정 | 본인만 (현 프로젝트) |
| Auto Memory | `~/.claude/projects/<project>/memory/` | Claude가 자동으로 기록하는 학습 메모 | 본인만 (프로젝트 단위) |

**읽기 타이밍과 우선순위**

- **시작 시 전문 읽기**: 작업 디렉터리에서 상위로 재귀적으로 발견되는 CLAUDE.md, CLAUDE.local.md, `.claude/rules/` 하위의 파일
- **온디맨드 읽기**: 자식 디렉터리의 CLAUDE.md (Claude가 해당 디렉터리의 파일을 읽었을 때 읽기)
- **앞부분만 읽기**: Auto Memory의 `MEMORY.md`는 앞 200행만

우선순위는 "더 구체적인 것이 우선"이다. `CLAUDE.local.md`는 `.gitignore`에 자동 추가되기 때문에, 개인 설정의 배치 장소로 적합하다.

본 글에서는 이후부터 가장 일반적으로 사용되는 **프로젝트 메모리(`./CLAUDE.md`)**를 중심으로 해설한다.

  
## **써야 할 것 · 쓰지 말아야 할 것:**
컨텍스트의 제약을 감안하면, CLAUDE.md에 쓰는 내용은 엄선해야 한다. 판단 기준은 단순하다.

> **"이 줄을 삭제하면, Claude가 실수를 저지르는가?"**
>
> 답이 No라면, 그 줄은 삭제 후보다.

**써야 할 내용**

| 항목 | 왜 필요한가 | 예시 |
|---|---|---|
| Claude가 추측할 수 없는 커맨드 | 프로젝트 고유의 커맨드는 코드에서 읽어낼 수 없다 | `pnpm test:e2e -- --headed` |
| 디폴트와 다른 코드 스타일 | 명시하지 않으면 일반적인 관례를 따라버린다 | "default export가 아닌 named export를 사용" |
| 테스트 방법과 선호하는 테스트 러너 | 테스트 전략은 코드에서 읽어내기 어렵다 | "Vitest를 사용. 단위 테스트를 우선 실행" |
| 브랜치 명명 규칙이나 PR 규약 | 리포지터리 고유의 워크플로 | "브랜치명은 `feat/`, `fix/` 프리픽스" |
| 프로젝트 고유의 아키텍처 결정 | 코드만으로는 설계 의도가 전달되지 않는다 | "API 라우트는 반드시 `/api/v1` 프리픽스를 사용" |
| 환경의 버릇이나 필수 환경 변수 | 문서화되지 않는 경우가 많다 | "로컬 개발에는 Redis가 필요" |
| 비자명한 주의사항 | 과거의 실패에서 얻은 프로젝트 고유의 지식 | "Stripe의 webhook 핸들러는 서명 검증이 필수" |

**쓰지 말아야 할 내용**

| 항목 | 왜 불필요한가 | 대체 수단 |
|---|---|---|
| 코드를 보면 알 수 있는 것 | Claude는 코드를 직접 읽을 수 있다 | — |
| 언어의 표준적인 관례 | Claude는 이미 알고 있다 | — |
| 상세한 API 문서 | 너무 길어서 컨텍스트를 압박한다 | URL 링크나 `@` 임포트로 참조 |
| 자주 바뀌는 정보 | 금방 낡아버린다 | Auto Memory에 맡긴다 |
| 긴 설명이나 튜토리얼 | 명령 예산을 대량으로 소비한다 | 별도 파일로 분리해서 참조 |
| 파일별 상세한 설명 | Claude가 스스로 탐색할 수 있다 | — |
| "깔끔한 코드를 쓴다" 등의 자명한 지시 | 명령 예산의 낭비 | — |
| 코드 스타일의 상세 규칙 | 린터가 더 확실하게 처리할 수 있다 | ESLint, Prettier 등에 위임 |

스타일 규칙은 린터에 맡기고, CLAUDE.md에는 "코드 스타일은 ESLint와 Prettier로 강제. 린터 에러가 나면 수정할 것"이라고 한 줄 쓰면 충분하다.

---

## **효과적인 구성과 포맷:**

**/init으로 출발점 만들기**

`/init` 커맨드를 실행하면, Claude Code가 프로젝트 구조를 분석해서 CLAUDE.md의 템플릿을 자동 생성한다. 단, 자동 생성된 내용에는 자명한 정보나 불필요한 기술이 포함되는 경우가 많기 때문에, **출발점으로 삼되, 반드시 수작업으로 꼼꼼히 검토해야 한다.**

**권장되는 섹션 구성**

1. **프로젝트 개요** — 1~2행으로 프로젝트의 전체상을 나타낸다
2. **코드 스타일** — 디폴트와 다른 규칙만
3. **커맨드** — 빌드, 테스트, 린트, 배포 등
4. **아키텍처** — 주요 디렉터리와 그 역할
5. **주의사항** — 프로젝트 고유의 함정

**구체적인 예시 (ShopFront 프로젝트)**

Next.js의 EC 앱 "ShopFront"를 예로 든 실용적인 CLAUDE.md다. 본 글에서는 이 프로젝트를 일관된 구체적 예시로 사용한다.

```markdown
# ShopFront

Next.js 16의 EC 앱. App Router, Turbopack, Stripe 결제, Prisma ORM, Tailwind CSS를 사용.

## 코드 스타일

- TypeScript strict 모드. `any` 타입은 금지
- default export가 아닌 named export를 사용
- CSS는 Tailwind 유틸리티 클래스만. 커스텀 CSS 파일은 만들지 않는다
- 코드 스타일은 ESLint와 Prettier로 강제. 린터 에러가 나면 그 출력에 따라 수정할 것

## 커맨드

- `pnpm dev`: 개발 서버 시작 (Turbopack, 포트 3000)
- `pnpm test`: Vitest에 의한 단위 테스트
- `pnpm test:e2e`: Playwright에 의한 e2e 테스트
- `pnpm lint`: ESLint 실행
- `pnpm db:migrate`: Prisma 마이그레이션 실행
- `pnpm db:seed`: 테스트 데이터 투입

## 아키텍처

- `/app`: App Router의 페이지와 레이아웃
- `/app/api`: API 라우트 (모두 `/api/v1` 프리픽스)
- `/components/ui`: 재사용 가능한 UI 컴포넌트
- `/lib`: 유틸리티와 공유 로직
- `/prisma`: 데이터베이스 스키마와 마이그레이션

## 주의사항

- IMPORTANT: Next.js 16에서는 데이터 취득이 기본적으로 캐시되지 않는다. 상품 데이터 등의 정적 데이터에는 `"use cache"`를 명시적으로 부여할 것
- Stripe의 webhook 핸들러(`/app/api/webhooks/stripe`)는 서명 검증이 필수. 이 라우트에는 `"use cache"`를 사용하지 않는다
- 상품 이미지는 Cloudinary에 저장. 로컬 스토리지는 사용하지 않는다
- 인증 플로의 상세는 @docs/auth-flow.md를 참조
```

약 30행으로, Claude가 코드에서 읽어낼 수 없는 정보에만 집중하고 있다.

**기술상의 팁**

- 공식 문서에서는 **약 500행 이내**가 권장된다([Best Practices](https://code.claude.com/docs/en/best-practices)). 짧을수록 효과적이며, 대규모 프로젝트에서는 Progressive Disclosure로 분리한다
- "IMPORTANT", "YOU MUST" 등의 강조는 정말로 중요한 수 항목에 한정한다. 모든 것을 강조하면 효과가 희석된다
- 긴 세션에서는 Claude Code가 자동으로 대화를 요약(컴팩션)한다. "컴팩션 시에는 변경된 파일 목록과 테스트 커맨드를 유지할 것"과 같은 지시로 정보 손실을 방지할 수 있다

  
## **컨텍스트의 단계적 개시 (Progressive Disclosure):**
**Progressive Disclosure(단계적 개시)** 는 정보를 "항상 필요한 것"과 "필요할 때만 읽어들이는 것"으로 분리하는 패턴이다. Anthropic의 공식 문서에서는 [Skills 아키텍처에서의 progressive disclosure](https://code.claude.com/docs/en/skills)가 기재되어 있으며, 이하는 그 사고방식을 CLAUDE.md 운용 전체에 확장한 것이다.

**3층 아키텍처**

| 층 | 배치 위치 | 읽기 타이밍 | 기술하는 내용 |
|---|---|---|---|
| Layer 1 | `./CLAUDE.md` | 매 세션 자동 | 프로젝트 개요, 필수 커맨드, Layer 2·3으로의 포인터 |
| Layer 2 | `docs/*.md`, `.claude/rules/*.md` | 태스크에 관련된다고 판단되었을 때 | 함정 모음, 테스트 전략, API 규약 |
| Layer 3 | `.claude/skills/`, `.claude/agents/` | 필요 시 기동 (자동 또는 수동) | 전문 도메인 지식, 반복 워크플로 |

**Layer 2의 구현**

방법 1: `.claude/rules/` 디렉터리

`.claude/rules/` 하위의 Markdown 파일은 프로젝트 메모리로 자동으로 읽혀진다. YAML 프론트매터로 **경로 지정**을 하면, 특정 파일을 다룰 때만 적용된다.

```markdown
---
paths:
  - "app/api/**/*.ts"
---

# API 엔드포인트의 규약

- 모든 엔드포인트에서 입력 유효성 검사를 실시할 것
- 에러 응답은 통일 포맷(`lib/api-error.ts`)을 사용
- 페이지네이션 대응의 리스트에는 `limit`과 `offset` 파라미터를 포함한다
```

방법 2: `@` 임포트

CLAUDE.md에서 `@` 구문으로 파일을 참조할 수 있다. 상대 경로·절대 경로에 대응하며, 재귀적인 임포트는 최대 5계층까지 가능하다.

```markdown
## 상세 문서

IMPORTANT: 태스크 시작 전에, 관련 문서를 확인할 것.

- 인증 플로의 상세: @docs/auth-flow.md
- Stripe 연동의 주의점: @docs/stripe-gotchas.md
- 테스트 전략: @docs/testing-strategy.md
```

`.claude/rules/`는 경로 지정에 의한 자동 적용이 가능해서 팀 공유의 표준 규약에, `@` 임포트는 특정 토픽의 상세 문서 참조에 적합하다.

**Layer 3의 구현**

스킬은 `.claude/skills/` 하위에 `SKILL.md`로 배치하는, 온디맨드의 도메인 지식이나 워크플로 정의다. 공식 문서에서는 스킬의 내용을 **Reference content(참조 콘텐츠)**와 **Task content(태스크 콘텐츠)**의 [2종류로 분류](https://code.claude.com/docs/en/skills)하고 있으며, 각각 기동 방법이 다르다.

패턴 1: Claude가 자동 기동하는 스킬 (단계적 개시)

디폴트 설정의 스킬에서는 description이 항상 컨텍스트에 존재하며, Claude가 태스크와의 관련성을 판단해서 필요할 때 자동으로 풀 컨텐츠를 읽어들인다. 코딩 규약이나 도메인 지식 등 부작용이 없는 참조 정보에 적합하다.

```markdown
---
name: api-conventions
description: API 엔드포인트의 설계 규약. API 관련 작업 시 사용.
---

API 엔드포인트를 만들거나 수정할 때의 규칙:

- RESTful 명명 규칙을 따른다
- 에러 응답은 통일 포맷(`lib/api-error.ts`)을 사용
- 페이지네이션 대응의 리스트에는 `limit`과 `offset`을 포함한다
```

패턴 2: 유저가 수동 기동하는 스킬 (워크플로 제어)

`disable-model-invocation: true`를 설정한 스킬은, 유저가 `/name`으로 명시적으로 기동한다. description도 컨텍스트에 포함되지 않기 때문에, Claude는 이 스킬의 존재를 인식하지 못한다. 커밋이나 PR 작성 등 **부작용을 수반하는 워크플로**에 적합하다.

```markdown
---
name: fix-issue
description: GitHub의 Issue를 분석하여 수정한다
disable-model-invocation: true
---

$ARGUMENTS로 지정된 GitHub Issue를 분석하고 수정한다.

1. `gh issue view`로 Issue 상세를 취득
2. 문제를 이해하고 관련 파일을 검색
3. 수정을 구현하고 테스트를 실행
4. 커밋하고 PR을 작성
```

패턴 1은 description이 항상 컨텍스트에 존재하고 (소비용), Claude가 필요 시 풀 로드하는 단계적 개시다. 패턴 2는 description도 컨텍스트에 포함되지 않아서 (제로 비용), 유저의 명시적인 기동으로만 발동한다.

서브에이전트는 `.claude/agents/` 하위에 정의하는, 독자적인 컨텍스트 윈도우로 동작하는 전문 에이전트다. 메인 대화의 컨텍스트를 소비하지 않고 대량의 파일을 읽는 조사 태스크에 적합하다.

**ShopFront 프로젝트에서의 적용 예시**

```
shopfront/
├── CLAUDE.md                          # Layer 1: 30행의 개요 (앞서의 예)
├── docs/
│   ├── auth-flow.md                   # Layer 2: 인증 플로 상세
│   ├── stripe-gotchas.md              # Layer 2: Stripe 연동의 주의점
│   └── testing-strategy.md            # Layer 2: 테스트 전략
├── .claude/
│   ├── rules/
│   │   └── api-endpoints.md           # Layer 2: API 규칙 (경로 지정 포함)
│   ├── skills/
│   │   ├── api-conventions/SKILL.md   # Layer 3: API 규약 (Claude 자동 기동)
│   │   └── fix-issue/SKILL.md         # Layer 3: Issue 수정 (유저 수동 기동)
│   └── agents/
│       └── security-reviewer.md       # Layer 3: 보안 리뷰
```

  
## **안티패턴:**

**1. 너무 많이 담기**

500행을 크게 초과하는 파일은 컨텍스트를 압박하고 중요한 규칙이 묻혀버린다. 각 행에 "삭제하면 Claude가 실수하는가?"를 물어보고, No라면 삭제한다. 약 500행을 초과한다면 Progressive Disclosure로 분리해야 한다.

**2. 린터의 대체로 활용하기**

코드 스타일의 상세 규칙을 열거하면 명령 예산을 대량으로 소비한다. 스타일 강제는 ESLint·Prettier 등에 위임하고, Hooks로 포맷터를 자동 실행하는 구조도 유효하다.

**3. /init의 출력을 그대로 사용하기**

자동 생성된 내용에는 자명한 정보나 중복된 기술이 포함되는 경우가 많아서, 검토 없이 사용하면 컨텍스트의 질을 저하시킨다. `/init`은 출발점으로 삼되, 반드시 수작업으로 꼼꼼히 검토해야 한다.

**4. 부정만으로 이루어진 제약**

금지 사항만으로는 대신 무엇을 해야 하는지가 불명확하다. 대체 수단을 함께 기재해야 한다.

```markdown
# 나쁜 예: `--foo-bar` 플래그는 사용하지 말 것
# 좋은 예: `--baz` 플래그를 사용할 것 (`--foo-bar`는 비권장)
```

**5. 태스크 고유의 지시를 글로벌에 배치하기**

특정 태스크에서만 사용하는 지시를 최상위 레벨에 쓰면 "무시될 리스크"를 높인다. `.claude/rules/`에 경로 지정과 함께 배치하거나, 별도 문서로 빼내야 한다.

  

## **운용과 지속적인 개선:**

**피드백 루프**

Claude가 실수를 하면 수정한 뒤 CLAUDE.md에 규칙을 추가하고 git에 커밋한다. 예를 들어 Claude가 `console.log`로 디버그를 했다면 "`lib/logger`를 사용할 것"이라고 추가한다. PR 리뷰에서의 규약 위반도 CLAUDE.md에 반영한다. 이 루프를 통해, 실제로 문제가 생긴 케이스만을 축적한 실용적인 문서로 성장한다.

**정기적인 점검**

몇 주에 한 번, `/memory` 커맨드로 CLAUDE.md를 재검토한다. 낡아진 지시나, 지시 없이도 Claude가 올바르게 수행할 수 있게 된 항목을 삭제한다.

**Auto Memory와의 역할 분담**

CLAUDE.md가 "당신이 Claude에게 전달하는 지시"인 것에 반해, Auto Memory는 "Claude가 스스로 학습한 메모"다. Claude가 작업 중에 발견하는 패턴은 Auto Memory에 맡기고, CLAUDE.md에서는 생략할 수 있다.



## **정리:**

CLAUDE.md의 설계 원칙을 4가지로 집약한다.

1. **최소의 고시그널 토큰 집합을 목표로 한다** — Claude가 코드에서 읽어낼 수 없는 정보만을, 가장 간결한 표현으로 쓴다
2. **컨텍스트의 질을 의식한다** — 명령 수가 늘어날수록 준수율은 저하된다. 약 500행 이내로 유지하고, 각 행의 필요성을 계속해서 묻는다
3. **보편적으로 적용되는 내용에 한정한다** — 태스크 고유의 정보는 Progressive Disclosure로 분리한다
4. **피드백 루프로 키워나간다** — 처음부터 완벽한 것을 목표로 쓰는 것이 아니라, 실제 문제로부터 배워서 추가해 나간다

**체크리스트**

- 프로젝트 개요가 1~2행으로 기술되어 있다
- 코드 스타일은 디폴트와 다른 규칙만. 상세는 린터에 위임
- 빌드·테스트·린트 커맨드가 기재되어 있다
- 프로젝트 고유의 주의사항이 기재되어 있다
- 약 500행 이내로 수렴되어 있다 (짧을수록 효과적)
- 각 행이 "삭제하면 Claude가 실수하는가" 테스트에 합격한다
- 태스크 고유의 지시는 `.claude/rules/` 등으로 분리되어 있다

먼저 `/init`으로 템플릿을 생성하고, 불필요한 행을 삭제하며, 프로젝트 고유의 지식을 추가한다. 개발 중에 마주친 문제를 하나씩 추가해 나가는 피드백 루프가, 가장 좋은 CLAUDE.md를 만드는 방법이다.