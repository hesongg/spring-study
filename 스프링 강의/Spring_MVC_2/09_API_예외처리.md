### 스프링 웹 MVC - 백엔드 웹 개발 활용 기술

#### Reference) 
	* 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/SpringBoot-Practice-Exception
	
<br>


### API 예외 처리


<br>

#### API 예외 처리 - 시작

- 목표
	- API 예외 처리는 어떻게 해야할까?
	- HTML 페이지의 경우 지금까지 설명했던 것 처럼 4xx, 5xx와 같은 오류 페이지만 있으면 대부분의 문제를 해결할 수 있다.
	- 그런데 API의 경우에는 생각할 내용이 더 많다. 오류 페이지는 단순히 고객에게 오류 화면을 보여주고 끝이지만, 
		API는 각 오류 상황에 맞는 오류 응답 스펙을 정하고, JSON으로 데이터를 내려주어야 한다.
	- 지금부터 API의 경우 어떻게 예외 처리를 하면 좋은지 알아보자.


- API 예외를 처리하기위해 오류 페이지 컨트롤러도 JSON 응답을 할 수 있도록 수정해야 한다
	- 컨트롤러 수정 - 메서드 추가
		- ```@RequestMapping``` 의 ```produces``` 옵션 설정
			- ```produces = MediaType.APPLICATION_JSON_VALUE``` 의 뜻은 
				- 클라이언트가 요청하는 HTTP Header의 ```Accept``` 의 값이 ```application/json``` 일 때 해당 메서드가 호출된다는 것이다. 
				- 결국 클라이언트가 받고싶은 미디어타입이 ```json```이면 이 컨트롤러의 메서드가 호출된다
		
		- 응답 데이터를 위해서 ```Map``` 을 만들고 ```status``` , ```message``` 키에 값을 할당했다. 
			Jackson 라이브러리는 ```Map``` 을 JSON 구조로 변환할 수 있다.
		
		- ```ResponseBody``` / ```ResponseEntity``` 를 사용해서 응답하기 때문에 메시지 컨버터가 동작하면서 클라이언트에 JSON이 반환된다.

<br>

- 포스트맨으로 테스트 시 
	- HTTP Header에 Accept 가 application/json 인 것을 꼭 확인하자.
	- HTTP Header에 Accept 가 application/json 이 아니면, 기존 오류 응답인 HTML 응답이 출력되는 것을 확인할 수 있다.
	
<br>

#### API 예외 처리 - 스프링 부트 기본 오류 처리

- API 예외 처리도 스프링 부트가 제공하는 기본 오류 방식을 사용할 수 있다.
	- 스프링 부트가 제공하는 ```BasicErrorController``` 코드를 보자
		```java
		@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
		public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {}
		
		@RequestMapping
		public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
		```
		
		- ```/error``` 라는 동일한 경로를 처리하는 ```errorHtml()``` , ```error()``` 두 메서드를 확인할 수 있다
		
		- ```errorHtml()``` : ```produces = MediaType.TEXT_HTML_VALUE``` 
			- 클라이언트 요청의 Accept 해더 값이 ```text/html``` 인 경우에는 ```errorHtml()``` 을 호출해서 view를 제공한다.
		
		- ```error()``` : 그외 경우에 호출되고 ```ResponseEntity``` 로 ```HTTP Body```에 ```JSON``` 데이터를 반환한다.

- 스프링 부트의 예외 처리
	- 앞서 학습했듯이 스프링 부트의 기본 설정은 오류 발생시 ```/error``` 를 오류 페이지로 요청한다.
	- ```BasicErrorController``` 는 이 경로를 기본으로 받는다. ( ```server.error.path``` 로 수정 가능, 기본 경로 : ```/error``` )
	
- 스프링 부트는 ```BasicErrorController``` 가 제공하는 기본 정보들을 활용해서 오류 API를 생성해준다.

- 다음 옵션들을 설정하면 더 자세한 오류 정보를 추가할 수 있다.
	```server.error.include-binding-errors=always```
	```server.error.include-exception=true```
	```server.error.include-message=always```
	```server.error.include-stacktrace=always```
	
	- 물론 오류 메시지는 이렇게 막 추가하면 보안상 위험할 수 있다. 간결한 메시지만 노출하고, 로그를 통해서 확인 해야 함

<br>

- Html 페이지 vs API 오류

	- ```BasicErrorController``` 를 확장하면 JSON 메시지도 변경할 수 있다. 
		- 그런데 API 오류는 조금 뒤에 설명할 ```@ExceptionHandler``` 가 제공하는 기능을 사용하는 것이 더 나은 방법이므로 
			지금은 ```BasicErrorController``` 를 확장해서 JSON 오류 메시지를 변경할 수 있다 정도로만 이해해두자.
			
			
	- 스프링 부트가 제공하는 ```BasicErrorController``` 는 HTML 페이지를 제공하는 경우에는 매우 편리하다.
		- 4xx, 5xx 등등 모두 잘 처리해준다
		
		- API에서 발생하는 예외에 따라 그 결과가 달라질 수 있다.
			- API 수행해야 할 예외 처리는 매우 세밀하고 복잡하다. 
			- 따라서 이 방법은 HTML 화면을 처리할 때 사용하고, API 오류 처리는 뒤에서 설명할 ```@ExceptionHandler``` 를 사용하자.

<br>

#### API 예외 처리 - HandlerExceptionResolver 시작

- 목표
	- 예외가 발생해서 서블릿을 넘어 WAS까지 예외가 전달되면 HTTP 상태코드가 500으로 처리된다.
		- 발생하는 예외에 따라서 400, 404 등등 다른 상태코드로 처리하려고 함
		- 오류 메시지, 형식등을 API마다 다르게 처리하려고 함
		

