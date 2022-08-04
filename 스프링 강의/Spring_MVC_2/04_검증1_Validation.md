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
