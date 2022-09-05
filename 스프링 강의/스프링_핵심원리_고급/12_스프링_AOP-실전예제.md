### 스프링 핵심 원리 - 고급편

#### Reference) 
	* 스프링 핵심 원리 - 고급편 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/Spring-AOP
	
<br>


### 스프링 AOP - 실전 예쩨

<br>

#### 예제 만들기

- 예제 만들기
	
	- 지금까지 학습한 내용을 활용해서 유용한 스프링 AOP를 만들어보자.
		- ```@Trace``` 애노테이션으로 로그 출력하기
		- ```@Retry``` 애노테이션으로 예외 발생시 재시도 하기
		
	- 테스트 코드 ```ExamTest``` 작성 - 커밋한 소스 참고
		
	- 실행해보면 테스트가 5번째 루프를 실행할 때 리포지토리 위치에서 예외가 발생하면서 실패하는 것을 확인 가능

<br>

#### 로그 출력 AOP

- 먼저 로그 출력용 AOP를 만들어보자.
	- ```@Trace``` 가 메서드에 붙어 있으면 호출 정보가 출력되는 편리한 기능이다.

<br>

- ```@Trace```
	```java
	@Target(ElementType.METHOD)
	@Retention(RetentionPolicy.RUNTIME)
	public @interface Trace {
	}
	```
	
<br>

- ```TraceAspect```
	- ```@annotation(hello.aop.exam.annotation.Trace)``` 포인트컷을 사용해서 ```@Trace``` 가 붙은 메서드에 어드바이스를 적용

<br>

- ```ExamService``` - ```@Trace``` 추가

- ```ExamRepository``` - ```@Trace``` 추가

<br>

- 실행해보면 ```@Trace``` 가 붙은 메서드 호출시 로그가 잘 남는 것을 확인 가능

<br>

#### 재시도 AOP

- 재시도 AOP 구현
	- ```@Retry``` 애노테이션이 있으면 예외가 발생했을 때 다시 시도해서 문제를 복구

<br>

- ```@Retry```
	
	```java
	@Target(ElementType.METHOD)
	@Retention(RetentionPolicy.RUNTIME)
	public @interface Retry {
		int value() default 3;
	}
	```
	
	- 이 애노테이션에는 재시도 횟수로 사용할 값이 있다. 기본값으로 ```3``` 을 사용

<br>

- ```RetryAspec```
	
	```java
	@Slf4j
	@Aspect
	public class RetryAspect {

		@Around("@annotation(retry)")
		public Object doRetry(ProceedingJoinPoint joinPoint, Retry retry) throws Throwable {
			log.info("[retry] {} args={}", joinPoint.getSignature(), retry);

			int maxRetry = retry.value();

			Exception exceptionHolder = null;

			for (int retryCnt = 1; retryCnt <= maxRetry; retryCnt++) {
				try {
					log.info("[retry] try count={}/{}", retryCnt, maxRetry);
					return joinPoint.proceed();
				} catch (Exception e) {
					exceptionHolder = e;
				}
			}
			throw exceptionHolder;
		}
	}
	```

	- 재시도 하는 애스펙트이다.
	
	- ```@annotation(retry)``` , ```Retry retry``` 를 사용해서 어드바이스에 애노테이션을 파라미터로 전달한다.
	
	- ```retry.value()``` 를 통해서 애노테이션에 지정한 값을 가져올 수 있다.
	
	- 예외가 발생해서 결과가 정상 반환되지 않으면 ```retry.value()``` 만큼 재시도한다.

<br>

- ```ExamRepository``` - ```@Retry``` 추가
	
	```java
	public class ExamRepository {

		private static int seq = 0;

		/**
		 * 5번에 1번 실패하는 요청
		 */
		@Trace
		@Retry(4)
		public String save(String itemId) {
			seq++;
			if (seq % 5 == 0) {
				throw new IllegalStateException("예외 발생");
			}

			return "ok";
		}
	}
	```
	
	- ```ExamRepository.save()``` 메서드에 ```@Retry(value = 4)``` 를 적용했다. 
	- 이 메서드에서 문제가 발생하면 4번 재시도 한다

<br>

- 실행 결과
	- 실행 결과를 보면 5번째 문제가 발생했을 때 재시도 덕분에 문제가 복구되고, 정상 응답되는 것을 확인할 수 있다.

- 참고
	- 스프링이 제공하는 ```@Transactional``` 은 가장 대표적인 AOP이다.
