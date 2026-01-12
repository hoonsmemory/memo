# rules/naming.md

## 타입 네이밍(기본 권장)
- 요청 바디: *Command (예: CreateArticleCommand)
- 조회 응답: *View (예: ArticleView)
- 토큰 응답: AccessTokenCarrier
- 목록 래퍼: ArrayCarrier<T>
- 페이지 래퍼: PageCarrier<T>

## 표기
- id: string(UUID)
- 시간: string(YYYY-MM-DDThh:mm:ss.sss) 필요 시 Utc 접미(registeredTimeUtc)
