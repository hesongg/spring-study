### 스프링 웹 MVC

#### Reference) 
	* 스프링 MVC 1편 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/springboot_springmvc_study_1
	
<br>

### 스프링 MVC 기본 기능


<br>

- 참고) 프로젝트 Packaging - ```Jar```
	- Jar를 사용하면 항상 내장 서버(톰캣등)를 사용하고, webapp 경로도 사용하지 않음
	- 내장 서버 사용에 최적화 되어 있는 기능, 최근에는 주로 이 방식을 사용
	
<br>

#### 로깅 간단히 알아보기
	
- 스프링부트는 기본 로깅 라이브러리가 포함된다.
	- 로깅 인터페이스 라이브러리 : SLF4J - http://www.slf4j.org
	- 로깅 구현체 라이브러리 : Logback - http://logback.qos.ch
	
- 로그 선언
	```java
	// 밑에 중 선택해서 사용
	private static final Logger log = LoggerFactory.getLogger(getClass());
	
	private static final Logger log = LoggerFactory.getLogger(Xxx.class)
	```
	
	- @Slf4j : 롬복 사용 가능 -> 위에 로그 선언을 생략하고 사용 가능하다.
	
- 스프링부트 application.properties 설정으로 로그 레벨 설정 가능
	```properties
	#hello.springmvc 패키지와 그 하위 로그 레벨 설정
	logging.level.hello.springmvc=debug
	```
	
- *중요) 올바른 로그 사용법
	- ```log.debug("data="+data)```
		- 로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다.
		- 결과적으로 문자 더하기 연산이 발생한다.
		- ```이렇게 쓰면 안됨!!!```
		
	- ```log.debug("data={}", data)```
		- 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이	발생하지 않는다.
		
- 로그 사용시 장점
	- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
	
	- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절 가능
	
	- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다.
		특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.

	- 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다.
	

<br>






- ```@RestController```
	- @Controller 는 반환 값이 String 이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.
	
	- @RestController 는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다.
		- 따라서 실행 결과로 ok 메세지를 받을 수 있다. @ResponseBody 와 관련이 있는데, 뒤에서 더 자세히 설명...
		

- HTTP 메서드 매핑 축약 - 편리한 축약 애노테이션
	* @GetMapping
	* @PostMapping
	* @PutMapping
	* @DeleteMapping
	* @PatchMapping
	
	
- PathVariable(경로 변수) 사용
	```java
	/**
	 * PathVariable 사용
	 * 변수명이 같으면 생략 가능
	 * @PathVariable("userId") String userId -> @PathVariable userId
	 */
	@GetMapping("/mapping/{userId}")
	public String mappingPath(@PathVariable("userId") String data) {
		log.info("mappingPath userId={}", data);
		return "ok";
	}
	```
	
	- @RequestMapping 은 URL 경로를 템플릿화 할 수 있는데, @PathVariable 을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다.
	
	- @PathVariable 의 이름과 파라미터 이름이 같으면 생략할 수 있다.
		- ```@GetMapping("/mapping/{userId}")``` Url Mapping 애노테이션에 {userId} 파라미터 들어올 때,
		- ```@PathVariable("userId") String data``` == ```@PathVariable String userId```
	
	- PathVariable 다중 사용도 가능하다. (코드 참고)

	
- 최근 HTTP API는 다음과 같이 리소스 경로에 식별자를 넣는 스타일을 선호한다.
	- ```/mapping/userA```
	- ```/users/1```
	
	
- 특정 파라미터 / 헤더 값 / 헤더의 특정 Content-Type 값, Accept 값 등으로 매핑할 수도 있다.



- 요청 매핑 - API 예시
	* 회원 목록 조회:    GET    /users
    * 회원 등록:        POST   /users
    * 회원 조회:        GET    /users/{userId}
    * 회원 수정:        PATCH  /users/{userId}
    * 회원 삭제:        DELETE /users/{userId}
	
<br>

#### HTTP 요청 - 기본, 헤더 조회

