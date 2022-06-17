# 스프링 Transaction 

<br>

### 트랜잭션 이란?

- 트랜잭션(Transaction)은 데이터베이스의 상태를 변환시키는 하나의 논리적 기능을 수행하기 위한 작업의 단위 
  또는 ```한꺼번에 모두 수행되어야 할 일련의 연산```들을 의미한다.

- 참고) ```프록시``` 란?
  - target object와 Aspect를 하나로 합친 객체
  
  - Weaving 과정을 통해 생성
    - ```Weaving``` : Proxy 객체를 생성하는 과정
    
    - Weaving의 종류 
      - Run-time Weaving
        - 스프링에서는 AspectJ를 사용하지 않고 Spring AOP를 사용
        - Spring AOP는 Run-time Weaving
        - 소스코드나 클래스 정보 자체를 변경하지 않음
        - 실행 속도가 AspectJ에 비해 느림
        - 구현하기가 쉬움

      - Compile-time Weaving
        - Compile 시점에 Proxy 객체 생성
        - Load-time에 대한 절차가 없어 퍼포먼스 하락 없음
        - lombok 처럼 compile시 수행되는 plugin과 충돌 발생할 수 있음
        - AspectJ 라이브러리를 추가해서 사용하는 방식

      - Load-time Weaving
        - Load 시점에 AspectJ에 의해 Proxy 객체를 생성
        - applicationContext에 로드될 객체들이 모두 로드된 이후에 weaving을 하므로 퍼포먼스 하락이 있음
        - AspectJ 라이브러리를 추가해서 사용하는 방식
   
  - JDK Dynamic Proxy
    - Java Reflection을 이용하여 비교적 느림
    - 인터페이스를 구현한 객체를 대상으로 함
  - CGLIB Proxy
    - 스프링 부트에서는 디폴트로 사용되는 프록시 객체
    - 상속을 통해 Proxy를 구현
    - final 클래스일 경우 Proxy로 생성 불가
    - 인터페이스를 통하지 않은 일반 객체를 대상으로 함

  - 스프링 프레임워크 AOP의 기본 설정은 JDK Dynamic Proxy 이다. (스프링 부트는 CGLIB Proxy)
    - 두 프록시의 차이는 인터페이스 유무이다.
    - 다이나믹 프록시의 경우 인터페이스를 구현한 클래스가 필수로 존재해야하고, 리플렉션을 활용하므로 성능이 떨어진다는 단점이 있다.
    - CGLIB는 클래스만으로 프록시 객체를 동적으로 생성 가능하고, 리플렉션이 아닌 바이트 조작을 사용하여 타겟에 대한 정보를 알고있기 때문에
      다이나믹 프록시에 비해 성능이 좋다.
    - 스프링 프레임워크에서 xml 설정을 사용하고 있을 때, ```<aop:config>``` 태그 안에 ```proxy-target-class="true"``` 를 추가하면
      CGLIB를 사용하도록 한다.

- AOP 방식으로 사용하는 스프링 트랜잭션(선언적 트랜잭션 방식 및 xml aop )은 프록시 기반으로 동작하기때문에 public 메서드에만 적용된다.
    - 스프링의 트랜잭션 처리는 스프링 AOP를 기반으로 하고 있다.
      - 스프링 AOP는 프록시를 기반으로 동작한다. 
      
      - 프록시는 원래 객체를 상속받아 해당 객체를 감싸서 트랜잭션 로직을 추가하기 때문에 상속이 되지않는 private 객체는 트랜잭션 처리가 안된다.

      - 그리고 트랜잭션은 Proxy 외부에서 접근해야 AOP가 적용되기 때문에 프록시 객체 메서드에서 같은 객체에있는 메서드를 호출 시 
        이미 Bean의 내부에서 호출하기 때문에 Proxy를 통한 호출이아니어서 트랜잭션이 적용되지 않는다.
      
      - 여러 트랜잭션 설정 방식으로 private 객체를 만들어서 테스트 해보자..
      
    - private 메서드에 적용하고싶으면 ```AspectJ(non-proxy based)``` 를 알아보자.
  
<br>  
  
### 트랜잭션 특징

|ACID|설명|
|----|---|
|Atomicity(원자성)|트랜잭션의 연산은 데이터베이스에 모두 반영되든지 아니면 전혀 반영되지 않아야 한다.(All or Nothing)|
|Consistency(일관성)|트랜잭션이 성공적으로 완료하면 일관성 있는 데이터베이스 반영한다.|
|Isolation(독립성,격리성)|트랜잭션은 독립적으로 처리되며, 서로간의 트랜잭션은 간섭(연산) 할 수 없다.|
|Durablility(영속성,지속성)|성공적으로 완료된 트랜잭션의 결과는 영구적으로 지속되어야 한다.|

<br>

### propagation (전파 레벨)

- 트랜잭션에 전파 레벨을 설정할 수 있다.

