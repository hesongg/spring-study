### 스프링 핵심 원리 - 고급편

#### Reference) 
	* 스프링 핵심 원리 - 고급편 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/Spring-AOP
	
<br>


### 스프링 AOP - 포인트컷

<br>

#### 포인트컷 지시자

- 애스펙트J는 포인트컷을 편리하게 표현하기 위한 특별한 표현식을 제공한다.
	- 예) ```@Pointcut("execution(* hello.aop.order..*(..))")```

- 포인트컷 표현식은 AspectJ pointcut expression 즉 애스펙트J가 제공하는 포인트컷 표현식을 줄여서 말하는 것이다.

- 포인트컷 지시자
	- 포인트컷 표현식은 ```execution``` 같은 포인트컷 지시자(Pointcut Designator)로 시작한다. 
	- 줄여서 PCD라한다.

<br>

- 포인트컷 지시자의 종류
	- ```execution``` : 메소드 실행 조인 포인트를 매칭한다. 스프링 AOP에서 가장 많이 사용하고, 기능도복잡하다.
	
	- ```within``` : 특정 타입 내의 조인 포인트를 매칭한다.
	
	- ```args``` : 인자가 주어진 타입의 인스턴스인 조인 포인트
	
	- ```this``` : 스프링 빈 객체(스프링 AOP 프록시)를 대상으로 하는 조인 포인트
	
	- ```target``` : Target 객체(스프링 AOP 프록시가 가르키는 실제 대상)를 대상으로 하는 조인 포인트
	
	- ```@target``` : 실행 객체의 클래스에 주어진 타입의 애노테이션이 있는 조인 포인트
	
	- ```@within``` : 주어진 애노테이션이 있는 타입 내 조인 포인트
	
	- ```@annotation``` : 메서드가 주어진 애노테이션을 가지고 있는 조인 포인트를 매칭
	
	- ```@args``` : 전달된 실제 인수의 런타임 타입이 주어진 타입의 애노테이션을 갖는 조인 포인트
	
	- ```bean``` : 스프링 전용 포인트컷 지시자, 빈의 이름으로 포인트컷을 지정한다.

- ```execution``` 은 가장 많이 사용하고, 나머지는 자주 사용하지 않는다. 
	- 따라서 ```execution``` 을 중점적으로 이해

<br>

#### 예제 만들기

- 포인트컷 표현식을 이해하기 위해 예제 코드를 하나 추가

- ```ClassAop```, ```MethodAop```, ```MemberService```, ```MemberServiceImpl``` 커밋한 소스 참고

- ```ExecutionTest```
	- ```AspectJExpressionPointcut``` 이 바로 포인트컷 표현식을 처리해주는 클래스다. 
		- 여기에 포인트컷 표현식을 지정하면 된다. 
		- ```AspectJExpressionPointcut``` 는 상위에 ```Pointcut``` 인터페이스를 가진다.

<br>

#### execution1

- execution 문법
	```
	execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)
	```
	
	- 메소드 실행 조인 포인트를 매칭한다.
	- ?는 생략할 수 있다.
	- ```*``` 같은 패턴을 지정할 수 있다
	
<br>

- 가장 정확한 포인트컷
	- 먼저 ```MemberServiceImpl.hello(String)``` 메서드와 가장 정확하게 모든 내용이 매칭되는 표현식이다
	
		```java
		@Test
		void exactMatch() {
			//public java.lang.String hello.aop.member.MemberServiceImpl.hello(java.lang.String)
			pointcut.setExpression("execution(public String hello.aop.member.MemberServiceImpl.hello(String))");

			assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
		}
		```
		
		- ```AspectJExpressionPointcut``` 에 ```pointcut.setExpression``` 을 통해서 포인트컷 표현식을 적용할 수 있다.
		
		- ```pointcut.matches(메서드, 대상 클래스)``` 를 실행하면 지정한 포인트컷 표현식의 매칭 여부를 
			```true``` , ```false``` 로 반환한다.

	<br>
	
	- 매칭 조건
		- 접근제어자? : ```public```
		- 반환타입 : ```String```
		- 선언타입? : ```hello.aop.member.MemberServiceImpl```
		- 메서드이름 : ```hello```
		- 파라미터 : ```(String)```
		- 예외? : 생략

	- ```MemberServiceImpl.hello(String)``` 메서드와 포인트컷 표현식의 모든 내용이 정확하게 일치한다.
		- 따라서 ```true``` 를 반환한다.

<br>

