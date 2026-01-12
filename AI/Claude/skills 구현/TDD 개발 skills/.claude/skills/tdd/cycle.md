# Red-Green-Refactor 사이클 상세

## 개요

TDD의 핵심은 **Red-Green-Refactor** 사이클입니다.
이 사이클을 올바르게 따르는 것이 TDD 성공의 핵심입니다.

```
    ┌──────────────────────────────────────────────────┐
    │                                                  │
    │   ┌─────────┐                                    │
    │   │   RED   │  1. 실패하는 테스트 작성                │
    │   │  (빨강)  │  2. 테스트 실행 → 실패 확인             │
    │   └────┬────┘                                    │
    │        │                                         │
    │        ▼                                         │
    │   ┌─────────┐                                    │
    │   │  GREEN  │  3. 최소한의 구현                     │
    │   │  (초록)  │  4. 테스트 실행 → 통과 확인             │
    │   └────┬────┘                                    │
    │        │                                         │
    │        ▼                                         │
    │   ┌─────────┐                                    │
    │   │REFACTOR │  5. 코드 개선                      │
    │   │ (파랑) │  6. 테스트 실행 → 여전히 통과 확인  │
    │   └────┬────┘                                    │
    │        │                                         │
    │        └─────────────────────────────────────────┘
    │                       반복
    │
    └──────────────────────────────────────────────────┘
```

---

## 1단계: Red (실패하는 테스트 작성)

### 목적
- 구현하려는 기능을 **테스트로 정의**
- 테스트가 **실제로 동작하는지 확인** (실패해야 함)

### 절차

1. 시나리오에서 하나의 테스트 케이스 선택
2. 테스트 코드 작성
3. 테스트 실행
4. **실패 확인** ← 중요!

### 예시

```java
// 시나리오: "email이 없으면 400 Bad Request를 반환한다"

@Test
void email이_없으면_400_Bad_Request를_반환한다() {
    // Arrange
    var command = new CreateUserCommand(
        null,           // email 없음
        "username",
        "password123"
    );

    // Act
    ResponseEntity<Void> response = client.postForEntity(
        "/users/signup",
        command,
        Void.class
    );

    // Assert
    assertThat(response.getStatusCode().value()).isEqualTo(400);
}
```

### 테스트 실행 결과 (Red)

```
✗ email이_없으면_400_Bad_Request를_반환한다

Expected: 400
Actual: 500 (또는 다른 값)

BUILD FAILED
```

### 왜 실패를 확인해야 하는가?

- 테스트가 **올바르게 작성되었는지** 확인
- 테스트가 **실제로 검증하고 있는지** 확인
- 이미 구현되어 있다면 테스트가 통과할 것 → 중복 구현 방지

---

## 2단계: Green (테스트 통과)

### 목적
- 테스트를 **통과하는 최소한의 코드** 작성
- 완벽한 코드가 아니어도 됨

### 절차

1. 테스트를 통과하는 코드 작성
2. 테스트 실행
3. **통과 확인**

### 예시

```java
// Controller에 검증 로직 추가
@PostMapping("/users/signup")
public ResponseEntity<Void> signUp(@RequestBody CreateUserCommand command) {
    // 최소한의 구현: email null 체크
    if (command.email() == null) {
        return ResponseEntity.badRequest().build();
    }

    // 기존 로직...
    userService.create(command);
    return ResponseEntity.noContent().build();
}
```

### 테스트 실행 결과 (Green)

```
✓ email이_없으면_400_Bad_Request를_반환한다

BUILD SUCCESSFUL
```

### "최소한의 구현"이란?

```java
// 좋은 예: 테스트 통과에 필요한 최소 코드
if (command.email() == null) {
    return ResponseEntity.badRequest().build();
}

// 나쁜 예: 아직 필요 없는 추가 구현
if (command.email() == null) {
    throw new InvalidCommandException("email", "이메일은 필수입니다");
}
// ↑ 에러 메시지 검증 테스트가 없다면 불필요
```

