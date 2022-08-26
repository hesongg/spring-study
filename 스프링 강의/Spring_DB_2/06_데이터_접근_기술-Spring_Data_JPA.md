### 스프링 DB - 데이터 접근 활용 기술

#### Reference) 
	* 스프링 DB 2편 - 데이터 접근 활용 기술 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/Spring-DB-Itemservice
	
<br>


### 데이터 접근 기술 - 스프링 데이터 JPA


<br>

#### 스프링 데이터 JPA 주요 기능

- 스프링 데이터 JPA는 JPA를 편리하게 사용할 수 있도록 도와주는 라이브러리이다.
	- 수많은 편리한 기능을 제공하지만 가장 대표적인 기능은 다음과 같다.
		- 공통 인터페이스 기능
		- 쿼리 메서드 기능
		
<br>

- 공통 인터페이스 기능
	- ```JpaRepository``` 인터페이스를 통해서 기본적인 CRUD 기능 제공환다.
	- 공통화 가능한 기능이 거의 모두 포함되어 있다.
	- ```CrudRepository``` 에서 ```fineOne()``` -> ```findById()``` 로 변경되었다.
	
	<br>
	
	- ```JpaRepository``` 사용법
		```java
		public interface ItemRepository extends JpaRepository<Member, Long> {}
		```
		
		- ```JpaRepository``` 인터페이스를 인터페이스 상속 받고, 제네릭에 관리할 ```<엔티티, 엔티티ID>``` 를 주면 된다.
		- 그러면 ```JpaRepository``` 가 제공하는 기본 CRUD 기능을 모두 사용할 수 있다.
	
	<br>
	
	- 스프링 데이터 JPA가 구현 클래스를 대신 생성
		- ```JpaRepository``` 인터페이스만 상속받으면 스프링 데이터 JPA가 프록시 기술을 사용해서 구현 클래스를 만들어준다. 
			- 그리고 만든 구현 클래스의 인스턴스를 만들어서 스프링 빈으로 등록한다.
			- 따라서 개발자는 구현 클래스 없이 인터페이스만 만들면 기본 CRUD 기능을 사용할 수 있다

<br>

- 쿼리 메서드 기능
	- 스프링 데이터 JPA는 인터페이스에 메서드만 적어두면, 메서드 이름을 분석해서 쿼리를 자동으로 만들고 실행해주는 기능을 제공한다.

	- 순수 JPA 리포지토리
		```java
		public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
			return em.createQuery("select m from Member m where m.username = :username and m.age > :age")
				.setParameter("username", username)
				.setParameter("age", age)
				.getResultList();
		}
		```
		
		- 순수 JPA를 사용하면 직접 JPQL을 작성하고, 파라미터도 직접 바인딩 해야 한다.
	
	- 스프링 데이터 JPA
		```java
		public interface MemberRepository extends JpaRepository<Member, Long> {
			List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
		}
		```
		
		- 스프링 데이터 JPA는 메서드 이름을 분석해서 필요한 JPQL을 만들고 실행해준다. 
			- 물론 JPQL은 JPA가 SQL로 번역해서 실행한다.
			
		- 물론 그냥 아무 이름이나 사용하는 것은 아니고 다음과 같은 규칙을 따라야 한다
	
	<br>
	
	- 스프링 데이터 JPA가 제공하는 쿼리 메소드 기능
		- 조회: ```find…By``` , ```read…By``` , ```query…By``` , ```get…By```
			- 예:) ```findHelloBy``` 처럼 ...에 식별하기 위한 내용(설명)이 들어가도 된다.
		
		- COUNT: ```count…By``` 반환타입 ```long```
		- EXISTS: ```exists…By``` 반환타입 ```boolean```
		- 삭제: ```delete…By``` , ```remove…By``` 반환타입 ```long```
		- DISTINCT: ```findDistinct``` , ```findMemberDistinctBy```
		- LIMIT: ```findFirst3``` , ```findFirst``` , ```findTop``` , ```findTop3```
		
		- 쿼리 메소드 필터 조건
			- 스프링 데이터 JPA 공식 문서 참고
				- https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.querymethods.query-creation
				- https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limitquery-result
				
	<br>
	
	- JPQL 직접 사용하기
		```java
		public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long> {
			//쿼리 메서드 기능
			List<Item> findByItemNameLike(String itemName);
			
			//쿼리 직접 실행
			@Query("select i from Item i where i.itemName like :itemName and i.price <= :price")
			List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);
		}
		```
		
		- 쿼리 메서드 기능 대신에 직접 JPQL을 사용하고 싶을 때는 ```@Query``` 와 함께 JPQL을 작성하면 된다.
			- 이때는 메서드 이름으로 실행하는 규칙은 무시된다.
		
		- 참고로 스프링 데이터 JPA는 JPQL 뿐만 아니라 JPA의 네이티브 쿼리 기능도 지원하는데, JPQL 대신에 SQL을 직접 작성할 수 있다.

