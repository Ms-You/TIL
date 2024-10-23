## Webflux ##
Webflux는 반응형 웹 애플리케이션 개발을 지원하는 모듈로 Spring 5.0 버전부터 포함되었음

Webflux의 목표는 비동기 및 이벤트 기반으로 더 높은 확장성과 성능을 제공하는 애플리케이션을 개발하는 것임

Reactor는 Reactive Programming을 구현하기 위한 라이브러리 중 하나임

<br />

### Reactive Programming ###
Reactive Programming은 비동기 데이터 스트림을 위한 프로그래밍 패러다임으로, 스레드 대신 데이터 스트림에 초점을 맞추며 이벤트 기반으로 실시간으로 데이터가 생성되거나 변경될 때마다 반응하는 방식임

반응형 프로그래밍은 대부분 비동기 방식으로 동작하며, 명령형 프로그래밍은 대부분 동기 방식으로 동작함

<br />

### Reactive Streams ###
- Non-Blocking, BackPressure를 이용한 비동기 데이터 처리의 표준
```
public interface Publisher<T> {
  public void subscribe(Subscriber<? super T> s);  // Subscriber를 Publisher에 등록하고 데이터를 수신할 준비가 되었음을 Publisher에게 알림
}

public interface Subscriber<T> {
  public void onSubscribe(Subscription s);  // Subscription을 통해 받아들일 데이터 양을 정함
  public void onNext(T t);  // Publisher가 데이터를 전달할 때마다 호출
  public void onError(Throwable t);  // 데이터 전송 중 에러가 발생함
  public void onComplete();  // 모든 데이터가 성공적으로 전송됨
}

public interface Subscription {
  public void request(long n);  // n개의 데이터 요청
  public void cancel();  // 구독 취소
}
```

<br />

### Reactor ###
Reactor 라이브러리는 Reactive Programming을 위한 표준 API인 Reactive Streams의 구현체로 비동기 지원을 위해 함수형 프로그래밍 형태로 구현되어 있음

Reactor는 Publisher-Subscriber 패턴을 중심으로 동작함
- Publisher는 데이터를 생성하고 전달하는 컴포넌트
  - 기본적으로 Subscriber가 등록되기 전까지는 아무런 일도 하지 않음
  - Subscriber가 등록되는 시점부터 데이터를 push함
- Subscriber는 데이터를 받아서 처리하는 컴포넌트
- Subscription은 Publisher가 생성한 구독 정보
  - request() 메서드에 의해 BackPressure가 가능함

또한 Reactive Streams의 Publisher 구현체로  Mono와 Flux가 존재함 (Publisher<T>를 상속받고 있음)
  - 따라서 Subscriber가 subscribe() 하기 전까지는 아무런 반응이 없음

<br/>

#### Mono ####
- Mono<T>: 0~1 개의 데이터를 전달함
  - onNext() 이벤트가 발생하면 이 때부터 방출함
  - onComplete() 이벤트가 발생하면 완료하고, onError() 이벤트가 발생하면 에러를 발생함
- 단일 결과를 처리할 때 유용함

#### Flux ####
- Flux<T>: 0~n 개의 T 타입 원소를 방출함
  - onNext() 이벤트가 발생하면 이 때부터 방출함
- 여러 개의 결과를 처리하거나 스트리밍 데이터를 다룰 때 적합함

<br />

Mono와 Flux를 구분해서 사용하면 데이터의 흐름을 효과적으로 관리할 수 있다고 생각함

Mono는 오직 0~1개의 데이터를 전달하는데, 이를 Subscriber는 Publisher가 전달하는 데이터가 0개면 하던 작업을 진행하고, 1개면 이를 받은 후에 하던 작업을 진행.

Flux는 0~n개의 데이터를 전달하는데, Subscriber는 아직 받아야 될 데이터가 있으니까 이를 알려주는 목적으로 구분되는 것이 아닐까?

이는 Subscriber가 어떻게 반응해야 하는지 알려주기 때문에 효과적이라고 생각함


### Cold / Hot Publisher ###
<b style="color: blue">Cold Publisher</b>는 각 구독에 대해 독립적인 데이터 흐름을 생성하며, 구독이 없으면 데이터가 생성되지 않음
- 기본적으로 Mono와 Flux는 Cold로 동작
- 각 구독에 대해 항상 데이터를 새로 생성함
  - 즉, 각 Subscriber는 독립적으로 데이터를 요청하며, 각 Subscriber가 구독할 때마다 새로 생성된 데이터 스트림을 가짐

