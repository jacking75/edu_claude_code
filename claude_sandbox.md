# **Claude Code를 안전하게 사용하기 위해 지금 당장 설정해야 할 10가지 항목**
  
>>  출처: https://qiita.com/miruky/items/51db293a7a7d0d277a5d   

---

Claude Code는 매우 강력한 AI 코딩 에이전트이지만, **터미널 명령 실행 및 파일 읽기/쓰기**가 가능하기 때문에 보안 설정을 소홀히 하면 예상치 못한 위험을 초래할 수 있습니다.  
이 글에서는 Claude Code를 안전하게 사용하기 위해 **지금 당장 설정해야 할 10가지 항목**을 정리했습니다. 본 글은 2026년 3월 시점의 Claude Code 공식 문서를 기반으로 작성되었습니다.  

---

**① 샌드박스를 활성화한다**

**가장 중요한 설정**입니다.
샌드박스를 활성화하면 Claude Code가 실행하는 Bash 명령이 OS 수준에서 격리되어, 파일 시스템 및 네트워크 접근이 제한됩니다.

```json
// .claude/settings.json
{
  "sandbox": {
    "enabled": true
  }
}
```

macOS에서는 **Seatbelt**, Linux에서는 **Bubble Wrap**을 사용한 샌드박스가 적용됩니다. 활성화 후에는 `/sandbox` 명령어로 상태를 확인할 수 있습니다.

샌드박스는 2025년 후반에 도입된 비교적 새로운 기능입니다. 아직 활성화하지 않으셨다면 최우선으로 설정하시기 바랍니다.

---

**② 샌드박스의 탈출구를 막는다**

샌드박스에는 기본적으로 "탈출구(escape hatch)"가 존재하며, 특정 명령이 샌드박스 외부에서 실행될 수 있습니다. 이를 완전히 차단합시다.

```json
{
  "sandbox": {
    "enabled": true,
    "allowUnsandboxedCommands": false
  }
}
```

`allowUnsandboxedCommands: false`로 설정하면, `dangerouslyDisableSandbox` 파라미터를 통한 샌드박스 우회가 **완전히 비활성화**됩니다.

---

**③ 위험한 명령을 deny 규칙으로 차단한다**

`permissions.deny`를 사용하여 Claude Code에서 실행하지 않을 명령을 명시적으로 차단할 수 있습니다.

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(curl *)",
      "Bash(wget *)",
      "Bash(git push *)",
      "Bash(chmod 777 *)"
    ]
  }
}
```

`curl`과 `wget`은 기본적으로 차단되어 있지만, 명시적으로 deny 규칙에 추가해 두면 의도치 않은 허가 설정 오버라이드를 방지할 수 있습니다.

규칙의 평가 순서는 **deny → ask → allow** 입니다. deny 규칙이 최우선으로 적용됩니다.

---

**④ 기밀 파일에 대한 접근을 거부한다**

`.env` 파일이나 시크릿, 인증 정보가 포함된 파일에 대한 접근을 차단합니다.

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Read(**/*.pem)",
      "Read(**/*.key)"
    ]
  }
}
```

샌드박스의 `denyRead`와 조합하면, Claude Code의 파일 툴뿐만 아니라 Bash 명령을 통한 읽기도 차단할 수 있습니다.

```json
{
  "sandbox": {
    "filesystem": {
      "denyRead": ["~/.aws/credentials", "~/.ssh"]
    }
  }
}
```

---

**⑤ 네트워크 접근을 제한한다**

샌드박스의 네트워크 제어를 사용하여, 허용할 도메인을 화이트리스트 방식으로 지정합니다.

```json
{
  "sandbox": {
    "network": {
      "allowedDomains": [
        "github.com",
        "*.githubusercontent.com",
        "*.npmjs.org",
        "registry.yarnpkg.com",
        "pypi.org"
      ]
    }
  }
}
```

이를 통해 Claude Code가 **의도치 않은 외부 서버로 데이터를 전송하는 위험**을 대폭 줄일 수 있습니다. 프롬프트 인젝션 공격에 의한 데이터 탈취(exfiltration) 대책으로 특히 유효합니다.

---

**⑥ bypassPermissions 모드를 비활성화한다**

`--dangerously-skip-permissions` 플래그는 모든 권한 검사를 건너뛰는 모드입니다. 팀 개발에서는 특히 이 플래그를 **완전히 사용 불가** 상태로 만들어야 합니다.

```json
{
  "permissions": {
    "disableBypassPermissionsMode": "disable"
  }
}
```

이 설정을 Managed Settings(조직 설정)에 배치하면, 사용자 측에서 덮어쓸 수 없습니다.

---

**⑦ PreToolUse 훅으로 커스텀 안전 검사를 추가한다**

Hooks는 Claude Code의 라이프사이클 각 지점에서 커스텀 스크립트를 실행할 수 있는 강력한 기능입니다. `PreToolUse` 훅을 사용하면 **툴 실행 전에 커스텀 검사**를 삽입할 수 있습니다.

```json
// .claude/settings.json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/validate-command.sh"
          }
        ]
      }
    ]
  }
}
```

훅 스크립트 예시(위험한 명령 차단):

