## 2026-05-29 16:54 KST - 로컬 Markdown 렌더링 대응

로컬 file:// 환경에서도 Markdown 문서를 볼 수 있도록 docs-data.js를 생성했다.
viewer.html이 GitHub Pages에서는 실제 .md 파일을 먼저 fetch하고, 실패하거나 로컬 파일 환경이면 내장 문서 데이터를 사용하도록 수정했다.
CDN Markdown 렌더러가 로딩되지 않는 경우에도 기본 렌더러로 최소 표시가 가능하도록 fallback을 추가했다.
docs-data.js에는 학습 문서 36개가 포함되고 working_log.md는 제외되는지 검증했다.

## 2026-05-29 16:48 KST - 인포그래픽 이미지 뷰어 추가

상단 인포그래픽을 원본 이미지 파일로 직접 열지 않고 image-viewer.html로 열리도록 변경했다.
image-viewer.html은 이미지를 브라우저 창 안에 맞춰 표시하도록 max-width, max-height, object-fit: contain을 적용했다.
목록으로 돌아가기와 원본 열기 버튼을 제공해 화면 맞춤 보기와 원본 확인을 분리했다.

## 2026-05-29 16:42 KST - Markdown 웹 뷰어 추가

viewer.html을 추가해 저장소의 Markdown 문서를 웹 페이지 안에서 렌더링해 볼 수 있게 했다.
index.html의 학습 문서 링크를 모두 viewer.html?file=문서명.md 형식으로 변경했다.
뷰어에는 허용 문서 목록, 원본 Markdown 링크, 목록으로 돌아가기 버튼, 코드 블록 스타일을 추가했다.
학습 문서 36개와 뷰어 링크 36개가 일치하는지 검증했다.

## 2026-05-29 16:25 KST - index.html 문서 허브 정비

index.html의 깨진 한글 텍스트와 기존 랜딩 페이지 구조를 Claude Code 학습 자료실 형태로 재구성했다.
저장소의 마크다운 문서 36개를 모두 링크하고, 추천 학습 순서와 주제별 자료 묶음을 추가했다.
입문, CLAUDE.md, 확장/자동화, 팀 운용, 환경/보안 자료를 나누어 처음 보는 사람이 탐색하기 쉽게 정리했다.
인포그래픽 이미지를 첫 화면 시각 자료로 연결하고 외부 공식 문서 링크도 함께 배치했다.
