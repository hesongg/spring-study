### 프로퍼티 파일 사용 방법 예시

<br>

- 1. ResourceBundle
    - ```java.util.ResourceBundle``` 을 이용하여 프로퍼티 파일의 값을 읽어오는 방식
    - classpath의 props 라는 폴더에 Resource.properties 프로퍼티 파일을 정의 해놓는다고 가정
    - ```java.util.ResourceBundle``` 을 import 한다.
    - ```private static ResourceBundle resourceBundle = null;``` 로 리소스번들 변수를 생성
    - ```resourceBundle = ResourceBundle.getBundle("props.Resource");``` props/ 경로의 Resource.properties 파일을 리소스번들로 얻는다.
    - ```String result = resourceBundle.getString(keyName).trim();``` result String을 설정함

- 2. 프로퍼티 클래스에서 @Value 어노테이션으로 매핑
    - 프로퍼티 전용 bean 클래스를 구현하여 사용한다고 가정하자.

    - 일단 해당 클래스 구현과 별개로 @Configuration(빈 환경설정) 클래스에 ```PropertySourcesPlaceholderConfigurer``` 인스턴스를 생성하는 @Bean 메서드를 static으로 정의해놓는다
        - (중요) static으로 정의해놓아야 설정 값을 불러오기 전에 PropertySourcesPlaceholderConfigurer 를 빈으로 설정 해놓을 수 있다.
            ```java
            @Bean
            public static PropertySourcesPlaceholderConfigurer setPspc(){
                return new PropertySourcesPlaceholderConfigurer();
            }
            ```
            
    - 프로퍼티 클래스 구현
        - 해당 클래스를 bean으로 관리하기위해 ```@Component``` 어노테이션을 붙인다.
        - 사용할 프로퍼티 파일 source를 명시하는 ```@PropertySource``` 어노테이션을 붙인다.
            - ```@PropertySource(value="classpath:props/Resource.properties", ignoreResourceNotFound=true)```
                - ignoreResourceNotFound=true : 해당 프로퍼티 파일에서 특정 값을 찾지 못했을 때, 오류로 처리하지않고 무시함
        - 클래스의 프로퍼티 값은 다음과 같이 작성한다.
            ```java
            @Value("${check.value:N}")
            String checkYn;
            ```
            - Resource.properties 파일에서 check.value 의 값을 찾아서 String checkProperties 변수에 설정시켜준다. 기본 값은 N으로 설정한다.

    - 프로퍼티 클래스 작성을 위와 같이 했으면 다음과 같이 사용할 수 있다.
        - 사전에 @Configuration 클래스(또는 다른 빈 설정)에서 해당 프로퍼티 클래스를 컴포넌트스캔하여 빈으로 등록했다고 가정한다.
        - 사용 예시
            ```java
            public class PropertiesTest {
                @Autowired
                GetProperties getProperties;
                
                public static void main(String[] args){
                    String cv = getProperties.checkYn;
                }
            }
            ```
    
- ```Enviroment API```를 이용한다던지, xml에서 프로퍼티 파일을 사용하는 등 다양한 방법이 있음

- 스프링부트에서는 PropertySourcesPlaceholderConfigurer 가 기본적으로 정의되어있어 더 간편하게 사용할 수 있는거 같다.