- 가장 많이 생략한 포인트컷
	
	```java
	@Test
    void allMatch() {
        pointcut.setExpression("execution(* *(..))");

        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
    }
	```
		
	- 매칭 조건
		- 접근제어자? : 생략
		- 반환타입 : ```*```
		- 선언타입? : 생략
		- 메서드이름 : ```*```
		- 파라미터 : ```(..)```
		- 예외? : 없음

	<br>
	
	- ```*``` 은 아무 값이 들어와도 된다는 뜻이다.
	
	- 파라미터에서 ```..``` 은 파라미터의 타입과 파라미터 수가 상관없다는 뜻이다. 
		- ```( 0..* )``` 파라미터는 뒤에 자세히 정리하겠다.

<br>

- 내용 관련 작성한 코드는 커밋한 ```ExecutionTest``` 참고

- 메서드 이름 매칭 관련 포인트컷 

	- 메서드 이름 앞 뒤에 ```*``` 을 사용해서 매칭할 수 있다.

<br>

- 패키지 매칭 관련 포인트컷	
	
	- ```hello.aop.member.*(1).*(2)```
		- (1): 타입
		- (2): 메서드 이름
	
	- 패키지에서 . , .. 의 차이를 이해해야 한다.
		- ```.``` : 정확하게 해당 위치의 패키지
		- ```..``` : 해당 위치의 패키지와 그 하위 패키지도 포함

<br>

#### execution2

- 타입 매칭 - 부모 타입 허용

	- ```typeExactMatch()``` 는 타입 정보가 정확하게 일치하기 때문에 매칭된다.
	
	- ```typeMatchSuperType()``` 을 주의해서 보아야 한다.
	
	 - ```execution``` 에서는 ```MemberService``` 처럼 부모 타입을 선언해도 그 자식 타입은 매칭된다. 
		- 다형성에서 ```부모타입 = 자식타입``` 이 할당 가능하다는 점을 떠올려보면 된다.

<br>

- 타입 매칭 - 부모 타입에 있는 메서드만 허용

	- ```typeMatchInternal()``` 의 경우 ```MemberServiceImpl``` 를 표현식에 선언했기 때문에 
		그 안에 있는 ```internal(String)``` 메서드도 매칭 대상이 된다.
	
	- ```typeMatchNoSuperTypeMethodFalse()``` 를 주의해서 보아야 한다.
		- 이 경우 표현식에 부모 타입인 ```MemberService``` 를 선언했다. 
		
		- 그런데 자식 타입인 ```MemberServiceImpl``` 의 ```internal(String)``` 메서드를 매칭하려 한다. 
			- 이 경우 매칭에 실패한다. ```MemberService``` 에는 ```internal(String)``` 메서드가 없다!
		
		- 부모 타입을 표현식에 선언한 경우 부모 타입에서 선언한 메서드가 자식 타입에 있어야 매칭에 성공한다.
		
		- 그래서 부모 타입에 있는 ```hello(String)``` 메서드는 매칭에 성공하지만, 
			부모 타입에 없는 ```internal(String)``` 는 매칭에 실패한다.

<br>

- 파라미터 매칭

	- execution 파라미터 매칭 규칙은 다음과 같다.
		- ```(String)``` : 정확하게 String 타입 파라미터
		
		- ```()``` : 파라미터가 없어야 한다.
		
		- ```(*)``` : 정확히 하나의 파라미터, 단 모든 타입을 허용한다.
		
		- ```(*, *)``` : 정확히 두 개의 파라미터, 단 모든 타입을 허용한다.
		
		- ```(..)``` : 숫자와 무관하게 모든 파라미터, 모든 타입을 허용한다. 참고로 파라미터가 없어도 된다. 
			- ```0..*``` 로 이해하면 된다.
		
		- ```(String, ..)``` : String 타입으로 시작해야 한다. 숫자와 무관하게 모든 파라미터, 모든 타입을 허용한다.
			- 예) ```(String)``` , ```(String, Xxx)``` , ```(String, Xxx, Xxx)``` 허용

<br>

#### within

- within 지시자는 특정 타입 내의 조인 포인트에 대한 매칭을 제한한다. 
	- 쉽게 이야기해서 해당 타입이 매칭되면 그 안의 메서드(조인 포인트)들이 자동으로 매칭된다.

- 문법은 단순한데 ```execution``` 에서 타입 부분만 사용한다고 보면 된다.

<br>

