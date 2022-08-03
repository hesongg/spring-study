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
