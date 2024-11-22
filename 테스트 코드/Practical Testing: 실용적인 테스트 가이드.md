### Practical Testing: 실용적인 테스트 가이드
**코드** : [practice-testing-cafekiosk](https://github.com/hoonsmemory/practice-testing-cafekiosk)
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

















