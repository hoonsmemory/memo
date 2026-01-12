# 테스트 네이밍 규칙

## 개요

좋은 테스트 이름은 **테스트가 실패했을 때 무엇이 잘못되었는지 즉시 알 수 있게** 해줍니다.
시나리오 문장을 그대로 테스트 메서드명으로 사용하는 것을 권장합니다.

---

## 권장: 한국어 서술형 네이밍

### 장점
- 시나리오와 1:1 매핑
- 비개발자도 이해 가능
- 실패 시 문제 즉시 파악

### 예시

```java
@Test
void 올바르게_요청하면_200_OK_상태코드를_반환한다() { }

@Test
void email_속성이_지정되지_않으면_400_Bad_Request_상태코드를_반환한다() { }

@Test
void email_속성이_올바른_형식을_따르지_않으면_400_Bad_Request_상태코드를_반환한다() { }

@Test
void 비밀번호를_올바르게_암호화한다() { }

@Test
void 판매자가_아닌_사용자의_접근_토큰을_사용하면_403_Forbidden_상태코드를_반환한다() { }

@Test
void 상품_목록을_등록_시점_역순으로_정렬한다() { }
```

### 시나리오 → 메서드명 변환

| 시나리오 | 메서드명 |
|---------|---------|
| 올바르게 요청하면 200 OK를 반환한다 | `올바르게_요청하면_200_OK를_반환한다` |
| email이 없으면 400을 반환한다 | `email이_없으면_400_Bad_Request를_반환한다` |
| 두 번째 페이지를 올바르게 반환한다 | `두_번째_페이지를_올바르게_반환한다` |

**규칙**: 공백 → 밑줄(`_`)

---

## 대안: 영어 네이밍 스타일

프로젝트 컨벤션에 따라 영어를 사용할 수도 있습니다.

### should 스타일

```java
@Test
void should_return_200_when_request_is_valid() { }

@Test
void should_return_400_when_email_is_missing() { }

@Test
void should_encrypt_password_correctly() { }
```

### given_when_then 스타일

```java
@Test
void given_valid_request_when_signup_then_returns_204() { }

@Test
void given_null_email_when_signup_then_returns_400() { }

@Test
void given_existing_email_when_signup_then_returns_400() { }
```

### it 스타일 (RSpec/Jest 영향)

```java
@Test
void it_returns_200_for_valid_request() { }

@Test
void it_returns_400_when_email_is_null() { }
```

---

## 테스트 클래스 네이밍

### API 테스트

```java
// 패턴: {HTTP메서드}_{경로}_{설명}_specs
class POST_seller_signUp_specs { }
class GET_seller_products_specs { }
class POST_shopper_issueToken_specs { }

// 또는 한국어
class 판매자_회원가입_API_테스트 { }
class 판매자_상품_목록_조회_API_테스트 { }
```

### 도메인/서비스 테스트

```java
// 패턴: {클래스명}Test
class UserServiceTest { }
class OrderCalculatorTest { }
class PasswordValidatorTest { }

// 또는 한국어
class 주문_금액_계산기_테스트 { }
```

---

## 네이밍 패턴별 예시

### 정상 케이스

```java
// 한국어
void 올바르게_요청하면_201_Created_상태코드를_반환한다()
void 상품을_성공적으로_등록한다()
void 사용자_정보를_올바르게_반환한다()

// 영어
void should_return_201_when_request_is_valid()
void should_create_product_successfully()
void should_return_user_info_correctly()
```

### 검증 실패

```java
// 한국어
void email_속성이_지정되지_않으면_400_Bad_Request_상태코드를_반환한다()
void email_속성이_올바른_형식을_따르지_않으면_400_Bad_Request_상태코드를_반환한다()
void password_속성이_8자_미만이면_400_Bad_Request_상태코드를_반환한다()

// 영어
void should_return_400_when_email_is_null()
void should_return_400_when_email_format_is_invalid()
void should_return_400_when_password_is_less_than_8_chars()
```

### 권한/보안

```java
// 한국어
void 접근_토큰이_없으면_401_Unauthorized_상태코드를_반환한다()
void 판매자가_아닌_사용자가_접근하면_403_Forbidden_상태코드를_반환한다()
void 비밀번호를_올바르게_암호화한다()

// 영어
void should_return_401_when_access_token_is_missing()
void should_return_403_when_user_is_not_seller()
void should_encrypt_password_correctly()
```

