# Claude Code에서 OpenRouter 사용하기

## 사전 준비

1. **Claude Code 설치** (아직 설치하지 않았다면)
   - macOS/Linux/WSL: `curl -fsSL https://claude.ai/install.sh | bash`
   - Windows PowerShell: `irm https://claude.ai/install.ps1 | iex`

2. **OpenRouter API 키 발급**: [https://openrouter.ai/settings/keys](https://openrouter.ai/settings/keys) 에서 API 키를 생성합니다.
  
 
## 설정 방법
쉘 프로필 파일(`~/.zshrc` 또는 `~/.bashrc`)에 다음 환경 변수를 추가합니다:

```bash
export OPENROUTER_API_KEY="여기에-OpenRouter-API-키-입력"
export ANTHROPIC_BASE_URL="https://openrouter.ai/api"
export ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"
export ANTHROPIC_API_KEY=""   # 반드시 빈 문자열로 설정해야 합니다
```

저장 후 터미널을 재시작하거나 `source ~/.zshrc` (또는 `source ~/.bashrc`)를 실행합니다.

> **주의사항:**
> - `ANTHROPIC_API_KEY`는 반드시 **빈 문자열(`""`)** 로 설정해야 합니다. 설정하지 않으면(null) Claude Code가 Anthropic 서버로 직접 인증을 시도할 수 있습니다.
> - 이전에 Anthropic 계정으로 로그인한 적이 있다면, Claude Code 세션에서 `/logout` 명령을 실행하여 캐시된 인증 정보를 먼저 지워야 합니다.
> - 프로젝트 레벨의 `.env` 파일에 넣지 마세요. Claude Code 네이티브 설치는 `.env` 파일을 읽지 않습니다.


## 다른 모델 사용 (선택 사항)
OpenRouter를 통해 Claude 외의 다른 모델도 사용할 수 있습니다. `ANTHROPIC_MODEL` 환경 변수를 추가하면 됩니다:

```bash
export ANTHROPIC_MODEL="stepfun/step-3.5-flash:free"   # 예시 (무료 모델)
```
  

## 실행 및 확인

```bash
cd /path/to/your/project
claude
```

접속 후 `/status` 명령으로 OpenRouter를 통해 연결되었는지 확인할 수 있고, [OpenRouter Activity Dashboard](https://openrouter.ai/activity)에서도 실시간으로 요청 내역을 확인할 수 있습니다.
  

## OpenRouter를 사용하면 좋은 점
OpenRouter를 중간 라우터로 사용하면 몇 가지 이점이 있습니다. Anthropic API에 장애나 레이트 리밋이 발생했을 때 다른 프로바이더로 자동 페일오버가 되어 작업이 중단되지 않습니다. 또한 팀 단위로 예산 제한을 설정하고 사용량을 모니터링할 수 있으며, Anthropic의 사용량 한도에 걸렸을 때 대안으로 활용하거나 무료 모델을 사용해 비용을 절감할 수도 있습니다.


이제 Windows 환경에서의 설정 방법도 포함하여 정리해 드리겠습니다.

---

## Windows에서 Claude Code + OpenRouter 설정 방법
Windows에서는 **두 가지 방식**으로 환경 변수를 설정할 수 있습니다.

### 방법 1: PowerShell 프로필에 추가 (권장)
PowerShell을 열고 프로필 파일을 편집합니다:

```powershell
notepad $PROFILE
```

> 만약 프로필 파일이 없다는 메시지가 나오면, 먼저 생성합니다:
> ```powershell
> New-Item -Path $PROFILE -Type File -Force
> notepad $PROFILE
> ```

열린 메모장에 다음 내용을 추가하고 저장합니다:

```powershell
$env:OPENROUTER_API_KEY = "여기에-OpenRouter-API-키-입력"
$env:ANTHROPIC_BASE_URL = "https://openrouter.ai/api"
$env:ANTHROPIC_AUTH_TOKEN = $env:OPENROUTER_API_KEY
$env:ANTHROPIC_API_KEY = ""
```

저장 후 PowerShell을 재시작하면 적용됩니다.

---

### 방법 2: Windows 시스템 환경 변수에 등록 (영구 설정)
이 방법은 PowerShell뿐 아니라 모든 터미널에서 적용됩니다.

1. `Win + R` → `sysdm.cpl` 입력 → Enter
2. **고급** 탭 → **환경 변수** 클릭
3. **사용자 변수**에서 **새로 만들기**를 눌러 아래 4개를 각각 추가합니다:

| 변수 이름 | 변수 값 |
|---|---|
| `OPENROUTER_API_KEY` | `여기에-OpenRouter-API-키-입력` |
| `ANTHROPIC_BASE_URL` | `https://openrouter.ai/api` |
| `ANTHROPIC_AUTH_TOKEN` | `여기에-OpenRouter-API-키-입력` (동일한 키) |
| `ANTHROPIC_API_KEY` | (빈 값으로 두기) |

4. **확인**을 누르고 터미널을 재시작합니다.


### 이전 로그인 정보 제거
이전에 Anthropic 계정으로 로그인한 적이 있다면, Claude Code 세션에서 `/logout` 명령을 먼저 실행해야 합니다.


### 실행 및 확인

```powershell
cd C:\path\to\your\project
claude
```

접속 후 `/status` 명령으로 OpenRouter를 통해 연결되었는지 확인할 수 있습니다.


### (선택) 다른 모델 사용하기
OpenRouter를 통해 Claude 외의 다른 모델도 사용할 수 있습니다. 다음 환경 변수를 추가하면 됩니다:

```powershell
# PowerShell 프로필에 추가
$env:ANTHROPIC_MODEL = "stepfun/step-3.5-flash:free"   # 예시: 무료 모델
```

모델별 성능 티어를 세부 지정하려면 다음과 같이 설정할 수도 있습니다:

```powershell
$env:ANTHROPIC_DEFAULT_SONNET_MODEL = "원하는-모델-ID"   # 기본 모델
$env:ANTHROPIC_DEFAULT_OPUS_MODEL = "원하는-모델-ID"     # 고성능 모델
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL = "원하는-모델-ID"    # 경량/빠른 모델
```

대체 모델을 사용할 때는 해당 모델이 **tool use(도구 사용)** 를 지원하는지 확인하는 것이 중요합니다. Claude Code는 파일 읽기, 명령 실행, 코드 편집 등에 tool use 기능을 많이 활용하기 때문입니다. [OpenRouter 모델 페이지](https://openrouter.ai/models?supported_parameters=tools)에서 tool use 지원 여부를 필터링할 수 있습니다.    


  
## PowerShell의 `$PROFILE`
PowerShell이 시작될 때 **자동으로 실행되는 스크립트 파일의 경로**를 담고 있는 자동 변수입니다. Linux/macOS의 `~/.bashrc`나 `~/.zshrc`와 같은 역할을 합니다.

### 실제 경로
PowerShell에서 `$PROFILE`을 입력하면 경로를 확인할 수 있습니다:

```powershell
$PROFILE
```

일반적으로 다음과 같은 경로가 출력됩니다:

```
C:\Users\사용자이름\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
```

### 하는 일
이 `.ps1` 파일 안에 작성한 코드는 PowerShell을 열 때마다 자동으로 실행됩니다. 그래서 환경 변수 설정, 자주 쓰는 별칭(alias), 커스텀 함수 등을 여기에 넣어두면 매번 수동으로 입력할 필요가 없습니다.

### 파일이 없는 경우
`$PROFILE`은 경로만 가리키고 있을 뿐, 파일이 기본적으로 존재하지는 않습니다. 파일 존재 여부를 확인하고 없으면 생성하는 방법은 다음과 같습니다:

```powershell
# 존재 여부 확인
Test-Path $PROFILE

# False가 나오면 파일 생성
New-Item -Path $PROFILE -Type File -Force
```

### 프로필의 종류
사실 `$PROFILE`에는 4가지 종류가 있습니다:

```powershell
$PROFILE | Select-Object *
```

이렇게 입력하면 전체 목록을 볼 수 있습니다:

| 속성 | 적용 범위 |
|---|---|
| `$PROFILE.CurrentUserCurrentHost` | 현재 사용자 + 현재 호스트 (기본값, `$PROFILE`과 동일) |
| `$PROFILE.CurrentUserAllHosts` | 현재 사용자 + 모든 호스트 |
| `$PROFILE.AllUsersCurrentHost` | 모든 사용자 + 현재 호스트 |
| `$PROFILE.AllUsersAllHosts` | 모든 사용자 + 모든 호스트 |

대부분의 경우 그냥 `$PROFILE` (= `CurrentUserCurrentHost`)을 사용하면 충분합니다. 앞서 OpenRouter 환경 변수를 이 파일에 넣으라고 안내한 이유도, PowerShell을 열 때마다 자동으로 환경 변수가 설정되도록 하기 위해서입니다.




## `ANTHROPIC_API_KEY` 을 빈 값으로 하는 이유

### 왜 빈 값으로 설정하는가
Claude Code는 인증 시 여러 환경 변수를 확인하는데, 우선순위가 있습니다. `ANTHROPIC_AUTH_TOKEN`이 설정되어 있으면 이것을 인증 토큰으로 사용합니다. 즉, OpenRouter API 키가 담긴 `ANTHROPIC_AUTH_TOKEN`이 실제 인증에 사용되므로, `ANTHROPIC_API_KEY`는 비어 있어도 괜찮습니다.

### 빈 값으로 설정하지 않으면 생기는 문제
오히려 `ANTHROPIC_API_KEY`를 **비우지 않으면** 문제가 생깁니다. 만약 이전에 Anthropic API 키가 이 변수에 남아 있다면, Claude Code가 OpenRouter 대신 Anthropic 서버로 직접 인증을 시도하면서 충돌이 발생할 수 있습니다. 그래서 OpenRouter 공식 문서에서도 반드시 빈 문자열(`""`)로 명시적으로 설정하라고 안내하고 있습니다.

### 주의할 점: "빈 문자열"과 "미설정"은 다르다

```powershell
# 빈 문자열로 설정 (올바른 방법)
$env:ANTHROPIC_API_KEY = ""

# 변수 자체를 삭제 (문제가 될 수 있음)
Remove-Item Env:ANTHROPIC_API_KEY
```

변수를 아예 삭제(미설정/null)하면 Claude Code가 기본 동작으로 Anthropic 서버에 로그인을 시도할 수 있습니다. 반면 빈 문자열로 설정하면 "이 변수는 존재하지만 값이 없다"는 것을 Claude Code가 인식하고, 대신 `ANTHROPIC_AUTH_TOKEN`을 사용하게 됩니다.

요약하면, `ANTHROPIC_API_KEY=""`는 "Anthropic 직접 인증을 쓰지 않겠다"는 명시적 선언이라고 보시면 됩니다.  