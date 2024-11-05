## NoArgsConstructor(access=AccessLevel.PROTECTED) ##

### AccessLevel.PROTECTED 사용 이유 ###
1. 외부에서의 조작 방지
    - 엔티티의 필드나 메서드를 외부에서 직접 접근하지 못하게 하면서 같은 패키지 내의 클래스나 상속 관계에 있는 클래스에서는 접근할 수 있음
    - 무분별한 객체 생성에 대해 한 번 더 체크할 수 있음
2. 엔티티의 Proxy 조회 때문
    - JPA에서 엔티티의 필드에 접근하기 위해 리플렉션을 사용함
    - 만약, JPA에서 접근하는 필드가 PRIVATE일 경우 기본 생성자를 통해 접근할 수 있어야 함
    - 기본 생성자가 PROTECTED일 경우 JPA가 접근할 수 있으면서도 필드의 직접적인 접근을 제한할 수 있게 됨
3. 상속이 용이함
    - PROTECTED로 선언할 경우 상속이 용이하기 때문에 하위 클래스에서 상위 클래스의 필드에 접근할 수 있음

<br />
<br />

### AccessLevel.PRIVATE 사용하지 않는 이유 ###
- 필드에 대한 직접 접근을 제한하지만, JPA 등에서 리플렉션을 통해 접근할 수 없음

<br />

### AccessLevel.PUBLIC 사용하지 않는 이유 ###
- 모든 클래스에서 접근이 가능하므로, 엔티티의 필드가 외부에서 수정될 위험이 있음
- 또한, 무분별한 객체 생성 및 Setter 메서드를 통한 값 주입이 가능해지기 때문에 사용하지 않음

<br />

```JAVA
@Getter
@NoArgsConstructor(access=AccessLevel.PROTECTED)
@Builder
@Entity
public class User {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long userId;
  private String name;
}

@Getter
@NoArgsConstructor(access=AccessLevel.PROTECTED)
@Builder
@Entity
public class Post {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long postId;
  private String title;
  
  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "userId")
  private User user;
}

// Test
@SpringBootTest
class Test {
  @Autowired
  private EntityManager em;

  @BeforeEach
  void generate() {
    User user = User.builder()
            .name("user-name")
            .build();

    em.persist(user);

    Post post = Post.builder()
            .title("post-title")
            .user(user)
            .build();

    em.persist(post);
  }

  @Test
  void proxyTest() {
    Post post = em.find(Post.class, 1L);
    System.out.println("post id: " + post.getPostId());
    System.out.println("user id: " + post.getUser().getUserId());
  }
}

/**
 * User와 Post 모두 protected일 때)
 * - 테스트 통과
 * User는 private, Post는 protected일 때)
 * - 에러 발생 (User 프록시 객체를 생성하지 못함)
 * User는 protected, Post는 private일 때)
 * - 테스트 통과
 */
```