## HandlerExceptionResolver - API 예외처리
RESTful API를 개발할 때 고려해야 할 사항 중 Response Status를 잘 남겨야 하는 사항이 있다.  
일반적으로 성공한 요청에 대해서는 적절한 상태를 나타내지만 예외가 발생하면 그렇지 않은 경우가 있다.  
따라서 스프링 MVC는 HandlerExceptionResolver 인터페이스를 제공하는데 상속받아 직접 구현하거나 애너테이션을 적용해 예외를 해결하면서 적절한 Response Status를 지정해줄 수 있다.  

**HandlerExceptionResolver 호출 시점**  
<img width="883" alt="스크린샷 2024-11-07 오후 1 56 41" src="https://github.com/user-attachments/assets/98407ad3-7bda-4a5d-ade7-b4000273decc">
HandlerExceptionResolver는 이름에서 유추할 수 있듯이 Handler 밖으로 예외가 던져진 경우 호출되어 처리한다.  
따라서 예외가 발생해도 서블릿 컨테이너까지 예외가 전달되지 않고, 스프링 MVC에서 예외 처리는 끝이 난다. 결과적으로 WAS 입장에서는 정상 처리가 된 것이다.  
이렇게 예외를 HandlerExceptionResolver에서 모두 처리할 수 있다는 것이 핵심이다.  

<br>

**HandlerExceptionResolver 분석**  
HandlerExceptionResolver.java  
```java
public interface HandlerExceptionResolver {

	@Nullable
	ModelAndView resolveException(HttpServletRequest request, 
                                      HttpServletResponse response, 
                                      @Nullable Object handler, 
                                      Exception ex);

}
```
HandlerExceptionResolver를 상속받으면 handler에 대한 정보와 exception 정보를 가지고 예외를 해결할 수 있다.  
리턴값으로 ModelAndView 를 반환하는 이유는 마치 try, catch를 하듯이 Exception 을 처리해서 정상 흐름처럼 변경하는 것이 목적이다.  
이름 그대로 Exception 을 Resolver(해결)하는 것이 목적이다.  

**리턴 값에 따른 동작 방식**  
HandlerExceptionResolver 의 리턴 값에 따른 DispatcherServlet 의 동작 방식은 다음과 같다.  
**빈 ModelAndView** : new ModelAndView() 처럼 빈 ModelAndView 를 반환하면 뷰를 렌더링 하지 않고, 정상 흐름으로 서블릿이 리턴된다.  
**ModelAndView 지정** : ModelAndView 에 View , Model 등의 정보를 지정해서 반환하면 뷰를 렌더링 한다.  
**null** : null 을 반환하면, 다음 ExceptionResolver 를 찾아서 실행한다. 만약 처리할 수 있는 ExceptionResolver 가 없으면 예외 처리가 안되고, 기존에 발생한 예외를 서블릿 밖으로 던진다.  

**HandlerExceptionResolverComposite.java**
```java
public class HandlerExceptionResolverComposite implements HandlerExceptionResolver, Ordered {

	@Nullable
	private List<HandlerExceptionResolver> resolvers;

	@Override
	@Nullable
	public ModelAndView resolveException(HttpServletRequest request, 
                                             HttpServletResponse response, 
                                             @Nullable Object handler, 
                                             Exception ex) {
		if (this.resolvers != null) {
			for (HandlerExceptionResolver handlerExceptionResolver : this.resolvers) {
				ModelAndView mav = handlerExceptionResolver.resolveException(request, response, handler, ex);
				if (mav != null) {
					return mav;
				}
			}
		}
		return null;
	}
}
```
스프링 부트는 기본적으로 우선순위가 가장 높은 ExceptionHandlerExceptionResolver와 ResponseStatusExceptionResolver, DefaultHandlerExceptionResolver를 제공한다.  
개발자는 상황에 맞게 개발을 잘 해놓으면 예외 발생 시 스프링 프레임워크가 유연하게 적용하여 해결한다.  

