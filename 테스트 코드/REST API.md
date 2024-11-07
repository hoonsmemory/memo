## REST API
**코드** : [rest-api-practice](https://github.com/hoonsmemory/rest-api-practice)

**아키텍쳐**
* Client-Server
* Stateless
* Cache 
* Uniform Interface 
* Layered System
* Code-On-Demand (optional) 

<br>

**Uniform Interface**   
Url자원 식별
자원은 url로 식별 가능해야한다.

표현을 통한 자원 조작
HTTP 표준 메서드를 통해 자원을 저장, 변경, 조회, 삭제를 할 수 있어야 한다.

**Self-descriptive message**   
* 메시지 스스로 메시지에 대한 설명이 가능해야 한다.
* 서버가 변해서 메시지가 변해도 클라이언트는 그 메시지를 보고 해석이 가능하다.
* 확장 가능한 커뮤니케이션
HTTP Header에 타입을 명시하고 각 메시지(자원)들은 MIME types에 맞춰 표현되어야 합니다. 예를 들어 json를 반환한다면 application/json으로 명시해주어야합니다.MIME types는 문서, 파일 등의 특성과 형식을 나타내는 표준입니다. IETF의 RFC6838에 정의 및 표준화되어 있습니다. Font/tf, 'text/plain, text/csv등을 말합니다.
예를 들어 json타입의 데이터를 보낼 때는 헤더의 'Content-Type' ='application/json' 을 명시해야 함을 말합니다.


**HATEOAS**  
* 하이퍼미디어(링크)를 통해 애플리케이션 상태 변화가 가능해야 한다.
* 링크 정보를 동적으로 바꿀 수 있다. (Versioning 할 필요 없이!) 


**Self-descriptive message 해결 방법**   
* 방법 1: 미디어 타입을 정의하고 IANA에 등록하고 그 미디어 타입을 리소스 리턴할 때 Content-Type으로 사용한다. 
* 방법 2: profile 링크 헤더를 추가한다 . ( 발표 영상 41 분 50초 ) 
    * 브라우저들이 아직 스팩 지원을 잘 안해
    * 대안으로 HAL 의 링크 데이터에 p rofile 링크 추가 

**HATEOAS 해결 방법**   
* 방법1: 데이터에 링크 제공
    * 링크를 어떻게 정의할 것인가? —> HAL
* 방법2: 링크 헤더나 Location을 제공 

<br>

Serialize
Json -> 객체 : DeSerialization
객체 -> Json : Serialization —> 일반적으로 빈으로 등록하면 BeanSerializer 가 변환해준다. (Errors 같은건 변환할 수 없다.)

**커스텀 JSON Serializer 만들기**  

● extends JsonSerializer<T> (Jackson JSON 제공)  
● @JsonComponent (스프링 부트 제공)  

**ObjectMapper 커스터마이징**  
spring.jackson.deserialization.fail-on-unknown-properties=true (객체로 변환하면서 없는 데이터를 받았을 경우 에러를 준다. ex 객체는 test란 값이 없는데 test란 값이 json에 있을 경우)  

<br>

**테스트**
스프링 부트 슬라이스 테스트 
@WebMvcTest  
○ MockMvc 빈을 자동 설정 해준다. 따라서 그냥 가져와서 쓰면 됨.  
○ 웹 관련 빈만 등록해 준다. (슬라이스)  

MockMvc  
● 스프링 MVC 테스트 핵심 클래스  
● 웹 서버를 띄우지 않고도 스프링 MVC (DispatcherServlet)가 요청을 처리하는 과정을 확인할 수 있기 때문에 컨트롤러 테스트용으로 자주 쓰임.  
  
@MockBean  
● Mockito를 사용해서 mock 객체를 만들고 빈으로 등록해 줌.  
● (주의) 기존 빈을 테스트용 빈이 대체 한다.  
  
**스프링 부트 통합 테스트 @WebMvcTest 빼고 다음 애노테이션 추가**  
○ @SpringBootTest  
○ @AutoConfigureMockMvc  
Repository @MockBean 코드 제거  

**REST Docs 자동 설정** 
@AutoConfigureRestDocs 

RestDocMockMvc 커스터마이징 요청 응답 포맷팅이 필요한 경우 사용
RestDocsMockMvcConfigurationCustomizer 구현한 빈 등록 
@TestConfiguration 

**TestConfiguration 등록하는 방법**
클래스 위에 추가 : @Import(RestDocsConfiguration.class)


**HATEOAS**  
현재 상태에 적합한 URL 정보를 전달해줘야 한다.  
링크와 API 문서 구현  

Location URI 만들기  
HATEOS가 제공하는 linkTo(), methodOn() 사용  
linkTo() 메소드 : 컨트롤러나, 핸들러 메서드로부터 URI 정보 읽어올 때 쓰는 메서드 입니다. 스프링 HATEOAS 프로젝트에서 제공하는 유틸리티라고 생각하시면 되요.  

Header 의 Location 정보는 ResponseEntity.created(uri정보)에 의해 만들어진다.  

**링크 예시**  
로그인하지 않은 사용자일 경우  
self profile: 이벤트 목록 조회 API 문서로 링크  
get-an-event: 이벤트 하나 조회하는 API 링크  
next: 다음 페이지 (optional) prev: 이전 페이지 (optional)   

로그인한 사용자일 경우(Bearer 헤더에 유효한 AccessToken이 들어있는 경우)
* self 
* profile: 이벤트 목록 조회 API 문서로 링크 
* get-an-event: 이벤트 하나 조회하는 API 링크 
* create-new-event: 이벤트를 생성할 수 있는 API 링크
* next: 다음 페이지 (optional) 
* prev: 이전 페이지 (optional) 


Resource 만들기(Resource<T> 사용)
* extends ResourceSupport의 문제 
    * @JsonUnwrapped로 해결
    * extends Resource<T>로 해결 

Resource를 만들면 add해서 링크를 추가할 수 있다.


**스프링 REST Docs**
Sping MVC Test를 사용해서 문서의 일부분(조각 : snippets)을 생성하는 라이브러리  
Asciidoctor를 사용한다.  
[Swagger와 차이](https://velog.io/@monkeydugi/Spring-Rest-Docs-%EC%B1%84%ED%83%9D-%EC%9D%B4%EC%9C%A0)  

**REST Docs 코딩 andDo(document(“doc-name”, snippets))**
snippets  
* links()  
    * requestParameters() + parameterWithName()  
    * pathParameters() + parametersWithName()  
    * requestParts() + partWithname()  
    * requestPartBody()  
    * requestPartFields()  
    * requestHeaders() + headerWithName()   
    * requestFields() + fieldWithPath()  
    * responseHeaders() + headerWithName()   
    * responseFields() + fieldWithPath()  
    * relaxedResponseFields —> 문서의 일부분만 확인해도 되게끔 설정해주는 prefix (단점 : 정확한 테스트 문서를 생성하지 못한다.)  
    * ...   
