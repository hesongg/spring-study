### 스프링 웹 MVC - 백엔드 웹 개발 활용 기술

#### Reference) 
	* 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/SpringMVC-Practice-Validation1
	
<br>


### 검증2 - Bean Validation

<br>

#### Bean Validation - 소개

- 밑과 같은 검증 로직을 모든 프로젝트에 적용할 수 있게 공통화하고, 표준화 한 것이 바로 ```Bean Validation``` 이다.
	```java
	public class Item {
	
		private Long id;
		
		@NotBlank
		private String itemName;
		
		@NotNull
		@Range(min = 1000, max = 1000000)
		private Integer price;
		...
	}
	```
	- ```Bean Validation```을 잘 활용하면, 애노테이션 하나로 검증 로직을 매우 편리하게 적용할 수 있다.

- Bean Validation 이란?
	- ```Bean Validation```은 특정한 구현체가 아니라 ```Bean Validation 2.0(JSR-380)```이라는 기술 표준이다.
	- 검증 애노테이션과 여러 인터페이스의 모음
	
	- ```Bean Validation```을 구현한 기술중에 일반적으로 사용하는 구현체는 ```하이버네이트 Validator```이다. 
		- 이름이 하이버네이트가 붙어서 그렇지 ORM과는 관련이 없음.

	- 하이버네이트 Validator 관련 링크
		- 공식 사이트: http://hibernate.org/validator/
		- 공식 메뉴얼: https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/
		- 검증 애노테이션 모음: https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec
	
<br>

#### Bean Validation - 시작

- 먼저 스프링과 통합하지 않고, 순수한 Bean Validation 사용법 부터 테스트 코드로 알아보자.

- 의존관계 추가
	- ```Bean Validation```을 사용하려면 다음 의존관계를 추가해야 함
		- ```build.gradle``` - ```implementation 'org.springframework.boot:spring-boot-starter-validation'```
		
		- ```spring-boot-starter-validation``` 의존관계를 추가하면 라이브러리가 추가 된다.
			- ```Jakarta Bean Validation```
				- ```jakarta.validation-api``` : Bean Validation 인터페이스
				- ```hibernate-validator``` : 구현체
	
<br>

- 검증 애노테이션
	- ```@NotBlank``` : 빈값 + 공백만 있는 경우를 허용하지 않는다.
	- ```@NotNull``` : null 을 허용하지 않는다.
	- ```@Range(min = 1000, max = 1000000)``` : 범위 안의 값이어야 한다.
	- ```@Max(9999)``` : 최대 9999까지만 허용한다.

- 참고) 
	- ```javax.validation.constraints.NotNull```
	- ```org.hibernate.validator.constraints.Range```
	
	- ```javax.validation``` 으로 시작하면 특정 구현에 관계없이 제공되는 표준 인터페이스
	- ```org.hibernate.validator``` 로 시작하면 하이버네이트 validator 구현체를 사용할 때만 제공되는 검증 기능
	
	-  실무에서 대부분 하이버네이트 validator를 사용하므로 자유롭게 사용해도 된다.
	
- 검증기 생성
	- 다음 코드와 같이 검증기를 생성한다. 이후 스프링과 통합하면 우리가 직접 이런 코드를 작성하지는 않으므로, 이렇게 사용하는구나 정도만 참고하자.
		```java
		ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
		Validator validator = factory.getValidator();
		```
- 검증 실행
	- 검증 대상( ```item``` )을 직접 검증기에 넣고 그 결과를 받는다. 
	
	- ```Set``` 에는 ```ConstraintViolation``` 이라는 검증 오류가 담긴다. 따라서 결과가 비어있으면 검증 오류가 없는 것이다.
		
	```java
	Set<ConstraintViolation<Item>> violations = validator.validate(item);
	```
	
	- ```ConstraintViolation``` 출력 결과를 보면, 검증 오류가 발생한 객체, 필드, 메시지 정보등 다양한 정보를 확인할 수 있다.
	
- 정리
	- 빈 검증기(```Bean Validation```)를 직접 사용하는 방법을 알아보았다. 
	- 스프링은 이미 개발자를 위해 빈 검증기를 스프링에 완전히 통합해두었다.
	
<br>


#### 스프링 MVC는 어떻게 Bean Validator를 사용하는가?

- 스프링 부트가 ```spring-boot-starter-validation``` 라이브러리를 넣으면 자동으로 ```Bean Validator```를 인지하고 스프링에 통합한다.

- 스프링 부트는 자동으로 ```글로벌 Validator```로 등록한다.
	- ```LocalValidatorFactoryBean``` 을 글로벌 Validator로 등록한다. 
	- 이 Validator는 ```@NotNull``` 같은 애노테이션을 보고 검증을 수행한다. 
	- 이렇게 글로벌 Validator가 적용되어 있기 때문에 ```@Valid``` 또는 ```@Validated``` 만 적용하면 된다.
	- 검증 오류가 발생하면 ```FieldError``` / ```ObjectError``` 를 생성해서 ```BindingResult``` 에 담아준다.
	
