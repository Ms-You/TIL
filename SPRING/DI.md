## 의존성 주입 (DI - Dependency Injection) ##
의존성 주입이란 한 오브젝트가 의존하는 오브젝트를 직접 생성하는 것이 아니라 외부에서 넘겨받는 것을 의존성 주입이라 함

```
public class FoodService {
  private final ChefServiceImpl chefServiceImpl;

  public void FoodService() {
    this.chefServiceImpl = new ChefServiceImpl();
  }
}
```
위 코드에서 FoodService 클래스는 ChefServiceImpl에 의존하고 있음

만약 ChefServiceImpl를 사용하는 서비스가 100개 일 때, ChefServiceImpl 대신 MasterChefServiceImpl로 변경하고자 하면 100개의 클래스를 돌아다니면서 ChefServiceImpl 대신 MasterChefServiceImpl를 생성하도록 일일이 수정해야 함

또한 위와 같은 코드로는 단위 테스트가 어려움

Mock 클래스를 사용하더라도 생성자 내부에서 ChefServiceImpl를 생성하기 때문임

이런 문제점을 해결하는 것이 의존성 주입임

<br />

### 의존성 주입 방식 ###
의존성 주입 방식에는 필드 주입, 수정자 주입, 생성자 주입 3가지 방식이 있음

#### 필드 주입 ####
```
public class FoodService {
  @Autowired
  private ChefService chefService;  // 필드를 이용한 의존성 주입
}
```
<b>장점</b>
- @Autowired만 선언해주면 되므로 사용하기 간편함
- 여러 필드에 대해 동시에 의존성을 주입할 수 있어 코드의 가독성이 높음

<b>단점</b>
- 외부에서 접근할 수 없어 테스트를 위한 Mock 객체를 주입하기 어려워 테스트 코드에서 사용하기 어려움
- final 키워드를 사용할 수 없어 객체의 불변성을 보장할 수 없음
- 객체의 생성 시점과 의존성 주입 시점이 다르기 때문에 순환 참조 에러를 사전에 발견할 수 없음

<br />
<br />

#### 수정자 주입 ####
```
public class FoodService {
  private ChefService chefService;

  public void setChefService(ChefService chefService) {   // 수정자를 이용한 의존성 주입
    this.chefService = chefService;
  }
}
```
<b>장점</b>
- 객체 생성 이후에 의존성을 주입하기 떄문에 객체의 생명주기에 관계없이 의존성을 변경할 수 있음
- Mock 객체를 주입하기 쉬워 단위 테스트가 용이함

<b>단점</b>
- final 키워드를 사용할 수 없어 객체의 불변성을 보장할 수 없음
- 객체의 생성 시점과 의존성 주입 시점이 다르기 때문에 순환 참조 에러를 사전에 발견할 수 없음

<br />
<br />

#### 생성자 주입 ####
```
public class FoodService {
  private final ChefService chefService;

  @Autowired
  public FoodService(ChefServie chefService) {  // 생성자를 이용한 의존성 주입
    this.chefService = chefService;
  }
}

// 또는

@RequiredArgsConstructor
public class FoodService {
  private final ChefService chefService;

  ...
}
```
<b>장점</b>
- 객체를 생성할 때 최초 한 번만 의존성을 주입하므로 객체의 불변성을 보장할 수 있음
- Mock 객체를 생성자에 주입하기 쉬워 단위 테스트가 용이함
- 객체 생성 시점과 의존성 주입 시점이 동시에 이루어지므로 사전에 순환 참조 에러를 발견할 수 있음

<b>단점</b>
- 의존성이 많아질 경우 생성자 매개변수가 길어질 수 있어 코드의 가독성이 떨어질 수 있음

<br />
<br />

#### 순환 참조 에러 ####
서로 다른 빈들이 서로를 참조하고 있는 과정에서 서로를 반복하여 호출하다가 결국에 스택 메모리 영역이 가득 차서 StackOverflowError 를 발생시키고 강제로 종료되는 에러로 런타임 시에 발견할 수 있음

```
@Component
public class A {
  private final B b;

  @Autowired
  public A(B b) {
    this.b = b;
  }

  public void testA() {
    System.out.println("test method of Class A");
    b.test();
  }
}

@Component
public class B {
  private final A a;

  @Autowired
  public B(A a) {
    this.a = a;
  }

  public void testB() {
    System.out.println("test method of Class B");
    a.test();
  }
}
```

- 필드 주입과 수정자 주입은 객체 생성 시점과 의존성 주입 시점이 다르기 때문에 클래스를 먼저 생성하고 의존성을 주입하므로 순환 참조 에러를 사전에 감지할 수 없음

- 생성자 주입 방식에서는 모든 의존성이 생성자의 매개변수로 전달되기 때문에, 스프링 프레임워크가 객체를 생성할 때 모든 의존성을 확인함
- 따라서 순환 참조 에러가 발생하면 스프링 컨테이너가 객체를 생성할 수 없기 때문에 오류가 발생함

