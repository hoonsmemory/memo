## AOP 사용 주의사항
### 프록시 방식의 AOP 한계
<img width="888" alt="스크린샷 2024-11-07 오후 2 16 38" src="https://github.com/user-attachments/assets/29f9b31a-66e9-4f76-9d6b-bc6f578b8519">

스프링은 프록시 방식의 AOP를 사용한다.  
따라서 AOP를 적용하려면 항상 프록시를 통해서 대상 객체(Target)을 호출해야 한다.  
이렇게 해야 프록시에서 먼저 어드바이스를 호출하고, 이후에 대상 객체를 호출한다.  
만약 프록시를 거치지 않고 대상 객체를 직접 호출하게 되면 AOP가 적용되지 않고, 어드바이스도 호출되지 않는다.  
  
프록시 방식의 AOP는 메서드 내부 호출에 프록시를 적용할 수 없다.  
아래 코드로 테스트를 해보자.  
  
CallServiceV0.java
```java
package hello.aop.internalcall;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class CallServiceV0 {

    /**
     * 내부 호출을 할 경우 호출된 메서드는 프록시를 거치지 않는다.
     */
    public void external() {
        log.info("call external");
        internal(); //내부 메서드 호출(this.internal();
    }

    public void internal() {
        log.info("call internal");
    }
}
```
위 코드는 간단히 테스트하기 위해 만들어진 클래스로 external() 메서드와 internal() 메서드를 가지고 있다.  
external() 메서드는 로그 호출 후 internal() 메서드를 호출한다.  
  
CallLogAspect.java  
```java
package hello.aop.internalcall;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Slf4j
@Aspect
public class CallLogAspect {

    @Before("execution(* hello.aop.internalcall..*.*(..))")
    public void doLog(JoinPoint joinPoint) {
        log.info("aop={}", joinPoint.getSignature());

    }
}
```
위 코드는 hello.aop.internalcall 패키지 이하의 메서드들이 호출되기 전 로그를 남길 수 있도록 만든 Aspect 이다.  

CallServiceV0Test.java
```java
package hello.aop.internalcall;

import hello.aop.AopApplication;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.Import;


@Slf4j
@Import(CallLogAspect.class)
@SpringBootTest(classes = AopApplication.class)
class CallServiceV0Test {

    @Autowired CallServiceV0 callServiceV0;

    @Test
    void external() {
        callServiceV0.external();
    }

    @Test
    void internal() {
        callServiceV0.internal();
    }
}
```
위 테스트 코드를 통해 external() 메서드와 internal() 메서드를 호출해보겠다.

**external() 메서드 테스트 결과**  
1. //프록시 호출  
2. CallLogAspect : aop=void hello.aop.internalcall.CallServiceV0.external()  
3. CallServiceV0 : call external  
4. CallServiceV0 : call internal  
 
  
**internal() 메서드 테스트 결과**  
1. //프록시 호출  
2. CallLogAspect     : aop=void hello.aop.internalcall.CallServiceV0.internal()  
3. CallServiceV0     : call internal  
  
callServiceV0.external() 안에서 internal() 을 호출할 때 CallLogAspect 어드바이스가 호출되지 않았다.  
자바 언어에서 메서드 앞에 별도의 참조가 없으면 this 라는 뜻으로 자기 자신의 인스턴스를 가리킨다.
결과적으로 자기 자신의 내부 메서드를 호출하는 this.internal() 이 되는데, 여기서 this 는 실제 대상 객체(target)의 인스턴스를 뜻한다.
결과적으로 이러한 내부 호출은 프록시를 거치지 않는다. 따라서 어드바이스도 적용할 수 없다.

<br>

### 프록시와 내부 호출 - 대안1 자기 자신 주입
내부 호출을 해결하는 가장 간단한 방법은 자기 자신을 의존관계 주입 받는 것이다.  
  
CallServiceV1.java
```java
package hello.aop.internalcall;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class CallServiceV1 {


    CallServiceV1 callServiceV1;

    @Autowired
    public void setCallServiceV1(CallServiceV1 callServiceV1) {
        log.info("CallServiceV1 setter = {}", callServiceV1.getClass());
        this.callServiceV1 = callServiceV1;
    }

    public void external() {
        log.info("call external");
        callServiceV1.internal(); //자기 자신을 의존관계 주입 받기
    }

    public void internal() {
        log.info("call internal");
    }
}
```
@Autowired를 통해 주입받은 객체는 주입받기 전 이미 AOP가 적용된 프록시 객체이므로  internal() 메서드를 호출하더라도 실제 객체가 호출되지 않는다.  

**참고: 생성자 주입은 본인을 생성하면서 주입해야 하기 때문에 순환 사이클이 만들어져 문제가 발생한다. 반면에 수정자 주입은 스프링이 생성된 이후에 주입할 수 있기 때문에 오류가 발생하지 않는다.**  

<br>

### 프록시와 내부 호출 - 대안2 지연 조회
스프링 빈을 지연해서 조회하면 되는데, ObjectProvider(Provider) 혹은  ApplicationContext 를 사용하면 된다.  

CallServiceV2.java
```java
package hello.aop.internalcall;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class CallServiceV2 {

    CallServiceV2 callServiceV2;

    //private final ApplicationContext applicationContext; //applicationContext 기능이 너무 많기 때문에  ObjectProvider 사용
    private final ObjectProvider<CallServiceV2> callServiceV2ObjectProvider;
    public CallServiceV2(ObjectProvider<CallServiceV2> callServiceV2ObjectProvider) {
        this.callServiceV2ObjectProvider = callServiceV2ObjectProvider;
    }
    
    public void external() {
        log.info("call external");
        // CallServiceV2 callServiceV2 = callServiceV2ObjectProvider.getBean(CallServiceV2.class);
        /**
         * ObjectProvider 는 객체를 스프링 컨테이너에서 조회하는 것을 스프링 빈 생성 시점이 아니라
         * 실제 객체를 사용하는 시점으로 지연할 수 있다.
         */
        CallServiceV2 callServiceV2 = callServiceV2ObjectProvider.getObject();
        callServiceV2.internal();
    }

    public void internal() {
        log.info("call internal");
    }
}
```
ObjectProvider 는 객체를 스프링 컨테이너에서 조회하는 것을 스프링 빈 생성 시점이 아니라 실제 객체를 사용하는 시점으로 지연할 수 있다.  

callServiceProvider.getObject() 를 호출하는 시점에 스프링 컨테이너에서 빈을 조회한다.  
여기서는 자기 자신을 주입 받는 것이 아니기 때문에 순환 사이클이 발생하지 않는다.  

### 프록시와 내부 호출 - 대안3 구조 변경
가장 나은 대안은 내부 호출이 발생하지 않도록 구조를 변경하는 것이다.  
실제 이 방법을 가장 권장한다.  

CallServiceV3.java  
```java
package hello.aop.internalcall;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import javax.annotation.Resource;


@Slf4j
@Component
public class CallServiceV3 {


    //@Autowired
    @Resource(name = "internalService")
    InternalService internalService;
    
    public void external() {
        log.info("call external");
        internalService.internal(); //구조를 분리하여 InternalService를 호출
    }

}
```

InternalService.java  
```java
package hello.aop.internalcall;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class InternalService {

    public void internal() {
        log.info("call internal");
    }
}
```

내부 함수를 호출하지 않고 별도의 클래스를 만들어 호출하게 하는 방법이다.  
내부 호출 자체가 사라지고, callService --> internalService 를 호출하는 구조로 변경되었다. 덕분에 자연스럽게 AOP가 적용된다.  
