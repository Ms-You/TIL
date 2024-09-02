# JPA 관계 설정 #
객체와 테이블을 매핑하고 연관 관계를 설정하기 위함

## 연관 관계 방향 ##
관계 설정은 단방향, 양방향으로 관계를 맺을 수 있음

데이터베이스 테이블의 경우 방향이 외래키를 통해 조인이 가능해 방향이 필요 없지만, 객체에서는 필드를 통해 다른 객체를 참조하기 때문에 방향이 필요함

즉, 방향은 객체 관계에서만 존재하며, 테이블 관계는 항상 양방향임

1. <b style="color: orange">단방향</b>
- 회원 → 주문처럼 한 쪽만 참조가 필요하다면 단방향으로 설정 (member.getOrder();)

```
@Entity
public class Project {
  @OneToOne
  @JoinColumn(name = "project_award_id")
  private ProjectAward projectAward;
}

// ProjectAward에서는 Project 객체를 참조하지 않음
```

<br />

2. <b style="color: orange">양방향</b>
- 엄밀하게 양방향은 아니고 두 객체가 서로 단방향으로 참조해서 양방향처럼 사용하는 것을 말함
- 회원 → 주문 (member.getOrder();), 주문 → 회원 (order.getMember();) 과 같이 둘 다 참조해야 한다면 양방향으로 설정

<br />

### ※ 단방향 관계가 필요한 이유 ###
양방향으로 관계를 설정했을 경우, 한 객체에서 다른 객체를 쉽게 참조 할 수 있음

하지만 연관 관계를 많이 가지고 있는 객체의 경우, 모든 엔티티를 양방향으로 설정하면 불필요한 연관 관계로 인해서 엔티티의 복잡성이 늘어나게 됨

따라서 우선 단방향으로 연관 관계를 설정하고 나중에 필요한 경우에만 양방향으로 설정하는 것이 좋음

<br />
<br />

## 연관 관계의 주체 ##
객체를 양방향 연관 관계로 설정하려면 연관 관계의 주체를 정해야 함

연관 관계의 주체를 지정하는 것은 객체의 제어(조회, 저장, 수정, 삭제)에 대한 권한을 어떤 객체가 갖는지 정하는 것임

연관 관계의 주체가 아닌 객체는 조회만 가능함

보통 외래키를 갖는 엔티티가 연관 관계의 주체가 됨

@JoinColumn 어노테이션을 통해 두 엔티티 간의 관계에서 외래키를 매핑할 수 있음
- @JoinColumn 어노테이션의 name 속성은 테이블에서 외래키 컬럼의 이름을 지정함
- refrerenceColumnName 속성은 외래키가 참조하는 대상 엔티티의 컬럼을 지정하는데, 디폴트로 설정되어 있어 기본적으로 대상 엔티티의 기본키를 참조함

mappedBy 속성은 양방향 연관 관계에서 주체가 아닌 종속체 엔티티에서 사용함

관계의 주체가 되는 엔티티에서의 종속체 필드 명을 지정하여, JPA가 어느 엔티티가 관계의 주체인지 알 수 있게 함

```
@Entity
public class Member {
  @ManyToOne
  @JoinColumn(name = "team_id")  // 외래키 매핑
  private Team team;
}

@Entity
public class Team {
  @OneToMany(mappedBy = "team")  // Member의 team 필드를 지정하여 team 필드에 의해 관리된다는 것을 의미
  private List<Member> members;
}
```

