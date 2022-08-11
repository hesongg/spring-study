### 스프링 웹 MVC

#### Reference) 
	* 스프링 MVC 1편 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/springboot_springmvc_study_1
	
<br>

### 스프링 MVC 웹 페이지

<br>

#### 타임리프 

- 타임리프 사용 선언
	```html
	<html xmlns:th="http://www.thymeleaf.org">
	```

- 타임리프로 css 절대 경로 바라보게 - html에 ```th:href="@{}"``` 부분추가
	- th:href 추가 시 밑에 상대 경로로 지정한 href 부분은 타임리프 태그로 치환되서 사용하지 않게 된다.
	```html
	<link th:href="@{/css/bootstrap.min.css}"
			href="../css/bootstrap.min.css" rel="stylesheet">
	```

- 타임리프로 버튼 onclick 시 리다이렉트 - button 태그에 해당 내용 추가
	```th:onclick="|location.href='@{/basic/items/add}'|"```

- 타임리프로 루프 돌려서 item 꺼내기 - html tr, td 태그 수정
	- ```th:each="item : ${items}"``` 
		- 해당 타임리프 each 문법으로 Model의 items에서 루프를 돌려서 아이템을 꺼내 사용할 수 있다.

	- ```th:href="@{/basic/items/{itemId}(itemId=${item.id})}"```
		- ```th:href``` 태그 : item id 를 클릭했을 때 url 요청을 설정한다.
		- ```"@{}"``` 문법 : url 설정할 때 쓰이는 문법
		- ```{itemId}(itemId=${item.id})``` : 루프 돌리는 값 item 의 id를 itemId 이라는 변수에 할당한다는 의미이다.

	```html
	<tr th:each="item : ${items}">
		<td><a href="item.html" th:href="@{/basic/items/{itemId}(itemId=${item.id})}" th:text="${item.id}">회원id</a></td>
		<td><a href="item.html" th:href="@{/basic/items/{itemId}(itemId=${item.id})}" th:text="${item.itemName}">상품명</a></td>
		<td th:text="${item.price}">10000</td>
		<td th:text="${item.quantity}">10</td>
	</tr>
	```

##### 타임리프 핵심

- 핵심은 ```th:xxx``` 가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체한다. th:xxx 이 없으면 기존 html의 xxx 속성이 그대로 사용된다.

- HTML을 파일로 직접 열었을 때, th:xxx 가 있어도 웹 브라우저는 th: 속성을 알지 못하므로 무시한다.

- 따라서 HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다.


- URL 링크 표현식 - ```@{...}```
	```th:href="@{/css/bootstrap.min.css}"```
	- @{...} : 타임리프는 URL 링크를 사용하는 경우 @{...} 를 사용한다. 이것을 URL 링크 표현식이라 한다. 
	- URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다

- 리터럴 대체 문법 - ```|...|```
	- ```|...|``` :이렇게 사용한다.

	- 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다.
		- ```<span th:text="'Welcome to our application, ' + ${user.name} + '!'">```

	- 다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용할 수 있다.
		- ```<span th:text="|Welcome to our application, ${user.name}!|">```

	- 결과를 다음과 같이 만들어야 하는데
		- ```location.href='/basic/items/add'```

	- 그냥 사용하면 문자와 표현식을 각각 따로 더해서 사용해야 하므로 다음과 같이 복잡해진다.
		- ```th:onclick="'location.href=' + '\'' + @{/basic/items/add} + '\''"```

	- 리터럴 대체 문법을 사용하면 다음과 같이 편리하게 사용할 수 있다.
		- ```th:onclick="|location.href='@{/basic/items/add}'|"```


- 반복 출력 - ```th:each```
	- ```<tr th:each="item : ${items}">```

	- 반복은 ```th:each``` 를 사용한다. 이렇게 하면 모델에 포함된 ```items``` 컬렉션 데이터가 ```item``` 변수에 하나씩 포함되고, 
		반복문 안에서 item 변수를 사용할 수 있다.

	- 컬렉션의 수 만큼 ```<tr>..</tr>``` 이 하위 태그를 포함해서 생성된다


- 변수 표현식 - ```${...}```
	- ```<td th:text="${item.price}">10000</td>```

	- 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다.

	- 프로퍼티 접근법을 사용한다. ( ```item.getPrice()``` )


- 내용 변경 - ```th:text```
	- ```<td th:text="${item.price}">10000</td>```

	- 내용의 값을 ```th:text``` 의 값으로 변경한다.

	- 여기서는 10000을 ```${item.price}``` 의 값으로 변경한다


- 속성 변경 - ```th:value```
	- ```th:value="${item.id}"```
	- 모델에 있는 item 정보를 획득하고 프로퍼티 접근법으로 출력한다. ( ```item.getId``` )
	- ```value``` 속성을 ```th:value``` 속성으로 변경한다.


- URL 링크 표현식2 - ```@{...}```
	- ```th:href="@{/basic/items/{itemId}(itemId=${item.id})}"```

	- 상품 ID를 선택하는 링크를 확인해보자.

	- URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다.

	- 경로 변수( {itemId} ) 뿐만 아니라 쿼리 파라미터도 생성한다.
		- 예) ```th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"```
			- 생성 링크: ```http://localhost:8080/basic/items/1?query=test```


- URL 링크 간단히
	- ```th:href="@{|/basic/items/${item.id}|}"```
	- 상품 이름을 선택하는 링크를 확인해보자.
	- 리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다.


