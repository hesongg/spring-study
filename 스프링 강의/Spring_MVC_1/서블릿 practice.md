### 스프링 웹 MVC

#### Reference) 
	* 스프링 MVC 1편 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
https://github.com/hesongg/springboot_servlet_practice
	

<br>


#### 서블릿 코드 작성

- 프로젝트 생성
	- 패키징은 war, 편리함을 위해 start.spring.io 에서 스프링 부트 프로젝트로 생성함

- post man? 설치함


- @ServletComponentScan
	- 스프링부트 애플리케이션에 해당 애노테이션을 달아준다.
	- 스프링이 서블릿을 찾아서 자동으로 등록해준다.
	
- @WebServlet(name = "helloServlet", urlPatterns = "/hello")
	- 해당 클래스를 서블릿으로 설정
	
	- url 패턴이 매칭되면 HttpServlet 인터페이스에서 상속받아 구현해놓은 service 메서드가 실행된다.
	

- HttpServletRequest 객체가 지원하는 기능
	- 파라미터, message body 데이터 직접 조회 가능

	- 임시 저장소 기능
		- ```request.setAttribute(name, value)``` / ```request.getAttribute(name)```
	
	- 세션 관리 기능
		- ```request.getSession(create: true)```
	
