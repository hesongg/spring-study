### 스프링 웹 MVC

#### Reference) 
	* 스프링 MVC 1편 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
https://github.com/hesongg/springboot_servlet_practice
	
<br>

#### 서블릿, JSP, MVC 패턴

- 서블릿
	- 자바 코드에서 HTML 화면을 구현해야 한다.

- JSP
	- jsp파일에서 서블릿보다는 편하게 html 화면을 그릴 수 있다.
	- 서블릿과 다른점은 HTML을 중심으로하고 자바 코드를 부분부분 작성 가능하다.
	- jsp 사용 시 ```<%@ page contentType="text/html;charset=UTF-8" language="java" %>``` 파일에 이 부분 필수 추가
	- import 시 ```<%@ page import="java.util.List" %>``` 이와 같이 사용
	- 자바 코드 사용시 ```<% ~ %>``` 태그 안에 사용해야한다.
		```java
		<%
			MemberRepository memberRepository = MemberRepository.getInstance();

			List<Member> members = memberRepository.findAll();
		%>
		```
	- 자바 코드 출력시 ```<%= ~~ %>``` 태그 사용

- 서블릿과 JSP 한계
	- 서블릿으로 개발 시 뷰(View) 화면을 위한 HTML 만드는 작업이 자바 코드에 섞여 매우 지저분하고 복잡하다.
	- JSP 사용 시 HTML 작성은 편하고, 동적으로 변경이 필요한 부분만 자바 코드 작성이 가능하다
	
	- jsp 코드를 보면 java 코드, 데이터를 조회하는 리포지토리 등 다양한 코드가 모두 jsp에 노출되어 있다.
	jsp가 너무 많은 역할을 하기 때문에 코드가 길어지면 유지보수하기 매우 힘들다.
	
<br>

- MVC 패턴의 등장
	- 비즈니스 로직은 서블릿 처럼 다른 곳에서 처리하고, JSP는 목적에 맞게 HTML로 화면을 그리는 일에 집중하도록 한다.

	- 하나의 서블릿이나 JSP만으로 비즈니스 로직과 뷰 렌더링까지 모두 처리하면 너무 많은 역할을 하게되고, 유지보수가 어려움
	
	- 변경의 라이프 사이클 : UI 와 비즈니스 로직간의 변경의 라이프 사이클이 다르다.
		- UI의 일부와 비즈니스 로직을 수정하는 일은 각각 다르게 발생할 확률이 높다.
		- 이렇게 변경의 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수하기 좋지 않음.
		
	- 특히 JSP 같은 뷰 템플릿은 화면을 렌더링하는데 최적화 되어있기 때문에 이 부분의 업무만 담당하는 것이 가장 효과적이다.
	

- MVC : Model View Controller
	- Model : 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 
		뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링 하는 일에 집중할 수 있다
	- View : 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다. 여기서는 HTML을 생성하는 부분을 말한다
	- Controller : HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다
	

- ```dispatcher.forward(request, response);``` : 서버 내부에서 호출이 발생하는, 다른 서버나 서블릿으로 이동할 수 있는 기능


- ```/WEB-INF``` : 이 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다. 우리가 기대하는 것은 항상 컨트롤러를 통해서 JSP를 호출하는 것이다.


- rediect 와 foward 차이
	- 리다이렉트는 실제 클라이언트(웹 브라우저)에 응답이 나갔다가, 클라이언트가 redirect 경로로 다시 요청한다.
		따라서 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경
	- 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 인지할 수 없음
	
	
- HttpServletRequest를 Model로 사용 가능. 
	- request가 제공하는 setAttribute() 를 사용하면 request 객체에 데이터를 보관해서 뷰에 전달할 수 있다.
	- 뷰는 request.getAttribute() 를 사용해서 데이터를 꺼내면 된다.
	- JSP에서 ```<%= request.getAttribute("member")%>``` 로 모델에 저장한 member 객체를 꺼낼 수 있지만, 
	너무 복잡해진다.
		- JSP는 ```${}``` 문법을 제공하는데, 이 문법과 jstl을 사용하면 request의 attribute에 담긴 데이터를 편리하게 조회할 수 있다.

<br>

- MVC 컨트롤러의 단점
	- 포워드 중복 : View로 이동하는 코드가 항상 중복 호출되어야 한다. 이 부분을 메서드로 공통화한다해도, 해당 메서드도 항상 직접 호출되어야 한다.
		```java
		String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
		```
		
	- ViewPath에 중복
		```String viewPath = "/WEB-INF/views/members.jsp";```
		
		- prefix : '/WEB-INF/views/'
		- suffix : '.jsp'
		
		- 만약 jsp가 아닌 thymeleaf 같은 다른 뷰로 변경한다면 전체 코드를 다 변경해야한다.
	
	- 사용하지 않는 코드
		- ```HttpServletRequest request, HttpServletResponse response```
		- 이런 코드는 사용할때도 있고, 안할 때도 있다. 위와 같은 코드는 테스트케이스 작성도 어려움
	
	- 공통 처리가 어렵다.
		- 기능이 복잡해질수록 컨트롤러에서 공통으로 처리해야하는 부분이 점점 더 많이 증가할 것이다.
		- 단순히 공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 항상 해당 메서드를 호출해야한다.
		- 호출하는 것 자체도 중복이 됨
		
	- 정리하면 공통 처리가 어렵다는 문제가 있다.
		- 이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다.
		- ```프론트 컨트롤러(Front Controller) 패턴``` 을 도입하면 이런 문제를 해결 가능
			- 스프링 MVC의 핵심
	

