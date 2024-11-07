## 스프링 MVC
예전 스프링 프레임워크를 사용하기 전에는 클라이언트의 요청을 처리하기 위해 각 요청마다 1:1 구조로 서블릿을 구현해야만 했다.  
규모가 작은 사이트라면 문제가 적겠지만 규모가 큰 사이트라면 수많은 서블릿을 관리해야 하는 수고로움이 있었다.  
그래서 이러한 문제점을 해결하기 위해 스프링 MVC를 구현하게 되었고, 물론 다른 요구사항에 맞춰서도 기능을 확장시켜왔다.  
스프링을 사용한다는 것은 스프링 컨테이너(IoC, DI...)를 사용한다는 의미도 있지만 스프링 MVC를 사용한다고 봐도 무방하다.  

<br>

### MVC 용어
Model : 도메인 객체 또는 DTO로 화면에 전달할 또는 화면에서 전달 받은 데이터를 담고 있는 객체이다.  
View : 데이터를 보여주는 화면으로 클라이언트가 서비스를 요청을 하게 되면 결괏값을 확인할 수 있다.  
Controller : 사용자 입력을 받아 모델 객체의 데이터를 변경하거나, 모델 객체를 뷰에 전달하는 역할을 한다.  

<br>

### 스프링 MVC 구조
<img width="878" alt="스크린샷 2024-11-07 오후 1 42 27" src="https://github.com/user-attachments/assets/f35d37d1-467b-4900-94d3-73b1a0757866">

**동작 순서**
1. 핸들러 조회 : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회 : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행 : 핸들러 어댑터를 실행한다.
4. 핸들러 실행 : 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환 : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. viewResolver 호출 : 뷰 리졸버를 찾고 실행한다.
7. View반환 : 뷰리졸버는 뷰의 논리이름을 물리이름으로 바꾸고,렌더링역할을 담당하는 뷰 객체를 반환한다. 
8. 뷰렌더링 : 뷰를 통해서 뷰를 렌더링한다.

<br>

### DispacherServlet
**DispacherServlet 코드 요약**  
```java
protected void doDispatch(HttpServletRequest request, 
                          HttpServletResponse response) throws Exception {
    
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    ModelAndView mv = null;
    
    //1. 핸들러 조회
    mappedHandler = getHandler(processedRequest); if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return; 
    }
    
    //2.핸들러 어댑터 조회-핸들러를 처리할 수 있는 어댑터
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
    
    //3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환 mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
private void processDispatchResult(HttpServletRequest request, 
                                   HttpServletResponse response, 
                                   HandlerExecutionChain mappedHandler, 
                                   ModelAndView mv, 
                                   Exception exception) throws Exception {
    //뷰 렌더링 호출
    render(mv, request, response);
}
protected void render(ModelAndView mv, 
                      HttpServletRequest request, 
                      HttpServletResponse response) throws Exception {
    View view;
    String viewName = mv.getViewName(); 
    
    //6. 뷰 리졸버를 통해서 뷰 찾기,7.View 반환
    view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
    
    //8. 뷰 렌더링
    view.render(mv.getModelInternal(), request, response);
}
```
DispatcherServlet은 요청마다 1:1 구조로 가졌던 HttpServlet을 대신하여 모든 요청을 DispatcherServlet이 받아 알맞게 응답을 해주는 기능을 해준다.  
이 때 DispatcherServlet은 전략 패턴을 사용하는데 예를 들면, 클래스 레벨에 @Controller 혹은 @RequestMapping 을 적용하게 되면 RequestMappingHandlerMapping, RequestMappingHandlerAdapter 를 전략으로 매칭되어 요청에 대한 처리를 하기도 하고, @ResponseBody 가 적용되어 있다면 핸들러를 실행 후 Converter를 사용해서 응답 본문을 만들고 응답을 보내기도 한다.  

<br>