- 애노테이션 기반의 스프링 컨트롤러는 다양한 파라미터를 지원

- ```HttpServletRequest```
- ```HttpServletResponse```
- ```HttpMethod``` : HTTP 메서드를 조회한다. ```org.springframework.http.HttpMethod```
- ```Locale``` : Locale 정보를 조회한다.
- ```@RequestHeader MultiValueMap<String, String> headerMap```
  - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
- ```@RequestHeader("host") String host```
  - 특정 HTTP 헤더를 조회한다.
  - 속성
    - 필수 값 여부: required
    - 기본 값 속성: defaultValue
- ```@CookieValue(value = "myCookie", required = false) String cookie```
  - 특정 쿠키를 조회한다.
  - 속성
    - 필수 값 여부: required
    - 기본 값: defaultValue
- 참고) ```MultiValueMap```
  - MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
  - HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
    - ```keyA=value1&keyA=value2```

- 참고)
  - @Conroller 의 사용 가능한 파라미터 목록은 다음 공식 메뉴얼에서 확인할 수 있다.
    - https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annarguments
  - @Conroller 의 사용 가능한 응답 값 목록은 다음 공식 메뉴얼에서 확인할 수 있다.
    - https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types


<br>


#### HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

- 참고) 리소스는 ```/resources/static``` 아래에 두면 스프링 부트가 자동으로 인식한다.
	- Jar 를 사용하면 webapp 경로를 사용할 수 없다. 정적 리소스도 클래스 경로에 함께 포함해야 한다.
	
- HTTP 요청 파라미터 - ```@RequestParam```
	- 스프링이 제공하는 ```@RequestParam``` 을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다.
	
	- @RequestParam : 파라미터 이름으로 바인딩
	- @RequestParam의 name(value) 속성이 파라미터 이름으로 사용
		- @RequestParam("username") String memberName -> request.getParameter("username")
	
	- HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능
	
	- String , int , Integer 등의 단순 타입이면 @RequestParam 도 생략 가능 
		- -> 너무 애노테이션을 안쓰는것도 좋지않은거같다. 명시적인 용도로 어느정도는 사용하는 것이 좋은듯
		
	- @RequestParam 애노테이션을 생략하면 스프링 MVC는 내부에서 required=false 를 적용한다.
		- @RequestParam.required
			- 파라미터 필수 여부
			- 기본값이 파라미터 필수( true )이다.
			
			- 주의! - 파라미터 이름만 사용
				- ```/request-param?username=```
				- 파라미터 이름만 있고 값이 없는 경우 빈문자로 통과
			
			- 주의! - 기본형(primitive)에 null 입력
				- ```/request-param``` 요청
				- @RequestParam(required = false) int age
					- null 을 int 에 입력하는 것은 불가능(500 예외 발생)
					- 따라서 null 을 받을 수 있는 Integer 로 변경하거나, 또는 defaultValue 사용
	
	
	- 파라미터에 값이 없는 경우 ```defaultValue``` 를 사용하면 기본 값을 적용할 수 있다. 이미 기본 값이 있기 때문에 required 는 의미가 없다.
		- defaultValue 는 빈 문자 파라미터의 경우에도 설정한 기본 값이 적용된다.
		
		
	- 파라미터를 Map, MultiValueMap으로 조회할 수 있다.
		- ```@RequestParam Map```
			- Map(key=value)
		
		- ```@RequestParam MultiValueMap```
			- MultiValueMap(key=[value1, value2, ...] 
				- ex) ```(key=userIds, value=[id1, id2])```
		
		- 파라미터의 값이 1개가 확실하다면 Map 을 사용해도 되지만, 그렇지 않다면 MultiValueMap 을 사용하자.


		
- ```@ResponseBody``` : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력


<br>


#### HTTP 요청 파라미터 - @ModelAttribute

