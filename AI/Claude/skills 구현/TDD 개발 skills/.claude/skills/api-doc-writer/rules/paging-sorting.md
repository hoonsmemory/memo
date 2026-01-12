# rules/paging-sorting.md

## 정렬
- "등록 시점 역순" 같은 정책은 정책 섹션에 명시한다.
- 목록/탐색 응답에는 정렬 관련 필드(registeredTimeUtc 등)가 있다면 그 필드를 기준으로 설명한다.

## 페이징
- continuationToken 기반이면 아래를 명시한다.
  - 페이지 크기(예: 최대 10개)
  - continuationToken이 비어있을 때의 의미(예: 첫 페이지)
  - 마지막 페이지 처리(예: 더 이상 없으면 continuationToken이 빈 값인지, 누락인지)
- 무효 continuationToken 처리(400/200/첫페이지 등)는 프로젝트 컨벤션이 필요하면 질문한다.