---

## 3단계: Refactor (리팩토링)

### 목적
- 코드 품질 개선
- **테스트는 계속 통과**해야 함

### 절차

1. 코드 개선점 파악
2. 리팩토링 수행
3. 테스트 실행
4. **여전히 통과 확인**

### 리팩토링 대상

- 중복 코드 제거
- 메서드/클래스 분리
- 변수/메서드 이름 개선
- 가독성 향상

### 예시

```java
// Before: Controller에 검증 로직이 직접 있음
@PostMapping("/users/signup")
public ResponseEntity<Void> signUp(@RequestBody CreateUserCommand command) {
    if (command.email() == null) {
        return ResponseEntity.badRequest().build();
    }
    if (command.username() == null) {
        return ResponseEntity.badRequest().build();
    }
    // ...
}

// After: 검증 로직을 분리
@PostMapping("/users/signup")
public ResponseEntity<Void> signUp(
    @RequestBody @Valid CreateUserCommand command  // @Valid 사용
) {
    userService.create(command);
    return ResponseEntity.noContent().build();
}

// CreateUserCommand에 검증 어노테이션 추가
public record CreateUserCommand(
    @NotNull String email,
    @NotNull String username,
    @NotNull String password
) {}
```

### 리팩토링 후 테스트 실행

```
✓ email이_없으면_400_Bad_Request를_반환한다
✓ 올바르게_요청하면_204_No_Content를_반환한다

BUILD SUCCESSFUL  ← 기존 테스트가 여전히 통과
```

---

## 사이클 반복

### 전체 흐름 예시

```
시나리오 파일:
- [ ] 올바르게 요청하면 204 No Content를 반환한다
- [ ] email이 없으면 400을 반환한다
- [ ] email 형식이 잘못되면 400을 반환한다
```

**1회차 사이클**
```
Red:      "올바르게 요청하면 204를 반환한다" 테스트 작성 → 실패
Green:    기본 Controller 구현 → 통과
Refactor: (필요시)
체크:     - [x] 올바르게 요청하면 204 No Content를 반환한다
```

**2회차 사이클**
```
Red:      "email이 없으면 400을 반환한다" 테스트 작성 → 실패
Green:    email null 체크 추가 → 통과
Refactor: (필요시)
체크:     - [x] email이 없으면 400을 반환한다
```

**3회차 사이클**
```
Red:      "email 형식이 잘못되면 400을 반환한다" 테스트 작성 → 실패
Green:    email 형식 검증 추가 → 통과
Refactor: 검증 로직을 @Valid로 통합 → 모든 테스트 통과
체크:     - [x] email 형식이 잘못되면 400을 반환한다
```

---

## 흔한 실수

### 1. Red 단계 생략

```
❌ 테스트 없이 구현부터 작성
❌ 테스트 작성 후 실패 확인 생략
```

### 2. Green 단계에서 과도한 구현

```
❌ 다음 테스트까지 미리 구현
❌ 완벽한 에러 처리 구현
❌ 성능 최적화
```

### 3. Refactor 단계 생략

```
❌ 테스트 통과 후 바로 다음 테스트로
❌ 중복 코드 방치
```

### 4. 큰 단위로 진행

```
❌ 여러 시나리오를 한 번에 구현
❌ 전체 기능을 한 번에 테스트
```

---

## 사이클 속도

### 이상적인 사이클 시간

- Red → Green: **1-5분**
- Refactor: **필요할 때만**
- 전체 사이클: **5-10분**

### 사이클이 너무 길다면

- 시나리오를 더 작게 분할
- 구현 범위를 축소
- 복잡한 로직은 여러 테스트로 나누기

---

## 팁

1. **작은 단계**: 한 번에 조금씩
2. **자주 커밋**: 각 Green 후 커밋
3. **테스트 자주 실행**: 확신이 없으면 실행
4. **실패를 두려워하지 않기**: Red는 정상
