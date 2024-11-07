## @Transactinal
스프링 프레임워크는 @Transactional 을 제공해 비즈니스 로직에 들어있는 JDBC 기술을 걷어내 서비스 계층을 특정 기술에 종속되지 않도록 하였고 다양한 옵션을 통해 트랜잭션을 제어할 수 있다.  

### @Transactional 적용 전
```java
@Slf4j
@RequiredArgsConstructor
public class MemberService {

    private final DataSource dataSource;
    private final MemberRepository memberRepository;

    public void transfer(String fromId, String toId, int money) throws SQLException {
        Connection con = dataSource.getConnection();
        try {

            //트랜잭션 시작
            con.setAutoCommit(false);
            
            //비즈니스 로직
            Member fromMember = memberRepository.findById(con, fromId);
            Member toMember = memberRepository.findById(con, toId);

            memberRepository.update(con, fromId, fromMember.getMoney() - money);
            validation(toMember);
            memberRepository.update(con, toId, toMember.getMoney() + money);

            //성공시 커밋
            con.commit(); 
        } catch (Exception e) {
            //실패시 롤백
            con.rollback(); 
            throw new IllegalStateException(e);
        } finally {
            //리소스 초기화
            if (con != null) {
                try {
                    con.setAutoCommit(true); //커넥션 풀을 고려해서 디폴트값인 true로 변경한다.
                    con.close(); //비즈니스 로직이 종료된 후 최종적으로 서비스계층에서 커넥션을 닫아준다.
                } catch (Exception e) {
                    log.info("error", e);
                }
            }
        }
    }
}
```
@Transactional 을 적용하기 전에는 서비스 계층에 트랜잭션을 사용하기 위해서 DataSource, Connection,SQLException 같은 JDBC 기술에 의존해야만 했다.  
물론 템플릿 콜백 패턴을 적용한 TransactionTemplate 을 사용해 코드 수를 줄일 수 있지만 순수한 서비스 계층으로 분리하진 못한다.  

<br>

### @Transactional 적용 후
```java
@Slf4j
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional
    public void transfer(String fromId, String toId, int money) {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);

        memberRepository.update(fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(toId, toMember.getMoney() + money);
    }
    
}​
```
@Transactional 을 적용해 불필요한 코드들을 없애고 순수한 서비스 계층으로 비즈니스의 핵심 로직만 관리할 수 있게 되었다.

<br>

### 트랜잭션 AOP 적용 전체 흐름
<img width="885" alt="스크린샷 2024-11-07 오후 2 25 31" src="https://github.com/user-attachments/assets/d26cafd0-33ba-4d10-bd2e-e7f2c7ad212d">

1. 클라이언트의 요청으로 AOP 프록시가 호출된다.  
2. 스프링 컨테이너를 통해 트랜잭션 매니저를 획득한다.  
3. 트랜잭션 매니저는 getTransaction 메서드를 호출해 트랜잭션을 시작한다.  
4. 트랜잭션이 시작되면 가장 먼저 데이터소스를 사용해서 데이터베이스 커넥션을 획득한다.  
5. 커넥션을 수동 커밋 모드로 변경(con.setAutoCommit(false))한다.  
6. 트랜잭션을 동기화하기 위해 트랜잭션 동기화 매니저에 보관한다. (쓰레드 로컬 사용)  
7. AOP 프록시가 실제 서비스를 호출한다.  
8. 데이터 접근 계층에서 DataSourceUtils.getConnection() 을 호출해서 트랜잭션 동기화 매니저에 보관된 커넥션을 꺼내서 사용한다.  
9. 획득한 커넥션을 사용해서 SQL을 데이터베이스에 전달해서 실행한다.  
10. 실행 결과에 따라서 트랜잭션 매니저는 커밋하거나 롤백을 한다.  
11. 전체 리소스를 정리한다.  
11-1. 트랜잭션 동기화 매니저를 정리한다. (쓰레드 로컬은 사용 후 꼭 정리해야 한다.)  
11-2. con.setAutoCommit(true) 로 되돌린다. (커넥션 풀을 고려해야 한다.)  
11-3. con.close 메서드를 호출해서 커넥션을 종료한다. (커넥션 풀을 사용하는 경우 con.close() 를 호출하면 커넥션 풀에 반환된다.)  

