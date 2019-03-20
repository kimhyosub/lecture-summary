# Spring Web Flux (Toby님)

> 함수형 스타일 WebFlux

스프링이 웹 요청을 처리하는 방식
 * 요청 매핑 
  - 웹 요청을 어느 핸들러에게 보낼지 결정
  - URL, 헤더
  - @RequstMapping
 * 요청 바인딩
  - 핸들러에게 전달할 웹 요청 준비
  - 웹 URL, 헤더, 쿠키, 바디
 * 핸들러 실행
  - 전달 받은 요청 정보를 이용해 로직을 수행하고 결과를 리턴
 * 핸들러 결과 처리
  - 핸들러의 리턴 값으로 웹 응답 생성
  ```
  @RestController
  public class MyController { 
    @GetMapping("/hello/{name}") // 요청 매핑
    String Hello(@PathVariable String name){ // 요청 바인딩
      return "Hello" + name; // 핸들러 실행, 핸들러 결과 처리(응답 생성)
    }
  }
  ```
  > RouterFunction
   * 함수형 스타일의 요청 매핑
  ```
  @FunctionalInterface
  public interface RouterFunction<T extends ServerResponse> {
    Mono<HandlerFunction<T>> route<ServerRequest request);
  }
  ```
  > HandlerFunction
   * 함수형 스타일의 웹 핸들러(컨트롤러 메소드)
   * 웹 요청을 받아 웹 응답을 돌려주는 함수
   
> 함수형 WebFlux가 웹 요청을 처리하는 방식
 * 요청 매핑 (Router Function)
 * 요청 바인딩 (Handler Function)
 * 핸들러 실행 (Handler Function)
 * 핸들러 결과 처리 (Handler Function)


> WebFlux 함수형 Hello/{name} 작성
 * 함수 2개 만들면 됨
  - HandlerFunction을 먼저 만들고
  - RouterFunction에서 path 기준 매핑을 해준다

HandlerFunction helloHandler = req -> { 
  * HandlerFunction.hanle()의 람다식 
  * Mono<T>handle(ServerRequest request);
  String name = req.pathVariable("name"); 
  * ServerRequest.pathVariable()로 {name} 추출
  Mono<String> result = Mono.just("Hello" + name); 
  * 로직 적용후 결과 값을 Mono에 담는다
  Mono<ServerResponse> res = ServerResponse.ok().body(result, String.class); 
  * 웹 응답을 ServerResponse로 만든다.
  * HTTP 응답에는 응답코드, 헤더, 바디가 필요
  * ServerResponse의 빌더를 활용
  * Mono에 담긴 ServerResponse 타입으로 리턴
  return res;
};

> 간단하게 작성시
HandlerFunction helloHandler = req -> ok().body(fromObject("Hello" + req.pathVariable("name")));
 * ServerRequest에서 로직에 필요한 데이터 추출 & 바인딩
 * 애플리케이션 로직을 적용하고 결과 값을 만든다.
 * BodyInserters.fromObject()의 도움을 받아 Mono에 담는다
 * 로직의 결과 값을 바디에 담고 상태코드를 추가해 웹 응답(ServerResponse)으로 만든다.
 * content-type이나 기타 헤더들은 스프링이 알아서 만들어준다.
 * 함수(람다 식) 형식으로 만들어 둔다
 
> RouterFunction으로
RouterFunction router = req -> RequestPredicates.path("/hello/{name}").test(req) ? Mono.just(helloHandler) : Mono.empty()
 * 웹 요청 정보 중에서 URL 결로 패턴 검사

> HandlerFunction과 RouterFunction 조합
 * RouterFunctions.route(predicate, handler)
 ```
 RouterFunction router = RouterFunctions.route(RequestPredicates.path("/hello/{name}")
   , req ->ServerResponse.ok().body(fromObject("Hello" + req.pathVariable("name"))));
 ```  
 > RouterFunction의 등록
  * RouterFunction 타입의 @Bean으로 만든다
 ``` 
 @Bean
 RounterFunction helloPathVarRouter(){
  return route(RequestPredicates.path("/hello/{name}")
   , req ->ServerResponse.ok().body(fromObject("Hello" + req.pathVariable("name"))));
 }
 ```
> 핸들러 내부 로직이 복잡하다면 분리한다.
 * 핸들러 코드만 람다 식을 따로 선언하거나
 * 메소드를 정의하고 메소드 참조로 가져온다
```
HandlerFunction handler = req -> {
  String res = myService.hello(req.pathVariable("name"));
  return ok().body(fromObject(res));
};

return route(path("/hello/{name}"), handler);


@Component
public class Hellohandler {
  @Autowired MyService myService;
  
  Mono<ServerResponse>hello(ServerRequest req){
    String res = myService.hello(req.pathVariable("name"));
    return ok.body(fromObject(res));
  }
}

@Bean
RouterFunction helloRouter(@Autowired HelloHandler helloHandler) {
  return route(path("/hello/{name}"), helloHandler::hello);
}
```
> RouterFunction의 조합
 * 핸들러 하나에 @Bran하나씩 만들어야하나?
 * RouterFunction의 and(), andRoute() 등으로 하나의 @Bean에 n개의 RouterFunction을 선언할 수 있다.
 