```bash
#!/bin/bash
# .claude/hooks/validate-command.sh
COMMAND=$(jq -r '.tool_input.command' < /dev/stdin)

# rm -rf 차단
if echo "$COMMAND" | grep -q 'rm -rf'; then
  echo "Blocked: rm -rf commands are not allowed" >&2
  exit 2  # exit 2로 툴 실행 차단
fi

# 프로덕션 환경 접속 차단
if echo "$COMMAND" | grep -q 'prod'; then
  echo "Blocked: production access is not allowed" >&2
  exit 2
fi

exit 0
```

훅의 종류는 명령(command) 외에도 **HTTP webhook**, **LLM 프롬프트 평가**, **에이전트형** 등 총 4가지가 있습니다.

---

**⑧ `/permissions` 명령으로 주기적으로 권한을 점검한다**

Claude Code에는 현재 권한 설정을 일람할 수 있는 `/permissions` 명령이 있습니다. 다음 사항을 정기적으로 확인하세요.

- 세션 중에 허가한 "Always allow" 규칙이 누적되어 있지 않은지
- 불필요한 허가 규칙이 남아 있지 않은지
- deny 규칙이 의도한 대로 설정되어 있는지

또한 `/status` 명령으로 어떤 설정 파일이 로드되어 있는지 확인할 수 있습니다. 설정 파일에 오류가 있을 경우도 여기서 감지됩니다.

---

**⑨ devcontainer로 완전 격리 환경을 구축한다**

가장 안전한 방법은 Claude Code를 **컨테이너 안에서 실행**하는 것입니다. Anthropic 공식 devcontainer 레퍼런스 구현이 제공되고 있습니다.

주요 특징은 **방화벽** 기반의 네트워크 제어(기본 deny 정책), 호스트 머신으로부터의 **완전한 격리**, 그리고 VS Code Remote Containers 확장 기능으로 손쉽게 설정 가능하다는 점입니다.

```bash
# 방법 1: Claude Code 리포지토리의 devcontainer 레퍼런스 구현 사용
git clone https://github.com/anthropics/claude-code.git
# VS Code에서 열어 "Reopen in Container" 선택

# 방법 2: devcontainer features로 기존 프로젝트에 추가
# https://github.com/anthropics/devcontainer-features 참조
```

devcontainer 환경에서는 `--dangerously-skip-permissions`를 사용한 자동화도 **비교적 안전하게** 수행할 수 있습니다(단, 신뢰할 수 있는 리포지토리에서만 권장).

---

**⑩ 팀용: Managed Settings로 조직 정책을 강제한다**

팀이나 조직에서 Claude Code를 사용하는 경우, **Managed Settings**로 정책을 일괄 관리할 수 있습니다. Managed Settings는 최고 우선순위를 가지며, 사용자나 프로젝트 설정으로 덮어쓸 수 없습니다.

Managed Settings에는 **2가지 방식**이 있습니다.

| 방식 | 개요 | 적용 상황 |
|---|---|---|
| **Server-managed settings** (public beta) | Claude.ai 관리 콘솔에서 설정 배포. MDM 불필요 | 원격 근무·BYOD 환경 |
| **Endpoint-managed settings** | MDM(Jamf, Intune 등)으로 디바이스에 직접 배치 | 보안 중시 조직 |

Server-managed settings는 Claude for Teams / Enterprise 플랜에서 이용 가능하며, 사용자 인증 시 설정이 자동 배포됩니다.

Endpoint-managed settings의 배치 경로는 macOS의 경우 `/Library/Application Support/ClaudeCode/managed-settings.json`, Linux는 `/etc/claude-code/managed-settings.json`, Windows는 `C:\Program Files\ClaudeCode\managed-settings.json`입니다.

조직 관리자용 주요 설정:

```json
{
  "permissions": {
    "disableBypassPermissionsMode": "disable"
  },
  "allowManagedPermissionRulesOnly": true,
  "allowManagedHooksOnly": true,
  "allowManagedMcpServersOnly": true,
  "sandbox": {
    "enabled": true,
    "allowUnsandboxedCommands": false,
    "network": {
      "allowManagedDomainsOnly": true,
      "allowedDomains": ["github.com", "*.npmjs.org"]
    }
  }
}
```

| 설정 키 | 효과 |
|---|---|
| `disableBypassPermissionsMode` | `--dangerously-skip-permissions` 사용 금지 |
| `allowManagedPermissionRulesOnly` | Managed 이외의 allow/deny/ask 규칙 무효화 |
| `allowManagedHooksOnly` | 사용자·프로젝트 훅 무효화, 관리자 설정 훅만 허용 |
| `allowManagedMcpServersOnly` | 관리자가 허가한 MCP 서버만 사용 가능 |
| `allowManagedDomainsOnly` | 관리자가 지정한 도메인만 네트워크 접근 가능 |

---

**마치며**

Claude Code는 매우 편리한 툴이지만, **"편리함"과 "안전성"은 트레이드오프** 관계에 있습니다.
특히 팀 개발에서는 먼저 ①~④의 기본 설정을 완료한 후, 단계적으로 ⑤ 이후를 도입해 나가는 것을 추천드립니다.  