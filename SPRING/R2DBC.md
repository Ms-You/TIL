## R2DBC (Reactive Relational DataBase Connectivity) ##
Webflux 기반 프로젝트에서 JPA를 사용하게 되면?

JPA는 기본적으로 동기-블로킹 방식으로 동작하기 때문에 JPA를 사용하는 부분에서 스레드가 블로킹 되기 때문에 애플리케이션의 전체적인 성능이 떨어지게 됨

R2DBC는 반응형 프로그래밍을 위한 비동기 데이터베이스 연결 표준으로 비동기-논블로킹을 지원하기 때문에 Webflux에서 성능 저하 없이 사용할 수 있음

<br />

#### 특징 ####
1. 비동기 처리
    - R2DBC는 비동기로 DB와 통신하므로 I/O 작업 중 애플리케이션이 블로킹되지 않음

2. 논블로킹
    - R2DBC는 논블로킹 API를 제공하므로 DB 쿼리 실행 시 다른 작업을 수행할 수 있음

3. 리액티브 스트림
    - R2DBC는 리액티브 스트림을 기반으로 데이터 흐름을 처리하므로 데이터의 흐름을 제어하고, 백프레셔를 관리할 수 있음

<br />
<br />

R2DBC는 JPA와 달리 연관 관계 매핑을 지원하지 않음

그러므로 Lazy Loading, JoinColumn 등이 불가능함

Join이 필요할 경우, 직접 쿼리를 작성해야 함

@Id 어노테이션 설정 필드는 값을 할당하느냐 할당하지 않느냐에 따라 SQL 쿼리가 달라짐
  - Id 필드가 null인 상태로 save 메서드를 실행하면 Insert
  - Id 필드값이 설정된 상태로 save 메서드를 실행하면 Update
```
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table("member")
public class Member {
    @Id
    @Column(value = "member_id")
    private Long memberId;

    private String name;

    private String email;

    private String password;
}
```

<br />
<br />

### DatabaseClient ###
DatabaseClient는 R2DBC에서 제공하는 API로, DB와 비동기적으로 상호작용할 수 있도록 설계된 클라이언트임

DatabaseClient를 통해 SQL 쿼리를 실행하고 결과를 처리할 수 있으며, 리액티브 프로그래밍 모델을 따름

또한, 비동기-논블로킹 방식으로 동작하므로 높은 동시성을 요구하는 애플리케이션에 적합함

<br />

#### R2DBC와 DatabaseClient의 관계 ####
DatabaseClient는 R2DBC의 핵심 구성 요소로, R2DBC API를 통해 DB와 상호작용 하는 방법을 제공함

<b>R2DBC는 비동기 DB 연결을 위한 표준이고,</b>

<b>DatabaseClient는 이 표준을 구현한 클라이언트</b>

<br />

#### 특징 ####
1. 비동기 쿼리 실행
    - SQL 쿼리를 비동기적으로 실행할 수 있으며, 결과를 Mono, Flux와 같은 리액티브 타입으로 반환함

2. 트랜잭션 지원
    - DB 트랜잭션을 지원해서 트랜잭션의 특징 중 원자성을 보장할 수 있음

3. 유연한 결과 처리
    - 쿼리 결과를 다양한 형태로 처리 가능하며, 단일 결과 또는 여러 결과를 쉽게 다룰 수 있음

<br />


#### 문법 ####
<b>fetch(): </b>
- SQL 쿼리를 실행하고 그 결과를 가져오기 위해 사용
- SELECT문과 같이 결과 데이터를 반환하는 쿼리에서 사용

<b>fetch().one(): </b>
- 쿼리 결과에서 첫 번째 행만 가져오고, 그 이후의 행은 무시
- 만약 쿼리 결과가 없으면 빈 Mono가 반환됨 (Void)

<b>fetch().first(): </b>
- 쿼리 결과에서 첫 번째 행만 가져오지만, 항상 값을 포함하는 Mono를 반환함
- 만약 쿼리 결과가 없으면 기본값(null)이 포함된 Mono가 반환됨

<b>fetch().all(): </b>
- 쿼리 결과의 모든 행을 Flux로 반환함

<b>fetch().rowsUpdated(): </b>
- INSERT, UPDATE, DELETE 문과 같이 결과 데이터가 없는 쿼리에서 업데이트 된 행의 수를 반환하여 해당 작업이 성공적으로 수행 되었음을 확인할 수 있음

<br />
<br />

ex) 예시 코드
```
@RequiredArgsConstructor
@Repository
public class CategoryRepository {
    private final DatabaseClient databaseClient;

    // Id 필드가 null이면 Insert, null이 아니면 Update
    public Mono<Void> save(Category category) {
      String insertQuery = """
          INSERT INTO category (tier, name, code, parent_code) 
          VALUES (:tier, :name, :code, :parentCode)
      """;

      String updateQuery = """
          UPDATE category
          SET tier = :tier, name = :name, code = :code, parent_code = :parentCode
          WHERE category_id = :categoryId
      """;

      String query = category.getCategoryId() == null ? insertQuery : updateQuery;

      DatabaseClient.GenericExcuteSpec executeSpec = database.sql(query)
          .bind("tier", category.getTier())
          .bind("name", category.getName())
          .bind("code", category.getCode())
          .bind("parentCode", category.getParentCode());

      if (category.getCategoryId() != null) {
        executeSpec = executeSpec.bind("categoryId", category.getCategoryId());
      }

      return executeSpec
          .fetch()
          .rowsUpdated()
          .then();
    }

    public Flux<Category> findAll() {
      String query = """
          SELECT *
          FROM category c
      """;

      return databaseClient.sql(query)
          .fetch()
          .all()
          .map(row -> Category.builder()
              .categoryId((Long) row.get("category_id"))
              .tier((Integer) row.get("tier"))
              .name((String) row.get("name"))
              .code((String) row.get("code"))
              .parentCode((String) row.get("parent_code"))
              .build());
    }
}
```

<br />
<br />

아래의 두 return 문은 차이가 있음
```
String query = """
          SELECT *
          FROM category c
      """;

return databaseClient.sql(query)
    .map(row -> Category.builder()
        .categoryId((Long) row.get("category_id"))
        .tier((Integer) row.get("tier"))
        .name((String) row.get("name"))
        .code((String) row.get("code"))
        .parentCode((String) row.get("parent_cide"))
        .build())
    .all();


return databaseClient.sql(query)
    .fetch()
    .all()
    .map(row -> Category.builder()
        .categoryId((Long) row.get("category_id"))
        .tier((Integer) row.get("tier"))
        .name((String) row.get("name"))
        .code((String) row.get("code"))
        .parentCode((String) row.get("parent_cide"))
        .build());
```
- 처음 코드는 쿼리를 실행하고 각 행이 도착할 때마다 map() 연산을 수행한 다음, all() 메서드로 모든 결과를 반환함
- map() 연산은 각 행이 데이터베이스로부터 도착하는 즉시 호출되며, 모든 데이터가 도착하기 전에도 일부 결과를 처리할 수 있음
- <b style="color: orangered">map() 메서드는 내부적으로 쿼리를 실행하고 결과를 스트리밍하는 역할을 하기 때문에 fetch()를 명시적으로 호출하지 않아도 됨</b>

<br />

- 두 번째 코드는 SQL 쿼리를 실행하고(fetch()), 그 결과로 반환된 모든 행을 가져온 후(all()), 각 행에 대해 map() 연산을 수행함
- map() 연산은 데이터베이스로부터 모든 데이터를 가져온 후에 호출됨