> RouterFucntion의 매핑 조건의 중복
 * 타입 레벨 - 메소드 레벨의 @RequestMapping처럼 공통의 조건을 정의하는 것 가능
 * RouterFunction.nest()
```
 public Router(Function<?>routingFunction(){
  return nest( pathPrefix("/person")
      , nest(accept(APPLICATION_JSON)
      , route(GET("/id"), handler::getPerson).andRoute(method(HttpMethod.GET), handler::listPeople)
      ).andRoute(POST("/").and(contentType(APPLICATION_JSON)), handler::createPerson)
    );
```
  * /person/{id} 경로의 GET이면 getPerson 핸들러로 매핑
  * /person 경로에 GET이면 listPeople 핸들러로 매핑
  * /person 경로에 POST이고 contentType이 APPLICATION_JSON이면 createPerson 핸들러로
 }
 
> WebFlux 함수형 스타일의 장점
 * 모든 웹 요청 처리 작업을 명시적인 코드로 작성
  - 메소드 시그니처 관례와 타입 체크가 불가능한 애노테이션에 의종하는 @MVC 스타일보다 명확
  - 정확한 타입 체크 가능
 * 함수 조합을 통한 편리한 구성, 추상화에 유리
 * 테스트 작성의 편리함
  - 핸들로 로직은 물론이고 요청 매핑과 리턴 값 처리까지 단위 테스트로 작성 가능
  
> WebFlux 함수형 스타일의 단점
 * 함수형 스타일의 코드 작성이 편하지 않으면 코드 작성과 이해 모두 어려움
 * 익숙한 방식으로도 가능한데 뭐하러...

> **@Controller + @RequestMapping**

> @MVC WebFlux
 * 애노테이션과 메소드 형식의 관례를 이용하는 @MVC방식과 유사
 * 비동기 + 논블록킹 리엑티브 스타일로 작성

> ServerRequest, ServerResponse
 * WebFlux의 기본 요청, 응답 인터페이스 이용
 * 함수형 WebFlux와 HandlerFunction을 메소드로 만들었을때와 유사
 * 매핑만 애노테이션 방식을 이용
``` 
 @RestController
 public static class MyController{
  @RequestMapping("/hello/{name}")
  Mono<ServerResponse>hello(ServerRequest req){
    return ok().body(fromObject(req.pathVariable("name")));
  }
 }
``` 
> @MVC 요청 바인딩과 Mono / Flux 리턴 값
 * 가장 대표적인 @MVC WebFlux 작성 방식
 * 파라미터 바인딩은 @MVC 방식 그대로
 * 핸들러 로직 코드의 결과를 Mono / Fulx 타입으로 리턴
``` 
 @GetMapping("/hello/{name}")
 Mono<String>hello(@PathVariable String name){
  return Mono.just("Hello" + name);
 }
 
 @RequestMapping("/hello")
 Mono<String>hello(User user){
  return Mono.just("Hello" + user.getName());
 }
``` 
> @RequestBody 바인딩 (JSON, XML)
 * T
```
 @RequestMapping("/hello")
 Mono<String>hello(@RequestBody User user){
  return Mono.just("Hello" + user.getName());
 }
```
 * Mono<T>
```
 @RequestMapping("/hello")
 Mono<String>hello(@RequestBody Mono<User> user){
  return user.map(u->"Hello" + u.getName());
 }
```
 * Flux<T>
```
 @PostMaping("/hello")
 Flux<String>hello(@RequestBody Flux<User> user){
  return user.map(u->"Hello" + u.getName());
 }
```
 * Flux<ServerSideEvent>
 * void
 * Mono<void>
 
> **WebClient + Reactive Data**
 WebFlux와 리엑티브 기술

> WebFlux 만으로 성능이 좋아질까?
 * 비동기- 논블록킹 구조의 장점은 블록킹 IO를 제거하는 데서 나온다
 * HTTP 서버에서 논블록킹 IO는 오래 전부터 지원
 * 뭘 개선해야 하나?

> 개선할 블록킹 IO
 * 데이터 엑세스 리포지토리
 * HTTP API
 * 기타 네트워크를 사용하는 서비스
 
> JPA - JDBC 기반 RDB 연결
 * 현재 답이 없다
 * 블로킹 메소드로 점철된 JDBC API
 * 일부 DB는 논블록킹 드라이버가 존재하지만...
 * @Async 비동기 호출과 CFuture를 리엑티브로 연결하고 
   쓰레드풀 관리를 통해서 앱 연결 자원을 효율적으로 사용하도록 만드는 정도
 * JDK10 에서 Async JDBC가 등장할 수도
 
> Spring Data JPA의 비동기 쿼리 결과 방식
 * 리포지토리 메소드의 리턴 값을 @Async 메소드처럼 작성