<b style="color: red">Hot Publisher</b>는 구독자 수에 관계 없이 데이터 스트림이 즉시 시작되며, 이미 생성된 데이터 스트림을 여러 Subscriber가 공유할 수 있음
- 구독 전에 데이터 스트림을 시작할 수 있음
  - 즉, Subscriber의 존재와 관계없이 즉시 데이터 전송을 시작함
- 반복적으로 발생하는 데이터 스트림을 미리 동작해서 공유하면 중복을 방지할 수 있음
  - Flux.subscribeOn(Schedulers.single())을 통해서 하나의 스레드에서 수행하게 할 수 있음
  - Flux.publishOn()을 통해서 각 구독자 동작을 별도의 스레드에서 수행하면 효율적인 병렬 처리가 가능함

<br />
<br />

### Operators of Reactor ###
Publisher -> [Data 1] -> Op1 -> [Data2] -> Op2 -> [Data3] -> Subscriber

Mono와 Flux의 Operators를 이용해서 데이터를 조작할 수 있음

flatMap, concatMap, flatMapSequential 모두 비동기 작업을 수행하고 결과를 병합하는 역할을 하는 Operator

<br />

#### map ####
- map은 입력 요소를 변환하는 데 사용됨
- 각 요소에 대해 함수가 적용돼서 새로운 스트림을 반환함
- 단순한 변환에 사용되며, 비동기 작업이 아닌 경우에 적합함
- <b>반환 타입이 Mono, Flux가 아닌 경우에 사용</b>

<br />

#### flatMap ####
- 원본 스트림의 각 항목에 대해 변환 함수를 비동기적으로 적용하고, 그 결과를 평탄화를 통해 하나의 새로운 스트림으로 병합함
- 원본 스트림의 순서를 보장하지 않아 결과 스트림의 순서는 원본 스트림과 다름
- 비동기 작업을 수행하고, 결과를 평탄화하는 데 사용
- <b>각 요소가 반환하는 타입이 Mono, Flux인 경우에 사용</b>
```
public static void main(String[] args) {
  Flux<Integer> numbers = Flux.just(1, 2, 3);

  numbers
    .flatMap(number -> asyncOperation(number))
      .subscribe(result -> System.out.println("Result: " + result));
}

private static Mono<String> asyncOperation(int number) {
  return Mono.just("Processed " + number)
    .delayElement(java.time.Duration.ofSeconds(1));  // 비동기적 지연
}
```
- 원래의 입력 순서(1, 2, 3)와 다를 수 있음

<br />

#### concatMap ####
- 원본 스트림의 각 항목에 대해 변환 함수를 적용하고 결과를 병합
- flatMap과 달리 이전 작업이 완료될 때까지 다음 작업을 시작하지 않아 원본 스트림의 순서가 결과 스트림에서도 유지됨
- 게시글을 작성 시간에 따라 내림차순으로 정렬 후 조회할 때, flatMap을 사용해서 가공하면 결과가 내림차순과 다를 수 있지만, concatMap은 순차적으로 작업을 수행하기 때문에 결과가 내림차순과 동일함
```
public static void main(String[] args) {
  Flux<Integer> numbers = Flux.just(1, 2, 3);

  numbers
    .concatMap(number -> asyncOperation(number))
      .subscribe(result -> System.out.println("Result: " + result));
}

private static Mono<String> asyncOperation(int number) {
  return Mono.just("Processed " + number)
    .delayElement(java.time.Duration.ofSeconds(1));  // 비동기적 지연
}
```
- 원래의 입력 순서(1, 2, 3)와 같음

<br />

#### flatMapSequential ####
- 원본 스트림의 각 항목에 대해 변환 함수를 비동기적으로 적용하되, 반환된 모든 스트림을 원본의 순서대로 병합함
- flatMap과 concatMap의 중간적인 역할을 함
- 입력 요소에 대해 비동기 작업을 수행하고 작업이 완료되면 순서에 따라 결과를 발행함
  - 이전 작업이 완료되기 전까지 다음 작업을 시작하지 않음
