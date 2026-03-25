# Claude Code에서 C# 랭귀지 서버 사용하기

## **Claude Code의 LSP 지원이란?**
Claude Code는 2025년 12월부터 **LSP(Language Server Protocol)** 를 공식 지원하기 시작했습니다. LSP를 연동하면 Claude가 코드를 단순히 텍스트로 검색(`grep`)하는 방식에서 벗어나, 진짜 IDE처럼 코드의 의미를 이해하며 작업할 수 있게 됩니다. 구체적으로 Claude가 사용할 수 있는 LSP 기능은 다음과 같습니다.

- **goToDefinition** – 심볼이 정의된 위치로 이동
- **findReferences** – 심볼의 모든 참조 위치 탐색
- **hover** – 심볼의 타입 정보 및 문서 조회
- **documentSymbol / workspaceSymbol** – 파일 혹은 프로젝트 전체 심볼 목록 조회
- **goToImplementation** – 인터페이스/추상 메서드의 구현체 탐색
- **incomingCalls / outgoingCalls** – 함수 호출 계층 분석

이 기능들이 활성화되면 Claude는 대규모 C# 프로젝트에서도 훨씬 정확하게 코드 구조를 파악하고, 리팩토링이나 버그 수정을 더 스마트하게 수행할 수 있습니다.

---

## **csharp-ls vs roslyn-language-server, 무엇을 써야 하나?**
여기서 중요한 구분이 있습니다.

`csharp-ls`는 커뮤니티에서 개발한 독립적인 C# 언어 서버로, Roslyn 기반이지만 **Microsoft에서 공식 관리하지 않습니다**. 반면 `roslyn-language-server`는 **Microsoft가 공식으로 관리**하며 VS Code의 C# 확장 및 C# Dev Kit을 구동하는 바로 그 언어 서버입니다.

커뮤니티 경험에 따르면, `csharp-ls`를 설치하고 Claude Code의 공식 `csharp-lsp` 플러그인과 연동하려 했지만 **잘 동작하지 않는 경우가 많았고**, 실제로 안정적으로 작동한 것은 `roslyn-language-server`를 직접 수동으로 연동한 방법이었습니다.


## roslyn-language-server 사용해야할까? (2026.03)
현재 roslyn-language-server를 Claude Code에 연결하는 과정은 설정 파일을 3개나 직접 작성하고, 경로를 일일이 맞춰야 하고, 마켓플레이스까지 수동으로 등록해야 하는 꽤 번거로운 작업입니다.
**사용할 가치가 있는지 여부는 작업 스타일에 따라 갈립니다.**  

  
**쓸 가치가 있는 경우**  
- 대규모 C# 프로젝트에서 Claude Code를 집중적으로 사용할 때 — `goToDefinition`, `findReferences`, `incomingCalls` 같은 LSP 기능 덕분에 Claude가 파일을 grep으로 헤매지 않고 정확하게 심볼을 추적하므로 **토큰 낭비가 크게 줄어듭니다.**
- 한 번 설정해두면 그 이후로는 건드릴 일이 없으니 초기 비용만 감수하면 됩니다.

**그냥 안 쓰는 게 나은 경우**  
- 소규모 프로젝트이거나 Claude Code를 가끔만 쓰는 경우
- 설정에 시간 쓰는 것 자체가 스트레스인 경우
- 솔직히 LSP 없이도 Claude가 C# 코드를 꽤 잘 이해합니다. 못 쓸 수준이 전혀 아닙니다.


결론적으로 **"일단 LSP 없이 써보고, Claude가 엉뚱한 파일을 자꾸 뒤지거나 심볼을 못 찾아서 불편하다 싶을 때 그때 설정하는 것"** 이 가장 현실적인 접근이라고 생각합니다.
  
---
 
 
## **실제 권장 설정 방법 (roslyn-language-server 기준)**
아래는 현재 커뮤니티에서 검증된 방법입니다.

### **1단계: LSP 기능 활성화**

`~/.claude/settings.json`에 아래 내용을 추가합니다.

```json
{
  "env": { "ENABLE_LSP_TOOL": "1" }
}
```

### **2단계: roslyn-language-server 설치**

```bash
dotnet tool install --global roslyn-language-server --prerelease
```

### **3단계: 커스텀 플러그인 디렉토리 구성**
아래와 같은 디렉토리 구조를 만들어야 합니다.

```
~/.claude-custom-plugins/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    └── roslyn-ls/
        ├── .lsp.json
        └── .claude-plugin/
            └── plugin.json
```   
    
**3단계 상세 설명: 커스텀 플러그인 디렉토리 구성**  
3단계는 Claude Code가 roslyn-language-server를 플러그인으로 인식하고 설치할 수 있도록, **로컬 마켓플레이스 디렉토리와 그 안의 설정 파일 3개를 직접 손으로 만드는 작업**입니다. 각 파일이 어떤 역할을 하는지부터 이해하면 훨씬 명확해집니다.

  
**전체 디렉토리 구조와 각 구성요소의 역할**

```
~/.claude-custom-plugins/          ← 로컬 마켓플레이스 루트 (이름/위치 자유)
├── .claude-plugin/
│   └── marketplace.json           ← [파일 1] 마켓플레이스 카탈로그
└── plugins/
    └── roslyn-ls/                 ← 플러그인 폴더 (marketplace.json의 source와 이름 일치해야 함)
        ├── .lsp.json              ← [파일 2] LSP 서버 실행 설정
        └── .claude-plugin/
            └── plugin.json        ← [파일 3] 플러그인 메타데이터
```

