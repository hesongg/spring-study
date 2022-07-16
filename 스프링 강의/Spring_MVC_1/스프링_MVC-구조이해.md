### 스프링 웹 MVC

#### Reference) 
	* 스프링 MVC 1편 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
-
	
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

- 핸들러 조회 : 핸들러 매핑을 통해 요청 URLㅇ 매핑된 핸들러(컨트롤러)를 조회
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
