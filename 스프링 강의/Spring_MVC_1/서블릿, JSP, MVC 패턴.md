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
	
- MVC 패턴의 등장
	- 비즈니스 로직은 서블릿 처럼 다른 곳에서 처리하고, JSP는 목적에 맞게 HTML로 화면을 그리는 일에 집중하도록 한다.
	

