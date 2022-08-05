### 스프링 웹 MVC - 백엔드 웹 개발 활용 기술

#### Reference) 
	* 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/SpringMVC-Practice-Validation1
	
<br>

### 검증1 - Validation


<br>

#### 검증 직접 처리 -개발

- 사용자가 넘겨준 ```Item``` 값에 오류가 있는 경우, errors map에 오류 내용을 넣어 뷰 템플릿으로 반환해주는 방식으로 개발해봄
	- 오류 내용을 ```Model```의 ```addAttribute``` 메서드를 이용해서 model에 담는다.
	- 입력 폼이있는 뷰 템플릿으로 보내어서 타임리프 기능으로 오류 내용을 보여준다.

- html 수정 내용
	- css추가 ```head``` 태그 -> ```style``` 태그 안에 내용 추가함
		```html
		.field-error {
		 border-color: #dc3545;
		 color: #dc3545;
		}
		```
	
	- 오류 메시지 출력
		```html
		<div th:if="${errors?.containsKey('globalError')}">
			<p class="field-error" th:text="${errors['globalError']}">전체 오류 메시지</p>
		</div>
		```
		- 오류 메시지는 ```errors``` 에 내용이 있을 때만 출력하면 된다. 
			- 타임리프의 ```th:if``` 를 사용하면 조건에 만족할 때만 해당 HTML 태그를 출력할 수 있다. 
		
	##### 참고) Safe Navigation Operator
		
		- ```errors?.``` 는 ```errors``` 가 ```null```일때 
		```NullPointerException``` 이 발생하는 대신, ```null``` 을 반환하는 문법이다. 
			- ```th:if``` 에서 ```null``` 은 실패로 처리되므로 오류 메시지가 출력되지 않는다.
	
	- 필드 오류 처리 - 입력 폼 색상 적용 ```th:classappend```
		```html
		<input type="text" 
			th:classappend="${errors?.containsKey('itemName')} ? 'fielderror' : _" 
			class="form-control">
		```
		- ```classappend``` 를 사용해서 해당 필드에 오류가 있으면 ```field-error``` 라는 클래스 정보를 더해서 
			폼의 색깔을 빨간색으로 강조한다. 만약 값이 없으면 ```_ (No-Operation)``` 을 사용해서 아무것도 하지 않는다.
	
	- 필드 오류 처리 - 메시지
		```html
		<div class="field-error" th:if="${errors?.containsKey('itemName')}" th:text="${errors['itemName']}">
			상품명 오류
		</div>
		```

- 남은 문제점
	- 뷰 템플릿에서 중복되는 처리가 많다.
	- 타입 오류 처리가 안된다. 
		- 입력 시 숫자 타입에 문자가 들어오면 오류가 발생한다. 이러한 오류는 스프링MVC에서 컨트롤러에 진입하기도 전에 예외가 발생하기 때문에, 
			컨트롤러가 호출되지도 않고, ```400``` 예외가 발생하면서 오류 페이지를 띄워줌
	- 타입 오류가 발생해도 고객이 입력한 문자를 화면에 남겨야한다. 
		- 만약 컨트롤러가 호출된다고 가정해도 고객이 입력한 값을 보관할 수가 없다. 
		- 고객이 입력한 값도 어딘가에 별도로 관리가 되어야 한다.
		
<br>

#### BindingResult

- ```BindingResult``` 사용 시 주의점
	- 밑의 예시와 같이 ```BindingResult bindingResult``` 파라미터의 위치는 ```@ModelAttribute Item item``` 다음에 와야한다.
		```java
		@PostMapping("/add")
		public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) { ... }
		```

- 필드 오류 - ```FieldError```
	```java
	if (!StringUtils.hasText(item.getItemName())) {
		bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
	}
	```
		
	- 필드에 오류가 있으면 ```FieldError``` 객체를 생성해서 ```bindingResult``` 에 담아두면 된다.
	
	- ```FieldError``` 생성자 파라미터 
		- ```objectName``` : ```@ModelAttribute``` 이름
		- ```field``` : 오류가 발생한 필드 이름
		- ```defaultMessage``` : 오류 기본 메시지
	

- 글로벌 오류 - ```ObjectError```
	```java
	bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
	```

<br>

