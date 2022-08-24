### 스프링 DB - 데이터 접근 활용 기술

#### Reference) 
	* 스프링 DB 2편 - 데이터 접근 활용 기술 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/Spring-DB-Itemservice
	
<br>


### 데이터 접근 기술 - 스프링 JdbcTemplate


<br>

#### JdbcTemplate 소개와 설정

- 장점
	- 설정의 편리함
		- JdbcTemplate은 ```spring-jdbc``` 라이브러리에 포함되어 있는데, 
			이 라이브러리는 스프링으로 JDBC를 사용할 때 기본으로 사용되는 라이브러리이다. 
		- 그리고 별도의 복잡한 설정 없이 바로 사용할 수 있다.
	
	- 반복 문제 해결
		- JdbcTemplate은 템플릿 콜백 패턴을 사용해서, JDBC를 직접 사용할 때 발생하는 대부분의 반복 작업을 대신 처리해준다.
		- 개발자는 SQL을 작성하고, 전달할 파리미터를 정의하고, 응답 값을 매핑하기만 하면 된다.
		- 우리가 생각할 수 있는 대부분의 반복 작업을 대신 처리해준다.
			- 커넥션 획득
			- ```statement``` 를 준비하고 실행
			- 결과를 반복하도록 루프를 실행
			- 커넥션 종료, ```statement``` , ```resultset``` 종료
			- 트랜잭션 다루기 위한 커넥션 동기화
			- 예외 발생시 스프링 예외 변환기 실행

- 단점
	- 동적 SQL을 해결하기 어렵다.
	
<br>

- JdbcTemplate 설정
	- ```build.gradle```
		```
		//JdbcTemplate 추가
		implementation 'org.springframework.boot:spring-boot-starter-jdbc'
		//H2 데이터베이스 추가
		runtimeOnly 'com.h2database:h2'
		```
	
		- ```org.springframework.boot:spring-boot-starter-jdbc``` 를 추가하면 JdbcTemplate이 들어있는
			```spring-jdbc``` 가 라이브러리에 포함된다.
		
		- 여기서는 H2 데이터베이스에 접속해야 하기 때문에 H2 데이터베이스의 클라이언트 라이브러리(Jdbc Driver)도 추가하자.
			- ```runtimeOnly 'com.h2database:h2'```

<br>

#### JdbcTemplate 적용1 - 기본

- ```JdbcTemplate```을 사용해서 메모리에 저장하던 데이터를 데이터베이스에 저장

- 기본
	- ```JdbcTemplateItemRepositoryV1``` 은 ```ItemRepository``` 인터페이스를 구현
	
	- ```this.template = new JdbcTemplate(dataSource)```
		- ```JdbcTemplate``` 은 데이터소스( ```dataSource``` )가 필요하다.
		- ```JdbcTemplateItemRepositoryV1()``` 생성자를 보면 ```dataSource``` 를 의존 관계 주입 받고 생성자 내부에서 ```JdbcTemplate``` 을 생성한다. 
			- 스프링에서는 ```JdbcTemplate``` 을 사용할 때 관례상 이 방법을 많이 사용한다.
		- 물론 ```JdbcTemplate``` 을 스프링 빈으로 직접 등록하고 주입받아도 된다

<br>

