# 테스트 패턴 가이드

## 테스트 구조 패턴

### AAA 패턴 (Arrange-Act-Assert)

가장 기본적이고 권장되는 테스트 구조입니다.

```java
@Test
void 올바르게_요청하면_200_OK를_반환한다() {
    // Arrange (준비)
    var request = new CreateUserRequest("user@example.com", "username", "password123");

    // Act (실행)
    ResponseEntity<UserResponse> response = client.postForEntity("/users", request, UserResponse.class);

    // Assert (검증)
    assertThat(response.getStatusCode().value()).isEqualTo(200);
}
```

### Given-When-Then 패턴 (BDD 스타일)

행동 기반 개발(BDD) 스타일의 테스트 구조입니다.

```java
@Test
void 올바르게_요청하면_200_OK를_반환한다() {
    // Given (주어진 상황)
    var request = new CreateUserRequest("user@example.com", "username", "password123");

    // When (행동)
    ResponseEntity<UserResponse> response = client.postForEntity("/users", request, UserResponse.class);

    // Then (결과)
    assertThat(response.getStatusCode().value()).isEqualTo(200);
}
```

---

## 테스트 케이스 패턴

### 1. 정상 케이스 (Happy Path)

기대하는 정상 동작을 검증합니다.

```java
@Test
void 올바르게_요청하면_201_Created를_반환한다() {
    // Arrange
    var command = new CreateProductCommand("상품명", 10000, 100);

    // Act
    ResponseEntity<Void> response = client.postForEntity("/products", command, Void.class);

    // Assert
    assertThat(response.getStatusCode().value()).isEqualTo(201);
    assertThat(response.getHeaders().getLocation()).isNotNull();
}
```

### 2. 필수값 검증 (Null/Empty Check)

필수 필드가 없을 때의 동작을 검증합니다.

```java
@Test
void email이_없으면_400_Bad_Request를_반환한다() {
    // Arrange
    var command = new CreateUserCommand(null, "username", "password");

    // Act
    ResponseEntity<Void> response = client.postForEntity("/users", command, Void.class);

    // Assert
    assertThat(response.getStatusCode().value()).isEqualTo(400);
}

@Test
void email이_빈_문자열이면_400_Bad_Request를_반환한다() {
    // Arrange
    var command = new CreateUserCommand("", "username", "password");

    // Act
    ResponseEntity<Void> response = client.postForEntity("/users", command, Void.class);

    // Assert
    assertThat(response.getStatusCode().value()).isEqualTo(400);
}
```

### 3. 형식 검증 (Format Validation)

필드 형식이 잘못됐을 때의 동작을 검증합니다.

```java
@ParameterizedTest
@ValueSource(strings = {"invalid", "invalid@", "invalid@test", "@test.com"})
void email_형식이_잘못되면_400_Bad_Request를_반환한다(String invalidEmail) {
    // Arrange
    var command = new CreateUserCommand(invalidEmail, "username", "password");

    // Act
    ResponseEntity<Void> response = client.postForEntity("/users", command, Void.class);

    // Assert
    assertThat(response.getStatusCode().value()).isEqualTo(400);
}
```

### 4. 경계값 테스트 (Boundary Testing)

경계 조건에서의 동작을 검증합니다.

```java
@Test
void username이_2자이면_400_Bad_Request를_반환한다() {
    // 최소 3자 필요 - 경계값 아래
    var command = new CreateUserCommand("user@test.com", "ab", "password");

    ResponseEntity<Void> response = client.postForEntity("/users", command, Void.class);

    assertThat(response.getStatusCode().value()).isEqualTo(400);
}

@Test
void username이_3자이면_성공한다() {
    // 최소 3자 필요 - 정확히 경계값
    var command = new CreateUserCommand("user@test.com", "abc", "password123");

    ResponseEntity<Void> response = client.postForEntity("/users", command, Void.class);

    assertThat(response.getStatusCode().value()).isEqualTo(204);
}
```

### 5. 유일성 검증 (Uniqueness)

중복 데이터 처리를 검증합니다.

```java
@Test
void email이_이미_존재하면_400_Bad_Request를_반환한다() {
    // Arrange - 먼저 사용자 생성
    var existingEmail = "existing@test.com";
    client.postForEntity("/users",
        new CreateUserCommand(existingEmail, "user1", "password123"),
        Void.class);

    // Act - 같은 이메일로 다시 생성 시도
    ResponseEntity<Void> response = client.postForEntity("/users",
        new CreateUserCommand(existingEmail, "user2", "password123"),
        Void.class);

    // Assert
    assertThat(response.getStatusCode().value()).isEqualTo(400);
}
```

### 6. 권한/인증 검증 (Security)

```java
@Test
void 접근_토큰이_없으면_401_Unauthorized를_반환한다() {
    // Act - 토큰 없이 요청
    ResponseEntity<Void> response = client.getForEntity("/users/me", Void.class);

    // Assert
    assertThat(response.getStatusCode().value()).isEqualTo(401);
}

@Test
void 권한이_없는_사용자가_접근하면_403_Forbidden을_반환한다() {
    // Arrange - 구매자 토큰으로 설정
    fixture.loginAsShopper();

    // Act - 판매자 전용 API 접근
    ResponseEntity<Void> response = fixture.client()
        .getForEntity("/seller/products", Void.class);

    // Assert
    assertThat(response.getStatusCode().value()).isEqualTo(403);
}
```

### 7. 데이터 검증 (Data Verification)

저장된 데이터가 올바른지 검증합니다.

