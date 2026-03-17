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