- ```save()```
	- 데이터를 저장한다.
	
	- ```template.update()``` : 데이터를 변경할 때는 ```update()``` 를 사용하면 된다.
		- ```INSERT``` , ```UPDATE``` , ```DELETE``` SQL에 사용한다.
		- ```template.update()``` 의 반환 값은 ```int``` 인데, 영향 받은 로우 수를 반환한다.
		
	- 데이터를 저장할 때 PK 생성에 ```identity``` (auto increment) 방식을 사용하기 때문에, 
		PK인 ID 값을 개발자가 직접 지정하는 것이 아니라 비워두고 저장해야 한다. 
		- 그러면 데이터베이스가 PK인 ID를 대신 생성해준다.
		
	- 문제는 이렇게 데이터베이스가 대신 생성해주는 PK ID 값은 데이터베이스가 생성하기 때문에,
		데이터베이스에 INSERT가 완료 되어야 생성된 PK ID 값을 확인할 수 있다.
		
	- ```KeyHolder``` 와 ```connection.prepareStatement(sql, new String[]{"id"})``` 를 사용해서 id 를
		지정해주면 INSERT 쿼리 실행 이후에 데이터베이스에서 생성된 ID 값을 조회할 수 있다.
	
	- 물론 데이터베이스에서 생성된 ID 값을 조회하는 것은 순수 JDBC로도 가능하지만, 코드가 훨씬 더	복잡하다.
	
	- 참고로 뒤에서 설명하겠지만 ```JdbcTemplate```이 제공하는 ```SimpleJdbcInsert``` 라는 훨씬 편리한 기능이
		있으므로 대략 이렇게 사용한다 정도로만 알아두면 된다.
		
<br>

- ```update()```
	- 데이터를 업데이트 한다.
	- ```template.update()``` : 데이터를 변경할 때는 ```update()``` 를 사용하면 된다.
		- ```?``` 에 바인딩할 파라미터를 순서대로 전달하면 된다.
		- 반환 값은 해당 쿼리의 영향을 받은 로우 수 이다. 여기서는 ```where id=?``` 를 지정했기 때문에 영향 받은 로우수는 최대 1개이다.

<br>

- ```findById()```
	- 데이터를 하나 조회한다.
	- ```template.queryForObject()```
		- 결과 로우가 하나일 때 사용한다.
		- ```RowMapper``` 는 데이터베이스의 반환 결과인 ```ResultSet``` 을 객체로 변환한다.
		- 결과가 없으면 ```EmptyResultDataAccessException``` 예외가 발생한다.
		- 결과가 둘 이상이면 ```IncorrectResultSizeDataAccessException``` 예외가 발생한다.
	
	- ```ItemRepository.findById()``` 인터페이스는 결과가 없을 때 ```Optional``` 을 반환해야 한다. 
		- 따라서 결과가 없으면 예외를 잡아서 ```Optional.empty``` 를 대신 반환하면 된다.

<br>

- ```findAll()```
	- 데이터를 리스트로 조회한다. 그리고 검색 조건으로 적절한 데이터를 찾는다.
	- ```template.query()```
		- 결과가 하나 이상일 때 사용한다.
		- ```RowMapper``` 는 데이터베이스의 반환 결과인 ```ResultSet``` 을 객체로 변환한다.
		- 결과가 없으면 빈 컬렉션을 반환한다.
		- 동적 쿼리에 대한 부분은 바로 다음에 다룬다.

<br>

- ```itemRowMapper()```
	- 데이터베이스의 조회 결과를 객체로 변환할 때 사용
	- JDBC를 직접 사용할 때 ```ResultSet``` 를 사용했던 부분을 떠올리면 된다.
	- 차이가 있다면 다음과 같이 ```JdbcTemplate```이 다음과 같은 루프를 돌려주고,
		개발자는 ```RowMapper``` 를 구현해서 그 내부 코드만 채운다고 이해하면 된다.
		```java
		while(resultSet 이 끝날 때 까지) {
			rowMapper(rs, rowNum)
		}
		```
		
<br>


#### JdbcTemplate 적용2 - 동적 쿼리 문제

- 결과를 검색하는 ```findAll()``` 에서 어려운 부분은 사용자가 검색하는 값에 따라서 실행하는 SQL이 동적으로 달려져야 한다는 점