- ```WithinTest``` - 코드 참고
	
	- 주의
		- 그런데 ```within``` 사용시 주의해야 할 점이 있다. 
		
		- 표현식에 부모 타입을 지정하면 안된다는 점이다. 
			- 정확하게 타입이 맞아야 한다. 
			- 이 부분에서 ```execution``` 과 차이가 난다.

	- 부모 타입(여기서는 ```MemberService``` 인터페이스) 지정시 ```within``` 은 실패하고, 
		```execution``` 은 성공하는 것을 확인할 수 있다.

<br>

#### args

- ```args``` : 인자가 주어진 타입의 인스턴스인 조인 포인트로 매칭
	- 기본 문법은 ```execution``` 의 ```args``` 부분과 같다.
	
- ```execution```과 ```args```의 차이점
	- ```execution``` 은 파라미터 타입이 정확하게 매칭되어야 한다. 
		- ```execution``` 은 클래스에 선언된 정보를 기반으로 판단한다.
		
	- ```args``` 는 부모 타입을 허용한다. 
		- ```args``` 는 실제 넘어온 파라미터 객체 인스턴스를 보고 판단한다.

<br>

- ```ArgsTest``` - 코드 참고

	- ```pointcut()``` 
		- ```AspectJExpressionPointcut``` 에 포인트컷은 한번만 지정할 수 있다. 
		- 테스트에서는 테스트를 편리하게 진행하기 위해 포인트컷을 여러번 지정하기 위해 포인트컷 자체를 생성하는 메서드를 만들었다.
	
	- 자바가 기본으로 제공하는 ``` String```  은 ``` Object```  , ``` java.io.Serializable```  의 하위 타입이다.
	
	- 정적으로 클래스에 선언된 정보만 보고 판단하는 ``` execution(* *(Object))```  는 매칭에 실패한다.
	
	- 동적으로 실제 파라미터로 넘어온 객체 인스턴스로 판단하는 ``` args(Object)```  는 매칭에 성공한다. 
		- (부모 타입 허용)

<br>

- 참고
	- ```args``` 지시자는 단독으로 사용되기 보다는 뒤에서 설명할 파라미터 바인딩에서 주로 사용된다.

<br>

#### @target, @within

- 정의
	- ```@target``` : 실행 객체의 클래스에 주어진 타입의 애노테이션이 있는 조인 포인트
	- ```@within``` : 주어진 애노테이션이 있는 타입 내 조인 포인트

<br>

- 설명
	- ```@target , @within``` 은 다음과 같이 타입에 있는 애노테이션으로 AOP 적용 여부를 판단한다.
		- ```@target(hello.aop.member.annotation.ClassAop)```
		- ```@within(hello.aop.member.annotation.ClassAop)```
		
		- ```@ClassAop``` 애노테이션은 미리 생성해놓음
		
<br>
	