### 비즈니스 로직

```java
// 한국어
void 상품_목록을_등록_시점_역순으로_정렬한다()
void 같은_사용자의_식별자는_항상_같다()
void 다른_판매자가_등록한_상품은_조회할_수_없다()

// 영어
void should_sort_products_by_registration_time_desc()
void should_return_same_id_for_same_user()
void should_not_allow_access_to_other_sellers_products()
```

### 유일성/중복

```java
// 한국어
void email_속성에_이미_존재하는_이메일이_지정되면_400_Bad_Request_상태코드를_반환한다()
void username이_중복되면_가입에_실패한다()

// 영어
void should_return_400_when_email_already_exists()
void should_fail_when_username_is_duplicate()
```

### 페이지네이션

```java
// 한국어
void 첫_번째_페이지를_올바르게_반환한다()
void 두_번째_페이지를_올바르게_반환한다()
void 마지막_페이지에는_continuationToken이_없다()

// 영어
void should_return_first_page_correctly()
void should_return_second_page_correctly()
void should_not_have_continuation_token_on_last_page()
```

---

## 안티 패턴

### 나쁜 예시

```java
// ❌ 너무 짧음 - 무엇을 테스트하는지 불명확
void test1() { }
void testSignUp() { }

// ❌ 구현 세부사항 노출
void testUserRepositorySave() { }
void testValidationAnnotation() { }

// ❌ 모호한 표현
void testError() { }
void testSuccess() { }
void testValidation() { }

// ❌ 여러 시나리오를 하나에
void testEmailValidation() { }  // null, 형식, 중복 모두?
```

### 좋은 예시

```java
// ✅ 구체적이고 명확함
void email이_없으면_400_Bad_Request_상태코드를_반환한다() { }
void email_형식이_잘못되면_400_Bad_Request_상태코드를_반환한다() { }
void email이_이미_존재하면_400_Bad_Request_상태코드를_반환한다() { }
```

---

## @DisplayName 활용

JUnit 5에서는 `@DisplayName`으로 더 읽기 좋은 이름을 지정할 수 있습니다.

```java
@Test
@DisplayName("올바르게 요청하면 200 OK 상태코드를 반환한다")
void returns_200_when_valid_request() { }

@Test
@DisplayName("email 속성이 지정되지 않으면 400 Bad Request 상태코드를 반환한다")
void returns_400_when_email_is_null() { }
```

### 장점
- 메서드명은 짧게 유지
- 테스트 결과에 @DisplayName이 표시됨
- 공백, 특수문자 자유롭게 사용 가능

---

## 테스트 결과 출력 예시

### 좋은 네이밍의 테스트 결과

```
POST /seller/signUp
  ✓ 올바르게 요청하면 204 No Content 상태코드를 반환한다
  ✓ email 속성이 지정되지 않으면 400 Bad Request 상태코드를 반환한다
  ✗ email 속성이 올바른 형식을 따르지 않으면 400 Bad Request 상태코드를 반환한다
      Expected: 400
      Actual: 500
  ✓ username 속성이 지정되지 않으면 400 Bad Request 상태코드를 반환한다
```

실패한 테스트가 무엇을 검증하는지 즉시 알 수 있습니다.

### 나쁜 네이밍의 테스트 결과

```
UserControllerTest
  ✓ test1
  ✓ test2
  ✗ test3
      Expected: 400
      Actual: 500
  ✓ test4
```

test3이 무엇을 테스트하는지 코드를 봐야 알 수 있습니다.

---

## 프로젝트별 컨벤션

팀/프로젝트에서 하나의 스타일을 선택하고 일관되게 사용하세요:

| 스타일 | 장점 | 단점 |
|-------|------|------|
| 한국어 서술형 | 직관적, 시나리오와 1:1 | IDE 자동완성 불편 |
| should_xxx | 영어권 표준 | 길어질 수 있음 |
| given_when_then | BDD 명확 | 매우 길어짐 |
| @DisplayName | 유연함 | 중복 관리 필요 |

**권장**: 시나리오를 한국어로 작성했다면, 테스트명도 한국어로 통일