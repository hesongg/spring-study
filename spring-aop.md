# Spring aop

### xml 방식으로 구현한 간단한 aop 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:task="http://www.springframework.org/schema/task"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
						http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
						http://www.springframework.org/schema/aop 
						http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
						
	<!-- 서비스 호출전/후 AOP 설정 (around) -->
	<aop:config>
		<aop:aspect id="exAspect" ref="serviceExecutionAdvice">
			<aop:pointcut id="serviceCallPointcut"
				expression="execution(public * ex.*Caller.serviceCall*(..))" />
			<aop:around pointcut-ref="serviceCallPointcut" method="canExecute" />
		</aop:aspect>
	</aop:config>
	
	<!-- Aop Advice, 서비스 실행 전 후 필요한 메소드 실행 -->
	<bean id="serviceExecutionAdvice" class="ex.ServiceExecutionControl">
		<property name="serviceExecutionChecker" ref="serviceExecutionChecker"/> 
	</bean>
	
	<!-- 서비스 제어 bean -->
	<bean id="serviceExecutionChecker" class="ex.ServiceExecutionChecker" />
	
</beans>
```

- 자바 aop 설정, xml 태그 내용, pjp 등 내용 추가 