- ```BindingResult``` 를 타임리프에서 사용
	
	- 타임리프 스프링 검증 오류 통합 기능
		- 타임리프는 스프링의 ```BindingResult``` 를 활용해서 편리하게 검증 오류를 표현하는 기능을 제공한다.
		- ```#fields``` : ```#fields``` 로 ```BindingResult``` 가 제공하는 검증 오류에 접근할 수 있다.
		- ```th:errors``` : 해당 필드에 오류가 있는 경우에 태그를 출력한다. ```th:if``` 의 편의 버전이다.
		- ```th:errorclass``` : ```th:field``` 에서 지정한 필드에 오류가 있으면 ```class``` 정보를 추가한다.
	
	- 검증과 오류 메시지 공식 메뉴얼
		- https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html#validation-and-error-messages
	
	- 글로벌 오류 처리 예시
		```html
		<div th:if="${#fields.hasGlobalErrors()}">
			<p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">전체 오류 메시지</p>
		</div>
		```
		
	- 필드 오류 처리 예시
		```html
		<input type="text" id="itemName" th:field="*{itemName}"
			th:errorclass="field-error" class="form-control" placeholder="이름을 입력하세요">
		<div class="field-error" th:errors="*{itemName}">
			상품명 오류
		</div>
		```

<br>


#### BindingResult 2

- ```BindingResult``` 가 있으면 ```@ModelAttribute``` 에 데이터 바인딩 시 오류가 발생해도 컨트롤러가 호출된다
	
	- ```@ModelAttribute```에 바인딩 시 타입 오류가 발생하면?
		- ```BindingResult``` 가 없으면 : ```400``` 오류가 발생하면서 컨트롤러가 호출되지 않고, 오류 페이지로 이동한다.
		- ```BindingResult``` 가 있으면 : 오류 정보( ```FieldError``` )를 ```BindingResult``` 에 담아서 컨트롤러를 정상 호출한다.
		
<br>

##### BindingResult에 검증 오류를 적용하는 3가지 방법

- ```@ModelAttribute``` 의 객체에 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 ```FieldError``` 생성해서 ```BindingResult``` 에 넣어준다

- 개발자가 직접 넣어준다

- ```Validator 사용``` -> 뒤에서 설명

<br>

- ```FieldError``` 생성자
	- FieldError 는 두 가지 생성자를 제공한다.
		- 화면에서 입력한 값이 유지되게 하려면 ```rejectedValue``` 값을 화면에서 넘어온 값으로 설정하자.
			- 오류 발생 시 사용자 입력 값이 유지된다.
			```java
			bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수입니다."));
			```
		
		- FieldError 는 오류 발생시 사용자 입력 값을 저장하는 기능을 제공
		
		- ```rejectedValue``` 가 오류 발생시 사용자 입력 값을 저장하는 필드
		
		- ```bindingFailure``` 는 타입 오류 같은 바인딩이 실패했는지 여부를 적어주면 된다. 

#### 타임리프의 사용자 입력 값 유지

```html
th:field="*{price}"
```

- 타임리프의 ```th:field``` 는 매우 똑똑하게 동작하는데, 정상 상황에는 모델 객체의 값을 사용하지만, 
	오류가 발생하면 ```FieldError``` 에서 보관한 값을 사용해서 값을 출력한다.

<br>

#### 스프링의 바인딩 오류 처리

- 타입 오류로 바인딩에 실패하면 스프링은 ```FieldError``` 를 생성하면서 사용자가 입력한 값을 넣어둔다.
- 그리고 해당 오류를 ```BindingResult``` 에 담아서 컨트롤러를 호출한다. 
- 따라서 타입 오류 같은 바인딩 실패시에도 사용자의 오류 메시지를 정상 출력할 수 있다.
	
<br>