<br>

- 주의)
	- 다음과 같이 직접 글로벌 Validator를 직접 등록하면 스프링 부트는 Bean Validator를 글로벌 Validator 로 등록하지 않는다. 
	- 따라서 애노테이션 기반의 빈 검증기가 동작하지 않는다. 해당 부분은 제거하자.
		```java
		@SpringBootApplication
		public class ItemServiceApplication implements WebMvcConfigurer {
			// 글로벌 검증기 추가
			@Override
			public Validator getValidator() {
				return new ItemValidator();
			}
			// ...
		}
		```

<br>

#### 검증 순서

- 1. ```@ModelAttribute``` 각각의 필드에 타입 변환 시도
	- 1) 성공하면 다음으로
	- 2) 실패하면 ```typeMismatch``` 로 ```FieldError``` 추가
	
- 2. ```Validator``` 적용

- 바인딩에 성공한 필드만 ```Bean Validation``` 적용
	- BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않는다.
	- 타입 변환에 성공해서 바인딩에 성공한 필드여야 BeanValidation 적용이 의미 있다.
		- 모델 객체에 바인딩 받는 값이 정상으로 들어와야 검증도 의미가 있다.
	
	- ```@ModelAttribute``` -> 각각의 필드 타입 변환시도 -> 변환에 성공한 필드만 BeanValidation 적용
	
<br>

#### Bean Validation - 에러코드

- Bean Validation이 기본으로 제공하는 오류 메시지를 좀 더 자세히 변경하고 싶으면 어떻게 하면 될까?

- Bean Validation을 적용하고 ```bindingResult``` 에 등록된 검증 오류 코드를 보자.
	- 오류 코드가 애노테이션 이름으로 등록된다. 마치 ```typeMismatch``` 와 유사하다.

<br>

- NotBlank 라는 오류 코드를 기반으로 MessageCodesResolver 를 통해 다양한 메시지 코드가 순서대로 생성 됨

	- ```@NotBlank```
		```
		NotBlank.item.itemName
		NotBlank.itemName
		NotBlank.java.lang.String
		NotBlank
		```
	
	- ```@Range```
		```
		Range.item.price
		Range.price
		Range.java.lang.Integer
		Range
		```
		
- 에러 코드 등록 예시 
	- ```errors.properties```
		```properties
		#Bean Validation 추가
		NotBlank={0} 공백X
		Range={0}, {2} ~ {1} 허용
		Max={0}, 최대 {1}
		```
		
		- ```{0}``` 은 필드명이고, ```{1}```, ```{2}``` ...은 각 애노테이션 마다 다르다.
		
- BeanValidation 메시지 찾는 순서
	- 1. 생성된 메시지 코드 순서대로 ```messageSource``` 에서 메시지 찾기 (설정한 메시지 프로퍼티)
	- 2. 애노테이션의 ```message``` 속성 사용 ```@NotBlank(message = "공백! {0}")```
	- 3. 라이브러리가 제공하는 기본 값 사용 -> 공백일 수 없습니다.
	
<br>

#### Bean Validation - 오브젝트 오류

- Bean Validation에서 특정 필드( ```FieldError``` )가 아닌 해당 오브젝트 관련 오류( ```ObjectError``` )는 어떻게 처리할 수 있을까?
	- 다음과 같이 ```@ScriptAssert()``` 를 사용하면 된다.
		```java
		@Data
		@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000")
		public class Item {
			//...
		}
		```
		
		- 메시지 코드는 다음과 같이 생성된다.
			- ```ScriptAssert.item```
			- ```ScriptAssert```
			
	- 그런데 실제 사용해보면 제약이 많고 복잡하다. 그리고 실무에서는 검증 기능이 해당 객체의 범위를 넘어서는 경우들도 종종 등장하는데, 그런 경우 대응이 어렵다.
	
	- 따라서 오브젝트 오류(글로벌 오류)의 경우 ```@ScriptAssert``` 을 억지로 사용하는 것 보다는 오브젝트 오류 관련 부분만 직접 자바 코드로 작성하는 것을 권장한다.
		- 글로벌 오류일 경우, 메서드에서 파라미터로 설정한 ```bindingResult```의 ```reject()``` 사용 권장
			```java
			bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
			```

<br>

#### Bean Validation - 한계

- 예시) 데이터를 등록할 때와 수정할 때는 요구사항이 다를 수 있다.
	- 수정하는 경우에서, 새로 생긴 요구사항에 따라 어노테이션을 변경할 경우.. 등록에서 문제가 발생
	- 결과적으로 등록과 수정에서 검증 조건의 충돌이 발생하고, 
		등록과 수정은 같은 BeanValidation 을 적용할 수 없다. 이 문제를 어떻게 해결할 수 있을까?

