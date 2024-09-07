## REST API ##
REST (Representational State Transfer)
- HTTP URI를 통해 자원을 명시하고, HTTP Method를 통해 명시된 자원의 행위를 적용
- REST는 '자원', '행위', '표현'으로 구성된 아키텍처로 볼 수 있음
  - Resource - URI
  - Verb - HTTP Method
  - Representations
- REST API는 플랫폼 독립적이기 때문에 HTTP 기반이면 웹 브라우저에 상관 없이 동일하게 동작함
- REST API를 사용하면 Uniform Interface의 Self-Descriptive와 HATEOAS 같은 특징 덕분에 서버와 클라이언트 간에 협업이 용이하다는 장점이 있음
- 하지만 정해진 표준 규약이 없기 때문에 일관된 REST API를 설계하기 어려움
- 또한, Uniform Interface를 만족하기 어렵기 때문에 완벽한 REST API를 설계하기 어렵다는 단점이 있음

<br />
<br />

### REST API의 특징 ###
1. <b>Client-Server</b>
- 리소스를 관리하는 서버가 존재하고, 다수의 클라이언트가 리소스를 소비하려고 네트워크를 통해 서버에 접근하는 구조

2. <b>Stateless</b>
- 클라이언트가 서버에 요청을 보낼 때 이전 요청의 영향을 받지 않음
- 클라이언트는 서버에 요청할 때마다 요청에 리소스를 받기 위한 모든 정보를 포함해야 함
- 서버는 작업을 위한 상태 정보를 보관하거나 관리하지 않고 클라이언트의 요청만 처리하면 됨

3. <b>Cacheable</b>
- 서버에서 리소스를 응답할 때 캐시가 가능한지 아닌지를 명시할 수 있어야 함
- HTTP 에서는 cache-control이라는 헤더에 리소스의 캐시 여부를 명시할 수 있음

4. <b>Layered System</b>
- 클라이언트가 서버에 요청할 때 여러 개의 레이어로 된 서버를 거칠 수 있음
- 서버가 인증 서버, 로드 밸런서, 암호화 계층을 통해 애플리케이션에 도착할 때, 레이어들은 요청과 응답에 어떠한 영향을 미치지 않음
- 클라이언트는 서버의 레이어 존재 유무를 알지 못함

5. <b>Uniform Interface</b>
- 클라이언트와 서버 간의 상호작용을 단순하고 일관되게 유지해줌

<br />

### Uniform Interface의 특징 ###
1. <b>Self-Descriptive</b>
- 서버가 응답해주는 메시지 자체에 필요한 모든 정보를 포함하여 응답 받는 클라이언트가 메시지의 모든 요소가 어떤 의미인지 알 수 있도록 하는 원칙
- 즉, 클라이언트는 응답에서 리소스에 대한 메타 데이터, 상태 코드, 링크 등을 통해 자원의 의미와 사용 방법을 이해할 수 있음
- Self-Descriptive를 위해서 서버는 응답해주고자 하는 데이터의 Content-type header에 있는 MediaType을 정확히 명시해줘야 함
- 예를 들어, 응답해주고자 하는 데이터의 형태가 JSON이면 MediaType을 "application/json"으로 명시해줘야 함

<br />

2. <b>HATEOAS (Hypermedia as the Engine of Application State)</b>
- 애플리케이션의 상태는 링크를 통해서 전이되어야 한다는 원칙
- 클라이언트가 애플리케이션의 상태를 전환하는 데 필요한 모든 정보를 서버가 제공함
- 즉, 클라이언트는 서버의 응답에서 다음에 수행할 수 있는 작업에 대한 링크를 제공받아, 자원 간의 관계를 탐색할 수 있음
- 예를 들어, 클라이언트가 서버에 주문 정보를 요청했을 때 서버는 주문 정보와 함께 '주문 취소 링크'나 '주문 상세 보기 링크'도 응답해줘야 함
```
@GetMapping("order/{id}")
public Order getOrderInfo(@PathVariable Long id) {
  Order order = getOrder(id);

  if (order != null) {
    // 주문 취소 링크
    Link cancelLink = WebMvcLinkBuilder.linkTo(WebMvcLinkBuilder.methodOn(OrderController.class).cancelOrder(id)).withRel("cancel-order");
    order.add(cancelLink);

    // 주문 상세 보기 링크
    Link orderDetailsLink = WebMvcLinkBuilder.linkTo(WebMvcLinkBuilder.methodOn(OrderController.class).getOrderDetails(id)).withRel("order-details");
    order.add(orderDetailsLink);
  }

  return order;
}
```

<br />
<br />

### REST API 디자인 가이드 ###
1. URI는 정보의 자원을 표현해야 함
- 리소스 명은 동사보다는 명사를 사용할 것
  - DELETE /members/delete/1 (X)
  - delete와 같은 행위에 대한 표현이 들어가서는 안됨

2. 자원에 대한 행위는 HTTP Method로 표현함
- 위의 잘못된 URI는 아래와 같이 변경될 수 있음
  - DELETE /members/1 (O)

→ URI는 자원을 표현하는데 집중하고, 행위에 대한 정의는 HTTP Method를 통해 하는 것이 RESTful한 API를 설계하는 중심 규칙임

<br />
<br />

#### URI 설계 시 주의할 점 ####
1. <b>/는 계층 관계를 나타내는 데 사용함</b>
- http://localhost:8080/houses/apartments
- http://localhost:8080/animals/mammals/whales

2. <b>URI 마지막 문자로 /를 포함하지 않아야 함</b>
- URI에 포함되는 모든 글자는 리소스의 유일한 식별자로 사용되어야 하며, URI가 다르다는 것은 리소스가 다르다는 것이고, 리소스가 다르면 URI도 달라져야 함
- REST API는 명확한 URI를 통해 통신해야 하므로 혼동을 주지 않기 위해 URI 경로 마지막에 /를 사용하지 않음
- http://localhost:8080/houses/apartments/ (X)
- http://localhost:8080/houses/apartments (O)

3. <b> - 은 URI 가독성을 높이는데 사용함</b>
- URI를 쉽게 읽고 해석하기 위해 긴 URI 경로를 사용한다면 -을 통해 가독성을 높일 수 있음
- http://localhost:8080/rest-api

4. <b> _ 은 URI에 사용하지 않음</b>
- 밑줄은 보기 어렵거나 밑줄 때문에 문자가 가려지기도 하므로 가독성을 위해 -을 대신 사용

5. <b>URI 경로에는 소문자가 적합함</b>
- 대소문자에 따라 다른 리소스로 인식되기 때문에 URI 경로에 대문자 사용은 피해야 함

6. <b>파일 확장자는 URI에 포함시키지 않음</b>
- REST API에서는 메시지 바디 내용의 포맷을 나타내기 위한 파일 확장자를 URI에 포함시키지 않고, Accept header를 사용함
- http://localhost:8080/members/photo.jpg (X)

<br />
<br />

#### 리소스 간의 관계를 표현하는 방법 ####
REST 리소스 간에는 연관 관계가 있을 수 있고, 이런 경우 아래와 앝은 표현 방법을 사용함
- / 리소스명/리소스 ID/관계가 있는 다른 리소스명
- GET : /users/{userid}/devices (소유의 관계를 표현할 때)

만약 관계가 복잡하다면 이를 서브 리소스에 명시적으로 표현하는 방법이 있음

예를 들어 사용자가 좋아하는 동물 목록을 표현해야 할 경우 아래처럼 사용 가능함
- GET : /users/{userid}/likes/animals (관계가 애매하거나 구체적 표현이 필요할 때)

<br />
<br />