- 참고
	- 타임리프는 순수 HTML 파일을 웹 브라우저에서 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다. 
		JSP를 생각해보면, JSP 파일은 웹 브라우저에서 그냥 열면 JSP 소스코드와 HTML이 뒤죽박죽 되어서 정상적인 확인이 불가능하다. 
		오직 서버를 통해서 JSP를 열어야 한다.
	- 이렇게 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 ```네츄럴 템플릿 (natural templates)```이라 한다.

<br>

- 속성 변경 - ```th:action```
	- ```th:action```
		- HTML form에서 action 에 값이 없으면 현재 URL에 데이터를 전송한다.

	- 상품 등록 폼의 URL과 실제 상품 등록을 처리하는 URL을 똑같이 맞추고 HTTP 메서드로 두 기능을 구분한다.
		- 상품 등록 폼:  ```GET  /basic/items/add```
		- 상품 등록 처리: ```POST /basic/items/add```
		- 이렇게 하면 하나의 URL로 등록 폼과, 등록 처리를 깔끔하게 처리할 수 있다.


<br>

#### @ModelAttribute

##### @ModelAttribute - 요청 파라미터 처리

- ```@ModelAttribute``` 는 ```Item``` 객체를 생성하고, 요청 파라미터의 값을 프로퍼티 접근법(setXxx)으로 입력해준다.

##### @ModelAttribute - Model 추가

- @ModelAttribute 는 중요한 한가지 기능이 더 있는데, 바로 모델(Model)에 @ModelAttribute 로 지정한 객체를 자동으로 넣어준다. 

- 코드에서, ```model.addAttribute("item", item)``` 부분이 주석처리 되어 있어도 잘 동작하는 것을 확인할 수 있다.

- 모델에 데이터를 담을 때는 이름이 필요하다. 이름은 ```@ModelAttribute``` 에 지정한 ```name(value)``` 속성을 사용한다. 

- 만약 다음과 같이 @ModelAttribute 의 이름을 다르게 지정하면 다른 이름으로 모델에	포함된다.
	- ```@ModelAttribute("hello") Item item``` -> 이름을 ```hello``` 로 지정
	- ```model.addAttribute("hello", item);``` -> 모델에 ```hello``` 이름으로 저장
		
<br>

#### 리다이렉트

- 상품 수정은 마지막에 뷰 템플릿을 호출하는 대신에 상품 상세 화면으로 이동하도록 리다이렉트를 호출한다.

- 스프링은 ```redirect:/...``` 으로 편리하게 리다이렉트를 지원한다.
	- ```redirect:/basic/items/{itemId}```
	
- 컨트롤러에 매핑된 ```@PathVariable``` 의 값은 ```redirect``` 에도 사용 할 수 있다.
	- ```redirect:/basic/items/{itemId}``` -> ```{itemId}``` 는 ```@PathVariable Long itemId``` 의 값을 그대로 사용한다.


- 참고
	- ```HTML Form``` 전송은 ```PUT```, ```PATCH```를 지원하지 않는다. ```GET```, ```POST```만 사용할 수 있다.
		- ```PUT```, ```PATCH```는 ```HTTP API``` 전송시에 사용
		
	- 스프링에서 HTTP POST로 Form 요청할 때 히든 필드를 통해서 PUT, PATCH 매핑을 사용하는 방법이 있지만, HTTP 요청상 POST 요청이다.
	
<br>

#### PRG Post/Redirect/Get 

- 웹 브라우저의 새로 고침은 마지막에 서버에 전송한 데이터를 다시 전송한다. -> ```PRG``` 패턴으로 해결 가능

- 데이터 등록 처리 이후에 뷰 템플릿이 아니라 상품 상세 화면으로 리다이렉트 하도록 코드를 작성하자.
	- 이런 문제 해결 방식을 ```PRG Post/Redirect/Get``` 라 한다.
	
- 주의
	- ```"redirect:/basic/items/" + item.getId()``` 
		- redirect에서 ```+item.getId()``` 처럼 URL에 변수를 더해서 사용하는 것은 URL 인코딩이 안되기 때문에 위험하다. 
		- ```RedirectAttributes``` 를 사용하자.

<br>

#### RedirectAttributes

- ```RedirectAttributes``` 를 사용하면 ```URL 인코딩```도 해주고, ```pathVarible``` , ```쿼리 파라미터```까지 처리해준다.
	- 컨트롤러에 추가한 자바 코드
		```java
		@PostMapping("/add")
		public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
			Item savedItem = itemRepository.save(item);
			
			redirectAttributes.addAttribute("itemId", savedItem.getId());
			redirectAttributes.addAttribute("status", true);
			
			return "redirect:/basic/items/{itemId}";
		}
		```
		- 실행해보면 다음과 같은 리다이렉트 결과가 나온다. : ```http://localhost:8080/basic/items/3?status=true```
	
		- ```redirect:/basic/items/{itemId}```
		- ```pathVariable``` 바인딩: ```{itemId}```
		- 나머지는 쿼리 파라미터로 처리: ```?status=true```
		
	- 뷰 템플릿에 추가한 코드 - 메시지 추가
		```html
		<h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>
		```
		
		- ```th:if``` : 해당 조건이 참이면 실행
		- ```${param.status}``` : 타임리프에서 쿼리 파라미터를 편리하게 조회하는 기능
			- 원래는 컨트롤러에서 모델에 직접 담고 값을 꺼내야 한다. 그런데 쿼리 파라미터는 자주 사용해서 타임리프에서 직접 지원한다.