<br>

#### Bean Validation - groups

- 동일한 모델 객체를 등록할 때와 수정할 때 각각 다르게 검증하는 방법이 있다.

- 방법 2가지
	- BeanValidation의 ```groups``` 기능을 사용한다.
	- Item을 직접 사용하지 않고, ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어서 사용한다.


- BeanValidation groups 기능 사용
	- 이런 문제를 해결하기 위해 Bean Validation은 groups라는 기능을 제공한다.
	- 예를 들어서 등록시에 검증할 기능과 수정시에 검증할 기능을 각각 그룹으로 나누어 적용할 수 있다.
	
	- ```SaveCheck```, ```UpdateCheck``` 각각 2개의 인터페이스를 만든다.
	
	- 도메인 객체의 검증 어노테이션에 ```groups``` 옵션을 적용한다.
		```java
		...
		@NotNull(groups = UpdateCheck.class) //수정시에만 적용
		private Long id;
		
		@NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
		private String itemName;
		...
		```
	
	- 컨트롤러의 저장 로직 및 수정 로직의 ```@Validated``` 어노테이션의 ```value``` 옵션을 설정한다.
		- 저장 로직
			```java
			@PostMapping("/add")
			public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
				//...
			}
			```
		- 수정 로직
			```java
			@PostMapping("/{itemId}/edit")
			public String editV2(@PathVariable Long itemId, @Validated(UpdateCheck.class) @ModelAttribute Item item, BindingResult bindingResult) {
				//...
			}
			```
	
	- 참고) ```@Valid``` 에는 ```groups```를 적용할 수 있는 기능이 없다. 
		- 따라서 ```groups```를 사용하려면 ```@Validated``` 를 사용해야 한다.
		
- 정리
	- groups 기능을 사용해서 등록과 수정시에 각각 다르게 검증을 할 수 있었다. 
	- 그런데 groups 기능을 사용하니 ```Item``` 은 물론이고, 전반적으로 복잡도가 올라갔다.
	- 사실 groups 기능은 실제 잘 사용되지는 않는데, 그 이유는 실무에서는 주로 다음에 등장하는 등록용 폼 객체와 수정용 폼 객체를 분리해서 사용하기 때문이다.
	
<br>


#### Form 전송 객체 분리 - 소개

- 실무에서는 ```groups``` 를 잘 사용하지 않는다.
	- 바로 등록시 폼에서 전달하는 데이터가 ```Item``` 도메인 객체와 딱 맞지 않기 때문이다.
	- 실무에서는 회원 등록시 회원과 관련된 데이터만 전달받는 것이 아니라, 약관 정보도 추가로 받는 등 
		```Item```(도메인 객체) 과 관계없는 수 많은 부가 데이터가 넘어온다
	
	- 그래서 보통 ```Item``` 을 직접 전달받는 것이 아니라, 복잡한 폼의 데이터를 컨트롤러까지 전달할 별도의 객체를 만들어서 전달한다. 
	- 예를 들면 ```ItemSaveForm``` 이라는 폼을 전달받는 전용 객체를 만들어서 ```@ModelAttribute``` 로 사용한다. 
	- 이것을 통해 컨트롤러에서 폼 데이터를 전달 받고, 이후 컨트롤러에서 필요한 데이터를 사용해서 ```Item``` 을 생성한다.

<br>
	
- 폼 데이터 전달에 Item 도메인 객체 사용
	- ```HTML Form -> Item -> Controller -> Item -> Repository```
		- 장점: Item 도메인 객체를 컨트롤러, 리포지토리 까지 직접 전달해서 중간에 Item을 만드는 과정이 없어서 간단하다.
		- 단점: 간단한 경우에만 적용할 수 있다. 수정시 검증이 중복될 수 있고, groups를 사용해야 한다.


- 폼 데이터 전달을 위한 별도의 객체 사용
	- ```HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository```
		- 장점: 전송하는 폼 데이터가 복잡해도 거기에 맞춘 별도의 폼 객체를 사용해서 데이터를 전달 받을 수 있다. 
			보통 등록과, 수정용으로 별도의 폼 객체를 만들기 때문에 검증이 중복되지 않는다.
		- 단점: 폼 데이터를 기반으로 컨트롤러에서 Item 객체를 생성하는 변환 과정이 추가된다.

	- 웬만하면 이 방식을 사용할 것..

<br>


#### Form 전송 객체 분리 - 개발

