## OSIV(Open Sesion In View)
![Interceptor](https://github.com/user-attachments/assets/3c8307ed-61b9-40cb-9eac-4982f5076d87)
spring.jpa.open-in-view : true (기본값)  
이 기본값을 뿌리면서 애플리케이션 시작 시점에 warn 로그를 남기는 것은 이유가 있다. OSIV 전략은 트랜잭션 시작처럼 최초 데이터베이스 커넥션 시작 시점부터 API 응답이 끝날 때 까지 영속성 컨텍스트와 데이터베이스 커넥션을 유지한다.  
그래서 지금까지 View Template이나 API 컨트롤러에서 지연 로딩이 가능했던 것이다. 지연 로딩은 영속성 컨텍스트가 살아있어야 가능하고, 영속성 컨텍스트는 기본적으로 데이터베이스 커넥션을 유지한다. 이것 자체가 큰 장점이다.  
  
그런데 이 전략은 너무 오랜시간동안 데이터베이스 커넥션 리소스를 사용하기 때문에, 실시간 트래픽이 중요한 애플리케이션에서는 커넥션이 모자랄 수 있다.  
이것은 결국 장애로 이어진다. 예를 들어서 컨트롤러에서 외부 API를 호출하면 외부 API 대기 시간 만큼 커넥션 리소스를 반환하지 못하고, 유지해야 한다.  

**스프링에서는 만약 트랜잭션 밖에서 에러가 발생하면 JpaTransactionManager.doRollback 메서드를 통해 EntityManager.clear() 을 해준다.**  
![Interceptor](https://github.com/user-attachments/assets/040f7687-5890-49e4-a7f9-a2b7ca2c320d)
spring.jpa.open-in-view: false  
OSIV를 끄면 트랜잭션을 종료할 때 영속성 컨텍스트를 닫고, 데이터베이스 커넥션도 반환한다. 따라서 커넥션 리소스를 낭비하지 않는다.  
OSIV를 끄면 모든 지연로딩을 트랜잭션 안에서 처리해야 한다. 따라서 지금까지 작성한 많은 지연 로딩 코드를 트랜잭션 안으로 넣어야 하는 단점이 있다.  
그리고 view template에서 지연로딩이 동작하지 않는다. 결론적으로 트랜잭션이 끝나기 전에 지연 로딩을 강제로 호출해 두어야 한다.  