**ExceptionHandlerExceptionResolver** : @ExceptionHandler가 적용된 예외를해결하고, API 예외 처리는 대부분 이 기능으로 해결한다.  
**ResponseStatusExceptionResolver** : HTTP 상태 코드를 지정해준다. 예) @ResponseStatus(value = HttpStatus.NOT_FOUND)  
**DefaultHandlerExceptionResolver** : 스프링 내부 기본 예외를 처리한다.  

<br>

### 테스트
HandlerExceptionResolver를 상속받아 직접 구현하는 방법도 있지만 직접 구현하게 되면 상당히 복잡하다.  
그리고 스프링에서는 강력한 기능을 가진 ExceptionHandlerExceptionResolver를 제공하므로 보다 쉽게 개발할 수 있다.  

ApiExceptionController.java  
```java
@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {

        if(id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }

        return new MemberDto(id, "hello " + id);
    }
}
```
멤버 요청 시 ID를 잘못된 입력을 했을 경우 IllegalArgumentException을 발생하도록 하는 테스트 코드를 구현했다.  

**테스트 결과**  
<img width="839" alt="스크린샷 2024-11-07 오후 1 59 53" src="https://github.com/user-attachments/assets/fe863dff-42e1-44ba-b5c9-7ebdcab55f9a">

테스트 결과 사용자의 입력문제로 400 에러가 나와야하는데 서버 에러인 500 에러가 나왔고 RESTful API 고려사항에 어긋났음을 알 수 있다.  
이를 해결하기 위해서는 @ControllerAdvice를 적용하고 IllegalArgumentException에 대해 재정의를 해주어야 한다.  
@ControllerAdvice는 @Controller 전역에서 발생할 수 있는 예외를 잡아 처리해주는 애노테이션이다.  

<br>

**ExceptionControllerAdvice.java**  
```java
@Slf4j
@RestControllerAdvice
public class ExceptionControllerAdvice {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExceptionHandler(IllegalArgumentException e) {
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("BadUserId", e.getMessage());
    }
}
``` 
**@RestControllerAdvice** : @ResponseBody를 적용하고 있음으로 API에 대한 예외 처리를 할 경우 @RestControllerAdvice를 적용해주면 된다.  
**@ResponseStatus** :  응답 코드로 보낼 값을 지정한다. (ResponseEntity를 반환해 응답 코드를 나타낼 수도 있다.)  
**@ExceptionHandler** : 적용할 Exception을 지정한다.  

<br>

**대상 컨트롤러 지정 방법**  
1.컨트롤러의 애노테이션을 보고 해당 애노테이션이 적용된 부분만 실행  
@ControllerAdvice(annotations = RestController.class)  

2. 해당 패키지안에서만 실행  
@ControllerAdvice("org.example.controllers")  

3. 해당 클래스만 실행  
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})  

**실행 흐름**  
1 .컨트롤러를 호출한 결과 IllegalArgumentException 예외가 컨트롤러 밖으로 던져진다.  
2. 예외가 발생했으로 ExceptionResolver 가 작동한다. 가장 우선순위가 높은 ExceptionHandlerExceptionResolver 가 실행된다.  
3. ExceptionHandlerExceptionResolver 는 해당 컨트롤러에 IllegalArgumentException 을 처리할 수 있는 @ExceptionHandler 가 있는지 확인한다.  
4. illegalExceptionHandler메서드를 실행한다 @ResponseBody 가 적용되었음으로 HTTP 컨버터가 사용되고, 응답이 JSON으로 반환된다.  
5. @ResponseStatus(HttpStatus.BAD_REQUEST) 를 지정했으므로 HTTP 상태 코드 400으로 응답한다.  
  
**테스트 결과**
<img width="882" alt="스크린샷 2024-11-07 오후 2 02 29" src="https://github.com/user-attachments/assets/395885b1-da46-42bf-8611-cad372a74acc">
ControllerAdvice 를 적용해 Response Status를 500에서 400으로 변경해보았다.  
리턴값으로 code, message 필드를 가진 객체로 리턴을 했지만 아래에 공식 메뉴얼을 보면 마치 스프링의 컨트롤러의 파라미터 응답처럼 다양한 파라미터와 응답을 지정할 수 있는걸 알 수 있다.  














