### 스프링 웹 MVC

#### Reference) 
	* 스프링 MVC 1편 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
https://github.com/hesongg/springboot_servlet_practice
	
<br>

#### 프론트 컨트롤러 패턴

- 프론트 컨트롤러 패턴 특징
	- 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음
	- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
	- 입구를 하나로
	- 공통 처리 가능
	- 프론트컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨


- 스프링 웹 MVC와 프론트 컨트롤러
	- 스프링 웹 MVC의 핵심도 바로 FrontController
	- 스프링 웹 MVC의 ```DispatcherServlet```이 FrontController 패턴으로 구현되어 있음
	
<br>
	
#### 프론트 컨트롤러 패턴 구현 코드 작성

- 패턴을 구현해본 버전 별로 정리힌다.

- V1 버전
	- 서비스를 수행하는 추상 메서드를 가진 ControllerV1 인터페이스를 생성
	- 각 기능 컨트롤러에서는 ControllerV1 인터페이스를 상속받아서 비즈니스 로직 수행 후 jsp로 foward하는 로직 작성
	- 프론트 컨틀롤러 서블릿 V1 생성
		- 제일 처음 요청을 받는 컨트롤러 (url 패턴을 *로 맵핑 '/front-controller/v1/*')
		- 생성자에서 전역변수 map에  각 요청 URL에 따라 요청을 수행할 컨트롤러 인스턴스를 넣어줌
		- 프론트 컨트롤러에서 servlet 메서드 ```service``` 에서 URI에 따라 map에서 컨트롤러를 찾아서 서비스를 수행함
		
- V2 버전
	- dispatch.foward (view로 화면이동) 해주는 객체 MyView 추가
	- ControllerV2 인터페이스 추가 : 추상 메서드의 리턴 타입을 MyView 로 함
	- 비즈니스 로직 수행 컨트롤러 추가 : 생성한 인터페이스를 상속받아서 로직을 수행하고 이동할 화면 값으로 MyView 인스턴스를 생성해서 리턴함
	- 프론트 컨트롤러에서는 비즈니스 로직 수행 컨트롤러에서 view를 리턴받아 MyView의 render 메서드를 수행함 (뷰 렌더링)
	- 결과적으로 프론트 컨트롤러에서 공통 로직을 다 처리한다.
	- MyView를 리턴타입으로 갖는 공통 인터페이스 역할의 중요성을 알 수 있다.
	

- V3 버전
	- 서블릿 종속성 제거 진행
		- 컨트롤러 입장에서 HttpServletReqeust, HttpServletResponse 가 필요하지 않다.
		- request 객체를 Model 로 사용하도록 -> 컨트롤러가 Servlet 기술을 사용하지 않도록
		- 구현 코드 단순화 및 테스트 코드 작성 용이
	
	- 뷰 이름 중복 제거 진행
	
	- 코드 작성
		- 비즈니스 로직 컨트롤러
			- 파라미터를 Map으로 받는 것으로 변경하여 서블릿 종속성 제거
			- 리턴 타입을 ModelView 로함
			- 비즈니스 로직 수행 후 수행 결과를 ModelView의 Model map에 담은 후, ModelView를 리턴함
		
		- ModelView 객체 생성
			- String 타입 viewName 과, String 을 key, Object를 value로 가지는 model Map을 가지고있음
		
		- 프론트 컨트롤러
			- 서비스 수행 전, HttpServletRequest의 요청 파라미터 키값을 Map에 맵핑 후 서비스 수행 시 해당 Map을 파라미터로 전달함
			- 서비스 수행 후, 전달받은 ModelView 에서 viewName을 뽑아 viewResolver 로 MyView 객체를 생성함
			- 생성한 MyView에 model 값와 HttpServletRequest / Response를 전달하며 뷰 렌더링 호출
			
		- MyView
			- JSP에서는 request의 attribute에 데이터가 세팅되어야하기 때문에 Model 값을 request.setAttribute 한 후
				RequestDispatcher 의 foward 호출함. 이 때 dispatcher의 path는 프론트 컨트롤러에서 뷰 리졸버로 구현 해준 viewPath이다.
	

- V4 버전
	- 프론트컨트롤러에서 컨트롤러 호출하는 파라미터에 Model 추가함
	- 비즈니스 로직 컨트롤러에서는 파라미터 Model에 결과값을 세팅하고, 뷰 논리 이름만 리턴해주면 된다.


- V5 버전 : 유연한 컨트롤러
	- 어댑터 패턴
		- 어댑터 패턴을 이용해서 프론트 컨트롤러가 다양한 방식의 컨트롤러를 처리할 수 있도록 변경해보자.
	
	- 핸들러 어댑터 : 중간에 어댑터 역할을하는 어댑터가 추가. (핸들러 어댑터)
		- 여기서 어댑터 역할을 해주기 때문에 다양한 종류의 컨트롤러 호출 가능
	
	- 핸들러 : 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경.
		- 컨트롤러의 개념뿐만아니라, 이제 어댑터가 있기때문에 어떠한 것이든 해당하는 종류의 어댑터만 있으면 다 처리할 수 있다.
	