- Propagation.REQUIRED
  - ```default``` 값이기 때문에 생략 가능
  - 부모 트랜잭션(해당 메서드를 호출한 곳의 트랜잭션)이 있을 경우, 
    부모 트랜잭션 내에서 실행하며 부모 트랜잭션이 없을 경우 새로운 트랜잭션 생성
  - 예외가 발생하면 롤백이 되고 호출한 곳에도 롤백이 전파된다.

- Propagation.REQUIRES_NEW
  - 매번 새로운 트랜잭션을 시작(새로운 연결을 생성하고 실행)
  - 호출한 곳에서 이미 트랜잭션이 설정되어 있다면(기존의 연결), 기존의 트랜잭션은 메소드가 종료할 때까지 잠시 대기 상태로 두고 자신의 트랜잭션을 실행한다.       <br>(부모 트랜잭션 상관X -> 트랜잭션 실패의 경우 예외 처리를 하지 않으면 부모 트랜잭션에게 예외가 전파될 수 있다.)

- Propagation.NESTED
  - 해당 메소드가 부모 트랜잭션에서 진행될 경우 별개로 커밋되거나 롤백될 수 있다.
    - Propagation.REQUIRES_NEW 와 다르게 부모 트랜잭션이 중심인 것 같다.
  
  - 부모 트랜잭션이 있으면 중첩 트랜잭션을 생성
  - 중첩 트랜잭션이 끝나도 모든 커밋은 부모 트랜잭션의 끝에서 이루어 진다.
  - 중첩 트랜잭션 내부에서 롤백이 발생할 때 롤백을 부모 트랜잭션까지 전파하지 않는다.
  - 중첩 트랜잭션이 끝난 후 부모 트랜잭션에서 롤백이 발생하면 모든 트랜잭션을 롤백한다.
  - 둘러싼 부모 트랜잭션이 없을 경우 Propagation.REQUIRED와 동일하게 작동한다. (새로운 트랜잭션 생성)
  - 유의해야 할 점은 DB가 SAVEPOINT 기능을 지원해야 사용이 가능(Oracle)하다고 한다.

- Propagation.MANDATORY
  - 부모 트랜잭션에 합류한다. 만약 부모 트랜잭션이 없다면 예외 발생

- Propagation.SUPPORT
  - 부모 트랜잭션이 존재하면 부모 트랜잭션으로 동작하고, 없을경우 non-transactional 하게 동작한다.

- Propagation.NOT_SUPPORT
  - non-transactional 로 실행되며 부모 트랜잭션이 존재하면 일시 정지한다.

- Propagation.NEVER
  - non-transactional 로 실행되며 부모 트랜잭션이 존재하면 Exception이 발생한다.

<br>

### isolation (격리수준)

- 트랜잭션에서 일관성이 없는 데이터를 허횽하도록 하는 수준

- READ_UNCOMMITED (level 0)
- READ_COMMITED (level 1)
- REPEATABLE_READ (level 2)
- SERIALIZABLE (level 3)

- 격리 수준이 높아질수록 동시성(Concurrency)는 높아지고 성능은 안좋아진다.
  - 균형을 잘 맞추는 것이 중요

- READ_UNCOMMITED (커밋되지 않는 읽기, level 0)
  - 트랜잭션 처리 중이거나 아직 commit되지 않은 데이터를 다른 트랜잭션이 읽는 것을 허용한다.
  - ```dirty read``` 발생
    - 다른 트랜잭션에서 처리하는 작업이 완료되지 않았는데도 다른 트랜잭션에서 읽을 수 있는 현상

- READ_COMMITED (커밋된 읽기, level 1)
  - dirty read 방지 : 트랜잭션이 commit 되어 확정된 데이터만을 읽도록 허용
  - A라는 데이터가 B로 변경되는 동안 다른 사용자는 해당 데이터에 접근할 수 없음
  - "REPEATABLE READ" 정합성에는 어긋난다. -> ```"NON-REPEATABLE READ"```
    - 하나의 트랜잭션 내에서 같은 SELECT 쿼리를 여러번 실행했을 때, 그 사이에 커밋이 일어나면 결과가 달라짐
    

- REPEATABLE_READ (반복 가능한 읽기, level 2)
  - 트랜잭션이 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 shared lock이 걸린다.
  - 선행 트랜잭션이 읽은 데이터는 트랜잭션이 종료될 때까지 후행 트랜잭션이 갱신하거나 삭제하는 것을 불허함으로써 
    같은 데이터를 두 번 조회했을 때 일관성 있는 결과를 보여준다.
  - Phantom READ 발생
    - 한 트랜잭션내에서 동일한 쿼리를 두 번 수행했는데, 첫 번째 쿼리에서 존재하지 않던 유령(Phantom) 레코드가 두 번째 쿼리에서 나타나는 현상
  