- 롬복 ```@Data```
	- @Getter , @Setter , @ToString , @EqualsAndHashCode , @RequiredArgsConstructor 를 자동으로 적용해준다.
	
	- 참고) 롬복의 ```@RequiredArgsConstructor```
		- final 필드에 대해 생성자를 만들어주는 lombok의 annotation 이다.
		- Setter로 값을 변경하는 필드 값의 경우 final을 가질 수 없으므로, 생성자의 파라미터 추가 대상에 해당되지 않는다.
		
	
- 스프링MVC는 ```@ModelAttribute``` 가 있으면 다음을 실행한다.
	- ```HelloData``` 객체를 생성한다.
	- 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩) 한다.
		- 예) 파라미터 이름이 username 이면 ```setUsername()``` 메서드를 찾아서 호출하면서 값을 입력한다
		
- 프로퍼티
	- 객체에 ```getUsername()``` , ```setUsername()``` 메서드가 있으면, 이 객체는 ```username``` 이라는 프로퍼티를 가지고 있다.
	- ```username``` 프로퍼티의 값을 변경하면 ```setUsername()``` 이 호출되고, 조회하면 ```getUsername()``` 이 호출된다.
	

- ```@ModelAttribute``` 는 생략할 수 있다.
	- 그런데 @RequestParam 도 생략할 수 있으니 혼란이 발생할 수 있다.
	
	- 스프링은 해당 생략시 다음과 같은 규칙을 적용한다.
		- String , int , Integer 같은 단순 타입 = @RequestParam
		- 나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)
	
	- 참고) ```argument resolver```는 뒤에서 학습
	

<br>

#### HTTP 요청 메시지 - 단순 텍스트

- 요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 @RequestParam , @ModelAttribute 를 사용할 수 없다.
	- (물론 HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정된다.)

<br>

- 스프링 MVC는 다음 파라미터를 지원한다.
	- InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
	- OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력

<br>
	
- 스프링 MVC는 다음 파라미터를 지원한다.
	- ```HttpEntity``` : HTTP header, body 정보를 편리하게 조회
		- 메시지 바디 정보를 직접 조회
		- 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X, @ModelAttribute X
	
	- ```HttpEntity```는 응답에도 사용 가능
		- 메시지 바디 정보 직접 반환
		- 헤더 정보 포함 가능
		- view 조회X

	- HttpEntity 를 상속받은 다음 객체들도 같은 기능을 제공한다.
		- ```RequestEntity```
			- HttpMethod, url 정보가 추가, 요청에서 사용
		
		- ```ResponseEntity```
			- HTTP 상태 코드 설정 가능, 응답에서 사용
				```java
				return new ResponseEntity<String>("Hello World", responseHeaders,HttpStatus.CREATED);
				```
				
	-  참고
		- 스프링MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달해주는데, 
			이때 ```HTTP 메시지 컨버터( HttpMessageConverter )```라는 기능을 사용한다.
			이것은 조금 뒤에 HTTP 메시지 컨버터에서 자세히 설명한다.


	- ```@RequestBody```
		- @RequestBody 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 
		- 참고로 헤더 정보가 필요하다면 HttpEntity 를 사용하거나 @RequestHeader 를 사용하면 된다.
		- 이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 ```@RequestParam``` , ```@ModelAttribute``` 와는 전혀 관계가 없다.

<br>

- ```요청 파라미터 vs HTTP 메시지 바디```
	- 요청 파라미터를 조회하는 기능 : ```@RequestParam``` , ```@ModelAttribute```
	- HTTP 메시지 바디를 직접 조회하는 기능 : ```@RequestBody```
	
	
- ```@ResponseBody```
	- ```@ResponseBody``` 를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다. 
	- 물론 이 경우에도 view를 사용하지 않는다

<br>


#### HTTP 요청 메시지 - JSON

- ```@RequestBody``` 객체 파라미터
	```java
	@RequestBody HelloData data
	```
	
	- @RequestBody 에 직접 만든 객체를 지정할 수 있다


