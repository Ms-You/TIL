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

## 다중성 ##
### 일대일 관계 (One-to-One) ###
두 엔티티가 서로 1:1로 연결되는 관계

#### 일대일 단방향 ####
```
@Entity
public class Project {
  @Id @GeneratedValue
  private Long id;

  @OneToOne
  @JoinColumn(name = "project_award_id")
  private ProjectAward projectAward;
}

@Entity
public class ProjectAward {
  @Id @GeneratedValue
  private Long id;

  private String grade;
  ...
}
```

#### 일대일 양방향 ####
```
@Entity
public class Project {
  @Id @GeneratedValue
  private Long id;

  @OneToOne
  @JoinColumn(name = "project_award_id")
  private ProjectAward projectAward;
}

@Entity
public class ProjectAward {
  @Id @GeneratedValue
  private Long id;

  @OneToOne(mappedBy = "projectAward")
  private Project project;

  private String grade;
  ...
}
```

<br />

### 다대일 관계 (Many-to-One) ###
여러 개의 엔티티가 하나의 엔티티와 연결되는 관계로 연관 관계의 주체가 '다' 쪽에 있음

#### 다대일 단방향 ####
```
// Member 엔티티가 연관 관계의 주체
@Entity
public class Member {
  @Id @GeneratedValue
  private Long id;

  @ManyToOne
  @JoinColumn(name = "team_id")
  private Team team;
}

@Entity
public class Team {
  @Id @GeneratedValue
  private Long id;

  private String name;
  ...
}
```

#### 다대일 양방향 ####
```
// Member 엔티티가 연관 관계의 주체
@Entity
public class Member {
  @Id @GeneratedValue
  private Long id;

  @ManyToOne
  @JoinColumn(name = "team_id")
  private Team team;
}

@Entity
public class Team {
  @Id @GeneratedValue
  private Long id;

  @OneToMany(mappedBy = "team")
  private List<Member> members = new ArrayList<>();

  private String name;
  ...
}
```
<br />

### 다대다 관계 (Many-to-Many) ###
여러 개의 엔티티가 다수의 엔티티와 연결되는 관계

<b style="color:orange">@JoinTable 어노테이션</b>
- 두 엔티티 간의 관계를 나타내는 중간 테이블을 정의함
- 연관 관계의 주체가 되는 엔티티에서 사용함
- name 속성: 생성할 중간 테이블의 이름을 지정
- joinColumns 속성: 중간 테이블 입장에서 현재 엔티티를 참조하는 왜래키를 정의함
- inverseJoinColumns 속성: 중간 테이블 입장에서 반대되는 엔티티를 참조하는 외래키를 정의함



#### 다대다 단방향 관계 ####
학생은 수업을 참조하지만, 수업은 학생을 알지 못하는 경우
```
@Entity
public class Student {
  @Id @GeneratedValue
  private Long id;

  @ManyToMany
  @JoinTable(
    name = "student_course",  // 생성할 중간 테이블 명
    joinColumns = @JoinColumn(name = "student_id"), // 현재 엔티티의 외래키
    inverseJoinColumns = @JoinColumn(name = "course_id")  // 반대되는 엔티티의 외래키
  )
  private Set<Course> courses = new HashSet<>();
}

// Course 엔티티는 Student를 참조하지 않음
@Entity
public class Course {
  @Id @GeneratedValue
  private Long id;

  private String title;
  ...
}
```

#### 다대다 양방향 관계 ####
```
@Entity
public class Student {
  @Id @GeneratedValue
  private Long id;

  @ManyToMany
  @JoinTable(
    name = "student_course",
    joinColumns = @JoinColumn(name = "student_id"),
    inverseJoinColumns = @JoinColumn(name = "course_id")
  )
  private Set<Course> courses = new HashSet<>();
}

@Entity
public class Course {
  @Id @GeneratedValue
  private Long id;

  @ManyToMany(mappedBy = "courses")
  private Set<Student> students = new HashSet<>();

  private String title;
  ...  
}
```

#### 다대다 관계는 실무에서 피하는 경우가 많음 ####
JPA에서 사용하기에는 복잡성과 데이터 관리의 어려움으로 인해 실무에서는 피하는 경우가 많음

다대다 관계는 중간 테이블을 통해 구현되지만, 이 구조로 인해 데이터의 중복이 발생할 수 있음
- 학생과 수업의 관계에서 학생이 여러 수업을 듣고, 수업도 여러 학생을 가질 경우, 중간 테이블에 많은 중복 데이터가 생길 수 있음

또한 관계가 복잡해지면 쿼리가 복잡해지고 대량의 데이터가 존재할 경우, 조인 쿼리가 성능 저하를 일으킬 수 있음

→ 다대다 관계는 두 개의 다대일, 일대다 관계로 분리해서 구현하는 것이 좋음
- Student와 Course 사이에 Enrollment와 같은 중간 엔티티를 만들어서 학생과 수업 엔티티 간의 관계를 명시적으로 관리할 수 있음