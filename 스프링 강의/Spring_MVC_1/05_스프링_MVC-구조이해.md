### 스프링 웹 MVC

#### Reference) 
	* 스프링 MVC 1편 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/springboot_servlet_practice
	
<br>

#### 스프링 MVC 구조

- DispatcherServlet 구조
	- 스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어 있다.
	- 스프링 MVC 의 프론트 컨트롤러 : 디스패처 서블릿 (스프링 MVC의 핵심)

	- DispatcherServlet 서블릿 등록
		- ```DispatcherServlet```도 부모 클래스에서 ```HttpServlet```을 상속받아서 사용하고, 서블릿으로 동작한다.
		- 스프링부트는 DispatcherServlet을 서블릿으로 자동 등록하면서 모든 경로(urlPatterns="/")에 대해서 매핑한다.
			- 참고) 더 자세한 경로가 우선순위가 높다. 그래서 기존에 등록한 서블릿도 함께 동작한다.
		
	- 요청 흐름
		- 서블릿이 호출되면 HttpServlet이 제공하는 ```service()```가 호출된다.
		- 스프링 MVC는 DispatcherServlet 의 부모인 FrameworkServlet 에서 service() 를 오버라이드 해두었다.
		- FrameworkServlet.service() 를 시작으로 여러 메서드가 호출되면서 ```DispacherServlet.doDispatch()``` 가 호출된다

<br>	

#### 동작순서

![image](https://user-images.githubusercontent.com/77953474/180109452-f8c6c145-f510-4ff6-a872-34c7b81c8663.png)

- 핸들러 조회 : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회
- 핸들러 어댑터 조회 : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회
- 핸들러 어댑터 실행 : 핸들러 어댑터 실행
- 핸들러 실행 : 핸들러 어댑터가 실제 핸들러를 실행
- ModelAndVieww 반환 : 핸들러 어댑터는 핸들러가 변환하는 정보를 ModelAndView로 변환해서 반환한다.
- viewResolver 호출 : 뷰 리졸버를 찾고 실행
	- JSP의 경우 ```InternalResourceViewResolver```가 자동 등록되고, 사용됨
- View 반환 : 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
	- JSP의 경우 ```InternalResourceView(JstlView)```를 반환하는데, 내부에 ```foward()```로직이 있다.
- 뷰 렌더링

<br>

#### 인터페이스 살펴보기

- 스프링 MVC의 큰 강점은 DispatcherServlet 코드의 변경 없이, 원하는 기능을 변경하거나 확장할 수있다는 점이 있다.

- 여러 기능을 확장 가능하도록 인터페이스로 제공한다.

- 이 인터페이스들만 구현해서 DispatcherServlet 에 등록하면 커스텀 컨트롤러를 구현 할 수 있다.

- 주요 인터페이스 목록
	- 핸들러 매핑 : ```org.springframework.web.servlet.HandlerMapping```
	- 핸들러 어댑터 : ```org.springframework.web.servlet.HandlerAdapter```
	- 뷰 리졸버 : ```org.springframework.web.servlet.ViewResolver```
	- 뷰 : ```org.springframework.web.servlet.View```


- 스프링 구조를 알아야되는 이유..
	- 스프링 MVC 자체 기능을 확장하여 이용하거나 하는 일은 없다. 이미 대부분의 기능이 다 구현되어있기때문에 잘 찾아서 사용해야 함..
	그럼에도 전체적인 구조를 잘 알아야하는 이유는 문제가 생겼을 때 전체적인 구조에서 어떤 부분에서 문제가 발생하였는지 문제를 쉽게 파악하고,
	문제를 해결할 수 있기 때문이다. 그리고 확장 포인트가 필요할 때, 어떤 부분을 확장해야할지 감을 잡을 수 있다.
	
<br>

#### 핸들러 매핑과 핸들러 어댑터

- Controller 인터페이스
	- 컨트롤러 인터페이스는 ```@Controller``` 애노테이션과 전혀 다르다.
	- ModelAndView를 반환하고있는 ```handleRequest``` 메서드를 가지고있다.

- 위 인터페이스로 구현한 컨트롤러가 호출되려면 2가지가 필요하다.
	- ```HandlerMapping (핸들러 매핑)```
		- 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
		- 예) ```스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑```이 필요
	- ```HandlerAdapter (핸들러 어댑터)```
		- 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
		- 예) ```Controller``` 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야 한다.
	