### DispatcherServlet의 전략
```java
protected void initStrategies(ApplicationContext context) {
	initMultipartResolver(context);
	initLocaleResolver(context);
	initThemeResolver(context);
	initHandlerMappings(context);
	initHandlerAdapters(context);
	initHandlerExceptionResolvers(context);
	initRequestToViewNameTranslator(context);
	initViewResolvers(context);
	initFlashMapManager(context);
}
```
DispatcherServlet은 런타임 시점에 스프링 컨테이너(ApplicationContext)로 부터 필요한 전략을 갖게된다.  
각 전략은 설정파일을 통해 빈으로 등록할 수도 있고, 물론 설정을 하지 않으면 기본 전략을 사용하게 된다.  

예시로 HandlerMapping에 관련한 전략을 확인해보겠다.
```java
private void initHandlerMappings(ApplicationContext context) {
    this.handlerMappings = null;

    if (this.detectAllHandlerMappings) {
        // Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
        Map<String, HandlerMapping> matchingBeans =
                BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
        if (!matchingBeans.isEmpty()) {
            this.handlerMappings = new ArrayList<>(matchingBeans.values());
            // We keep HandlerMappings in sorted order.
            AnnotationAwareOrderComparator.sort(this.handlerMappings);
        }
    }
    else {
        try {
            HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
            this.handlerMappings = Collections.singletonList(hm);
        }
        catch (NoSuchBeanDefinitionException ex) {
            // Ignore, we'll add a default HandlerMapping later.
        }
    }

    // Ensure we have at least one HandlerMapping, by registering
    // a default HandlerMapping if no other mappings are found.
    if (this.handlerMappings == null) {
        this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
        if (logger.isTraceEnabled()) {
            logger.trace("No HandlerMappings declared for servlet '" + getServletName() +
                    "': using default strategies from DispatcherServlet.properties");
        }
    }
}
```
우선 handlerMappings 는 List 타입으로 전략에 사용할 빈들을 담는다.  
detectAllHandlerMappings 는 boolean 타입으로 여러 가지 전략을 담을 때는 true(default)를 주고 하나의 전략만 적용하고 싶으면 false로 두면 된다.  

설정 파일에 등록한 빈이 없을 경우 DispatcherServlet.properties 파일에서 기본 전략을 가져와 객체를 생성한 뒤 handlerMappings 에 담는다.  
HandlerMapping에 대한 기본 전략은 3가지로 BeanNameUrlHandlerMapping, RequestMappingHandlerMapping, RouterFunctionMapping 이 있으며  
주로 @RequestMapping 을 사용하기 때문에 RequestMappingHandlerMapping을 전략으로 사용한다.


DispatcherServlet만 보더라도 스프링은 객체지향적으로 굉장히 잘 완성된 프레임워크라는걸 알 수 있다.  
이런 코드를 분석하다보면 좋은 코드를 짤 수 있을 것라 생각한다.  

<br>

### RequestMappingHandlerAdapter 동작 방식
<img width="873" alt="스크린샷 2024-11-07 오후 1 46 07" src="https://github.com/user-attachments/assets/aac01147-dc3d-4bf0-9f98-7d82756ab7c8">
HandlerApdapter의 종류로 HttpRequestHandlerAdapter, SimpleControllerHandlerAdapter 등이 있지만  
대부분은 가장 강력한 기능을 갖춘 RequestMappingHandlerAdapter를 전략으로 사용한다.  
  
RequestMappingHandlerAdapter는 핸들러의 실행을 주관하기도 하지만 핸들러가 실행하는데 필요한 파라미터와 핸들러가 실행 후 리턴 값으로 보낼 파라미터를 상황에 맞춰 변환 시켜주는 역할을 한다.  
따라서 스프링은 매우 다양한 파라미터를 사용할 수 있게 되었다.  

**ArgumentResolver** : 요청에 대한 파라미터를 변환할 수 있도록 도와주는 역할  
**ReturnValueHandler** : 응답에 대한 파라미터를 변환할 수 있도록 도와주는 역할  

사실 두 리졸버는 직접적으로 변환을 시켜주는 것이 아니다.  
리졸버는 핸들러에 적용된 애너테이션이나 파라미터 타입, 리턴 타입 등을 보고 어떻게 하면 데이터를 상황에 맞춰 변환할 수 있는지 지원을 해주는 역할을 하고, 직접적으로 변환을 시켜주는 역할은 HttpMessageConverter가 한다.