```
public static void main(String[] args) {
  Flux<Integer> numbers = Flux.just(1, 2, 3);

  numbers
    .flatMapSequential(number -> asyncOperation(number))
      .subscribe(result -> System.out.println("Result: " + result));
}

private static Mono<String> asyncOperation(int number) {
  return Mono.just("Processed " + number)
    .delayElement(java.time.Duration.ofSeconds(1));  // 비동기적 지연
}
```
- 원래의 입력 순서(1, 2, 3)와 같음

<br />

=> concatMap과 flatMapSequential은 모두 원본 순서를 보장하지만 처리 방식에 차이가 있음
- concatMap
  - 각 작업이 순차적으로 실행되며 이전 작업이 완료될 때까지 다음 작업을 수행하지 않기 때문에, 순서 보장과 동시성 활용 모두 단순함
- flatMapSequential
  - 비동기적으로 작업을 수행하지만 결과는 순서에 따라 방출되므로, 작업이 완료되는 순서와는 상관 없이 원본 스트림의 순서를 유지함

ex) 작업 A(1초), B(3초), C(1초)가 있을 때
1. concatMap:
  - 처리 순서: A -> B -> C
  - 총 걸리는 시간: 1초 + 3초 + 1초 = 5초

2. flatMap:
  - 처리 순서: A와 B, C가 동시에 시작됨
  - 총 걸리는 시간: 3초

3. flatMapSequential:
  - 처리 순서: A -> B -> C
  - 총 걸리는 시간: 1초 + 3초 + 1초 = 5초

<br />

#### doOnNext ####
- 보통 로그를 남길 때 주로 사용
- 일반적으로 스트림과 관계 없는 side effect에 불과하므로 flatMap과 분리하면 비즈니스 로직에 온전히 집중할 수 있음
```
// flatMap만 사용한 코드
public Flux<Integer> justFlatMap() {
  return Flux.just(1, 2, 3)
    .flatMap(num -> {
      log.info("num: {}", num);

      return reactiveOperation(num);
    });
}

// doOnNext 사용 코드
public Flux<Integer> withDoOnNext() {
  return Flux.just(1, 2, 3)
    .doOnNext(num -> log.info("num: {}", num))
    .flatMap(num -> reactiveOperation(num));
}
```

<br />

#### filter ####
- element에 predicate를 적용해 값이 true라면 그대로 방출하고, false라면 방출하지 않음
- flatMap 내부에서 if 구무늘 통해 empty signal을 내보내고 있다면 flatMap으로만 작성한 코드를 다음과 같이 filter로 대체할 수 있음
```
// flatMap만 사용한 코드
public Flux<Integer> justFlatMap() {
  return Flux.just(-1, 0, 1)
    .flatMap(num -> {
      if (num < 0) {
        return Mono.empty();
      }
      return reactiveOperation(num);
    });
}

// filter 사용 코드
public Flux<Integer> withFilter() {
  return Flux.just(-1, 0, 1)
    .filter(num -> num >= 0)
    .flatMap(num -> reactiveOperation(num));
}
```

<br />

#### defaultIfEmpty와 switchIfEmpty ####
- filter는 empty Publisher를 만들 가능성이 있으므로 의도치 않은 동작을 할 수 있음
- 이런 경우 기본 값을 제공해주는 defaultIfEmpty나 switchIfEmpty 같은 오퍼레이터를 사용하면 좋음
```
// 아래 코드는 flatMap의 if 구문 안에서 empty가 아닌 Publisher를 반환함
public Flux<Integer> onlyFlatMap() {
  return Flux.just(-1, 0, 1)
    .flatMap(num -> {
      if (num < 0) {
        return Mono.error(new RuntimeException());
      }
      return reactiveOperation(num);
    });
}

// switchIfEmpty 사용 코드
public Flux<Integer> useSwitchIfEmpty() {
  return Flux.just(-1, 0, 1)
    .filter(num -> num >= 0)
    .flatMap(num -> reactiveOperation(num))
    .switchIfEmpty(Mono.error(new RuntimeException()));
}

// defaultIfEmpty 사용 코드
public Flux<Integer> useDefaultIfEmpty() {
  return Flux.just(-1, 0, 1)
    .filter(num -> nnum >= 0)
    .flatMap(num -> reactiveOperation(num))
    .defaultIfEmpty(0);
}
```
- reactiveOperation()가 empty Publisher를 반환하면 처음 코드에서는 그대로 empty가 방출됨
- 두 번째 코드에서는 error signal이 방출됨
- defaultIfEmpty는 empty Publisher가 방출될 때 기본값을 방출함
- switchIfEmpty는 empty Publisher가 방출될 때 대체 Publisher를 사용해서 다른 동작을 수행할 수 있음

