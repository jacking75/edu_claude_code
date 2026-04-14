# Claude Code Sandbox 기능 소개 (2026년 4월)

> 원문: [dev.classmethod.jp](https://dev.classmethod.jp/articles/claude-code-sandbox-202604/)

---

## 들어가며

Claude Code의 Sandbox 기능을 소개한다. 등장 초기에 비해 최근(2026년 4월 기준) 상당히 완성도가 높아진 인상이라 다시 한번 정리해본다.

Claude Code에서 CLI 명령어를 실행할 때, Plan 모드나 AcceptEdits 모드에서는 명령어마다 허가 다이얼로그가 표시되었다. 또한 프로젝트 외부 파일이나 외부 네트워크 접근을 어디까지 허용할지 `settings.json`으로 관리했다. 이 방식은 매번 허가를 부여하는 부담이 사람에게 있고, 승인이 점점 모호해지는 문제가 있었다.

Sandbox 기능을 사용하면 파일 시스템과 네트워크의 경계를 사전에 설정할 수 있고, 안전한 명령어는 자동으로 실행 가능해진다.

참고: [Sandboxing - Claude Code Docs](https://code.claude.com/docs/en/sandboxing)

---

## 지원 플랫폼

macOS는 파일 접근과 네트워크 통신을 세밀하게 제한할 수 있는 **Seatbelt**가 내장되어 있어 추가 설치가 필요 없다. Linux와 WSL2는 **bubblewrap**을 사용한다. WSL1은 bubblewrap이 WSL2의 커널 기능을 필요로 하기 때문에 미지원이다. Windows 네이티브도 현시점에서는 미지원이다.

---

## Claude Code Sandbox 기능이란

Claude Code의 Sandbox 기능은 크게 다음 3가지를 제어할 수 있다.

- **CLI 명령어 자동 실행**
- **파일 시스템(디렉토리/파일 단위) 읽기·쓰기**
- **네트워크 접근(로컬/외부)**

사전에 경계를 정의함으로써, 그 범위 내에서는 허가 다이얼로그 없이 실행 가능해진다. Sandbox 범위 내의 안전한 명령어는 승인 없이 실행되므로 **승인 피로(approval fatigue)를 줄일 수 있다**. 접근 가능한 디렉토리와 호스트는 사전 지정 가능하며, Sandbox 외부로의 접근 시도는 OS 레벨에서 차단된다.

---

## CLI 명령어 자동 실행

Claude Code Sandbox에는 2가지 모드가 있다. (Plan 모드, AcceptEdits 모드와는 별개)

**Auto-allow 모드** — Sandbox 내의 CLI 명령어가 자동으로 허가된다. 외부 호스트 접근이 필요한 명령어 등 Sandbox로 처리할 수 없는 경우는 일반 허가 플로우로 폴백한다. 명시적인 거부 규칙은 항상 우선 적용된다.

**Regular permissions 모드** — Sandbox 내에서도 모든 명령어가 일반 허가 플로우를 거친다. CLI 명령어를 더 세밀하게 제어하고 싶을 때 사용한다.

두 모드 모두 파일 시스템과 네트워크 제한은 동일하다. 차이는 Sandbox 내 명령어를 자동 승인할지 여부뿐이다. Claude Code 사용에 익숙해져서 승인 피로가 생기기 시작하면 Auto-allow 모드를 검토하면 좋다.

`/sandbox` 명령어로 전환 가능하다. 또한 `settings.json`의 `sandbox.autoAllowBashIfSandboxed` 옵션으로 시작 시 기본 설정을 지정할 수 있다. 기본값 `true`가 Auto-allow 모드, `false`가 Regular permissions 모드다.

```json
// .claude/settings.json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true
  }
}
```

---

## 파일 시스템 권한 제어

파일 시스템의 경우 기본적으로 **쓰기는 현재 디렉토리와 하위 디렉토리에만** 허용된다. 읽기는 Linux 사용자가 갖는 권한 범위 내에서 가능하다. 명시적인 허가가 있는 경우 범위 외 접근도 가능하다.

세밀한 설정을 위해 4가지 프로퍼티로 특정 디렉토리의 읽기 허가/불허, 쓰기 허가/불허를 설정할 수 있다.

```json
// .claude/settings.json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": ["/tmp", "."],
      "denyWrite": ["/etc"],
      "allowRead": ["~/.zshrc"],
      "denyRead": ["~/.aws/credentials"]
    }
  }
}
```

---

## 네트워크 접근 권한 제어

네트워크 접근 설정에서는 도메인 허가/불허, 로컬 환경과의 통신, 프록시 경유 통신 등을 설정할 수 있다. localhost 포트 바인딩 허가도 `network.allowLocalBinding`으로 설정 가능하다.

```json
// .claude/settings.json
{
  "sandbox": {
    "enabled": true,
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"],
      "allowLocalBinding": true
    }
  }
}
```

위처럼 설정하면 로컬에서 실행 중인 서비스에 대한 접근을 허가할 수 있다. 또한 네트워크 접근이 필요한 CLI 도구는 최초 실행 시 호스트 접근 허가를 요청한다. 허가하면 이후 Sandbox 내에서 동작한다.

---

## Sandbox와 Permission의 역할 분담

Sandbox 외에도 Claude Code에는 파일 읽기·편집 허가를 제어하는 **Permission**이 있다.

Permission은 Bash, Read, Edit, WebFetch, MCP 등의 도구에 적용된다. Sandbox는 OS 레벨에서 파일 시스템 및 네트워크 레벨로 Bash 명령어가 접근할 수 있는 범위를 제한하며, Bash 명령어와 자식 프로세스에도 적용된다.

주의할 점은, **Sandbox에서 차단되어 있어도 도구(Tool) 경유라면 허가될 수 있다**는 것이다. 예를 들어 Sandbox에서 특정 디렉토리 접근을 금지해도, Read 도구를 통해 해당 파일을 읽을 수 있는 경우가 있다. 도구 사용 승인이 표시되므로, 실수로 승인하지 않도록 주의하자.

---

## 시작하는 방법

**사전 준비 (Linux/WSL2의 경우)**

macOS는 Seatbelt 내장이므로 추가 설치 불필요. Linux/WSL2는 다음 패키지를 설치한다.

```bash
# Ubuntu/Debian
sudo apt-get install bubblewrap socat

# Fedora
sudo dnf install bubblewrap socat
```

**`/sandbox` 명령어로 활성화**

Claude Code 채팅에서 `/sandbox`를 실행하면 모드 선택 메뉴가 열린다.

```
1. Sandbox BashTool, with auto-allow
2. Sandbox BashTool, with regular permissions
3. No Sandbox
```

**Sandbox 강제 적용 방법**

기본적으로 Sandbox 시작에 실패한 경우(의존성 부족, 미지원 플랫폼 등) 경고만 표시하고 Sandbox 없이 계속 실행된다. Sandbox 시작 실패 시 Claude Code 자체를 종료시키려면 `sandbox.failIfUnavailable`을 `true`로 설정한다.

```json
// managed-settings.json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true
  }
}
```

조직 단위로 Sandbox를 활용할 때는 MDM으로 단말에 `managed-settings.json`을 배포하면 Sandbox를 필수 요건으로 강제할 수 있다.

---

## Sandbox 우회 동작에 대해

Sandbox 제한으로 명령어가 실패한 경우, Claude Code는 에스케이프 해치(`dangerouslyDisableSandbox` 파라미터)를 사용해 Sandbox 외부에서 재시도를 제안하는 경우가 있다. 이 경우 일반 허가 플로우가 적용된다. 조직 관리 환경에서 이 에스케이프 해치를 비활성화하려면 `allowUnsandboxedCommands: false`를 설정한다.

출시 초기에는 에스케이프 해치가 자동으로 동작해서 신뢰성이 걱정되었지만, 현재는 외부에서 에스케이프 해치를 비활성화할 수 있어 DevContainer의 대안으로 사용할 수 있는 수준이 되어가고 있는 인상이다.

---

## 유스케이스별 실습

상세 설정은 [Claude Code settings 문서](https://code.claude.com/docs/en/settings#sandbox-settings) 참고. 여기서는 용도별 설정 예시를 소개한다.

**파일 쓰기 허용 경로 추가**

기본적으로 Sandbox 내 CLI 명령어는 현재 작업 디렉토리 하위에만 쓰기 가능하다. `terraform`, `npm` 등의 CLI 도구가 작업 디렉토리 외부에 쓰는 경우 `sandbox.filesystem.allowWrite`로 대상 경로를 추가한다.

```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": ["/tmp"]
    }
  }
}
```

GitHub 코드를 참조시킬 때 별도 디렉토리에 `git clone`해서 분석하는 경우가 있어, 그를 위해 `/tmp`를 등록해두면 좋다.

경로 프리픽스에 따른 해석 방식은 다음과 같다.

| 프리픽스 | 해석 대상 | 예시 |
|---|---|---|
| `/` | 파일 시스템 루트부터의 절대 경로 | `/tmp` |
| `~/` | 홈 디렉토리로부터의 상대 경로 | `~/.zsh` |
| `./` 또는 프리픽스 없음 | 프로젝트 설정 → 프로젝트 루트, 사용자 설정 → `~/.claude` | `./output` |

여러 설정 스코프에 `allowWrite`가 있는 경우 배열은 **병합**된다. 상위 스코프를 덮어쓰는 것이 아니라 모든 스코프의 경로가 합쳐서 적용된다. 도구 전체를 Sandbox에서 제외하는 `excludedCommands`보다 `allowWrite`로 필요한 경로만 허가하는 방식이 권장된다.

설정 전에 Claude Code에 `/tmp`에 파일을 만들어 라고 요청하면 다음과 같이 차단된다.

```
> /tmp에 파일을 만들어. 도구는 사용하지 말고.

도구(Bash나 Write) 없이 파일을 생성할 수 없습니다. 파일 시스템 접근은 모두 도구를 통해야 합니다.
직접 실행하려면 터미널에서 다음 명령어를 사용하세요.
touch /tmp/test.txt

> ! touch /tmp/test.txt
  ⎿  touch: /tmp/test.txt: Operation not permitted
```

`/tmp`를 추가하면 쓰기 가능해진다. Mac의 경우 `/tmp` 직하에 쓰기가 불가능하므로 `$TMPDIR`을 사용한다.

```
> /tmp에 파일을 만들어. 도구는 사용하지 말고.

⏺ Bash(touch /tmp/hoge.txt && echo "작성 성공")
  ⎿  Error: Exit code 1
     touch: /tmp/hoge.txt: Operation not permitted

⏺ $TMPDIR을 사용하면:
⏺ Bash(echo $TMPDIR && touch $TMPDIR/hoge.txt && echo "작성 성공")
  ⎿  /tmp/claude-502
     작성 성공
⏺ $TMPDIR은 /tmp/claude-502로 해석되며, 이쪽은 쓰기 가능합니다.
  /tmp 직하 쓰기가 Operation not permitted가 되는 이유는 macOS의 SIP(System Integrity Protection)나 실제 파일 시스템 권한에 의해 /tmp 루트 직하 쓰기가 제한되기 때문입니다. 설정의 allowWrite와 무관하게 OS 레벨에서 차단됩니다.
  /tmp에 쓰기가 필요한 경우는 /private/tmp/임의서브디렉토리를 사용하거나 $TMPDIR을 사용하는 것이 확실합니다.
```

**denyRead + allowWrite로 홈 디렉토리 보호**

홈 디렉토리 하위는 거부하고 현재 디렉토리에만 파일 쓰기를 허가하는 설정이다.

```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "denyRead": ["~/"],
      "allowWrite": ["."]
    }
  }
}
```

이 설정으로 프롬프트를 실행하면 Claude Code 에이전트 자체는 홈 디렉토리 파일에 접근할 수 없음이 확인된다.

```
> 홈 디렉토리의 shell 관련 파일을 읽어줘

샌드박스 제한으로 인해 /Users/user.name 직하 접근이 차단되었습니다.
~/.zshrc 등의 셸 설정 파일을 읽을 수 없는 상태입니다.
/sandbox 명령어로 /Users/user.name의 읽기를 허용 목록에 추가하면 읽을 수 있게 됩니다.
```

단, 파일을 직접 지정하면 도구를 통한 접근 허가를 요청한다. 이를 허가하면 파일을 읽을 수 있었다. 이를 통해 Sandbox는 Claude Code의 프로세스와 자식 프로세스에만 제어가 적용되고, Read/Edit 등의 도구에는 관여하지 않음을 확인할 수 있다. 도구 쪽 처리를 제한하려면 Permission을 사용해 차단해야 한다.

**보너스:** `denyRead` 설정에 `/`를 지정하면 CLI 명령어의 내부적인 파일 읽기가 불가해진다. 예를 들어 curl 명령어는 SSL 인증서 파일 읽기가 필요하므로 동작하지 않게 된다. curl 자체는 `-k` 옵션(SSL 검증 스킵)으로 사용 가능하지만, 다른 CLI 명령어에도 영향이 생길 수 있으므로 제한 범위 설정에 주의가 필요하다.

**로컬 환경 통신 제어**

기본 설정에서는 로컬 소켓 바인딩이 불가능해 로컬 앱 실행이 실패한다.

```json
{
  "sandbox": {
    "enabled": true
  }
}
```

다음과 같은 간단한 프로그램을 `python3 server.py`로 실행시키면 차단된다.

```python
from http.server import HTTPServer, SimpleHTTPRequestHandler as H
HTTPServer(("localhost", 8000), H).serve_forever()
```

```
⏺ 샌드박스 내에서는 bind가 차단되므로, 샌드박스를 해제하고 재실행합니다.

⏺ Bash(python3 server.py)
  ─────────────────────────────────────────────────
   Bash command (unsandboxed)
     python3 server.py
   이 명령어는 승인이 필요합니다
   진행하시겠습니까?
   ❯ 1. Yes
     2. Yes, and don't ask again for: python3:*
     3. No
```

다음 설정을 추가하면 서버가 정상적으로 실행된다.

```json
{
  "sandbox": {
    "enabled": true,
    "network": {
      "allowLocalBinding": true
    }
  }
}
```

**커스텀 프록시로 통신 로그 수집**

조직의 보안 정책으로 통신 검사나 로깅이 필요한 경우 커스텀 프록시를 지정할 수 있다.

```json
{
  "sandbox": {
    "network": {
      "allowedDomains": ["google.com"],
      "httpProxyPort": 8080
    }
  }
}
```

기존 보안 인프라(감사 로그, 통신 필터링 시스템 등)와 연동하고 싶을 때 활용할 수 있다. 위처럼 설정하고 `mitmproxy`를 로컬에서 실행하면 통신이 프록시 쪽으로 흐른다.

```bash
# 설치
brew install mitmproxy
# 프록시 실행 (별도 터미널)
mitmproxy --listen-port 8080

# Claude Code에서 지시
> google.com의 데이터를 샘플로 가져와줘
```

---

## 소감

승인 피로 없이 에이전트의 자율성을 높일 수 있는 실용적인 기능이라고 느낀다. 특히 엔터프라이즈 환경 도입 시 **네트워크 격리 + 커스텀 프록시 조합**은 보안 요건을 충족하는 데 유효하다. WSL1 미지원, Windows 네이티브 미지원 등의 제한은 있지만 향후 지원이 확대되기를 기대한다.

주의사항으로, **Sandbox에서 파일 읽기를 금지해도 도구(Tool)를 통한 읽기 가능 여부는 Permission에 의해 결정된다.** Sandbox로 막았으니 괜찮다고 생각하면 예상치 못한 함정이 될 수 있으므로 주의하자.  