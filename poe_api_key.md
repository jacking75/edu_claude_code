# Poe의 API Key로 Claude Code 사용하기  
  
## Windows
아래를 파워쉘에서 실행한다.  

### POE API 키 저장
  
```  
[Environment]::SetEnvironmentVariable("POE_API_KEY", "나의apikey", "User")
```    
  

### Claude Code가 Poe를 사용하도록 설정
  
```  
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://api.poe.com", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", "나의apikey", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "", "User")
```
  
  
### 설정이 잘 되었는지 쉘을 종료 후 재 시작한 후 아래 명령어로 확인한다
  
```
Write-Host "ANTHROPIC_BASE_URL: $env:ANTHROPIC_BASE_URL"
Write-Host "ANTHROPIC_AUTH_TOKEN: $($env:ANTHROPIC_AUTH_TOKEN.Substring(0, 10))..."
Write-Host "ANTHROPIC_API_KEY: '$env:ANTHROPIC_API_KEY'"
```  
  
작업할 디렉토리에 가서 claude로 실행한다. 