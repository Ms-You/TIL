## @RequestBody ##
HTTP 요청의 본문(body)을 자바 객체로 변환하는 데 사용됨

주로 RESTful에서 클라이언트가 서버에 JSON 또는 XML 형식으로 데이터를 전송할 때 사용

<br />

### Jackson 라이브러리 ###
자바에서 JSON 데이터를 처리하기 위해 사용되는 라이브러리

HTTP 요청 및 응답의 JSON 변환을 자동으로 처리해줌

<br />

<b>직렬화</b>: 자바 객체를 JSON 형식으로 변환

<b>역직렬화</b>: JSON 형식을 자바 객체로 변환하는 과정으로 @RequestBody는 역직렬화 과정을 자동으로 처리함

<br />
Jackson 라이브러리는 직렬화 할 때는 getter, 역직렬화 할 때는 setter를 사용함

직렬화 시 Jackson은 객체의 필드 값을 읽기 위해 해당 필드에 대한 getter 메서드를 호출함

역직렬화 시 Jackson은 JSON 데이터를 객체의 필드에 설정하기 위해 해당 필드에 대한 setter 메서드를 호출함

<br />

### POST 요청 시 ###
@RequestBody 어노테이션은 POST, PUT, PATCH, DELETE 등의 요청 시 사용 가능함

그렇다면 @RequestBody 모델에 기본 생성자, getter, setter 메서드가 필요한가?
- @RequestBody에 사용되는 DTO 들은 setter가 필요 없음
- @RequestBody로 JSON 데이터가 전달되면 자바 객체로 역직렬화 해주는 것을 Jackson2HttpMessageConverter에서 해주는데 이 converter 내부적으로 ObjectMapper를 사용해서 자바 객체로 변환해주기 때문
- ObjectMapper는 기본 생성자와 getter 혹은 setter 혹은 public field를 보고 property 명을 찾음
- 따라서 기본 생성자와 getter만 있어도 property 명을 찾고, java.lang.reflect 패키지를 사용해 값을 직접 주입시켜 주기 때문에 굳이 setter가 필요 하지 않음

<br />
<br />

- ObjectMapper 내부에는 property와 생성자가 위임된 경우, 그 정보를 이용해서 직렬화/역직렬화에 사용하는 로직이 있음
- 즉, <b style="color: skyblue">Jackson 라이브러리는 필드와 생성자 정보를 이용해서 직렬화/역직렬화를 수행함</b>
- @JsonProperty, @JsonAutoDetect, @JsonCreator 어노테이션이 작성되면 property 값이 위임되어 getter, setter, 기본 생성자 없이도 Jackson 라이브러리가 잘 동작함
- 이러한 어노테이션들은 Jackson 라이브러리가 객체의 필드에 접근하는 방법을 제어함
- jackson-datatype-jdk8 을 임포트 해서 사용한다면, 이는 자동으로 기본 생성자가 없으면 다른 생성자를 이용하도록 하는 모듈을 ObjectMapper에 등록시켜줌
- 주의할 점은 생성자 인자가 하나일 경우에는 @JsonCreator 어노테이션을 선언해줘야 함

```
public class User {
  private String name;

  // 생성자의 인자가 하나일 때 @JsonCreator 어노테이션 사용
  @JsonCreator
  public User(@JsonProperty("name") String name) {
    this.name = name;
  }

  // getter
  public String getName() {
    return name;
  }
}
```

<br />

### GET 요청 시 ###
Get 요청의 경우 JSON 데이터가 아닌 Query Parameter 이므로 Jackson2HttpMessageConverter가 동작하지 않음

- 이 때는 스프링에서 WebDataBinder를 사용하는데 기본 값으로 값을 할당하는 방법이 JavaBean 방식임
- WebDataBinder는 요청 파라미터를 자바 객체로 바인딩하는 역할을 함
- 주의할 점으로는 DTO에 setter가 없다면 실패함
- 자바 빈 방식은 setter를 통해서 값을 할당하기 때문에 setter가 없으면 동작하지 않음

<br />

그렇다면 Get 요청 시 setter를 사용하지 않는 방법은 없는가?

- initDirectFieldAccess()를 사용하면 WebDataBinder가 직접 필드에 접근하여 값을 할당할 수 있음
- 모든 컨트롤러에 대해 WebDataBinder의 설정을 적용하기 위해 @ControllerAdvice를 사용하면 됨

```
class UserDTO {  // setter를 사용하지 않음
  private String name;
  private int age;

  public String getName() {
    return name;
  }

  public int getAge() {
    return age;
  }
}


@ControllerAdvice
public WebControllerAdvice {
  @InitBinder
  public void initBinder(WebDataBinder binder) {
    binder.initDirectFieldAccess();  // 필드에 직접 접근하여 값을 할당
  }
}


@RestController
@RequestMapping("/api")
public class UserController {
  
  @RequestMapping("/user")
  public String getUser(UserDTO userDTO) {
    return "User " + userDTO.getName() + ", Age: " + userDTO.getAge();
  }
}
```