스프링 컨테이너는 빈을 등록하기 전 메서드 혹은 클래스에 @Transactional 이 적용되어있는지 확인 후 적용이 되어있다면 AOP 프록시를 만들게 된다.  
이때 @Transactional이 여러 메서드 중 하나라도 적용되어 있다면 우선 AOP 프록시로 만들고 해당 메서드에 요청이 들어왔을 때 @Transactional 을 다시 한번 체크해서 트랜잭션을 적용시킬지 결정한다.  

<br>

### 트랜잭션 옵션
value, transactionManager
```java
public class TxService {
      @Transactional(value = "memberTxManager")
      public void member() {...}
      
      @Transactional(transactionManager = "orderTxManager")
      public void order() {...}
}
```
사용할 트랜잭션 매니저를 지정할 때는 value , transactionManager 둘 중 하나에 트랜잭션 매니저의 스프링 빈의 이름을 적어주면 된다. 사용하는 트랜잭션 매니저가 둘 이상이라면 위처럼 트랜잭션 매니저의 이름을 지정해서 구분하면 되고, 두 값을 생략하면 기본으로 등록된 트랜잭션 매니저를 사용하기 때문에 대부분 생략한다.

 

**rollbackFor**
```java
@Transactional(rollbackFor = MemberRollbackException.class)
public void member() {...}
```
예외 발생시 스프링 트랜잭션의 기본 정책은 언체크 예외인 RuntimeException , Error 와 그 하위 예외가 발생하면 롤백하고 체크 예외인 Exception 과 그 하위 예외들은 커밋한다. 하지만 UserRollbackException 처럼 임의의 Exception을 구현하고 rollbackFor에 적용할 경우 해당 예외가 발생되면 롤백의 대상이 된다. 만약 하위 예외도 있다면 하위 예외들도 대상에 포함된다.


**noRollbackFor**
```java
@Transactional(noRollbackFor = MemberNoRollbackException.class)
public void member() {...}
```
noRollbakcFor을 쓰면 롤백 기본 정책에 추가로 어떤 예외가 발생했을 때 롤백하면 안되는지 지정할 수 있다.

 

**isolation**

트랜잭션 격리 수준을 지정할 수 있다. 기본 값은 데이터베이스에서 설정한 트랜잭션 격리 수준을 사용하는 DEFAULT 이다.

대부분 데이터베이스에서 설정한 기준을 따르며 애플리케이션 개발자가 트랜잭션 격리 수준을 직접 지정하는 경우는 드물다.

**DEFAULT** : 데이터베이스에서 설정한 격리 수준을 따른다.
**READ_UNCOMMITTED** : 커밋되지 않은 읽기
**READ_COMMITTED** : 커밋된 읽기
**REPEATABLE_READ** : 반복 가능한 읽기
**SERIALIZABLE** : 직렬화 가능


**timeout**
```java
@Transactional(timeout = 60)
public void member() {...}
```
트랜잭션 수행 시간에 대한 타임아웃을 초 단위로 지정한다. 기본 값은 트랜잭션 시스템의 타임아웃을 사용한다.

운영 환경에 따라 동작하는 경우도 있고 그렇지 않은 경우도 있기 때문에 꼭 확인하고 사용해야 한다.

 

**readOnly**
```java
@Transactional(readOnly = true)
public void member() {...}
```
readOnly=true 옵션을 사용하면 읽기 전용 트랜잭션이 생성되어 등록, 수정, 삭제가 안되고 읽기 기능만 작동한다.  
만약 읽기 외 다른 기능을 사용할 경우 예외가 발생될 수 있다. readOnly 기능은 드라이버나 데이터베이스에 따라 정상 동작하지 않는 경우도 있으니 적용 전에 테스트가 필요하다.  

<br>

### 트랜잭션 전파
복잡한 비즈니스를 가질수록 쓰기와 변경이 많이 일어난다. 스프링 트랜잭션은 상황에 따라 커밋과 롤백을 할 수 있도록 지원한다.

**전파 옵션**  

**REQUIRED**

가장 많이 사용하는 기본 설정이다. 기존 트랜잭션이 없으면 생성하고, 있으면 참여한다. 트랜잭션이 필수라는 의미로 이해하면 된다. (필수이기 때문에 없으면 만들고, 있으면 참여한다.)

기존 트랜잭션 없음: 새로운 트랜잭션을 생성한다.
기존 트랜잭션 있음: 기존 트랜잭션에 참여한다.

 

**REQUIRES_NEW**