- ```@target``` vs ```@within```
	- ```@target``` 은 인스턴스의 모든 메서드를 조인 포인트로 적용한다.
	- ```@within``` 은 해당 타입 내에 있는 메서드만 조인 포인트로 적용한다.

	<br>
	
	- 쉽게 이야기해서 ```@target``` 은 부모 클래스의 메서드까지 어드바이스를 다 적용하고, 
		```@within``` 은 자기 자신의 클래스에 정의된 메서드에만 어드바이스를 적용한다.

	![image](https://user-images.githubusercontent.com/77953474/188394402-9ce1f368-2ae4-423a-bc39-bb78df48efc4.png)

	
<br>

- ```AtTargetAtWithinTest``` - 코드 참고

	- ```parentMethod()``` 는 ```Parent``` 클래스에만 정의되어 있고, 
		```Child``` 클래스에 정의되어 있지 않기 때문에 ```@within``` 에서 AOP 적용 대상이 되지 않는다.
		
	- 실행결과를 보면 ```child.parentMethod()``` 를 호출 했을 때 ```[@within]``` 이 호출되지 않은 것을 확인할 수 있다.

<br>

- 참고
	- ```@target``` , ```@within``` 지시자는 뒤에서 설명할 파라미터 바인딩에서 함께 사용된다.

- 주의
	- 다음 포인트컷 지시자는 단독으로 사용하면 안된다. ```args, @args, @target```
	
	- 예제를 보면 ```execution(* hello.aop..*(..))``` 를 통해 적용 대상을 줄여준 것을 확인할 수 있다.
	
	- ```args , @args , @target``` 은 실제 객체 인스턴스가 생성되고 실행될 때 어드바이스 적용 여부를 확인할 수	있다.
	
	- 실행 시점에 일어나는 포인트컷 적용 여부도 결국 프록시가 있어야 실행 시점에 판단할 수 있다. 
		- 프록시가 없다면 판단 자체가 불가능하다. 
		
	- 그런데 스프링 컨테이너가 프록시를 생성하는 시점은 스프링 컨테이너가	만들어지는 애플리케이션 로딩 시점에 적용할 수 있다.
		
		- 따라서 args , ```@args , @target``` 같은 포인트컷 지시자가 있으면 스프링은 모든 스프링 빈에 AOP를 적용하려고 시도한다. 
	
		- 앞서 설명한 것 처럼 프록시가	없으면 실행 시점에 판단 자체가 불가능하다.
		
	- 문제는 이렇게 모든 스프링 빈에 AOP 프록시를 적용하려고 하면 스프링이 내부에서 사용하는 빈 중에는
		```final``` 로 지정된 빈들도 있기 때문에 오류가 발생할 수 있다.
		
	- 따라서 이러한 표현식은 최대한 프록시 적용 대상을 축소하는 표현식과 함께 사용해야 한다.

<br>

#### @annotation, @args

- ```@annotation```
	- 정의
		- ```@annotation``` : 메서드가 주어진 애노테이션을 가지고 있는 조인 포인트를 매칭

	- 다음과 같이 메서드(조인 포인트)에 애노테이션이 있으면 매칭한다.
		
		- 포인트 컷 설정 : ```@annotation(hello.aop.member.annotation.MethodAop)```
		
		- 적용되는 코드
			```java
			public class MemberServiceImpl {
				
				@MethodAop("test value")
				public String hello(String param) {
					return "ok";
				}
			}
			```

<br>

- ```@args```
	- 정의
		- ```@args``` : 전달된 실제 인수의 런타임 타입이 주어진 타입의 애노테이션을 갖는 조인 포인트
	
	- 설명
		- 전달된 인수의 런타임 타입에 ```@Check``` 애노테이션이 있는 경우에 매칭한다.
		
		- 포인트 컷 설정 : ```@args(test.Check)```
			- 이렇게 설정하면 전달된 인수 객체에 ```@Check``` 애노테이션이 선언되어있는 경우 매칭됨

<br>

#### bean

- 설명
	- 스프링 빈의 이름으로 AOP 적용 여부를 지정한다. 이것은 스프링에서만 사용할 수 있는 특별한	지시자이다.
	
	- ```bean(orderService) || bean(*Repository)```
		- ```*``` 과 같은 패턴을 사용할 수 있다.

<br>

#### 매개변수 전달

- 다음은 포인트컷 표현식을 사용해서 어드바이스에 매개변수를 전달할 수 있다.
	- ```this, target, args,@target, @within, @annotation, @args```

- 다음과 같이 사용
	```java
	@Before("allMember() && args(arg,..)")
	public void logArgs3(String arg) {
		log.info("[logArgs3] arg={}", arg);
	}
	```
	
	- 포인트컷의 이름과 매개변수의 이름을 맞추어야 한다. 여기서는 ```arg``` 로 맞추었다.
	
	- 추가로 타입이 메서드에 지정한 타입으로 제한된다. 
		- 여기서는 메서드의 타입이 ```String``` 으로 되어 있기 때문에 다음과 같이 정의되는 것으로 이해하면 된다.
			- ```args(arg,..) -> args(String,..)```

<br>

- ```ParameterTest``` - 다양한 포인트컷을 사용해서 매개변수 전달 예시 테스트 코드 작성
	
	```java
	@Slf4j
	@SpringBootTest
	@Import(ParameterTest.ParameterAspect.class)
	public class ParameterTest {

		@Autowired
		MemberService memberService;

		@Test
		void success() {
			log.info("memberService Proxy={}", memberService.getClass());
			memberService.hello("helloA");
		}

		@Aspect
		static class ParameterAspect {

			@Pointcut("execution(* hello.aop.member..*.*(..))")
			private void allMember() {
			}

			@Around("allMember()")
			public Object logArgs1(ProceedingJoinPoint joinPoint) throws Throwable {
				Object arg1 = joinPoint.getArgs()[0];
				log.info("[logArgs1]{}, arg={}", joinPoint.getSignature(), arg1);
				return joinPoint.proceed();
			}

			@Around("allMember() && args(arg,..)")
			public Object logArgs2(ProceedingJoinPoint joinPoint, Object arg) throws Throwable {
				log.info("[logArgs2]{}, arg={}", joinPoint.getSignature(), arg);
				return joinPoint.proceed();
			}

			@Before("allMember() && args(arg,..)")
			public void logArgs3(String arg) {
				log.info("[logArgs3] arg={}", arg);
			}

			@Before("allMember() && this(obj)")
			public void thisArgs(JoinPoint joinPoint, MemberService obj) {
				log.info("[this]{}, obj={}", joinPoint.getSignature(), obj.getClass());
			}

			@Before("allMember() && target(obj)")
			public void targetArgs(JoinPoint joinPoint, MemberService obj) {
				log.info("[target]{}, obj={}", joinPoint.getSignature(), obj.getClass());
			}

			@Before("allMember() && @target(annotation)")
			public void atTarget(JoinPoint joinPoint, ClassAop annotation) {
				log.info("[@target]{}, obj={}", joinPoint.getSignature(), annotation);
			}

			@Before("allMember() && @within(annotation)")
			public void atWithin(JoinPoint joinPoint, ClassAop annotation) {
				log.info("[@within]{}, obj={}", joinPoint.getSignature(), annotation);
			}

			@Before("allMember() && @annotation(annotation)")
			public void atAnnotation(JoinPoint joinPoint, MethodAop annotation) {
				log.info("[@annotation]{}, annotationValue={}", joinPoint.getSignature(), annotation.value());
			}

		}
	}
	```
	
	- ```logArgs1``` : ```joinPoint.getArgs()[0]``` 와 같이 매개변수를 전달 받는다.
	
	- ```logArgs2``` : ```args(arg,..)``` 와 같이 매개변수를 전달 받는다.
	
	- ```logArgs3``` : ```@Before``` 를 사용한 축약 버전이다. 추가로 타입을 ```String``` 으로 제한했다.
	
	- ```this``` : 프록시 객체를 전달 받는다.
	
	- ```target``` : 실제 대상 객체를 전달 받는다.
	
	- ```@target , @within``` : 타입의 애노테이션을 전달 받는다.
	
	- ```@annotation``` : 메서드의 애노테이션을 전달 받는다. 
		- 여기서는 ```annotation.value()``` 로 해당 애노테이션의 값을 출력하는 모습을 확인할 수 있다.

<br>

- 실행 결과 - 실제 테스트 순서는 다를 수 있음
	```
	memberService Proxy=class hello.aop.member.MemberServiceImpl$$EnhancerBySpringCGLIB$$82
	[logArgs1]String hello.aop.member.MemberServiceImpl.hello(String), arg=helloA
	[logArgs2]String hello.aop.member.MemberServiceImpl.hello(String), arg=helloA
	[logArgs3] arg=helloA
	[this]String hello.aop.member.MemberServiceImpl.hello(String), obj=class hello.aop.member.MemberServiceImpl$$EnhancerBySpringCGLIB$$8
	[target]String hello.aop.member.MemberServiceImpl.hello(String), obj=class hello.aop.member.MemberServiceImpl
	[@target]String hello.aop.member.MemberServiceImpl.hello(String), obj=@hello.aop.member.annotation.ClassAop()
	[@within]String hello.aop.member.MemberServiceImpl.hello(String), obj=@hello.aop.member.annotation.ClassAop()
	[@annotation]String hello.aop.member.MemberServiceImpl.hello(String), annotationValue=test value
	```

<br>

#### this, target

- 정의
	- ```this``` : 스프링 빈 객체(스프링 AOP 프록시)를 대상으로 하는 조인 포인트
	- ```target``` : Target 객체(스프링 AOP 프록시가 가르키는 실제 대상)를 대상으로 하는 조인 포인트

<br>

- 설명
	- ```this``` , ```target``` 은 다음과 같이 적용 타입 하나를 정확하게 지정해야 한다
		```
		this(hello.aop.member.MemberService)
		target(hello.aop.member.MemberService)
		```
	
	- ```*``` 같은 패턴을 사용할 수 없다.
	
	- 부모 타입을 허용한다.

<br>

- ```this vs target```
	- 단순히 타입 하나를 정하면 되는데, this 와 target 은 어떤 차이가 있을까?
	
	- 스프링에서 AOP를 적용하면 실제 ```target``` 객체 대신에 프록시 객체가 스프링 빈으로 등록된다.
	
	- ```this``` 는 스프링 빈으로 등록되어 있는 프록시 객체를 대상으로 포인트컷을 매칭한다.
	- ```target``` 은 실제 target 객체를 대상으로 포인트컷을 매칭한다.

<br>

- 프록시 생성 방식에 따른 차이
	
	- 스프링은 프록시를 생성할 때 JDK 동적 프록시와 CGLIB를 선택할 수 있다. 둘의 프록시를 생성하는 방식이 다르기 때문에 차이가 발생한다.
	
	- JDK 동적 프록시 : 인터페이스가 필수이고, 인터페이스를 구현한 프록시 객체를 생성한다.
	
	- CGLIB : 인터페이스가 있어도 구체 클래스를 상속 받아서 프록시 객체를 생성한다.

	<br>
	
	- JDK 동적 프록시
		- ```MemberService``` 인터페이스 지정
			- ```this(hello.aop.member.MemberService)```
				- proxy 객체를 보고 판단한다. 
				- ```this``` 는 부모 타입을 허용하기 때문에 AOP가 적용된다.
			
			- ```target(hello.aop.member.MemberService)```
				- ```target``` 객체를 보고 판단한다. 
				- ```target``` 은 부모 타입을 허용하기 때문에 AOP가 적용된다.
		
		- ```MemberServiceImpl``` 구체 클래스 지정
			- ```this(hello.aop.member.MemberServiceImpl)```
				- proxy 객체를 보고 판단한다. JDK 동적 프록시로
					만들어진 proxy 객체는 ```MemberService``` 인터페이스를 기반으로 구현된 새로운 클래스다. 
				
				- 따라서 ```MemberServiceImpl``` 를 전혀 알지 못하므로 AOP 적용 대상이 아니다.
			
			- ```target(hello.aop.member.MemberServiceImpl)``` 
				- target 객체를 보고 판단한다. 
				- target 객체가 ```MemberServiceImpl``` 타입이므로 AOP 적용 대상이다.
	
	<br>
	
	- CGLIB 프록시
		
		- ```MemberService``` 인터페이스 지정
			- ```this(hello.aop.member.MemberService)```
				- proxy 객체를 보고 판단한다. 
				- ```this``` 는 부모 타입을 허용하기 때문에 AOP가 적용된다.
				
			- ```target(hello.aop.member.MemberService)```
				- target 객체를 보고 판단한다.
				- target 은 부모 타입을 허용하기 때문에 AOP가 적용된다
		
		- ```MemberServiceImpl``` 구체 클래스 지정
			- ```this(hello.aop.member.MemberServiceImpl)```
				- proxy 객체를 보고 판단한다. 
				
				- CGLIB로 만들어진 proxy 객체는 ```MemberServiceImpl``` 를 상속 받아서 만들었기 때문에 AOP 적용가 적용된다. 
				
				- ```this``` 가 부모 타입을 허용하기 때문에 포인트컷의 대상이 된다.
			
			- ```target(hello.aop.member.MemberServiceImpl)```
				- target 객체를 보고 판단한다. 
				
				- target 객체가 ```MemberServiceImpl``` 타입이므로 AOP 적용 대상이다.

	- 정리
		- 프록시를 대상으로 하는 ```this``` 의 경우 구체 클래스를 지정하면 프록시 생성 전략에 따라서 다른 결과가 나올 수 있다는 점을 알아두자.

<br>

- ```ThisTargetTest``` - 테스트 코드 작성

	- ```@SpringBootTest(properties = "spring.aop.proxy-target-class=false")```
		- ```application.properties``` 에 설정하는 대신에 해당 테스트에서만 설정을 임시로 적용한다. 
		- 이렇게 하면 각 테스트마다 다른 설정을 손쉽게 적용할 수 있다.
		
	- ```spring.aop.proxy-target-class=false```
		- 스프링이 AOP 프록시를 생성할 때 JDK 동적 프록시를 우선 생성한다. 
		- 물론 인터페이스가 없다면 CGLIB를 사용한다.
	
	- ```spring.aop.proxy-target-class=true```
		- 스프링이 AOP 프록시를 생성할 때 CGLIB 프록시를 생성한다. 
		- 참고로 이 설정을 생략하면 스프링 부트에서 기본으로 CGLIB를 사용한다. 
			- 이 부분은 뒤에서 자세히 설명

	<br>
	
	- JDK 동적 프록시를 사용하면 ```this(hello.aop.member.MemberServiceImpl)``` 로 지정한 
		```[thisimpl]``` 부분이 출력되지 않는 것을 확인할 수 있다.
	
<br>

- 참고
	- ```this``` , ```target``` 지시자는 단독으로 사용되기 보다는 파라미터 바인딩에서 주로 사용된다.