- 결과적으로 4가지 상황에 따른 SQL을 동적으로 생성해야 한다. 동적 쿼리가 언듯 보면 쉬워 보이지만, 
	막상 개발해보면 생각보다 다양한 상황을 고민해야 한다. 
	- 예를 들어서 어떤 경우에는 ```where``` 를 앞에 넣고 어떤 경우에는 ```and``` 를 넣어야 하는지 등을 모두 계산해야 한다.
	- 그리고 각 상황에 맞추어 파라미터도 생성해야 한다.
	- 물론 실무에서는 이보다 훨씬 더 복잡한 동적 쿼리들이 사용된다.
	- 참고로 이후에 설명할 MyBatis의 가장 큰 장점은 SQL을 직접 사용할 때 동적 쿼리를 쉽게 작성할 수 있다는 점이있다.
	
<br>

#### JdbcTemplate 적용3 - 구성과 실행

- 설정 파일 추가 및 ```application.properties``` 파일에 ```datasource``` 정보 추가
	- 스프링 부트가 해당 설정을 사용해서 커넥션 풀과 ```DataSource``` , 트랜잭션 매니저를 스프링 빈으로 자동 등록한다. 
	

<br>

#### JdbcTemplate - 이름 지정 파라미터 1

- 순서대로 바인딩
	- JdbcTemplate을 기본으로 사용하면 파라미터를 순서대로 바인딩 한다.

	- 파라미터를 순서대로 바인딩 하는 것은 편리하기는 하지만, 순서가 맞지 않아서 버그가 발생할 수도 있으므로 주의해서 사용해야 한다.

<br>

- 이름 지정 바인딩
	- JdbcTemplate은 이런 문제를 보완하기 위해 ```NamedParameterJdbcTemplate``` 라는 이름을 지정해서 파라미터를 바인딩 하는 기능을 제공

	- 기본
		- ```this.template = new NamedParameterJdbcTemplate(dataSource)```
			- ```NamedParameterJdbcTemplate``` 도 내부에 ```dataSource``` 가 필요하다.
			- ```JdbcTemplateItemRepositoryV2``` 생성자를 보면 의존관계 주입은 ```dataSource``` 를 받고 
				내부에서 ```NamedParameterJdbcTemplate``` 을 생성해서 가지고 있다. 
				- 스프링에서는 JdbcTemplate 관련	기능을 사용할 때 관례상 이 방법을 많이 사용한다.
			- 물론 ```NamedParameterJdbcTemplate``` 을 스프링 빈으로 직접 등록하고 주입받아도 된다.
				
	- ```save()```
		- SQL에서 다음과 같이 ```?``` 대신에 ```:파라미터이름``` 을 받는 것을 확인할 수 있다.
			```java
			insert into item (item_name, price, quantity) " +
				"values (:itemName, :price, :quantity)"
			```
	- 추가로 ```NamedParameterJdbcTemplate``` 은 데이터베이스가 생성해주는 키를 매우 쉽게 조회하는 기능도 제공해준다.

	- 자세한건 커밋한 소스 참고

<br>

#### JdbcTemplate - 이름 지정 파라미터 2

- 이름 지정 파라미터
	- 파라미터를 전달하려면 ```Map``` 처럼 ```key``` , ```value``` 데이터 구조를 만들어서 전달해야 한다.
	- 여기서 ```key``` 는 ```:파리이터이름``` 으로 지정한, 파라미터의 이름이고 , ```value``` 는 해당 파라미터의 값이 된다.

	- 다음 코드를 보면 이렇게 만든 파라미터( param )를 전달하는 것을 확인할 수 있다.
		```java
		template.update(sql, param, keyHolder);
		```
	
	- 이름 지정 바인딩에서 자주 사용하는 파라미터의 종류는 크게 3가지가 있다.
		- ```Map```
		- ```SqlParameterSource```
			- ```MapSqlParameterSource```
			- ```BeanPropertySqlParameterSource```
	
<br>

- 1. ```Map```
	- 단순히 ```Map``` 을 사용한다.
	
	- ```findById()``` 코드에서 확인할 수 있다.
		```java
		Map<String, Object> param = Map.of("id", id);
		Item item = template.queryForObject(sql, param, itemRowMapper());
		```

<br>

