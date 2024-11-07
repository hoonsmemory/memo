### Practical Testing: 실용적인 테스트 가이드
**코드** : [practice-testing-cafekiosk](https://github.com/hoonsmemory/practice-testing-cafekiosk)

<br>

### 단위 테스트
![✓ 작은 코드 단위를 독립적으로 검증하는 테스트](https://github.com/user-attachments/assets/428b2fc0-c846-4ec8-8482-bf9ffa033756)
![테스트 케이스 세분화하기](https://github.com/user-attachments/assets/b96cb79e-9c7b-4d07-9a39-25d4de676231)
![테스트하기 어려운 영역을 구분하고 분리하기](https://github.com/user-attachments/assets/8895963d-599a-4480-9e25-bb251629a4f7)
![테스트하기 어려운 영역](https://github.com/user-attachments/assets/9db5dcf3-a505-456b-bd88-3b559d365908)
![✓ 같은 입력에는 항상 같은 결과](https://github.com/user-attachments/assets/083f2c2f-f1b5-4af6-b5bb-155f13f69b33)

<br>

### TDD: Test Driven Development
![Test Driven Development](https://github.com/user-attachments/assets/bffd6f4a-0564-4bbc-99f1-c8f6df1de737)
![Test Driven Development](https://github.com/user-attachments/assets/9cc1fb8e-656b-446a-abd0-97a430ed6032)  
  
**TDD의 중요성**  
![선 기능 구현, 후 테스트 작성](https://github.com/user-attachments/assets/365628dc-34e7-4872-8793-1459228f7f60)
![선 테스트 작성, 후 기능 구현](https://github.com/user-attachments/assets/b73a23d4-6237-468d-88a8-14d5fd071957)
![클라이언트 관점에서의 피드백을 주는 Test Driven](https://github.com/user-attachments/assets/6895fe5b-b636-4425-92c6-2bdd1804c39a)
![TDD 관점의 변화](https://github.com/user-attachments/assets/a181840c-39ef-4028-995e-3aa88d1d82b2)

<br>

### 테스트는 [ ]다.
![✓ 다양한 테스트 케이스를 통해](https://github.com/user-attachments/assets/2fed6512-bb8a-4b08-9cb6-67bf7704bb8d)
![DisplayName을 섬세하게](https://github.com/user-attachments/assets/5dd5ca2b-6ea9-41fe-873e-528fbbffb583)
![DisplyName을 섬세하게](https://github.com/user-attachments/assets/dfb803c8-f698-4ad1-a35c-eed400aeeb62)
![DisplayName을 섬세하게](https://github.com/user-attachments/assets/a7a03db5-981f-491c-b084-29d1befe8966)
![BDD, Behavior Driven Development](https://github.com/user-attachments/assets/9f42a35f-9832-455a-ae5f-3dca3db6fa08)
![Given  When  Then](https://github.com/user-attachments/assets/0236bd7d-e3e3-43fd-8128-d5aea2d8e7a9)
![Given  When  Then](https://github.com/user-attachments/assets/8b6cf816-a57e-402c-a8dd-ad9e77f7533f)

<br>

### Spring & JPA 기반 테스트
![✓ 여러 모듈이 협력하는 기능을 통합적으로 검증하는 테스트](https://github.com/user-attachments/assets/87467224-69cc-4721-b70b-585f17ed6b8d)
![JPA Java Persistence API](https://github.com/user-attachments/assets/db24c48f-90f1-46e0-b9d0-2b80a9ef0eaa)
![ORM Object-Relational Mapping](https://github.com/user-attachments/assets/af1a07df-411a-4ba6-9674-c3418b7253f2)
![Persistence Layer](https://github.com/user-attachments/assets/53af5c7a-5c16-4b0a-9490-2804837bd74f)
![Layered Architecture](https://github.com/user-attachments/assets/bdc17c56-a3f7-40c8-a8b4-e1c31c3e2a5a)
![Business Layer](https://github.com/user-attachments/assets/61c650a7-7255-44b2-b27f-bef42e94634c)
![Layered Architecture](https://github.com/user-attachments/assets/45fffd59-fe4b-4b2d-a9ea-c1cec0df93b9)
![Presentation Layer](https://github.com/user-attachments/assets/e5ed5461-1e44-40f7-ac73-83c8405c95be)
![Layered Architecture](https://github.com/user-attachments/assets/44ce9e0b-d09b-4a93-9ec1-3e5cc3a7bd79)
![MockMvc](https://github.com/user-attachments/assets/d1c143e5-a5b9-4ad0-9401-34e5ee723bc8)
Mock이란 테스트하고자하는 대상에 의존관계 등 방해가 되는 것들(준비해야하는 것이 많을 때)을 무시하고 테스트하고 싶은 것에 집중하고 싶을 때 사용.  

<br> 

### Mock을 마주하는 자세
![✓ Dummy](https://github.com/user-attachments/assets/408bca6f-3cec-4ec8-817f-8d241957b87e)
![Test Double](https://github.com/user-attachments/assets/7d61dc90-612e-47af-944f-3f7af14da8b4)  
**마틴 파울러의 ["Mocks Aren't Stubs"](https://chatgpt.com/share/671b4911-5a54-800e-889d-abfd0d6a2c9e) 글 정리**  

**스텁은 상태 기반 테스트에 사용**  
즉, 특정 메서드가 호출되었을 때의 반환값에만 관심이 있으며, 그 결과로 인해 객체의 상태나 반환값이 제대로 나오는지를 확인  

**목은 동작 기반 테스트에 사용**  
목은 특정 메서드가 호출되었는지, 호출 순서가 올바른지, 호출 횟수가 맞는지 등 행위나 동작을 검증하는 데 중점을 둔다.  

**@Mock**  
@Mock은 객체의 모의(Mock) 버전을 생성합니다.  
테스트 대상 객체가 의존하는 객체의 메서드 호출 시 실제 객체가 아닌 모의 객체의 메서드를 호출하게 되어, 테스트 환경에서 의존성을 격리할 수 있습니다.  
* 외부 의존성을 분리하여 테스트 대상 객체에 집중하고자 할 때.  
* 실제 데이터베이스나 네트워크 통신 등을 수행하지 않고, 정해진 결과를 반환하게 하여 빠른 테스트를 원할 때.  

**@Spy**  
@Spy는 실제 객체를 사용하면서 일부 메서드만 모의하여 행위를 변경할 수 있게 합니다.  
Mock 객체와 달리 실제 메서드가 호출되며, 필요한 메서드만 모의할 수 있습니다. 이는 객체의 특정 동작만 변경하고 나머지는 그대로 유지하고자 할 때 유용합니다.  
* 일부 메서드의 실제 동작을 확인하고 싶을 때.  
* 특정 메서드의 반환 값만 변경하고, 나머지 메서드들은 실제 구현을 사용하고 싶을 때.  

**@InjectMocks**  
@InjectMocks는 테스트 대상 객체를 생성하고, @Mock이나 @Spy로 생성한 객체를 해당 객체의 필드로 자동 주입하는 데 사용됩니다.  
이를 통해 의존성이 주입된 객체를 생성하여 단위 테스트를 진행할 수 있습니다. Spring의 @Autowired와 유사한 방식으로 모의 객체를 주입한다고 보면 됩니다.  
* 단일 테스트 대상 객체에 필요한 의존성을 모두 주입하여 테스트 준비를 자동화하고 싶을 때.  
* @Mock 또는 @Spy로 설정한 객체를 주입하여 실제 환경과 유사한 테스트를 구성하고 싶을 때.  

ChatGPT 질문 : [@Mock, @Spy, @InjectMocks 예제](https://chatgpt.com/share/671b5325-c91c-800e-a604-e663a319724e)

<br>

### 더 나은 테스트를 작성하기 위한 구체적 조언 
1. 하나의 테스트에는 하나의 조건의 테스트만 진행해야 한다.  
2. 완벽하게 제어하기  
3. 테스트 환경의 독립성을 보장하자  
4. 테스트 간 독립성을 보장하자(공유 필드 사용 X)  
5. 한 눈에 들어오는 Test Fixture 구성하기(given절에 대한 내용)  
    1. @BeforeEach, @BeforeAll : 각 테스트 입장에서 봤을 때 아예 몰라도 테스트 내용을 이해하는데 문제가 없는가?, 수정해도 모든 테스트에 영향을 주지 않는가? 고민 후 사용.  
    2. data.sql같은 미리 셋업하는 파일(기본절 텍스트 픽스처의 파편화) 사용을 지양하자.  
    3. 테스트용 빌더 메서드는 필요한 데이터만 매개변수로 두고 사용. (createProduct(String productNumber, ProductType type, ProductSellingStatus sellingStatus, String name, String price) ... )  
    4. 위처럼 빌더 메서드를 각 테스트할 클래스 안에서 만들어 사용.(하나의 클래스에서 구현 후 공유하여 사용할 경우 한 클래스에 수많은 메서드들로 복잡해진다.)  
6. Test Fixture 클렌징  
    1. deleteAllInBatch() 를 사용할 땐 외래키 등을 고려해서 순서를 잘 정해서 지워야한다. (deleteAll() : findAll()해서 각 엔티티 1건씩 제거, 연관관계인 엔티티도 모두 제거)  
    2. @Transactional 사용.(사이드 이펙트 주의 : test 상에서 사용할 경우 자동으로 롤백된다. (비즈니스 로직 등 트랜잭션이 생성되어야 할 곳에 적용되어있지 않다면, 변경 감지 등 그런 작업이 누락될 수 있다.))  
        1. @BeforeEach 사용 가능 질문 : https://www.inflearn.com/questions/947467  
7. @ParameterizedTest  
    1. 값을 여러개로 바꿔서 테스트하고 싶을 경우 사용.  
    2. Junit5 Parameterized Tests, Spock Data Tables  
        1. value 사용 질문 : https://www.inflearn.com/questions/1150252  
8. @DynamicTest  
    1. 여러 상황을 하나의 테스트로 묶어 시나리오 테스트하고 싶을 때 사용. 
9. 테스트 수행도 비용이다. 환경 통합하기  
    1. 통합 테스트를 수행할 때 여러번 Spring 서버를 띄우는 것을 효과적으로 개선하기 위해 TestSupport 추상클래스를 상속받아 Repository/Service 계층 테스트시에 통합된 환경을 구축하는게 더 좋은 것  

