- SERIALIZABLE (직렬화 가능, level 3)
  - 완벽한 읽기 일관성 모드 제공(가장 엄격하다)
  - 성능 측면에서 동시 처리 성능이 가장 낮다.
  - 거의 사용되지 않음
  - 트랜잭션이 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 shared lock이 걸리므로 
    다른 사용자는 그 영역에 해당하는 데이터에 대한 수정 및 입력이 불가능하다.

- 각 DB별 isolation
  ```
  MYSQL : REPEATABLE READ
  ORACLE : READ COMMITTED
  H2 : READ COMMITTED
  DB2 : READ COMMITTED
  ```

<br>

### 트랜잭션 설정 방식


- xml에서 AOP로 설정
  - 스프링 AOP기능을 이용해 트랜잭션의 범위, 롤백 규칙 등을 설정파일로 공통적으로 트랜잭션을 적용하는방식
  - 각 구현체에 비즈니스 로직을 구현할 때 따로 트랜잭션 미처리에 대한 문제를 피할 수 있음
  - 필요없는 부분까지 적용이 되기에 성능에 문제가 될 수 있음
  
  - 현재 실무에서 사용하는 방법이다.. 
    - 많은 비즈니스 모델의 공통 프레임워크 관리를 해야해서 이렇게했지만.. 상황에 따라 선언적 방식을 쓰는 것도 간편할 것 같다
  - xml 설정에 특정 dataSource를 참조하는 txManager bean을 만든 다음, AOP 에 적용한다
    ```xml
    <!-- 편의상 상단의 xml 스키마 내용은 생략-->

    <bean id = "txManager" class = "org.springframework.jdbc.datasource.Datasource.DataSourceTransactionManager">
      <property name="dataSource" ref="dataSource"/>   
    </bean>

    <tx:advice id = "txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="*" rollback-for = "Exception" timeout="40"/>
        </tx:attributes>
    </tx:advice>
    
    <aop:config>
        <!-- aop pointcut 설정. NonTransactional 어노테이션을 만들어서 해당 어노테이션이 달려있으면 트랜잭션을 안타게한다. -->
        <aop:pointcut id = "requiredTx" expression = "execution(* org.go.temp.*.*SVCImpl.*(..))
                                                       &amp;&amp; ! @annotation(org.temp.NonTransactional)"/>
        <!-- requiredTx 로 정의해놓은 pointcut에 txAdvice 를 적용한다. -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="requiredTx" />
    <aop:config>
    ```

- 선언적 트랜잭션(@Transactional)
    - 객체가 스프링 IoC 컨테이너에 빈으로 정의되었다면 이 빈 인스턴스는 XML 설정을 한 줄 추가하는 것으로 트랜잭션 처리 대상이 된다.
    ```xml
         <tx:annotation-driven transaction-manager="txManager"/>

        <bean id = "txManager" class = "org.springframework.jdbc.datasource.Datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource"/>   
        </bean>
    ```
    
    - Java 설정 사용하고싶으면 ```@Configuration``` 클래스에 ```@EnableTransactionManagement``` 어노테이션도 추가하면 된다.
        
    -
    
- 프로그래밍 방식 트랜잭션 설정
    - AOP 방식 트랜잭션이 비효율적인 경우 사용 가능
    
    - ```TransactionTemplate``` 사용
      - 실제로 업무에서 사용된 방식을 정리.. 원래 공통 설정 트랜잭션 기능을 제공하기위해 xml 설정(aop 방식)을 이용하여서 
        서비스가 실행될 때 Transaction 사용이 되도록 공통 처리가 되어있는데, 그 안에서 개별적으로 새로운 Transaction 사용이 경우가 있어
        Transaction 을 새로 만들어서 비즈니스 로직을 처리할 수 있는 메서드를 제공한다.
          ```java
          public static <T> T doExecuteNewTransaction(TransactionCallback<T> callback, int timeout) {
            //xml에 bean으로 정의해놓은 "txManager"(DataSourceTransactionManager) 를 사용한다.
            TransactionTemplate txTemplate = new TransactionTemplate((PlatformTransactionManager) StaticBeanService.getServiceBean("txManager"));
            
            // 전파레벨을 Propagation.REQUIRES_NEW 로 설정하여 완전히 새로운 트랜잭션으로 설정한다.
            txTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
            
            // 업무 특성에 맞는 트랜잭션명을 가져오기 위해 따로 정의함
            String txName = getTransactionName("TEMP");
            
            txTemplate.setName(txName);
            
            if(timeout != TIMEOUT_NOTDEFINED){
              txTemplate.setTimeout(timeout);
            }
            
            try {
              return txTemplate.execute(callback);  // 인수로 전달받은 비즈니스 메서드 실행하여 값 
            } catch (RuntimeException e) {
              throw e;
            }
          }
          ```
        
    
    - ```PlatformTransactionManager``` 구현체 직접 사용

- aspectJ ? 

#### 추가로..

- checked uncheckedException ?
- 
