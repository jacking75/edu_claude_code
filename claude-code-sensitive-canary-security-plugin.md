# Claude Code의 기밀 정보 유출을 막는 보안 플러그인 — sensitive-canary
  
[출처](https://zenn.dev/chataclaw/articles/dcb866d5d10d02 )  
  

## **들어가며:**
Claude Code는 너무 편리한 나머지, 자칫하면 기밀 정보를 Anthropic의 API에 전송해버릴 위험이 있다.

예를 들면 다음과 같은 상황이다:
- `.env` 파일을 "잠깐 확인해줘"라고 부탁하는 경우
- 터미널에 AWS 키를 붙여넣고 동작 확인을 요청하는 경우
- `cat .env.production`을 실행시키는 경우

이 모든 경우, API 키나 인증 정보가 Claude의 컨텍스트에 실려 Anthropic 서버로 전송된다.

"프롬프트에 직접 입력하지 않으면 괜찮겠지"라고 생각할 수 있지만, 그것만으로는 부족하다. **Claude Code는 자율적으로 파일을 읽어들인다.** 사용자가 명시적으로 지시하지 않아도, 컨텍스트로서 관련 파일을 자동으로 불러올 수 있다. 이 문제를 해결하기 위해 만들어진 것이 **[sensitive-canary](https://github.com/coo-quack/sensitive-canary)** 다.

---

## **sensitive-canary란:**
Claude Code에 두 가지 훅(Hook)을 삽입하여, 기밀 정보가 API로 전송되기 전에 차단하는 보안 플러그인이다. **설정 불필요, 외부 통신 없음, 설치 2개 커맨드**로 동작한다.

```
cat .env → 차단 ✅
AWS 키를 프롬프트에 붙여넣기 → 차단 ✅
echo $API_KEY (값이 실제 키인 경우) → 차단 ✅
Claude가 자율적으로 .env를 읽으려 할 때 → 차단 ✅
```

차단 시 다음과 같은 메시지가 출력된다:

```
🐦 sensitive-canary: sensitive data detected — blocked

  [Secret] AWS Access Key ID (aws-access-key): AKIA****MPLE
```

---

## **제로 설정 & 외부 통신 없음:**
sensitive-canary는 모든 처리가 로컬에서만 동작한다. 외부 서버와의 통신은 일절 없다. 탐지 및 차단 처리는 사용자의 머신 안에서만 완결되기 때문에, 플러그인 자체가 시크릿을 외부로 유출할 우려가 없다.

설정 파일도 불필요하다. 설치하는 순간부터 보호가 시작된다.

---

## **2가지 훅:**
**UserPromptSubmit 훅**은 프롬프트를 전송하기 전에 동작한다. 사용자의 메시지에 시크릿이나 PII(개인식별정보)가 포함되어 있으면, API로 전송하기 전에 차단한다.

**PreToolUse 훅**은 Claude가 Read 툴이나 Bash 툴을 사용하기 전에 동작한다. **이것이 핵심이다.** 프롬프트에 직접 시크릿을 작성하지 않아도, Claude는 컨텍스트로서 파일을 자율적으로 읽어들인다. 이 훅은 그것을 사전에 방지한다:

- `.env` / `.env.*` 파일을 파일명만으로 무조건 차단 (내용을 보기 전에 멈춤)
- 시크릿이나 PII를 포함하는 파일의 읽기
- `cat`, `head`, `tail` 등의 파일 읽기 커맨드 (대상 파일 내용을 사전에 스캔)
- 인라인 시크릿을 포함한 Bash 커맨드
- 환경 변수 참조 (`$SECRET_KEY` 등의 값이 실제 키인 경우 탐지)

---

**탐지 가능한 항목:**

**시크릿 (29종류)**

| 카테고리 | 예시 |
|---|---|
| 클라우드 | AWS Access Key ID |
| 소스 관리 | GitHub PAT, GitLab PAT |
| AI 서비스 | Anthropic API 키, OpenAI API 키 |
| 커뮤니케이션 | Slack Token, Discord Webhook, Telegram Bot Token |
| 결제 | Stripe Secret Key, 신용카드 번호 (Luhn 알고리즘으로 검증) |
| 이메일 | SendGrid, Mailgun, Mailchimp |
| 인증 | JWT, PEM 프라이빗 키, DB 연결 문자열 |

**PII(개인식별정보)**는 이메일 주소, 미국 SSN, 전화번호(미국·일본), 일본 우편번호(〒 프리픽스 필수), 프라이빗 IPv4 주소를 탐지한다.

탐지 패턴은 [gitleaks](https://github.com/gitleaks/gitleaks)와 [TruffleHog](https://github.com/trufflesecurity/trufflehog)의 룰 정의를 채택하고 있다. 엔트로피 필터링으로 오탐지를 억제하는 것도 특징이다.

---

## **설치 방법:**
Claude Code 세션 내에서 2개의 커맨드만 실행하면 된다:

```bash
# 마켓플레이스 등록
/plugin marketplace add coo-quack/sensitive-canary

# 플러그인 설치
/plugin install sensitive-canary@coo-quack
```

재시작 불필요, 설정 파일 불필요, API 키 불필요. 이것만으로 동작한다.

---

**의도적으로 허용하고 싶은 경우:**

프롬프트에 태그를 붙이기만 하면 된다:

```
[allow-secret] .env.example의 샘플 값을 확인해줘
[allow-pii] 이 유저 데이터의 스키마를 알려줘
[allow-all] 테스트용 인증 정보로 동작 확인해줘
```

태그는 해당 턴에서만 유효하다. 다음 프롬프트에는 이월되지 않는다.

---

## **마무리:**
Claude Code를 많이 사용할수록, 실수로 시크릿을 넘겨버릴 위험은 높아진다. "프롬프트에 직접 작성하지 않는다"는 자기 방어만으로는, AI가 자율적으로 읽어들이는 파일까지는 보호할 수 없다.

sensitive-canary는 제로 설정, 외부 통신 없이, 그 두 가지를 모두 커버한다.

- 공식 사이트 → https://coo-quack.github.io/sensitive-canary/
- GitHub 리포지토리 → https://github.com/coo-quack/sensitive-canary  