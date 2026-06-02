# Claude Code를 사내에서 안전하게 쓰기 위한 5가지 가드레일 

  
## **가드레일 1: `.claudeignore` 로 기밀 파일 제외:**
가장 중요하다. Claude Code가 프로젝트 내 파일을 읽을 때, **`.claudeignore` 에 기재된 것은 무시**된다.

프로젝트 루트에 `.claudeignore` 를 만들고:

```
# 인증 정보
.env
.env.*
*.key
*.pem
credentials.json

# 기밀 데이터
secrets/
private/

# 고객 데이터
data/customers/
data/users/

# 로그 (개인 정보 포함 가능성)
logs/

# 백업
*.bak
*.dump
```

이것으로 Claude Code가 이 파일들을 **절대로 읽지 않게 된다.**

---

## **가드레일 2: CLAUDE.md 에 "절대 해서는 안 되는 것"을 명시:**
`CLAUDE.md` 에 **금지 사항을 반드시 기재할 것.**

```
# 금지 사항

- ✋ 환경 변수의 값을 출력하지 않는다 (`console.log(process.env)` 등)
- ✋ 인증 정보를 하드코딩하지 않는다
- ✋ 개인 정보 (이메일·이름·전화번호) 를 로그에 출력하지 않는다
- ✋ DB에 대해 `DELETE` / `DROP` / `TRUNCATE` 를 허가 없이 실행하지 않는다
- ✋ 프로덕션 환경 (`NODE_ENV === 'production'`) 에 대한 작업은 확인 없이 하지 않는다
- ✋ HTTP 요청에 인증 정보를 평문으로 포함하지 않는다
```

이것을 기재하는 것만으로도, Claude Code가 "하려다 멈추는" 확률이 극적으로 올라간다.

---

## **가드레일 3: Hooks 로 위험한 조작을 강제 차단:**
`.claude/settings.json` 으로 **특정 조작을 Hook으로 차단**할 수 있다.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash.*(rm -rf|DROP TABLE|TRUNCATE|DELETE FROM users)",
        "hooks": [{"type": "command", "command": "echo '❌ 위험한 명령어는 금지' && exit 1"}]
      },
      {
        "matcher": "Bash.*git push.*main",
        "hooks": [{"type": "command", "command": "echo '⚠️ main 브랜치로의 직접 push는 확인 후 수동으로 실행해 주세요' && exit 1"}]
      }
    ]
  }
}
```

이것으로 Claude Code가 **위험한 조작을 실행하기 전에 멈춘다.**

---

## **가드레일 4: 프로덕션 환경 변수는 별도 관리:**
`.env.production` 을 Claude Code가 읽을 수 있는 위치에 두지 않는다.

우리 팀의 구성:

```
project/
├── .env                    # 로컬 개발용 (더미 값)     ← Claude OK
├── .env.example            # 설정 견본 (더미 값)        ← Claude OK
└── secrets/                # 프로덕션 값 (.claudeignore 대상)
    └── .env.production     # 프로덕션 인증 정보         ← Claude 읽기 불가
```

프로덕션 배포는 GitHub Actions나 Vercel 환경 변수 관리로 수행하고, Claude가 프로덕션 값에 접근하지 않는 설계로 한다.

---

## **가드레일 5: Skills 로 "승인 플로우"를 조립:**
기밀성이 높은 조작에는 **명시적인 승인 단계**를 Skill로 조립한다.

`.claude/skills/db-modify/skill.md`:

```
# db-modify

DB 스키마 변경 또는 데이터 업데이트 시의 절차:

1. **반드시 먼저** "지금부터 무엇을 할 것인지"를 한 줄로 선언
2. **영향 범위** 를 명확히 한다 (어떤 테이블·몇 건의 레코드)
3. **백업 취득** 명령어를 실행
4. **스테이징 환경** 에서 먼저 테스트
5. **사람의 최종 승인** 을 반드시 구한다
6. 프로덕션 실행 후, 결과를 보고

도중에 "사람의 승인이 필요하다"고 판단하면, 반드시 멈추고 확인해 주세요.
```

이것으로 "DB에 대한 변경"을 Claude가 단독으로 진행하지 않고, 사람의 판단을 구하게 된다.

---

## **전체 포함 템플릿 (복붙으로 사용 가능):**

리포지토리에 그대로 놓을 수 있는 템플릿:

```
project/
├── .claudeignore              ← 가드레일 1
├── CLAUDE.md                  ← 가드레일 2 (금지 사항 섹션)
├── .claude/
│   ├── settings.json          ← 가드레일 3 (Hooks)
│   └── skills/
│       └── db-modify/
│           └── skill.md       ← 가드레일 5 (승인 플로우)
└── secrets/                   ← 가드레일 4 (.claudeignore 대상)
    └── .env.production
```

---

## **정리 - "자유"와 "안전"을 양립하는 설계:**
Claude Code는 강력한 도구이지만, **사내에서 안전하게 사용하려면 "가드레일 설계"가 필수**다.

최소한:

1. `.claudeignore` 로 기밀 파일 제외
2. CLAUDE.md 에 금지 사항
3. Hooks로 위험 조작 차단
4. 프로덕션 환경 변수의 분리
5. Skills로 승인 플로우

이 5가지를 넣으면, **개인의 판단 실수로 발생하는 보안 사고**를 90% 줄일 수 있다.  