- ```HttpEntity``` , ```@RequestBody``` 를 사용하면 ```HTTP 메시지 컨버터```가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
	- HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해주는데, 우리가 방금 V2에서 했던 작업을 대신 처리해준다.
	- 자세한 내용은 뒤에 HTTP 메시지 컨버터에서 다룬다
	
	
- @RequestBody는 생략 불가능
	- 스프링은 @ModelAttribute , @RequestParam 과 같은 해당 애노테이션을 생략시 다음과 같은 규칙을 적용한다.
	
	- String , int , Integer 같은 단순 타입 = @RequestParam
	
	- 나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)

	- 따라서 이 경우 HelloData에 @RequestBody 를 생략하면 @ModelAttribute 가 적용되어버린다.
		- HelloData data @ModelAttribute HelloData data
	
	- ```생략하면 HTTP 메시지 바디가 아니라 요청 파라미터를 처리하게 된다.```
	
- 주의
	- HTTP 요청시에 ```content-type```이 ```application/json```인지 꼭! 확인해야 한다. 그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다.
	
	
- ```@ResponseBody```
	- 응답의 경우에도 @ResponseBody 를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다.
	- 물론 이 경우에도 HttpEntity 를 사용해도 된다.

- ```@RequestBody``` 요청
	- JSON 요청 -> HTTP 메시지 컨버터(JSON 컨버터) -> 객체

- ```@ResponseBody``` 응답
	- 객체 -> HTTP 메시지 컨버터(JSON 컨버터) -> JSON 응답
	
- 여기서 JSON 컨버터란 - ```MappingJackson2HttpMessageConverter```


<br>

#### HTTP 응답 - 정적 리소스, 뷰 템플릿


- 스프링(서버)에서 응답 데이터를 만드는 방법은 크게 3가지이다.
	- 정적 리소스
		- 예) 웹 브라우저에 정적인 HTML, css, js를 제공할 때는, 정적 리소스를 사용한다.
	
	- 뷰 템플릿 사용
		- 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.
	
	- HTTP 메시지 사용
		- HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.

<br>		

#### 정적 리소스

- 스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다.
	- ```/static``` , ```/public``` , ```/resources``` , ```/META-INF/resources```
		
<br>

#### 뷰 템플릿

- 뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.

- 일반적으로 HTML을 동적으로 생성하는 용도로 사용하지만, 다른 것들도 가능하다. 뷰 템플릿이 만들 수 있는 것이라면 뭐든지 가능

- 뷰 템플릿 경로
	- ```src/main/resources/templates```
		

- String을 반환하는 경우 - View or HTTP 메시지
	- ```@ResponseBody 가 없으면```  : response/hello 로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다.
	- ```@ResponseBody 가 있으면``` : 뷰 리졸버를 실행하지 않고, HTTP 메시지 바디에 직접 response/hello 라는 문자가 입력된다.
	

- HTTP 메시지
	- ```@ResponseBody``` , ```HttpEntity``` 를 사용하면, 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 응답 데이터를 출력할 수 있다.


- Thymeleaf 스프링 부트 설정
	- build.gradle : implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	
	- 스프링 부트가 자동으로 ThymeleafViewResolver 와 필요한 스프링 빈들을 등록한다. 그리고 다음 설정도 사용한다. 이 설정은 기본 값 이기 때문에 변경이 필요할 때만 설정하면 된다.
		- application.properties
			```properties
			spring.thymeleaf.prefix=classpath:/templates/
			spring.thymeleaf.suffix=.html
			```
	
	- 참고) 스프링 부트의 타임리프 관련 추가 설정은 다음 공식 사이트를 참고하자. (페이지 안에서 thymeleaf 검색)
		- https://docs.spring.io/spring-boot/docs/2.4.3/reference/html/appendix-applicationproperties.html#common-application-properties-templating

<br>


#### HTTP 응답 - HTTP API, 메시지 바디에 직접 입력

