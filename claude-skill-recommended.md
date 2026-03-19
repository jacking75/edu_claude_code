# Claude Code Skill 추천

## Context7 → 최신 API 문서 실시간 주입
클로드가 React 19, Next.js 15 같은 최신 라이브러리를 틀리게 쓰는 거 경험해봤죠? 😇
Context7은 최신 버전별 공식 문서를 실시간으로 프롬프트에 주입해줘요
**use context7** 한 마디만 추가하면 됨 🔥
  
claude.com/plugins/context7   

문제: Claude는 학습 데이터 한계로 React 19, Next.js 15 같은 최신 라이브러리를 틀리게 씀 
해결: Context7이 정확한 버전별 라이브러리 문서를 실시간으로 Claude 세션에 주입해서, 오래된 학습 데이터 대신 실제 최신 문서를 참고하게 함
```
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest
```  
  
사용 예시: "Supabase RLS 구현해줘 — use context7"  
  
  
## Ralph Loop → 자율 반복 코딩 에이전트
원래 AI 코딩은 매번 사람이 확인해야 하잖아요  
Ralph Loop는 /ralph-loop "프롬프트" `--max-iterations 10` 명령 하나로 클로드가 완료 조건 충족할 때까지 혼자 반복해줘요
자리 비워도 알아서 함 🫠

claude.com/plugins/ralph-loop  
  
문제: AI 코딩은 매번 사람이 리뷰·승인해야 해서 느림 
해결: 작업, 완료 조건, 최대 반복 횟수를 주면 Claude가 혼자 루프를 돌면서 자체 리뷰·수정을 반복해 목표 달성할 때까지 실행함  
```
`/plugin install ralph-loop@claude-plugins-official
```  
사용 예시:  
/ralph-loop "모든 Jest 테스트를 Vitest로 마이그레이션. 완료 시 <promise>DONE</promise> 출력" --max-iterations 40`
  
  
## Frontend Design → AI티 안 나는 UI 생성
AI가 만든 UI 특유의 느낌 알죠? 보라색 그라디언트 + Inter 폰트 + 카드 떡칠  
Frontend Design 스킬은 프로덕션 수준의 독특한 디자인 코드를 뽑아줘요 — AI스러운 느낌 없이요  
  
claude.com/plugins/frontend-design  
  
문제: AI가 만든 UI = 보라색 그라디언트 + Inter 폰트 + 카드 컴포넌트의 반복 😅   
해결: 명확한 디자인 방향, 독특한 타이포그래피, 체계적인 컬러 시스템을 적용해서 실제 프로덕션 수준의 컴포넌트 코드를 생성함  
```bash
/plugin install frontend-design@claude-plugins-official
```  
    
	
## Code Review Agents → 멀티 에이전트 PR 리뷰
혼자 코드 리뷰하면 놓치는 거 생기잖아요  
버그 탐지, 테스트, 타입, 코드 품질, 단순화 담당 에이전트들이 병렬로 PR을 훑고 신뢰도 기반 필터링으로 핵심만 보고해줘요 !  
노이즈 없이 핵심만 🎯  
  
claude.com/plugins/code-review  
  
문제: 혼자 코드 리뷰하면 놓치는 게 생김 
해결: 여러 전문 리뷰 에이전트가 병렬로 PR을 감사하며, 각각 버그 탐지·보안·성능·CLAUDE.md 준수를 검사한 뒤 신뢰도 80% 이상 이슈만 보고해서 노이즈를 대폭 줄임  
```bash
/plugin install code-review@claude-plugins-official
```  
    
    
## Claude-Mem → 세션 간 프로젝트 기억 유지
원래 대화 끊기면 맥락 다 날아가잖아요  
Claude-Mem은 ChromaDB 벡터 저장소로 대화를 자동 압축하고 다음 세션 시작 시 관련 컨텍스트를 자동으로 불러와줘요!!  
장기 프로젝트엔 이게 제일 체감 컸음 💾  
  
문제: Claude는 대화가 끝나면 프로젝트 맥락을 전부 잊어버림 
해결: 프로젝트 설정, 패턴, 결정 사항을 압축 저장해서 새 세션을 시작해도 이전 컨텍스트가 유지됨  
  
https://github.com/thedotmack/claude-mem  
  