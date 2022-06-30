### 스프링 부트 강의 학습내용 정리

#### Reference) 
- ```스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술 (인프런 김영한님 강의 학습한 내용 정리)```
	

#### 정리

- spring initializr(start.spring.io)
	- 프로젝트
		- 그래들
	- 자바
	- 스프링 부트 버전은 정식 릴리즈된 버전으로

- dependency 추가
	- spring web
	- 템플릿 엔진은 thymeleaf
	
- 인텔리제이에서 오픈 -> build.gradle 파일 open

- 프로젝트에서 src 밑에 main / test 나눠져있는건 거의 표준
	- test는 테스트 코드만 (중요함)
	
- resources는 자바 파일 제외한 나머지이다.

- 인텔리제이 참고)
	- 인텔리제이 세팅에 그래들 통해서 서버 띄우지말고 인텔리제이로 띄우도록 가능 (성능 향상)

- 스프링부트 쓰면 slf4j , logback 기본으로 import됨
	- slf4j는 인터페이스, 실제 로그 찍는 클래스는 로그백

- 테스트 라이브러리 junit 최근에는 5 버전을 쓰는 추세이다.

- @Controller로 선언한 컨트롤러에서 
	- @GetMapping("hello") 로하면
		- /hello uri 요청에 매핑됨
		- 해당 메소드에서 model.addAttribute("data", "hello!!");
			- 타임리프 ${data} 값으로 "hello!!" 넘겨줌
		- return "hello";
			- hello.html 로 매핑함

- 그래들
	- ./gradlew clean build : 클린 후 빌드하는 그래들 명령어
		- 빌드 후 build/lib 경로에서 java -jar 로 jar파일 실행시키면 톰캣 뜸

- 스프링 웹개발 기초
	- 정적 컨텐츠
	- MVC와 템플릿 엔진
	- API

- 만약 /hello-static.html 이라는 url 요청이 오게되면 컨트롤러에서 해당 요청에대해 매핑이 되는게있는지
	먼저 확인한 다음, 정적 컨테츠로 매핑한다(우선순위는 컨트롤러)
	
- viewResolver 
	- 타임리프 템플릿 엔진 처리, 설정한 모델 값 뷰에 설정해줌
	- 화면 렌더링해서 html을 웹브라우저에 넘겨줌


- 간단한 api 방식
	- @ResponseBody
		- http 응답 body에 값을 지정해줌
		
		- 스프링 컨테이너에서 @ResponseBody 가 있으면 viewResolver 대신에 HttpMessageConverter 가 동작하는데..
			- 응답이 문자(String)일때 그냥 문자를 응답 (StringHttpMessageConveter 동작)
			- 응답이 객체일때는 json으로 응답 (JsonConverter 동작)
				- 기본 객체 처리: MappingJackson2HttpMessageConverter
			- byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음
	
- test 경로에 테스트코드 작성
	- 개발 시 테스트코드를 먼저 작성하면서 개발하는 방식을 TDD라고 한다.
	- @Test를 여러개 동시에 수행할 때, 각 테스트는 서로 개별적으로 수행되어야 함(테스트코드 사이에 영향이 가도록 개발하면 안됨)
	

- @Autowired
	- 생성자에 해당 어노테이션이 붙어있으면 스프링 빈으로 등록되어있는 객체를 가져와서 연결해 세팅해준다.

- 스프링 빈 등록 방법
	- 컴포넌트 스캔과 자동 의존관계 설정
	- 자바 코드로 직접 스프링 빈 등록
	
	- 컴포넌트 스캔 원리
		- @Component 애노테이션이 있으면 스프링 빈으로 자동 등록된다.
		- @Controller 컨트롤러가 스프링 빈으로 자동 등록된 이유도 컴포넌트 스캔 때문이다.
		- @Component 를 포함하는 다음 애노테이션도 스프링 빈으로 자동 등록된다.
			- @Controller
			- @Service
			- @Repository
	
	- 대부분의 케이스에서 스프링 빈은 싱글톤으로 사용함
	
- 회원 웹 기능
	- 컨트롤러 메서드에서 return "redirect:/"; 를 사용해 리다이렉트 가능
	
	- 컨트롤러 메서드의 파라미터 model 객체의 addAttribute를 통해 list 데이터를 담은 후 화면으로 리다이렉트 시켜주면
		화면에서 타임리프 th:each 문법으로 리스트의 데이터를 하나씩 꺼내어 사용 가능 
		```html
		<tr th:each="member : ${members}">
			<td th:text="${member.id}"></td>
			<td th:text="${member.name}"></td>
		</tr>
		```

- Repository 구현
	- DataSource는 데이터베이스 커넥션을 획득할 때 사용하는 객체다. 
		- 스프링 부트는 데이터베이스 커넥션 정보를 바탕으로 DataSource를 생성하고 스프링 빈으로 만들어둔다. 그래서 DI를 받을 수 있다.
		
- Spring 테스트 시
	- @SpringBootTest
		- 스프링 테스트하기위한 어노테이션 -> 스프링 컨테이너와 테스트 함께 실행
	- @Transactional
		- 테스트 클래스에 해당 어노테이션을 붙이면 테스트가 끝난 후 테스트한 데이터 롤백 시켜줌 -> 다음 테스트에 영향을 주지 않음
	- @Commit
		- @Transactional 어노테이션이 있어도 커밋을 수행함
		
- 생성자가 하나만 있을 때는, @Autowired 태그 생략 가능

- JPA 
	- ORM (객체를 관계형 데이터베이스에 맵핑하는 기술)
	- JPA는 인터페이스, hibernate는 JPA 구현한 기술이라 볼 수 있다.
	
	- JPA 사용할 때 기본 properties 설정 후, 도메인 객체를 구현한다.
		- 도메인 클래스에 @Entity 어노테이션
			- JPA가 관리하는 엔티티라고 표시
		
		- @Id : 해당 변수가 테이블의 pk임을 표시
		- @GeneratedValue(strategy = GenerationType.IDENTITY)
			- DB가 ID를 자동 생성해주는 것을 아이덴티티 전략이라고 함
		- @Column(name = "username")
			- DB와 변수 명이 다를 때 @Column 어노테이션을 사용해서 설정 가능
			
	- JPA Repository 구현
		- private final EntityManager em;
			- JPA 사용 시, 스프링에서 자동으로 EntityManager를 생성해준다.
			- repository 생성자에서 스프링에서 자동으로 생성해준 EntityManager를 DI받으면 된다.
			
	- JPA 사용 시 모든 데이터 변경이 @Transactional 안에서 수행되어야 함
	
- 스프링 데이터 JPA
	- 스프링에서 제공하는 JpaRepository 를 상속받아 repositry 인터페이스를 만들어놓으면 
		스프링 데이터 JPA는 구현체를 만들어서 자동으로 bean으로 등록해준다.
		
	- 스프링 데이터 JPA Repository 구현 시 메서드 이름 규칙(ex. findBy...)으로 커스텀 select 메서드 구현 가능


- AOP
	- 공통 관심 사항과 핵심 관심 사항을 분리하는 스프링 기능
	
	- 프록시 방식
