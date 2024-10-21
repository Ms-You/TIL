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