- 컨트롤러 수정
	- ```Item``` 대신에 ```ItemSaveform``` 을 전달 받는다. 
	- 그리고 ```@Validated``` 로 검증도 수행하고, ```BindingResult``` 로 검증 결과도 받는다.


	- 주의
		- ```@ModelAttribute("item")``` 에 ```item``` 이름을 넣어준 부분을 주의하자. 
		- 이것을 넣지 않으면 ```ItemSaveForm``` 의 경우 규칙에 의해 ```itemSaveForm``` 이라는 이름으로 MVC Model에 담기게 된다.
		- 이렇게 되면 뷰 템플릿에서 접근하는 ```th:object``` 이름도 함께 변경해주어야 한다
		
	- 폼 객체의 데이터를 기반으로 저장 또는 수정할 Item 객체를 생성하자. 
		- 이렇게 폼 객체 처럼 중간에 다른 객체가 추가되면 변환하는 과정이 추가 됨
		
- Form 전송 객체 분리를 통해 등록과 수정에 딱 맞는 기능을 구성하고, 검증도 명확히 분리 가능하다.

<br>

#### Bean Validation - HTTP 메시지 컨버터

- ```@Valid``` , ```@Validated``` 는 ```HttpMessageConverter``` ( ```@RequestBody``` )에도 적용할 수 있다.

- 참고
	- ```@ModelAttribute``` 는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰 때 사용한다.
	- ```@RequestBody``` 는 HTTP Body의 데이터를 객체로 변환할 때 사용한다. 
		- 주로 API JSON 요청을 다룰 때 사용한다.
	
<br>

- API의 경우 3가지 경우를 나누어 생각해야 한다.
	- 성공 요청 : 성공
	- 실패 요청 : JSON을 객체로 생성하는 것 자체가 실패함
	- 검증 오류 요청 : JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패함
	
- API 실패 요청 테스트 - 객체 타입에 맞지 않는 데이터로 요청 시
	- ```HttpMessageConverter``` 에서 요청 JSON을 ```ItemSaveForm``` 객체로 생성하는데 실패한다.
	- 이 경우는 ```ItemSaveForm``` 객체를 만들지 못하기 때문에 컨트롤러 자체가 호출되지 않고 그 전에 예외가 발생한다. 
	- 물론 ```Validator```도 실행되지 않는다. 
	
- API 검증 오류 요청 테스트 - ```HttpMessageConverter``` 는 성공하지만 검증(Validator)에서 오류가 발생하는 경우를 확인
	- ```return bindingResult.getAllErrors();``` 는 ```ObjectError``` 와 ```FieldError``` 를 반환한다.
	
	- 스프링이 이 객체를 JSON으로 변환해서 클라이언트에 전달한다. 
		- 이런 JSON 데이터로 반환 됨
		```
		[
			{
				"codes": [
					"Max.itemSaveForm.quantity",
					"Max.quantity",
					"Max.java.lang.Integer",
					"Max"
				],
				"arguments": [
					{
						"codes": [
							"itemSaveForm.quantity",
							"quantity"
						],
						"arguments": null,
						"defaultMessage": "quantity",
						"code": "quantity"
					},
					9999
				],
				"defaultMessage": "9999 이하여야 합니다",
				"objectName": "itemSaveForm",
				"field": "quantity",
				"rejectedValue": 10000,
				"bindingFailure": false,
				"code": "Max"
			}
		]
		```
	
	- 여기서는 예시로 보여주기 위해서 검증 오류 객체들을 그대로 반환했다. 
		- 실제 개발할 때는 이 객체들을 그대로 사용하지 말고, 필요한 데이터만 뽑아서 별도의 API 스펙을 정의하고 그에 맞는 객체를 만들어서 반환해야 한다.


<br>

##### @ModelAttribute vs @RequestBody

- HTTP 요청 파리미터를 처리하는 ```@ModelAttribute``` 는 각각의 필드 단위로 세밀하게 적용된다. 
	그래서 특정 필드에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리할 수 있었다.
	
- ```@HttpMessageConverter``` 는 ```@@ModelAttribute``` 와 다르게 각각의 필드 단위로 적용되는 것이 아니라, 전체 객체 단위로 적용된다.
	- 따라서 메시지 컨버터의 작동이 성공해서 ```ItemSaveForm``` 객체를 만들어야 ```@Valid``` , ```@Validated``` 가 적용된다.

- ```@ModelAttribute``` 는 필드 단위로 정교하게 바인딩이 적용된다.
	- 특정 필드가 바인딩 되지 않아도 나머지 필드는 정상 바인딩 되고, Validator를 사용한 검증도 적용할 수 있다.

- ```@RequestBody``` 는 ```HttpMessageConverter``` 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후 단계 자체가 진행되지 않고 예외가 발생한다. 
	- 컨트롤러도 호출되지 않고, Validator도 적용할 수 없다.

- 참고)
	- ```HttpMessageConverter``` 단계에서 실패하면 예외가 발생한다. 
	- 예외 발생시 원하는 모양으로 예외를 처리하는 방법은 예외 처리 부분 참고