- errors 메시지 파일 생성
	- 먼저 스프링 부트가 해당 메시지 파일을 인식할 수 있게 다음 설정을 추가한다. 
		- 스프링 부트 메시지 설정 추가 - ```application.properties```
			```properties
			spring.messages.basename=messages,errors
			```
	
		- 이렇게하면 ```messages.properties``` , ```errors.properties``` 두 파일을 모두 인식한다. 
		- (생략하면 ```messages.properties``` 를 기본으로 인식한다.)
		
	- 예시 코드
		```java
		//range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
		new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"}, new Object[]{1000, 1000000}
		```
		
		- ```codes``` : ```required.item.itemName``` 를 사용해서 메시지 코드를 지정한다. 
			- 메시지 코드는 하나가 아니라 배열로 여러 값을 전달할 수 있는데, 순서대로 매칭해서 처음 매칭되는 메시지가 사용된다.
		
		- ```arguments``` : ```Object[]{1000, 1000000}``` 를 사용해서 코드의 ```{0}``` , ```{1}``` 로 치환할 값을 전달한다.

<br>

#### rejectValue, reject 메서드

- ```FieldError``` , ```ObjectError``` 는 다루기 너무 번거롭기 때문에 좀 더 자동화된 기능을 사용할 수 있다.

- 컨트롤러에서 ```BindingResult``` 는 검증해야 할 객체인 ```target``` 바로 다음에 온다. 
	- 따라서 ```BindingResult``` 는 이미 본인이 검증해야 할 객체인 ```target``` 을 알고 있다.

- ```BindingResult``` 가 제공하는 ```rejectValue()``` , ```reject()``` 를 사용하면 
	```FieldError``` , ```ObjectError``` 를 직접 생성하지 않고, 깔끔하게 검증 오류를 다룰 수 있다.
	
	- ```resultValue```
		```java
		void rejectValue(@Nullable String field, String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);
		```
	
		- 사용 예시
			```java
			bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null)
			```
			
		- 앞에서 ```BindingResult``` 는 어떤 객체를 대상으로 검증하는지 ```target```을 이미 알고 있다고 했다. 
			- 따라서 ```target```(item)에 대한 정보는 없어도 된다. 오류 필드명은 동일하게 ```price``` 를 사용했다.

		- 축약된 오류 코드
			- ```FieldError()``` 를 직접 다룰 때는 오류 코드를 ```range.item.price``` 와 같이 모두 입력했다. 
			- 그런데 ```rejectValue()``` 를 사용하고 부터는 오류 코드를 ```range``` 로 간단하게 입력했다. 
			- 이 부분을 이해하려면 ```MessageCodesResolver``` 를 이해해야 한다.
		
	- ```ObjectError``` 를 발생시켜야하면 ```reject()```를 사용하자.
		```java
		void reject(String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);
		```

<br>

#### MessageCodesResolver - BindingResult 동작 방식

- ```MessageCodesResolver```
	
	- 검증 오류 코드로 메시지 코드들을 생성한다.
	- ```MessageCodesResolver```는 인터페이스이고 ```DefaultMessageCodesResolver``` 는 기본 구현체이다.
	- 주로 다음과 함께 사용 : ```ObjectError``` , ```FieldError```

- ```DefaultMessageCodesResolver```의 기본 메시지 생성 규칙
	
	- 객체 오류
		- 객체 오류의 경우 다음 순서로 2가지 생성
			- 1.: ```code + "." + object name```
			- 2.: ```code```
		
		- 예) 오류 코드: ```required```, object name: ```item```
			- 1.: ```required.item```
			- 2.: ```required```
		
	- 필드 오류
		- 필드 오류의 경우 다음 순서로 4가지 메시지 코드 생성
			- 1.: ```code + "." + object name + "." + field```
			- 2.: ```code + "." + field```
			- 3.: ```code + "." + field type```
			- 4.: ```code```
		
		- 예) 오류 코드: ```typeMismatch```, object name ```"user"```, field ```"age"```, field type: ```int```
			- 1. ```"typeMismatch.user.age"```
			- 2. ```"typeMismatch.age"```
			- 3. ```"typeMismatch.int"```
			- 4. ```"typeMismatch"```
			