```java
@Test
void 비밀번호를_올바르게_암호화한다() {
    // Arrange
    String rawPassword = "password123";
    var command = new CreateUserCommand("user@test.com", "username", rawPassword);

    // Act
    client.postForEntity("/users", command, Void.class);

    // Assert
    User saved = userRepository.findByEmail("user@test.com").orElseThrow();
    assertThat(saved.getHashedPassword()).isNotEqualTo(rawPassword);
    assertThat(passwordEncoder.matches(rawPassword, saved.getHashedPassword())).isTrue();
}
```

### 8. 목록/정렬 검증 (List/Sorting)

```java
@Test
void 상품_목록을_등록_시점_역순으로_정렬한다() {
    // Arrange
    UUID id1 = fixture.registerProduct("상품1");
    UUID id2 = fixture.registerProduct("상품2");
    UUID id3 = fixture.registerProduct("상품3");

    // Act
    List<ProductView> products = fixture.getProducts();

    // Assert - 최신순 (id3, id2, id1)
    assertThat(products)
        .extracting(ProductView::id)
        .containsExactly(id3, id2, id1);
}
```

### 9. 페이지네이션 검증

```java
@Test
void 첫_번째_페이지를_올바르게_반환한다() {
    // Arrange
    fixture.registerProducts(25);  // 25개 상품 등록 (페이지당 10개)

    // Act
    PageCarrier<ProductView> page = fixture.getProductPage(null);

    // Assert
    assertThat(page.items()).hasSize(10);
    assertThat(page.continuationToken()).isNotNull();
}

@Test
void 두_번째_페이지를_올바르게_반환한다() {
    // Arrange
    fixture.registerProducts(25);
    String firstPageToken = fixture.getProductPage(null).continuationToken();

    // Act
    PageCarrier<ProductView> page = fixture.getProductPage(firstPageToken);

    // Assert
    assertThat(page.items()).hasSize(10);
}
```

---

## 테스트 픽스처 패턴

### 테스트 헬퍼 클래스

반복되는 테스트 준비 코드를 캡슐화합니다.

```java
public record TestFixture(
    TestRestTemplate client,
    UserRepository userRepository,
    ProductRepository productRepository
) {
    // 사용자 생성 헬퍼
    public void createUser(String email, String username, String password) {
        client.postForEntity("/users",
            new CreateUserCommand(email, username, password),
            Void.class);
    }

    // 토큰 발급 헬퍼
    public String issueToken(String email, String password) {
        var response = client.postForEntity("/auth/token",
            new IssueTokenQuery(email, password),
            AccessTokenCarrier.class);
        return response.getBody().accessToken();
    }

    // 인증된 상태로 설정
    public void loginAs(String email, String password) {
        String token = issueToken(email, password);
        client.getRestTemplate().getInterceptors().add(
            (request, body, execution) -> {
                request.getHeaders().setBearerAuth(token);
                return execution.execute(request, body);
            }
        );
    }

    // 상품 등록 헬퍼
    public UUID registerProduct(String name) {
        var response = client.postForEntity("/products",
            new RegisterProductCommand(name, "http://example.com/img.jpg", "설명", 10000, 100),
            Void.class);
        String location = response.getHeaders().getLocation().toString();
        return UUID.fromString(location.substring(location.lastIndexOf('/') + 1));
    }
}
```

### 테스트 데이터 생성기

무작위 테스트 데이터를 생성합니다.

```java
public class EmailGenerator {
    private static final AtomicInteger counter = new AtomicInteger();

    public static String generate() {
        return "user" + counter.incrementAndGet() + "@test.com";
    }
}

public class UsernameGenerator {
    private static final AtomicInteger counter = new AtomicInteger();

    public static String generate() {
        return "user" + counter.incrementAndGet();
    }
}
```

### 커스텀 어설션

도메인 특화 검증 로직을 재사용합니다.

```java
public class ProductAssertions {
    public static ThrowingConsumer<ProductView> matchesCommand(RegisterProductCommand command) {
        return product -> {
            assertThat(product.name()).isEqualTo(command.name());
            assertThat(product.imageUri()).isEqualTo(command.imageUri());
            assertThat(product.description()).isEqualTo(command.description());
            assertThat(product.priceAmount()).isEqualByComparingTo(command.priceAmount());
            assertThat(product.stockQuantity()).isEqualTo(command.stockQuantity());
        };
    }
}

// 사용
assertThat(product).satisfies(matchesCommand(command));
```

---

## ParameterizedTest 활용

### @ValueSource - 단순 값 목록

```java
@ParameterizedTest
@ValueSource(strings = {"", "ab", "user name", "user@name"})
void username이_올바른_형식이_아니면_400을_반환한다(String invalidUsername) {
    var command = new CreateUserCommand("user@test.com", invalidUsername, "password123");

    ResponseEntity<Void> response = client.postForEntity("/users", command, Void.class);

    assertThat(response.getStatusCode().value()).isEqualTo(400);
}
```

### @MethodSource - 복잡한 데이터

```java
@ParameterizedTest
@MethodSource("invalidPasswordProvider")
void password가_올바른_형식이_아니면_400을_반환한다(String invalidPassword) {
    var command = new CreateUserCommand("user@test.com", "username", invalidPassword);

    ResponseEntity<Void> response = client.postForEntity("/users", command, Void.class);

    assertThat(response.getStatusCode().value()).isEqualTo(400);
}

static Stream<String> invalidPasswordProvider() {
    return Stream.of(
        "",           // 빈 문자열
        "short",      // 8자 미만
        "abcd1234",   // 연속 문자 포함
        "12345678"    // 연속 숫자 포함
    );
}
```

### 커스텀 어노테이션

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@ParameterizedTest
@ValueSource(strings = {"invalid", "invalid@", "@test.com", "test@", ""})
public @interface InvalidEmailSource {}

// 사용
@InvalidEmailSource
void email이_잘못된_형식이면_400을_반환한다(String invalidEmail) {
    // 테스트 코드
}
```