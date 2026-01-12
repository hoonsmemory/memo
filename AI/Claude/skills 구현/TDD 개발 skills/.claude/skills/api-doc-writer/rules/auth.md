# rules/auth.md

## 인증 헤더 표기
- 인증 필요 시:
  Authorization: Bearer {accessToken}

## 역할 제한
- seller 전용 엔드포인트: shopper 토큰 사용 시 403
- shopper 전용 엔드포인트: seller 토큰 사용 시 403

## 테스트 제안(기본)
- 토큰 미사용: 401
- 역할 불일치: 403
- (선택) 토큰 포맷 이상/만료/서명 불일치: 프로젝트에서 다루면 추가