- 동작 방식
	- ```rejectValue()``` , ```reject()``` 는 내부에서 ```MessageCodesResolver``` 를 사용한다. 여기에서 메시지 코드들을 생성한다.
	
	- ```FieldError``` , ```ObjectError``` 의 생성자를 보면, 오류 코드를 하나가 아니라 여러 오류 코드를 가질 수 있다.
		- ```MessageCodesResolver``` 를 통해서 생성된 순서대로 오류 코드를 보관한다
		
		
	- FieldError - ```rejectValue("itemName", "required")```
		- 다음 4가지 오류 코드를 자동으로 생성
			- ```required.item.itemName```
			- ```required.itemName```
			- ```required.java.lang.String```
			- ```required```
	
	- ObjectError - ```reject("totalPriceMin")```
		- 다음 2가지 오류 코드를 자동으로 생성
			- ```totalPriceMin.item```
			- ```totalPriceMin```
	
	- 오류 메시지 출력
		- 타임리프 화면을 렌더링 할 때 ```th:errors``` 가 실행된다. 
		- 만약 이때 오류가 있다면 생성된 오류 메시지 코드를 순서대로 돌아가면서 메시지를 찾는다. 
		- 그리고 없으면 디폴트 메시지를 출력한다.


<br>

#### 오류 코드 관리 전략

- 핵심은 구체적인 것에서 -> 덜 구체적인 것으로
	- ```MessageCodesResolver``` 는 ```required.item.itemName``` 처럼 구체적인 것을 먼저 만들어주고,
		```required``` 처럼 덜 구체적인 것을 가장 나중에 만든다.
		- 메시지와 관련된 공통 전략을 편리하게 도입할 수 있다
		
	- 크게 중요하지 않은 메시지는 범용성 있는 ```requried``` 같은 메시지로 끝내고, 
		정말 중요한 메시지는 꼭 필요할 때 구체적으로 적어서 사용하는 방식이 더 효과적
		
	- 예시) 크게 객체 오류와 필드 오류를 나누고 범용성에 따라 레벨을 나누어두자.
		- ```itemName``` 의 경우 ```required``` 검증 오류 메시지가 발생하면 다음 코드 순서대로 메시지가 생성된다.
			- 1. ```required.item.itemName```
			- 2. ```required.itemName```
			- 3. ```required.java.lang.String```
			- 4. ```required```
			
		- 그리고 이렇게 생성된 메시지 코드를 기반으로 순서대로 ```MessageSource``` 에서 메시지를 찾는다
		
		- 구체적인 것에서 덜 구체적인 순서대로 찾는다. 
			- 메시지에 1번이 없으면 2번을 찾고, 2번이 없으면 3번을 찾음
			- 이렇게 되면 만약에 크게 중요하지 않은 오류 메시지는 기존에 정의된 것을 그냥 재활용 하면 된다.
			
<br>

#### ValidationUtils

- ValidationUtils 사용 전
	```java
	if (!StringUtils.hasText(item.getItemName())) {
		bindingResult.rejectValue("itemName", "required", "기본: 상품 이름은 필수입니다.");
	}
	```
	
- ValidationUtils 사용 후
	- 다음과 같이 한줄로 사용 가능하고 제공하는 기능은 ```Empty``` , 공백 같은 단순한 기능만 제공한다.
		```java
		ValidationUtils.rejectIfEmptyOrWhitespace(bindingResult, "itemName","required");
		```

<br>

#### 정리

- 1. ```rejectValue()``` 호출
- 2. ```MessageCodesResolver``` 를 사용해서 검증 오류 코드로 메시지 코드들을 생성
- 3. ```new FieldError()``` 를 생성하면서 메시지 코드들을 보관
- 4. ```th:erros``` 에서 메시지 코드들로 메시지를 순서대로 메시지에서 찾고, 노출

<br>

#### 스프링이 직접 만든 오류 메시지 처리

- 검증 오류 코드는 다음과 같이 2가지로 나눌 수 있다.
	- 개발자가 직접 설정한 오류 코드 -> ```rejectValue()```를 직접 호출
	- 스프링이 직접 검증 오류에 추가한 경우(주로 타입 정보가 맞지 않음)
	
- 스프링은 타입 오류가 발생하면 ```typeMismatch``` 라는 오류 코드를 사용한다. 
	- 이 오류 코드가 ```MessageCodesResolver``` 를 통하면서 몇 가지 메시지 코드를 생성하는 것이다.

- ```error.properties``` 에 해당 내용을 추가하면 타입 오류가 발생하였을 때, 원하는 메시지를 출력 가능하다.
	```properties
	typeMismatch.java.lang.Integer=숫자를 입력해주세요.
	typeMismatch=타입 오류입니다.
	```
	
<br>

#### Validator 분리1

