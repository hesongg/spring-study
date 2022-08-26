### 스프링 DB - 데이터 접근 활용 기술

#### Reference) 
	* 스프링 DB 2편 - 데이터 접근 활용 기술 (인프런 김영한님 강의 학습한 내용 정리)

#### 작성 코드
- https://github.com/hesongg/Spring-DB-Itemservice
	
<br>


### 데이터 접근 기술 - Querydsl


<br>

#### Querydsl 설정

- ```build.gradle```
	- ```dependencies``` 에 추가
		```gradle
		//Querydsl 추가
		implementation 'com.querydsl:querydsl-jpa'
		annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jpa"
		annotationProcessor "jakarta.annotation:jakarta.annotation-api"
		annotationProcessor "jakarta.persistence:jakarta.persistence-api"
		```
		
	- 밑의 내용도 추가
		```gradle
		//Querydsl 추가, 자동 생성된 Q클래스 gradle clean으로 제거
		clean {
			delete file('src/main/generated')
		}
		```
	
<br>

- 검증 - Q 타입 생성 확인 방법
	- Preferences -> Build, Execution, Deployment -> Build Tools -> Gradle
	
	- 여기에 가면 크게 2가지 옵션을 선택할 수 있다. 참고로 옵션은 둘다 같게 맞추어 두자.
		- 1. Gradle: Gradle을 통해서 빌드한다.
		- 2. IntelliJ IDEA: IntelliJ가 직접 자바를 실행해서 빌드한다
	
	- 옵션 선택1 - Gradle - Q타입 생성 확인 방법
		- Gradle IntelliJ 사용법
			- ```Gradle -> Tasks -> build -> clean```
			- ```Gradle -> Tasks -> other -> compileJava```
			
		- Gradle 콘솔 사용법
			- ```./gradlew clean compileJava```
	
		<br>
			
		- Q 타입 생성 확인
			- ```build -> generated -> sources -> annotationProcessor -> java/main``` 하위에
				- ```hello.itemservice.domain.QItem``` 이 생성되어 있어야 한다.
			
		- 참고: Q타입은 컴파일 시점에 자동 생성되므로 버전관리(GIT)에 포함하지 않는 것이 좋다.
			- gradle 옵션을 선택하면 Q타입은 ```gradle build``` 폴더 아래에 생성되기 때문에 여기를 포함하지 않아야한다.
			- 대부분 ```gradle build``` 폴더를 git에 포함하지 않기 때문에 이 부분은 자연스럽게 해결된다.

		- Q타입 삭제
			- ```gradle clean``` 을 수행하면 ```build``` 폴더 자체가 삭제된다. 따라서 별도의 설정은 없어도 된다.

	
	<br>
	
	- 옵션 선택2 - IntelliJ IDEA - Q타입 생성 확인 방법
		- ```Build -> Build Project``` 또는
		- ```Build -> Rebuild``` 또는
		- ```main()``` , 또는 테스트를 실행하면 된다
		
		- ```src/main/generated``` 하위에
			- ```hello.itemservice.domain.QItem``` 이 생성되어 있어야 한다.
		
		- Q타입 삭제
			- ```IntelliJ IDEA``` 옵션을 선택하면 ```src/main/generated``` 에 파일이 생성되고, 
				필요한 경우 Q파일을 직접 삭제해야 한다.
			
			- ```gradle``` 에 해당 스크립트를 추가하면 ```gradle clean``` 명령어를 실행할 때 
				```src/main/generated``` 의 파일도 함께 삭제해준다.
				```gradle
				//Querydsl 추가, 자동 생성된 Q클래스 gradle clean으로 제거
				clean {
					delete file('src/main/generated')
				}
				```
<br>

#### Querydsl 적용

- ```JpaItemRepositoryV3``` 에 구현 - 코드 참고

- 공통
	- Querydsl을 사용하려면 ```JPAQueryFactory``` 가 필요하다. ```JPAQueryFactory``` 는 JPA 쿼리인 JPQL을
		만들기 때문에 ```EntityManager``` 가 필요하다.
	
	- 설정 방식은 ```JdbcTemplate``` 을 설정하는 것과 유사하다.
	
	- 참고로 ```JPAQueryFactory``` 를 스프링 빈으로 등록해서 사용해도 된다.

<br>

- ```save(), update(), findById()```
	- 기본 기능들은 JPA가 제공하는 기본 기능을 사용한다.

<br>

- ```findAllOld```
	- Querydsl을 사용해서 동적 쿼리 문제를 해결한다.
	- ```BooleanBuilder``` 를 사용해서 원하는 ```where``` 조건들을 넣어주면 된다.
	- 이 모든 것을 자바 코드로 작성하기 때문에 동적 쿼리를 매우 편리하게 작성할 수 있다.

<br>

- ```findAll```
	- 앞서 ```findAllOld``` 에서 작성한 코드를 깔끔하게 리팩토링 했다. 
		```java
		List<Item> result = query
			.select(item)
			.from(item)
			.where(likeItemName(itemName), maxPrice(maxPrice))
			.fetch();
		```
		
		- ```BooleanExpression``` 을 리턴타입으로 하는 메서드 구현 (```likeItemName```, ```maxPrice```)
	
	- ```Querydsl```에서 ```where(A,B)``` 에 다양한 조건들을 직접 넣을 수 있는데,
		이렇게 넣으면 AND 조건으로 처리된다. 참고로 ```where()``` 에 ```null``` 을 입력하면 해당 조건은 무시한다.
		
	- 이 코드의 또 다른 장점은 ```likeItemName()``` , ```maxPrice()``` 를 다른 쿼리를 작성할 때 재사용 할 수 있다는	점이다.
		- 쉽게 이야기해서 쿼리 조건을 부분적으로 모듈화 할 수 있다. 자바 코드로 개발하기 때문에 얻을 수 있는 큰 장점이다.

<br>

- 예외 변환
	- ```Querydsl``` 은 별도의 스프링 예외 추상화를 지원하지 않는다. 
		- 대신에 JPA에서 학습한 것 처럼 ```@Repository``` 에서 스프링 예외 추상화를 처리해준다.

<br>

#### 정리

- Querydsl 장점
	- Querydsl 덕분에 동적 쿼리를 매우 깔끔하게 사용할 수 있다.
	
	- 쿼리 문장에 오타가 있어도 컴파일 시점에 오류를 막을 수 있다
	- 메서드 추출을 통해서 코드를 제사용할 수 있다.