- 2. ```MapSqlParameterSource```
	- ```Map``` 과 유사한데, SQL 타입을 지정할 수 있는 등 SQL에 좀 더 특화된 기능을 제공한다.
	- ```SqlParameterSource``` 인터페이스의 구현체이다.
	- ```MapSqlParameterSource``` 는 메서드 체인을 통해 편리한 사용법도 제공한다.
	
	- ```update()``` 코드에서 확인할 수 있다.
		```java
		SqlParameterSource param = new MapSqlParameterSource()
			.addValue("itemName", updateParam.getItemName())
			.addValue("price", updateParam.getPrice())
			.addValue("quantity", updateParam.getQuantity())
			.addValue("id", itemId); //이 부분이 별도로 필요하다.
		template.update(sql, param);
		```

<br>

- 3. ```BeanPropertySqlParameterSource```
	- 자바빈 프로퍼티 규약을 통해서 자동으로 파라미터 객체를 생성한다.
	- 예) ( ```getXxx() -> xxx, getItemName() -> itemName``` )
	- 예를 들어서 ```getItemName()``` , ```getPrice()``` 가 있으면 다음과 같은 데이터를 자동으로 만들어낸다.
		- ```key=itemName, value=상품명 값```
		- ```key=price, value=가격 값```

	- ```SqlParameterSource``` 인터페이스의 구현체이다.
	
	- ```save()``` , ```findAll()``` 코드에서 확인할 수 있다.
		```java
		SqlParameterSource param = new BeanPropertySqlParameterSource(item);
		KeyHolder keyHolder = new GeneratedKeyHolder();
		template.update(sql, param, keyHolder);
		```
	
	- 여기서 보면 ```BeanPropertySqlParameterSource``` 가 많은 것을 자동화 해주기 때문에 가장 좋아보이지만,
		```BeanPropertySqlParameterSource``` 를 항상 사용할 수 있는 것은 아니다.
	
	- 예를 들어서 ```update()``` 에서는 SQL에 ```:id``` 를 바인딩 해야 하는데, ```update()``` 에서 사용하는
		```ItemUpdateDto``` 에는 ```itemId``` 가 없다. 
		- 따라서 ```BeanPropertySqlParameterSource``` 를 사용할 수 없고, 대신에 ```MapSqlParameterSource``` 를 사용했다.

<br>

- ```BeanPropertyRowMapper```
	- 코드에서 ```V1``` 과 비교해서 변화된 부분이 하나 더 있다. 바로 ```BeanPropertyRowMapper``` 를 사용한 것이다.
		```java
		private RowMapper<Item> itemRowMapper() {
			return BeanPropertyRowMapper.newInstance(Item.class); //camel 변환 지원
		}
		```
	
	- ```BeanPropertyRowMapper``` 는 ```ResultSet``` 의 결과를 받아서 자바빈 규약에 맞추어 데이터를 변환
		- 데이터베이스에서 조회한 결과 이름을 기반으로 ```setId()``` , ```setPrice()``` 처럼 자바빈 프로퍼티 규약에 맞춘 메서드를 호출하는 것이다.

	- ```BeanPropertyRowMapper``` 는 언더스코어 표기법을 카멜로 자동 변환해준다.
		- 따라서 ```select item_name``` 으로 조회해도 ```setItemName()``` 에 문제 없이 값이 들어간다.
	
		- 정리하면 ```snake_case``` 는 자동으로 해결되니 그냥 두면 되고, 컬럼 이름과 객체 이름이 완전히 다른 경우에는 조회 SQL에서 별칭을 사용하면 된다.

<br>

#### JdbcTemplate - SimpleJdbcInsert

- ```JdbcTemplate```은 INSERT SQL를 직접 작성하지 않아도 되도록 ```SimpleJdbcInsert``` 라는 편리한 기능을 제공

- 기본
	- ```this.jdbcInsert = new SimpleJdbcInsert(dataSource)``` : 생성자를 보면 의존관계 주입은
		```dataSource``` 를 받고 내부에서 ```SimpleJdbcInsert``` 을 생성해서 가지고 있다
	