- 스프링에는 이미 필요한 핸들러 매핑과 핸들러 어댑터가 대부분 구현되어있다. 
	- 개발자가 직접 핸들러 매핑과 핸들러 어댑터를 만드는 일은 거의 없다.

- 스프링 부트가 자동 등록하는 핸들러 매핑과 핸들러 어댑터
	- 실제로는 더 많음, 일부 생략
	
	- HandlerMapping
		- 0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
		- 1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
			- 요청 URL로 빈 이름을 찾는다.
	
	- HandlerAdapter
		- 0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
		- 1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
		- 2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용) 처리
		
- ```@RequestMapping```
	- 가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 ```RequestMappingHandlerMapping``` , 
		```RequestMappingHandlerAdapter``` 이다.
	
	- 이 방식을 제일 많이 사용하고, 제일 중요함
	
<br>


#### 뷰 리졸버

- 뷰 리졸버 - InternalResourceViewResolver
	- 스프링 부트는 InternalResourceViewResolver 라는 뷰 리졸버를 자동으로 등록하는데, 
	- 이때 ```application.properties``` 에 등록한 다음의 설정 정보를 사용해서 등록한다.
		- ```spring.mvc.view.prefix```
		- ```spring.mvc.view.suffix``` 
	
	- 참고로 권장하지는 않지만 설정 없이 다음과 같이 전체 경로를 주어도 동작하기는 한다.
		- return new ModelAndView("/WEB-INF/views/new-form.jsp");
		

- 스프링 부트가 자동 등록하는 뷰 리졸버 (실제로는 더 많음, 일부 생략)
	- 1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
	- 2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.
	
- 참고
	- ```InternalResourceViewResolver``` 는 만약 JSTL 라이브러리가 있으면 ```InternalResourceView``` 를
		상속받은 ```JstlView``` 를 반환한다. JstlView 는 JSTL 태그 사용시 약간의 부가 기능이 추가된다.

	- 다른 뷰는 실제 뷰를 렌더링하지만, JSP의 경우 forward() 통해서 해당 JSP로 이동(실행)해야 렌더링이 된다. 
		JSP를 제외한 나머지 뷰 템플릿들은 forward() 과정 없이 바로 렌더링 된다.

	- Thymeleaf 뷰 템플릿을 사용하면 ```ThymeleafViewResolver``` 를 등록해야 한다. 
		최근에는 라이브러리만 추가하면 스프링 부트가 이런 작업도 모두 자동화해준다.
		
<br>

#### Spring MVC 코드 작성하며 배우기

- ```@Controller@```
	- 스프링이 자동으로 빈으로 등록 (내부에 @Component 애노테이션이있어서 컴포넌트 스캔의 대상이 된다.)
	- 스프링 MVC에서 애노테이션 기반 컨트롤러가 된다.
	
	- ```@RequestMapping``` : 요청 정보를 매핑. 해당 URL이 호출되면 이 메서드가 호출된다. 메서드의 이름은 임의로 지으면된다.
	
	-  ```ModelAndView``` : 모델과 뷰 정보를 담아서 반환하면 된다.
	
- ```RequestMappingHandlerMapping``` 은 스프링 빈 중에서 @RequestMapping 또는 @Controller 가 클래스 레벨에 붙어 있는 경우에 매핑 정보로 인식한다.
	- @Controller = @Component + @RequestMapping


- Model 파라미터
	- save() , members() 를 보면 Model을 파라미터로 받는 것을 확인할 수 있다. 스프링 MVC도 이런 편의기능을 제공한다.

- ViewName 직접 반환
 - 뷰의 논리 이름을 반환할 수 있다.

- @RequestParam 사용
	- 스프링은 HTTP 요청 파라미터를 @RequestParam 으로 받을 수 있다.
	- @RequestParam("username") 은 request.getParameter("username") 와 거의 같은 코드라 생각하면 된다.
	- GET 쿼리 파라미터, POST Form 방식을 모두 지원
	
- @RequestMapping -> @GetMapping, @PostMapping
	- @RequestMapping 은 URL만 매칭하는 것이 아니라, HTTP Method도 함께 구분할 수 있다
	- 이것을 @GetMapping , @PostMapping 으로 더 편리하게 사용할 수 있다.
	- 참고로 Get, Post, Put, Delete, Patch 모두 애노테이션이 준비되어 있다.