- 복잡한 검증 로직을 별도로 분리하자.

- 컨트롤러에서 검증 로직이 차지하는 부분은 매우 크다. 
	- 이런 경우 별도의 클래스로 역할을 분리하는 것이 좋다. 
	- 그리고 이렇게 분리한 검증 로직을 재사용 할 수도 있다.
	
- 스프링은 검증을 체계적으로 제공하기 위해 다음 인터페이스를 제공
	```java
	public interface Validator {
		boolean supports(Class<?> clazz);
		void validate(Object target, Errors errors);
	}
	```
	
	- ```supports() {}``` : 해당 검증기를 지원하는 여부 확인(뒤에서 설명)
	- ```validate(Object target, Errors errors)``` : 검증 대상 객체와 ```BindingResult``` 를 파라미터로 받음

<br>

#### Validator 분리2

- 스프링이 ```Validator``` 인터페이스를 별도로 제공하는 이유는 체계적으로 검증 기능을 도입하기 위해서다.
	- 그런데 앞에서는 검증기를 직접 불러서 사용했고, 이렇게 사용해도 된다. 
	- ```Validator``` 인터페이스를 사용해서 검증기를 만들면 스프링의 추가적인 도움을 받을 수 있다.
	
- ```WebDataBinder```를 통해서 사용하기
	- ```WebDataBinder``` 는 스프링의 파라미터 바인딩의 역할을 해주고 검증 기능도 내부에 포함한다
	
- ```Controller``` 에 다음 코드 추가
	```java
	@InitBinder
	public void init(WebDataBinder dataBinder) {
		log.info("init binder {}", dataBinder);
		dataBinder.addValidators(itemValidator);
	}
	```
	
	- 이렇게 ```WebDataBinder``` 에 검증기를 추가하면 해당 컨트롤러에서는 검증기를 자동으로 적용할 수 있다.
		- ```@InitBinder``` : 해당 컨트롤러에만 영향을 준다. 글로벌 설정 같은 경우 별도로 해야한다. (뒤에 설명)

- ```@Validated``` 적용
	- validator를 직접 호출하는 부분을 제거하고, 대신에 검증 대상 앞에 ```@Validated``` 를 사용할 수 있다.
		```java
		public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) { ... }
		```
		
	- 동작 방식
		- ```@Validated``` 는 검증기를 실행하라는 애노테이션이다.
		- 이 애노테이션이 붙으면 앞서 ```WebDataBinder``` 에 등록한 검증기를 찾아서 실행한다. 
		- 그런데 여러 검증기를 등록한다면 그 중에 어떤 검증기가 실행되어야 할지 구분이 필요하다. 
		- 이때 ```supports()``` 가 사용된다. - ```Validator``` 인터페이스를 상속받아 구현한 메서드
		- 여기서는 ```supports(Item.class)``` 호출되고, 결과가 ```true``` 이므로 ```ItemValidator``` 의 ```validate()``` 가 호출된다.


- 참고) 글로벌 설정 - 모든 컨트롤러에 다 적용 됨
	```java
	@SpringBootApplication
	public class ItemServiceApplication implements WebMvcConfigurer {
		public static void main(String[] args) {
			SpringApplication.run(ItemServiceApplication.class, args);
		}
		
		@Override
		public Validator getValidator() {
			return new ItemValidator();
		}
	}
	```
	
	- 이렇게 글로벌 설정을 추가할 수 있다. 기존 컨트롤러의 ```@InitBinder``` 를 제거해도 글로벌 설정으로 정상 동작하는 것을 확인 가능
	- 글로벌 설정을 하면 다음에 설명할 ```BeanValidator```가 자동 등록되지 않는다.

- 참고) ```@Validated / @Valid```
	- 검증시 ```@Validated``` / ```@Valid``` 둘다 사용가능하다.
	
	- ```javax.validation.@Valid``` 를 사용하려면 ```build.gradle``` 의존관계 추가가 필요하다.
		- ```implementation 'org.springframework.boot:spring-boot-starter-validation'```
	
	- ```@Validated``` 는 스프링 전용 검증 애노테이션이고, ```@Valid``` 는 자바 표준 검증 애노테이션이다.
	
	- 자세한 내용은 다음 ```Bean Validation```에서 설명
	
