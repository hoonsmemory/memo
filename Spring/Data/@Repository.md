## @Repository
@Repository 는 데이터 액세스 계층에서 사용되는 애너테이션이다.  
보통은 컴포넌트 스캔의 대상이 되기 위해 적용하지만 스프링 부트와 JPA 를 사용할 경우 예외 변환기(PersistenceExceptionTranslationPostProcessor) 를 자동으로 등록해서 @Repository 를 적용한 빈을 프록시로 변환한다.  
이렇게 변환된 프록시는 데이터 액세스 계층에서 예외가 발생하면 스프링 예외 추상화(DataAccessException)로 변환해준다.  
<br>
스프링 예외 추상화로 변환 시 이점은 JPA 뿐만 아니라 JdbcTemplete, MyBatis 와 같은 데이터 접근 기술에서 발생되는 예외를 통합적으로 관리한다는 점이다.  
따라서 데이터 접근 기술을 변경하더라도 예외 처리하는 로직을 수정할 필요가 없게 된다.  

### 테스트
```java
@Slf4j
@Component
public class ItemRepository {

    private final EntityManager em;

    public ItemRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public List<Item> findAll() {
        return em.createQuery("selectt i from Item i", Item.class)
                .getResultList();
    }
}


@Slf4j
@Repository
public class ItemRepository {

    private final EntityManager em;

    public ItemRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public List<Item> findAll() {
        return em.createQuery("selectt i from Item i", Item.class)
                .getResultList();
    }
}


@Slf4j
@SpringBootTest
class ItemRepositoryTest {
    @Autowired
    ItemRepository repository;

    @Test
    void selectItem() {
        log.info("repository = {}", repository.getClass());
        List<Item> all = repository.findAll();
    }
}
```
테스트는 select 함수가 오타 났을 때 발생하는 오류를 확인해보려고 한다.  
첫 번째 테스트 클래스는 @Component 를 적용하였고 두 번째 테스트 클래스는 @Repository 를 적용하였다.  
테스트코드에서는 클래스의 정보와 조회하는 코드를 작성하였다.  

**@Component 를 적용한 클래스  결과**  
```java
repository = class jpa.JpaItemRepository
java.lang.IllegalArgumentException: org.hibernate.hql.internal.ast.QuerySyntaxException
```
<br>

**@Repository 를 적용한 클래스  결과**  
```java
repository = class jpa.JpaItemRepository$$EnhancerBySpringCGLIB$$6f682d37
org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.hql.internal.ast.QuerySyntaxException
```
두 테스트를 비교해보면 @Repository 를 적용했을 때 프록시로 빈을 등록한 것과 스프링 예외 추상화로 변경된 것을 알 수 있다.  
<br>

참고로 @Controller 를 적용하면 스프링 프레임워크에 의해 URI와 @RequestMapping 을 비교해 원하는 핸들러를 찾을 수 있게 해준다.  
@Service 를 적용하면 별 다른 기능은 없고 @Component 처럼 단순 컴포넌트 스캔의 대상만 된다.