---

**[파일 1] `~/.claude-custom-plugins/.claude-plugin/marketplace.json`**   
이 파일은 **"이 마켓플레이스에는 어떤 플러그인들이 있고, 각 플러그인을 어디서 가져오는가"** 를 Claude Code에 알려주는 카탈로그입니다. `{{ }}` 안의 값들은 본인 환경에 맞게 교체해야 합니다.

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "local",
  "metadata": {
    "description": "Local plugins for development and testing.",
    "version": "1.0.0",
    "license": "MIT"
  },
  "owner": {
    "name": "홍길동"
  },
  "plugins": [
    {
      "name": "roslyn-ls",
      "version": "1.0.0",
      "source": "./plugins/roslyn-ls",
      "description": "LSP using roslyn-language-server.",
      "category": "development",
      "author": { "name": "홍길동" },
      "tags": ["csharp"],
      "lspServers": {
        "roslyn-ls": {
          "command": "/Users/홍길동/.dotnet/tools/roslyn-language-server",
          "args": [
            "--stdio",
            "--autoLoadProjects",
            "--logLevel",
            "Information",
            "--extensionLogDirectory",
            "/Users/홍길동/.dotnet/tools/roslyn-language-service/logs"
          ],
          "transport": "stdio",
          "extensionToLanguage": {
            ".cs": "csharp",
            ".csx": "csharp",
            ".cshtml": "csharp"
          },
          "initializationOptions": {},
          "settings": {},
          "startupTimeout": 120000
        }
      }
    }
  ]
}
```

**`command` 경로 확인 방법:**   
```bash
# 설치된 dotnet tool 경로 확인
which roslyn-language-server

# 보통 아래 경로 중 하나입니다
# Windows: C:/Users/이름/.dotnet/tools/roslyn-language-server.cmd
# macOS/Linux: /Users/이름/.dotnet/tools/roslyn-language-server
```

  
**[파일 2] `~/.claude-custom-plugins/plugins/roslyn-ls/.lsp.json`**  
이 파일은 **실제로 LSP 서버를 어떻게 실행하는지** 정의하는 파일입니다. `marketplace.json`의 `lspServers` 안의 내용과 동일하게 맞춰야 합니다.

```json
{
  "csharp": {
    "command": "/Users/홍길동/.dotnet/tools/roslyn-language-server",
    "args": [
      "--stdio",
      "--autoLoadProjects",
      "--logLevel",
      "Information",
      "--extensionLogDirectory",
      "/Users/홍길동/.dotnet/tools/roslyn-language-service/logs"
    ],
    "transport": "stdio",
    "extensionToLanguage": {
      ".cs": "csharp",
      ".csx": "csharp",
      ".cshtml": "csharp"
    },
    "initializationOptions": {},
    "settings": {},
    "startupTimeout": 120000
  }
}
```

  
**[파일 3] `~/.claude-custom-plugins/plugins/roslyn-ls/.claude-plugin/plugin.json`**   
이 파일은 플러그인 자체의 **메타데이터(이름, 버전, 작성자 등)** 를 담습니다. `marketplace.json`에 선언한 값들과 이름/버전이 일치해야 합니다.

```json
{
  "name": "roslyn-ls",
  "description": "LSP using roslyn-language-server.",
  "version": "1.0.0",
  "license": "MIT",
  "author": {
    "name": "홍길동"
  }
}
```

  
**실제로 파일을 만드는 명령어 (macOS/Linux 기준)**  
터미널에서 아래 명령어를 순서대로 실행하면 디렉토리와 빈 파일을 한 번에 만들 수 있습니다.

```bash
# 디렉토리 구조 생성
mkdir -p ~/.claude-custom-plugins/.claude-plugin
mkdir -p ~/.claude-custom-plugins/plugins/roslyn-ls/.claude-plugin

# 각 파일 생성 (내용은 위의 JSON을 붙여넣기)
touch ~/.claude-custom-plugins/.claude-plugin/marketplace.json
touch ~/.claude-custom-plugins/plugins/roslyn-ls/.lsp.json
touch ~/.claude-custom-plugins/plugins/roslyn-ls/.claude-plugin/plugin.json
```

  
**3개 파일의 관계 요약**  
세 파일은 서로 연결된 구조로 동작합니다. `marketplace.json`은 "roslyn-ls라는 플러그인이 `./plugins/roslyn-ls` 폴더에 있다"고 Claude Code에 알리고, `plugin.json`은 해당 플러그인의 신원을 확인해주며, `.lsp.json`은 C# 파일을 열 때 실제로 어떤 프로세스를 어떤 인자로 실행할지를 지정합니다. 세 파일의 `name`, `version`, `command` 경로가 서로 **일치**하지 않으면 4단계 등록/설치 시 오류가 발생하므로 특히 주의해야 합니다.  
  
  
### **4단계: Claude Code에 플러그인 등록**

```bash
# Claude 실행 후 플러그인 마켓플레이스 추가
/plugin marketplace add ~/.claude-custom-plugins

# 플러그인 설치
claude plugin install -s user roslyn-ls
```

---

## **주의할 점**
설정 과정이 상당히 번거롭고, 현재까지도 완전히 안정적이지는 않습니다. 특히 Windows 환경에서 경로 구분자(슬래시)에 민감하고, 설정 JSON 파일의 구조가 조금만 틀려도 "failed to load" 오류가 발생할 수 있습니다. 또한 일부 사용자는 IDE(VS Code 등)가 함께 열려 있어야 LSP 서버가 제대로 구동된다는 보고도 있습니다.