- @RestController
	- @Controller 대신에 @RestController 애노테이션을 사용하면, 해당 컨트롤러에 모두 @ResponseBody 가 적용되는 효과가 있다. 
	- 따라서 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다. 
	- 이름 그대로 Rest API(HTTP API)를 만들 때 사용하는 컨트롤러이다.
	
	- 참고) @ResponseBody 는 클래스 레벨에 두면 전체 메서드에 적용되는데, @RestController 에노테이션 안에 @ResponseBody 가 적용되어 있다.
	

<br>


#### HTTP 메시지 컨버터

- 스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.
	- HTTP 요청: ```@RequestBody``` , ```HttpEntity(RequestEntity)```
	- HTTP 응답: ```@ResponseBody``` , ```HttpEntity(ResponseEntity)```
	
	
- HTTP 메시지 컨버터 인터페이스 : ```org.springframework.http.converter.HttpMessageConverter```
	- HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다
		- ```canRead()``` , ```canWrite()``` : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
		- ```read()``` , ```write()``` : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능
		

- 스프링 부트 기본 메시지 컨버터 (일부 생략)
	- 0 = ByteArrayHttpMessageConverter
	- 1 = StringHttpMessageConverter
	- 2 = MappingJackson2HttpMessageConverter
	
	- 스프링 부트는 다양한 메시지 컨버터를 제공하는데, 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다. 
		- 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.
		
	- ```ByteArrayHttpMessageConverter``` : ```byte[]``` 데이터를 처리한다.
		- 클래스 타입: ```byte[]``` , 미디어타입: ```*/*```
		
		- 요청 예) ```@RequestBody byte[] data```
		
		- 응답 예) ```@ResponseBody return byte[]``` 쓰기 미디어타입 : ```application/octet-stream```
	
	- ```StringHttpMessageConverter``` : ```String``` 문자로 데이터를 처리한다.
		- 클래스 타입: ```String``` , 미디어타입: ```*/*```
		
		- 요청 예) ```@RequestBody String data```
		
		- 응답 예) ```@ResponseBody return "ok"``` 쓰기 미디어타입 : ```text/plain```
		
	- ```MappingJackson2HttpMessageConverter``` : ```application/json```
		- 클래스 타입: 객체 또는 ```HashMap``` , 미디어타입 ```application/json``` 관련
		
		- 요청 예) ```@RequestBody HelloData data```
		
		- 응답 예) ```@ResponseBody return helloData``` 쓰기 미디어타입 : ```application/json``` 관련
	

- HTTP 요청 데이터 읽기
	- HTTP 요청이 오고, 컨트롤러에서 ```@RequestBody``` , ```HttpEntity``` 파라미터를 사용한다.
	
	- 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 ```canRead()``` 를 호출한다. (여러 컨버터를 우선 순위 순으로 다 돌려봄)
	
	- 대상 클래스 타입을 지원하는가.
		- 예) ```@RequestBody``` 의 대상 클래스 ( ```byte[]``` , ```String``` , ```HelloData``` )
	
	- HTTP 요청의 ```Content-Type``` 미디어 타입을 지원하는가.
		- 예) ```text/plain``` , ```application/json``` , ```*/*```
	
	- ```canRead()``` 조건을 만족하면 ```read()``` 를 호출해서 객체 생성하고, 반환한다.
	
	
- HTTP 응답 데이터 생성
	- 컨트롤러에서 ```@ResponseBody``` , ```HttpEntity``` 로 값이 반환된다.
	
	- 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 ```canWrite()``` 를 호출한다.
	
	- 대상 클래스 타입을 지원하는가.
		- 예) return의 대상 클래스 ( ```byte[]``` , ```String``` , ```HelloData``` )
	
	- HTTP 요청의 ```Accept``` 미디어 타입을 지원하는가.(더 정확히는 @RequestMapping 의 produces )
		- 예) ```text/plain``` , ```application/json``` , ```*/*```
	
	- ```canWrite()``` 조건을 만족하면 ```write()``` 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.
	
<br>

#### 요청 매핑 헨들러 어뎁터 구조