```
 @Async
 CompletableFuture<User> findOneByFirstname(String firstname);
 
 @GetMapping
 Mono<User>findUser(String name){
  return Mono.fromCompletionStage(myRepository.findOneByFirstname(name));
 }
```
> 본격 리액티브 데이터 엑세스 기술
 * 스프링 데이터의 리액티브 리포지토리 이용
  - MongoDB
  - Cassandra
  - Redis
 * ReavtiveCrudRepository
``` 
 public interface ReactivePersonRepository extends ReactiveCrudRepository<Person, String>{
  Flux<Person>findByLastname(Mono<String> lastname);
  @Query({'fiistname':?0, 'lastname':?1})
  Mono<Person> findByFirstnameAndLastname(String firstname, String lastname);
 }
```
```
 @Autowired UserRepository userRepository
 
 @GgetMapping
 Flux<User> users(){
  return userRepository.findAll();
 }
``` 
 * 리포지토리 결과 Flux에 대해 추가 로직 적용
``` 
 public Mono<ServerResponse> getPerson(ServerRequest request) {
  int poersonId = Integer.valueOf(request.pathVariable("id"));
  Mono<ServerResponse> notFound = ServerResponse.notFound().build();
  Mono<Person> personMono = this.Repository.getPerson(personId);
  return personMono.flatMap(
    person-> ServerResponse.ok().contentType(APPLICATION_JSON).body(fromObject(person))).switchEmpty(notFound);
 }
```
> 논블록킹 API 호출은 WebClient
 * AsyncRestTemplate의 리액티브 버전
 * 요청을 Mono/Flux로 전달할 수 있고
 * 응답을 Mono/Flux형태로 가져온다
``` 
 @GetMapping("/webclient")
 Mono<String> webclient(){
  return WebClient.create("http://localhost:8080")
    .get()
    .uri("/hello/{name}", "Spring")
    .accept(MediaType.TEXT_PLAIN)
    .exchange()
    .flatMap(r -> r.bodyToMono(String.class))
    .map(d-> d.toUpperCase())
    .flatMap(d-> helloRepository.save(id));
 }
```
> 함수형 스타일의 코드가 읽기 어렵다면
 * 각 단계의 타입이 보이지 않기 때문이다.
 * 타입이 보이도록 코드를 재구성하고 익숙해지도록 연습이 필요하다
```
 @GetMapping("/webclient")
 Mono<String> webclient(){
   WebClient wc = WebClient.create("http://localhost:8080");
   UriSpec<RequestHeaderSpec<?>> uriSpec = wc.get();
   RequestHeaderSpec<?> headerSpec = uriSpec.uri("/hello/{name}", "Spring");
   RequestHeaderSpec<?> headerSpec2 = headerSpec.accept(MediaType.TEXT_PLAIN);
   Mono<ClientResponse> res = headerSpec2.exchange();
   Mono<String> data = res.flatMap(r->r.bodyToMono(String.class));
   Mono<String> upperData = data.map(d->d.toUpperCase());
   return upperData.flatMap(d-> helloRepository.save(d));
 }
``` 
 > 비동기-논블록킹 리액티브 웹 애플리케이션의 효과를 얻으려면
  * WebFlux + 리액티브 리포지토리
            + 리액티브 원격 API 호출
            + 리액티브 지원 외부 서비스
            + @Async 블록킹 IO
  * 코드에서 블록킹 작업이 발생하지 않도록 Flux 스트림
    또는 Mono 스트림에 데이터를 넣어서 전달
  
 > 리액티브 함수형은 꼭 성능 때문만?
  * 함수형 스타일 코드를 이용해 간결하고 읽기 좋고 조합하기 편한 코드 작성
  * 데이터 흐름에 다양한 오퍼레이터 적용
  * 연산을 조합해서 만든 동시성 정보가 노출되지 않는 추상화된 코드 작성
   - 동기, 비동기, 블록킹, 논블록킹 등을 유연하게 적용
  * 데이터의 흐름의 속도를 제어할 수 있는 메커니즘 제공
  
 > 논블록킹 IO에만 효과가 있나?
  * 시스템 외부에서 발생하는 이벤트에도 유용
  * 클라이언트로부터의 이벤트에도 활용 가능
 
 > ReacticeStreams
  * WebFlux가 사용하는 Reactor외에 RxJava2를 비롯한 다양한 리액티브 기술에 적용된 표준 인터페이스
  * 다양한 기술, 서비스 간의 상호 호환성에 유리
  * 자바9에 Flow API로 포함
  
 > 무엇을 공부해야 하나?
  * 자바 8+ 함수형 프로그래밍에 익숙해질 것
  * CompletableFuture와 같이 비동기 작업의 조합, 결합에 뛰어난 툴의 사용법을 익힐 것
  * ReactorCore 학습
   - Mono/Flux, 오퍼레이터, 스케줄러
  * WebFlux와 스프링의 리액티브 스택 공부
  * 비동기 논블록킹 성능과 관련된 벤치마킹, 모니터링, 디버깅 연구
  * 테스트
  
 > 스프링 리액티브 라이브 코딩은 youtube.com/tobyleetv