- ```SimpleJdbcInsert```
	```java
	this.jdbcInsert = new SimpleJdbcInsert(dataSource)
			.withTableName("item")
			.usingGeneratedKeyColumns("id");
		 // .usingColumns("item_name", "price", "quantity"); //생략 가능
	```
	
	- ```withTableName``` : 데이터를 저장할 테이블 명을 지정한다.
	- ```usingGeneratedKeyColumns``` : ```key``` 를 생성하는 PK 컬럼 명을 지정한다.
	- ```usingColumns``` : INSERT SQL에 사용할 컬럼을 지정한다. 특정 값만 저장하고 싶을 때 사용한다. 생략할 수 있다.

<br>

- ```SimpleJdbcInsert``` 는 생성 시점에 데이터베이스 테이블의 메타 데이터를 조회한다. 
	- 따라서 어떤 컬럼이 있는지 확인 할 수 있으므로 ```usingColumns``` 을 생략할 수 있다. 
	- 만약 특정 컬럼만 지정해서 저장하고 싶다면 ```usingColumns``` 를 사용하면 된다.

<br>

- ```save()```
	- ```jdbcInsert.executeAndReturnKey(param``` 을 사용해서 INSERT SQL을 실행하고, 생성된 키 값도 매우 편리하게 조회할 수 있다.

	```java
	public Item save(Item item) {
		SqlParameterSource param = new BeanPropertySqlParameterSource(item);
		Number key = jdbcInsert.executeAndReturnKey(param);
		item.setId(key.longValue());
		
		return item;
	}
	```

<br>

#### JdbcTemplate 기능 정리

- 주요 기능
	- JdbcTemplate이 제공하는 주요 기능은 다음과 같다.
	
	- ```JdbcTemplate```
		- 순서 기반 파라미터 바인딩을 지원한다.
		
	- ```NamedParameterJdbcTemplate```
		- 이름 기반 파라미터 바인딩을 지원한다. (권장)
		
	- ```SimpleJdbcInsert```
		- INSERT SQL을 편리하게 사용할 수 있다.
		
	- ```SimpleJdbcCall```
		- 스토어드 프로시저를 편리하게 호출할 수 있다.
		
		- 참고
			- 스토어드 프로시저를 사용하기 위한 ```SimpleJdbcCall``` 에 대한 자세한 내용은 다음 스프링 공식 메뉴얼을 참고
			- https://docs.spring.io/spring-framework/docs/current/reference/html/dataaccess.html#jdbc-simple-jdbc-call-1

<br>

#### JdbcTemplate 사용법 정리

- 참고
	- 스프링 JdbcTemplate 사용 방법 공식 메뉴얼
	- https://docs.spring.io/spring-framework/docs/current/reference/html/dataaccess.html#jdbc-JdbcTemplate


- 조회 및 변경 쿼리는 구현한 코드 참고..

- 기타 기능
	- 임의의 SQL을 실행할 때는 ```execute()``` 를 사용하면 된다. 테이블을 생성하는 DDL에 사용할 수 있다.

<br>

- 정리
	- 실무에서 가장 간단하고 실용적인 방법으로 SQL을 사용하려면 JdbcTemplate을 사용하면 된다.
	- JPA와 같은 ORM 기술을 사용하면서 동시에 SQL을 직접 작성해야 할 때가 있는데, 그때도 JdbcTemplate을 함께 사용하면 된다.
	- 그런데 JdbcTemplate의 최대 단점이 있는데, 바로 동적 쿼리 문제를 해결하지 못한다는 점이다. 
	- 그리고 SQL을 자바 코드로 작성하기 때문에 SQL 라인이 코드를 넘어갈 때 마다 문자 더하기를 해주어야 하는 단점도 있다.
	- 동적 쿼리 문제를 해결하면서 동시에 SQL도 편리하게 작성할 수 있게 도와주는 기술이 바로 MyBatis 이다
