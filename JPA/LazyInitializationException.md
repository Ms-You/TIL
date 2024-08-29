## LazyInitializationException ##
주로 지연 로딩 방식으로 설정된 연관된 엔티티에 접근하려 할 때 발생하는 에러

```
@Entity
public class Member {
  @Id
  private Long id;
  
  @OneToMany(mappedBy="member")
  private List<Order> orders;
}

@Entity
public class Order {
  @Id
  private Long id;
  
  @ManyToOne(fetch=FetchType.LAZY)
  @JoinColumn(name="member_id")
  private Member member;
}

// 서비스 메서드
@Transactional
public void test(Long memberId) {
	Member member = em.find(Member.class, memberId);
	em.close();
	
	List<Order> orders = member.getOrders();  // LazyInitializationException 발생
}
```

#### 발생 상황 ####
1. 비영속 상태
    - 엔티티 매니저가 닫힌 후, 즉 영속성 컨텍스트가 종료된 후에 지연 로딩된 엔티티를 접근하려 할 때 발생
    - 영속성 컨텍스트가 종료되면 해당 컨텍스트에 있는 엔티티의 상태를 관리할 수 없기 때문에 지연 로딩된 데이터를 접근할 수 없어 예외가 발생함

2. 트랜잭션이 종료된 후에 지연 로딩된 필드를 접근하려 할 때 발생
    - @Transactional 어노테이션이 선언된 메서드가 종료되면 Hibernate 세션도 함께 종료되기 때문

<br />

#### 예시 상황 ####

<br />

#### 안티 패턴 ####
1. <b style="color:skyblue">spring.jpa.open-in-view=true (기본값은 false)</b> [OSIV 활성화]
- @Transactional 어노테이션이 선언된 메서드가 종료되어도 컨트롤러 리턴 시점까지 세션을 유지시키는 방법
- DB 커넥션을 계속 사용하기 때문에 세션의 수명이 길어지므로 권장 X

2. <b style="color:skyblue">spring.jpa.properties.hibernate.enable_lazy_load_no_trans=true (기본값은 false)</b>
- 세션이 종료되어도 예외를 발생시키지 않고 다른 세션을 사용하여 데이터를 조회하는 방법
- 엔티티 연관 관계의 복잡성과 상황에 따라 커넥션 풀을 고갈시키는 장애를 유발할 수 있어 권장 X

<br />

#### 해결 방법 ####
1. <b style="color:orange">즉시 로딩</b>
- 연관된 엔티티를 즉시 로딩하도록 설정해서 필요한 데이터를 처음부터 로딩하게 함
- 성능 문제 때문에 권장 X

2. <b style="color:orange">명시적 Fetch Join</b>
- JPQL에서 JOIN FETCH를 사용해서 필요한 연관 엔티티를 조회하는 방법
```
public List<Member> findMembersWithOrders() {
  String query = "SELECT m 
  FROM Member m 
  JOIN FETCH m.orders";

  return em.createQuery(query, Member.class).getResultList();
}

// 서비스 메서드
@Transactional
public void test(Long memberId) {
  List<Member> members = findMembersWithOrders();

  Member member = members.stream()
    .filter(m -> m.getId().equals(memberId))
    .findFirst()
    .orElse(null);

  List<Order> orders = member.getOrders();
}
```

3. <b style="color:orange">DTO 사용</b> (권장 O)
- 연관 관계가 설정된 엔티티를 미리 조회하고 트랜잭션이 종료되는 시점에 리턴 타입으로 DTO로 변환하는 방법
```
public class MemberDTO {
  private Long id;
  private List<OrderDTO> orders;
}

public class OrderDTO {
  private Long id;
}

// 서비스 메서드
@Transactional
public MemberDTO getMemberWithOrders(Long memberId) {
  Member member = em.find(Member.class, memberId);

  List<OrderDTO> orderDTOs = member.getOrders().stream()
    .map(order -> new OrderDTO(order.getId()))
    .collect(Collectors.toList());

  return new MemberDTO(member.getId(), orderDTOs);
}
```
