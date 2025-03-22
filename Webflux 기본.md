### Webflux 기본내용 정리

webflux 에서 자바 future ?
- webflux 에서는 future 나 CompletableFuture 를 사용할 일이 적다.
- webflux 에서는 자체적으로 리액티브 스트림 (Mono, Flux) 를 지원하기 때문에 future 가 없어도 비동기 코드를 작성할 수 있다.
- `Reactor` 자체가 `CompletableFuture` 보다 더 강력한 기능을 제공한다. (delay, flatMap, zip, retry, backpressure 등)

<br>

여기서 Reactor 란 뭘까?
- 비동기 & 논블로킹 프로그래밍을 위한 리액티브 스트림 라이브러리이다.
- webFlux 의 핵심 라이브러리이며, `Mono`, `Flux` 를 제공하여 데이터 스트림을 처리할 수 있도록 한다.
- `CompletableFuture` 와 비슷하지만 더 많은 연산자 (`flatMap`, `delay`, `zip`, `retry` 등)를 제공
- 백프레셔(Backpressure) 를 지원해서 빠른 데이터 스트림을 소비자의 속도에 맞춰 조절 가능하다.

<br>

Mono ? Flux ?
- `Mono<T>` : 0개 또는 1개의 데이터를 비동기적으로 처리 
- `Flux<T>` : 0개 이상의 데이터를 스트림 형태로 처리

<br>

Webflux 와 File I/O
- File I/O 는 네트워크 I/O 와 마찬가지로 I/O intensive 한 작업이다.
- CPU 를 많이 쓰지는 않지만, 스레드가 I/O 작업 완료를 기다리는 동안 `블로킹` 된다.
  - WebFlux 의 이벤드 루프 (NIO 스레드) 를 점유하면 성능 문제가 생길 수 있다.
- 그래서 블로킹 작업의 경우 `Mono.fromCallable(() -> readFile()).subscribeOn(Schedulers.boundedElastic())` 같이 별도 스레드에서 실행시켜야 한다.

<br>

참고) 왜 파일 I/O 는 논블로킹이 불가할까?
- 대부분의 운영체제에서 디스크 I/O 는 시스템 콜을 통해 데이터를 읽고 써야하며, 그 과정이 동기적이라 blocking 이 된다고한다.
  - 정말 그런지 확인이 필요할 듯..
  - 자바에서는 NIO 기반 파일 I/O API 도 있다고하는데.. 사용하기 복잡하고,
  - 구조적으로 디스크 I/O 가 네트워크처럼 OS 레벨에서 완전 논블로킹이 어려운 구조여서 블로킹 I/O 를 별도 스레드에서 실행하는게 나은 것 같다.

<br>

Netty
- 비동기 이벤트 기반 네트워크 프레임워크
- spring webflux 에서는 Netty 를 기본 서버로 사용한다. (비동기 웹 요청 처리 가능)

<br>

기본 이벤트루프 스레드 개수
- 기본 값은 CPU 코어 수 X 2 라고한다.
- 스프링에서 직접 설정도 가능
- 스레드 개수가 너무 적으면 병목이 생기고, 너무 많으면 컨텍스트 스위칭 오버헤드가 발생할 수 있다고한다.

<br>

블로킹 작업을 별도의 스레드풀에서 실행하는 예시
```java
return Mono.fromCallable(() -> {
    Thread.sleep(2000); // 블로킹 작업 (예: DB, API 요청)
    return "User Data";
}).subscribeOn(Schedulers.boundedElastic()); // 블로킹 작업을 별도 스레드 풀에서 실행
```
- Mono.fromCallable()
  - 블로킹작업을 비동기적으로 감싸기
- subscribeOn()
  - mono 의 실행을 특정 스레드풀에서 수행하도록 지정한다.
- Schedulers.boundedElastic()
  - 블로킹 작업을 위한 별도 스레드 풀에서 실행된다.
- 결과적으로 이 코드는 webflux 의 기본 이벤트 루프(NIO 스레드)가 블로킹되지않도록 보호한다.

<br>

Webflux (Reactor) 를 고부하 환경에서 안전하게 쓰려면?
| **전략** | **설명** |
|---------------|-----------------|
| `boundedElastic()`은 블로킹 작업 전용으로만 사용 | 절대 모든 작업에 무분별하게 사용 금지 |
| 작업 시간 짧고 자주 쓰는 건 parallel() 사용 | CPU 바운드 연산은 여기서 |
| 병목이 우려되는 작업은 timeout()으로 보호 | 예: .timeout(Duration.ofSeconds(3)) |
| 요청이 많을 경우 백프레셔(backpressure) 적용 | limitRate() 등으로 조절 |
| 외부 시스템(API, DB 등)에도 타임아웃 설정 | WebClient → .responseTimeout(...) 등 |
| 스케줄러 수치 설정은 명시적으로 | System.setProperty() 또는 application.yml 사용 |