<br>

- 중요
	- 스프링 데이터 JPA는 JPA를 편리하게 사용하도록 도와주는 도구이다. 따라서 JPA 자체를 잘 이해하는 것이 가장 중요하다
	
<br>

#### 스프링 데이터 JPA 적용1

- 설정
	- 스프링 데이터 JPA는 ```spring-boot-starter-data-jpa``` 라이브러리를 넣어주면 된다.
	
	- 그런데 이미 앞에서 JPA를 설정하면서 spring-boot-starter-data-jpa 라이브러리를 넣어주었다.
		- 여기에는 JPA , 하이버네이트, 스프링 데이터 JPA( spring-data-jpa ), 
			그리고 스프링 JDBC 관련 기능도 모두 포함되어 있다.
		- 따라서 스프링 데이터 JPA가 이미 추가되어있으므로 별도의 라이브러리 설정은 하지 않아도 된다.
		
<br>

- 스프링 데이터 JPA 적용
	- 스프링 데이터 JPA가 제공하는 ```JpaRepository``` 인터페이스를 인터페이스 상속 받으면 기본적인 CRUD 기능을 사용할 수 있다.
	
	- 그런데 이름으로 검색하거나, 가격으로 검색하는 기능은 공통으로 제공할 수 있는 기능이 아니다. 
		- 따라서 쿼리 메서드 기능을 사용하거나 ```@Query``` 를 사용해서 직접 쿼리를 실행하면 된다.
		
	- 여기서는 데이터를 조건에 따라 4가지로 분류해서 검색한다. (커밋한 소스참고)
		- 모든 데이터 조회
		- 이름 조회
		- 가격 조회
		- 이름 + 가격 조회
		
	- 동적 쿼리를 사용하면 좋겠지만, 스프링 데이터 JPA는 동적 쿼리에 약하다. 
		- 이번에는 직접 4가지 상황을 스프링 데이터 JPA로 구현
		- 동적쿼리 문제는 이후에 Querydsl에서 해결
		
	- 참고
		- 스프링 데이터 JPA도 ```Example``` 이라는 기능으로 약간의 동적 쿼리를 지원하지만, 
			실무에서 사용하기는 기능이 빈약하다. 실무에서 JPQL 동적 쿼리는 Querydsl을 사용하는 것이 좋다.
	
	<br>
	
	```java
	public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long> {

		List<Item> findByItemNameLike(String itemName);

		List<Item> findByPriceLessThanEqual(Integer price);

		//쿼리 메서드 (아래 메서드와 같은 기능 수행)
		List<Item> findByItemNameLikeAndPriceLessThanEqual(String itemName, Integer price);

		//쿼리 직접 실행
		@Query("select i from Item i where i.itemName like :itemName and i.price <= :price")
		List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);
	}
	```
	
	- ```findAll()```
		- 코드에는 보이지 않지만 JpaRepository 공통 인터페이스가 제공하는 기능이다.
		- 모든 ```Item``` 을 조회한다.
		- 다음과 같은 JPQL이 실행된다.
			```jpql
			select i from Item i
			```

	- ```findByItemNameLike()```
		- 이름 조건만 검색했을 때 사용하는 쿼리 메서드이다.
		- 다음과 같은 JPQL이 실행된다.
			```jpql
			select i from Item i where i.name like ?
			```
	
	- ```findByPriceLessThanEqual()```
		- 가격 조건만 검색했을 때 사용하는 쿼리 메서드이다.
		- 다음과 같은 JPQL이 실행된다.
			```jpql
			select i from Item i where i.price <= ?
			```
	
	- ```findByItemNameLikeAndPriceLessThanEqual()```
		- 가격 조건만 검색했을 때 사용하는 쿼리 메서드이다.
		- 다음과 같은 JPQL이 실행된다.
			```jpql
			select i from Item i where i.itemName like ? and i.price <= ?
			```
	
	- ```findItems()```
		- 메서드 이름으로 쿼리를 실행하는 기능은 다음과 같은 단점이 있다.
		
		- 1. 조건이 많으면 메서드 이름이 너무 길어진다.
		- 2. 조인 같은 복잡한 조건을 사용할 수 없다.
			- 메서드 이름으로 쿼리를 실행하는 기능은 간단한 경우에는 매우 유용하지만, 복잡해지면 직접 JPQL 쿼리를 작성하는 것이 좋다.
		
		- 쿼리를 직접 실행하려면 ```@Query``` 애노테이션을 사용하면 된다.
			- 메서드 이름으로 쿼리를 실행할 때는 파라미터를 순서대로 입력하면 되지만, 
				쿼리를 직접 실행할 때는 파라미터를 명시적으로 바인딩 해야 한다.
		
			- 파라미터 바인딩은 ```@Param("itemName")``` 애노테이션을 사용하고, 애노테이션의 값에 파라미터 이름을	주면 된다