-  HTTP 메시지 컨버터는 스프링 MVC 어디쯤에서 사용되는 것일까?
	- 모든 비밀은 애노테이션 기반의 컨트롤러, 그러니까 @RequestMapping 을 처리하는 핸들러 어댑터인
	```RequestMappingHandlerAdapter``` (요청 매핑 헨들러 어뎁터)에 있다.

<br>

- ```ArgumentResolver```
	- 애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다.
		- ```HttpServletRequest``` , ```Model```, ```@RequestParam``` , ```@ModelAttribute``` 같은 애노테이션
		그리고 ```@RequestBody``` , ```HttpEntity``` 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을 가지고 있다.

	- 이렇게 파라미터를 유연하게 처리할 수 있는 이유가 바로 ```ArgumentResolver``` 덕분이다.

	- 애노테이션 기반 컨트롤러를 처리하는 ```RequestMappingHandlerAdapter``` 는 
	바로 이 ```ArgumentResolver``` 를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다.

	- 그리고 이렇게 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.
	
	- 스프링은 30개가 넘는 ArgumentResolver 를 기본으로 제공한다.
	
	- 참고) 가능한 파라미터 목록은 다음 공식 메뉴얼에서 확인 가능
		- https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annarguments
	
	
	- 정확히는 ```HandlerMethodArgumentResolver``` 인데 줄여서 ```ArgumentResolver``` 라고 부른다.
	
	- 동작 방식
		- 1. ArgumentResolver 의 ```supportsParameter()``` 를 호출해서 해당 파라미터를 지원하는지 체크
		- 2. 지원하면 ```resolveArgument()``` 를 호출해서 실제 객체를 생성
		- 3. 그리고 이렇게 생성된 객체가 컨트롤러 호출시 넘어간다.

<br>


- ```ReturnValueHandler```
	- ```HandlerMethodReturnValueHandler``` 를 줄여서 ```ReturnValueHandler``` 라 부른다.
	- ArgumentResolver 와 비슷한데, 이것은 응답 값을 변환하고 처리

	- 컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유가 바로 ```ReturnValueHandler``` 덕분이다.
	
	- 참고) 가능한 응답 값 목록은 다음 공식 메뉴얼에서 확인할 수 있다.
		- https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annreturn-types
		

<br>


- HTTP 메시지 컨버터
	- HTTP 메시지 컨버터를 사용하는 @RequestBody 도 컨트롤러가 필요로 하는 파라미터의 값에 사용된다.
	- @ResponseBody 의 경우도 컨트롤러의 반환 값을 이용한다.

	- ```요청의 경우``` 
		- ```@RequestBody``` 와 ```HttpEntity``` 를 처리하는 ```ArgumentResolver``` 가 있다. 
		- 이 ```ArgumentResolver``` 들이 ```HTTP 메시지 컨버터```를 사용해서 필요한 객체를 생성하는 것이다.
		
	- ```응답의 경우```
		- ```@ResponseBody``` 와 ```HttpEntity``` 를 처리하는 ```ReturnValueHandler``` 가 있다. 
		- 그리고 여기에서 ```HTTP 메시지 컨버터```를 호출해서 응답 결과를 만든다.
		
		
- 참고) 스프링 MVC 의 ArgumentResolver 사용
	- ```@RequestBody``` ```@ResponseBody``` 가 있으면 ```RequestResponseBodyMethodProcessor``` (ArgumentResolver) 를 사용하고,
	- ```HttpEntity``` 가 있으면 ```HttpEntityMethodProcessor``` (ArgumentResolver)를 사용한다.

<br>

- 확장 : 스프링은 다음을 모두 인터페이스로 제공한다. 따라서 필요하면 언제든지 기능을 확장할 수 있다.
	- ```HandlerMethodArgumentResolver```
	- ```HandlerMethodReturnValueHandler```
	- ```HttpMessageConverter```
	
	- 기능 확장은 ```WebMvcConfigurer``` 를 상속 받아서 스프링 빈으로 등록하면 된다