- ```HandlerExceptionResolver```
	- 스프링 MVC는 컨트롤러(핸들러) 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있는 방법을 제공한다. 
	- 컨트롤러 밖으로 던져진 예외를 해결하고, 동작 방식을 변경하고 싶으면 ```HandlerExceptionResolver``` 를 사용하면 된다. 
		줄여서 ```ExceptionResolver``` 라 한다.
		
<br>

##### ExceptionResolver 적용 전

![image](https://user-images.githubusercontent.com/77953474/184311925-8b8de43f-7a18-4152-9f89-ba72417b66ee.png)


<br>

##### ExceptionResolver 적용 후

![image](https://user-images.githubusercontent.com/77953474/184311957-6342081f-9d64-441c-b8e8-7423c9f9d2fb.png)


- 참고: ```ExceptionResolver``` 로 예외를 해결해도 ```postHandle()``` 은 호출되지 않는다

<br>

- ```HandlerExceptionResolver``` - 인터페이스
	```java
	public interface HandlerExceptionResolver {
		ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response,
			Object handler, Exception ex);
	}
	```
	
	- ```handler``` : 핸들러(컨트롤러) 정보
	- ```Exception ex``` : 핸들러(컨트롤러)에서 발생한 발생한 예외


- ```ExceptionResolver``` 가 ```ModelAndView``` 를 반환하는 이유는 마치 try, catch를 하듯이, ```Exception``` 을 처리해서 정상 흐름 처럼 변경하는 것이 목적이다. 
	- 이름 그대로 ```Exception``` 을 ```Resolver```(해결)하는 것이 목적이다.
	
<br>

##### 반환 값에 따른 동작 방식

- ```HandlerExceptionResolver``` 의 반환 값에 따른 ```DispatcherServlet``` 의 동작 방식은 다음과 같다.
	- 빈 ```ModelAndView``` : ```new ModelAndView()``` 처럼 빈 ```ModelAndView``` 를 반환하면 
		뷰를 렌더링 하지않고, 정상 흐름으로 서블릿이 리턴된다.
		
	- ```ModelAndView``` 지정 : ```ModelAndView``` 에 ```View``` , ```Model``` 등의 정보를 지정해서 반환하면 뷰를 렌더링 한다.
	
	- ```null``` : ```null``` 을 반환하면, 다음 ```ExceptionResolver``` 를 찾아서 실행한다. 만약 처리할 수 있는 ```ExceptionResolver``` 가 없으면 예외 처리가 안되고, 
		기존에 발생한 예외를 서블릿 밖으로 던진다.
		
<br>

##### ExceptionResolver 활용

- 예외 상태 코드 변환
	- 예외를 ```response.sendError(xxx)``` 호출로 변경해서 서블릿에서 상태 코드에 따른 오류를 처리하도록 위임
	- 이후 WAS는 서블릿 오류 페이지를 찾아서 내부 호출, 예를 들어서 스프링 부트가 기본으로 설정한 ```/error``` 가 호출됨

- 뷰 템플릿 처리
	- ```ModelAndView``` 에 값을 채워서 예외에 따른 새로운 오류 화면 뷰 렌더링 해서 고객에게 제공

- API 응답 처리
	- ```response.getWriter().println("hello");``` 처럼 HTTP 응답 바디에 직접 데이터를 넣어주는 것도 가능하다. 
		- 여기에 JSON 으로 응답하면 API 응답 처리를 할 수 있다.
		
<br>

- ```WebMvcConfigurer``` 를 통해 등록
	```java
	/**
	 * 기본 설정을 유지하면서 추가
	 */
	@Override
	public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver>	resolvers) {
		resolvers.add(new MyHandlerExceptionResolver());
	}
	```
		
	- ```configureHandlerExceptionResolvers(..)``` 메서드를 사용하면 
		스프링이 기본으로 등록하는 ```ExceptionResolver``` 가 제거되므로 주의 
	
		- ```extendHandlerExceptionResolvers``` 를 사용하자.
		
<br>

#### API 예외 처리 - HandlerExceptionResolver 활용

- 예외를 ```ExceptionHandler``` 에서 마무리하기
	- 예외가 발생하면 WAS까지 예외가 던져지고, WAS에서 오류 페이지 정보를 찾아서 다시 ```/error``` 를 호출하는 과정은 생각해보면 너무 복잡하다. 
	- ```ExceptionResolver``` 를 활용하면 예외가 발생했을 때 이런 복잡한 과정 없이 여기에서 문제를 깔끔하게 해결할 수 있다.
	
- 헤더의 accpet 값에 따라 예외를 처리하는 내용 구현 - 코드 참고

<br>

- 정리
	- ```ExceptionResolver``` 를 사용하면 컨트롤러에서 예외가 발생해도 ```ExceptionResolver``` 에서 예외를 처리해버린다.
		- 따라서 예외가 발생해도 서블릿 컨테이너까지 예외가 전달되지 않고, 스프링 MVC에서 예외 처리는 끝이 	난다.
	
	- 결과적으로 WAS 입장에서는 정상 처리가 된 것이다. 이렇게 예외를 이곳에서 모두 처리할 수 있다는 것이 핵심이다.
	
	- 서블릿 컨테이너까지 예외가 올라가면 복잡하고 지저분하게 추가 프로세스가 실행된다. 
		- 반면에 ```ExceptionResolver``` 를 사용하면 예외처리가 상당히 깔끔해진다.
	
	- 그런데 직접 ```ExceptionResolver``` 를 구현하려고 하니 상당히 복잡하다. 지금부터 스프링이 제공하는 ```ExceptionResolver``` 들을 알아보자.

<br>