항상 새로운 트랜잭션을 생성한다.
기존 트랜잭션 없음: 새로운 트랜잭션을 생성한다.

기존 트랜잭션 있음: 새로운 트랜잭션을 생성한다.

 

**SUPPORT**

트랜잭션을 지원한다는 뜻이다. 기존 트랜잭션이 없으면, 없는대로 진행하고, 있으면 참여한다.
기존 트랜잭션 없음: 트랜잭션 없이 진행한다.
기존 트랜잭션 있음: 기존 트랜잭션에 참여한다.

 

**NOT_SUPPORT**

트랜잭션을 지원하지 않는다는 의미이다.
기존 트랜잭션 없음: 트랜잭션 없이 진행한다.
기존 트랜잭션 있음: 트랜잭션 없이 진행한다. (기존 트랜잭션은 보류한다)

 

**MANDATORY**

의무사항이다. 트랜잭션이 반드시 있어야 한다. 기존 트랜잭션이 없으면 예외가 발생한다.

기존 트랜잭션 없음: IllegalTransactionStateException 예외 발생
기존 트랜잭션 있음: 기존 트랜잭션에 참여한다.

 

**NEVER**

트랜잭션을 사용하지 않는다는 의미이다. 기존 트랜잭션이 있으면 예외가 발생한다. 기존 트랜잭션도 허용하지 않는 강한 부정의 의미로 이해하면 된다.
기존 트랜잭션 없음: 트랜잭션 없이 진행한다.
기존 트랜잭션 있음: IllegalTransactionStateException 예외 발생

 

**NESTED**

중첩 트랜잭션은 외부 트랜잭션의 영향을 받지만, 중첩 트랜잭션은 외부에 영향을 주지 않는다. 중첩 트랜잭션이 롤백 되어도 외부 트랜잭션은 커밋할 수 있다. 외부 트랜잭션이 롤백 되면 중첩 트랜잭션도 함께 롤백된다. (중첩 트랜잭션은 JPA에서는 사용할 수 없다.)

기존  트랜잭션 없음: 새로운 트랜잭션을 생성한다.

기존  트랜잭션 있음: 중첩 트랜잭션을 만든다

<br>

### 물리 트랜잭션, 논리 트랜잭션
<img width="876" alt="스크린샷 2024-11-07 오후 2 27 41" src="https://github.com/user-attachments/assets/921ba05c-5ac5-4949-bb4d-c600b6af6607">

위 그림은 일반적으로 사용되는 전파 옵션인 REQUIRED 를 적용한 예이다.

물리 트랜잭션 : 데이터베이스 커넥션을 가르킨다. 하나의 요청에 여러 로직이 있더라도 각 로직이 모두 REQUIRED 옵션으로 적용되어 있다면 하나의 물리 트랜잭션으로 적용된다. 즉 하나의 커넥션을 가지고 요청이 처리되므로 동일한 정책을 가지게 되는데, 로직 중 예외가 발생되면 같은 커넥션을 가진 요청은 모두 롤백이 되거나 커밋이 된다. 반대로 각 로직이 REQUIRES_NEW으로 되어 있다면 각 로직마다 물리 트랜잭션을 가지므로 정책도 각각 달라진다.

 

논리 트랜잭션 : 로직 단위이다. 각 로직마다 전파 옵션을 가지며 REQUIRED 를 적용했다면 동일한 물리 트랜잭션 안에서 실행되므로 정책도 같아진다. 만약 물리 트랜잭션을 분리 시키고 싶다면 REQUIRES_NEW을 적용하면 된다.

**REQUIRES_NEW 예시**  
<img width="885" alt="스크린샷 2024-11-07 오후 2 28 08" src="https://github.com/user-attachments/assets/65df2062-c710-4107-bcd3-ab0e5c12a4e8">
만약 멤버를 저장하거나 변경에 사용된 데이터와 성공 유무를 로그로 저장하는 로직이 있다고 가정해보자.

멤버 저장을 하는 로직이 실패하더라도 로그를 저장하는 로직은 저장해야하므로 REQUIRES_NEW를 적용한다. 이렇게 하면 물리 트랜잭션은 2개로 분리되어 각각 다른 정책이 적용되므로 멤버 저장이 실패해도 로그 저장 로직은 롤백되지 않고 커밋을 한다. 반대로 로그를 저장하는 로직이 실패와 상관없이 멤버 저장 로직이 성공한다면 롤백되지 않고 커밋된다.