<br />

#### filterWhen ####
- Inner Publisher가 반환하는 boolean 값으로 필터를 적용하는 Operator
- Inner Publisher가 반환하는 boolean 값이 true면 통과, false 또는 complete면 차단함
```
// flatMap만 사용한 코드
public Flux<Integer> onlyFlatMap() {
  return Flux.just(1, 2, 3)
    .flatMap(id -> {
      return getNameById(id)
        .flatMap(name -> {
          if ("john".equals(name)) {
            return Mono.empty();
          }
          return reactiveOperation(id);
        })
    });
}

// filterWhen 사용 코드
public Flux<Integer> useFilterWhen() {
  return Flux.just(1, 2, 3)
    .filterWhen(id -> getName(id)
      .map(name -> !"john".equals(name)))
    .flatMap(id -> reactiveOperation(id));
}

// 메서드를 분리 후 filterWhen 사용 코드
public Flux<Integer> useFilterWhen() {
  return Flux.just(1, 2, 3)
    .filterWhen(this::isNotJohn)
    .flatMap(this::reactiveOperation);
}

private Mono<Boolean> isNotJohn(int id) {
  return getName(id)
    .map(name -> !"john".equals(name))
    .onErrorReturn(false);
}
```
- 주의할 점은 앞의 element의 Inner Publisher가 complete signal을 보낸 후에 다음 element에서 Inner Publisher를 구독함
- 즉, flatMap처럼 Inner Publisher를 비동기적으로 구독하지 못하기 때문에 동시성이 떨어져 성능이 저하될 수 있음
- Outer Publisher가 Flux이면 filterWhen은 비동기적으로 동작하지 않음

<br />

- <b>Outer Publisher</b>: flatMap, filterWhen 등의 연산자가 적용되는 원본 스트림을 생성하는 Publisher를 의미함
  - 예를 들어, outerFlux.flatMap(item -> anotherFlux) 에서 outerFlux가 Outer Publisher임
- <b>Inner Publisher</b>: 각 element에 적용되는 함수 내부에서 반환되는 새로운 스트림을 생성하는 Publisher를 의미함
  - 예를 들어, flux.flatMap(item -> innerFlux) 에서 각 element에 대해 반환되는 innerFlux가 Inner Publisher임

<br />

#### zip ####
- 동시에 여러 reactive operation을 호출하고자 할 때는 Mono.zip()을 많이 사용함
- Mono.zip()은 여러 source Mono를 동시에 구독한 후 튜플로 합쳐서 반환하는 operation임
- flatMap으로 여러 Mono Publisher의 결과물을 같이 사용하려면 어쩔 수 없이 중첩되게 사용해야 하는데 Mono.zip()이 이런 문제를 개선해줌
```
// flatMap 사용 코드
public Mono<String> onlyFlatMap() {
  return getId()
    .flatMap(id -> {
      return getName(id)
        .flatMap(name -> {
          return reactiveOperation(id, name)
        });
    })
}

// zip 사용 코드
public Mono<String> withZip() {
  return Mono.zip(getId(), getName())
    .flatMap(tuple -> {
      int id = tuple.getT1();
      String name = tuple.getT2();

      return reactiveOperation(id, name);
    });
}
```

<br />

#### zipWhen ####
- zip()은 여러 Mono가 독립적일 때 사용함
- 만약, Mono가 다른 Mono에 의존성이 있다면 zipWhen()을 사용
- zipWhen()은 한 Mono의 결과물로 다른 Mono의 결과물을 만들고, 이 두 Mono의 결과물이 모두 필요한 상황에서 사용할 수 있음
```
// flatMap 사용 코드
public Mono<String> onlyFlatMap() {
  return getId()
    .flatMap(id -> {
      return getName(id)    // getName()가 getId()의 결과에 의존하고 있음
        .flatMap(name -> {
          return reactiveOperation(id, name);
        });
    });
}

// zipWhen 사용 코드
public Mono<String> withZipWhen() {
  // flatMap의 중첩 없이 두 결과물의 튜플을 downStream으로 전달함
  return getId()
    .zipWhen(id -> getName(id))
    .flatMap(tuple -> {
      int id = tuple.getT1();
      String name = tuple.getT2();

      return reactiveOperation(id, name);
    });
}
```

<br />