<br>

#### 스프링 데이터 JPA 적용2

- ```JpaItemRepositoryV2``` 코드 참고

- 의존관계와 구조
	- ```ItemService``` 는 ```ItemRepository``` 에 의존하기 때문에 ```ItemService``` 에서
		```SpringDataJpaItemRepository``` 를 그대로 사용할 수 없다.
	
	- 물론 ```ItemService``` 가 ```SpringDataJpaItemRepository``` 를 직접 사용하도록 코드를 고치면 되겠지만,
		우리는 ```ItemService``` 코드의 변경없이 ```ItemService``` 가 ```ItemRepository``` 에 대한 의존을 유지하면서
		DI를 통해 구현 기술을 변경하고 싶다.

	- 새로운 리포지토리를 만들어서 이 문제를 해결
		- 여기서는 ```JpaItemRepositoryV2``` 가 ```MemberRepository``` 와 ```SpringDataJpaItemRepository``` 사이를 맞추기 위한 어댑터 처럼 사용된다.
	
	<br>
	
	- 클래스 의존 관계
		
		![image](https://user-images.githubusercontent.com/77953474/186885570-fa3a769f-ab6c-4abc-8e50-3121c1b9bb0d.png)

		
		- ```JpaItemRepositoryV2``` 는 ```ItemRepository``` 를 구현한다. 그리고 ```SpringDataJpaItemRepository``` 를 사용한다.

	<br>
	
	- 런타임 객체 의존 관계
		
		![image](https://user-images.githubusercontent.com/77953474/186885605-4f20b933-36da-4b82-8f91-5573e90497eb.png)
		
		- 런타임의 객체 의존관계는 다음과 같이 동작한다.
		- ```itemService``` -> ```jpaItemRepositoryV2```
			-> ```springDataJpaItemRepository(프록시 객체)```

	- 이렇게 중간에서 ```JpaItemRepository``` 가 어댑터 역할을 해준 덕분에 ```MemberService``` 가 사용하는
		```MemberRepository``` 인터페이스를 그대로 유지할 수 있고 클라이언트인 ```MemberService``` 의 코드를 변경하지 않아도 되는 장점이 있다.

<br>

- 예외 변환
	- 스프링 데이터 JPA도 스프링 예외 추상화를 지원한다. 
	- 스프링 데이터 JPA가 만들어주는 프록시에서 이미 예외 변환을 처리하기 때문에, ```@Repository``` 와 관계없이 예외가 변환된다.
