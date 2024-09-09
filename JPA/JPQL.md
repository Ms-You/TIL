## JPQL (Java Persistence Query Language) ##
JPA는 SQL을 추상화 한 JPQL이라는 엔티티 객체를 조회하는 객체 지향 쿼리 언어를 제공함

JPA는 엔티티 객체를 중심으로 개발하기 때문에 SQL을 사용하지 않지만, 검색 쿼리를 사용할 때 SQL을 사용해야 함

JPA는 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야 함

=> 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요함

JPA는 적절한 SQL을 만들어 DB에서 데이터를 조회하며 JPQL은 SQL로 변환됨

<b>JPQL 쿼리가 실행되면 flush와 transaction commit이 자동으로 호출됨</b>

<br />
<br />

### JPQL vs SQL ###
1. <b>쿼리 대상</b>
- SQL은 테이블과 컬럼을 대상으로 쿼리함
- JPQL은 엔티티와 그 속성을 대상으로 쿼리함

2. <b>별칭</b>
- JPQL에서 엔티티의 별칭은 필수로 명시해야 함
- 대신 별칭을 명시하는 AS 키워드는 생략할 수 있음
```
query = em.createQuery("SELECT m FROM Member m");
```

3. <b>반환 타입</b>
- SQL의 반환 타입은 일반적으로 행과 열로 반환됨
- JPQL의 반환 타입은 객체이며, 엔티티 객체로 매핑됨

4. <b>지속성 관리</b>
- SQL은 DB와 직접 상호작용하며, 트랜잭션 관리와 영속성 관리를 수동으로 처리해야 함
- JPQL은 영속성 컨텍스트를 통해 객체의 상태를 관리하고, 트랜잭션을 자동으로 처리함


<br />
<br />

### JPA에서의 쿼리 작성 ###
JPA에서의 쿼리 작성은 @Query 어노테이션을 사용할 수 있음

반환할 타입을 명확하게 지정할 수 있으면 TypedQuery, 명확하게 지정할 수 없으면 Query를 사용
```
// TypedQuery
public static void typedQuery() {
  String name = "Tom";
  String jpql = "SELECT m FROM Member WHERE m.name = :name"l

  TypedQuery<Member>query = em.createQuery(jpql, Member.class);
  query.setParameter("name", name);
}


// Query
public static void query() {
  String jpql = "SELECT m.name, m.age FROM Member m";
  Query query = em.createQuery(jpql);
}
```

<br />

- JPQL로 작성 (nativeQuery = false)
- 일반 SQL로 작성 (nativeQuery = true)

```
public interface MemberRepository extends JpaRepository<Member, Long> {

  // JPQL 쿼리 (from 뒤는 엔티티 명)
  @Query(value = "SELECT m FROM Member m", nativeQuery = false)
  public List<Member> selectAllJPQL();

  // SQL 쿼리 (from 뒤는 DB 테이블)
  @Query(value = "SELECT FROM member", nativeQuery = true)
  public List<Member> selectAllSQL();


  // JPQL 쿼리 (파라미터 사용)
  @Query(value = "SELECT m FROM Member m WHERE m.id > ?1", nativeQuery = false)
  public List<Member> selectJPQLById1(Long id);
  // 또는
  @Query(value = "SELECT m FROM Member m WHERE m.id > :id", nativeQuery = false)
  public List<Member> selectJPQLById2(@Param(value = "id") Long id);

  // JPQL 객체 파라미터 쿼리
  @Query(value = "SELECT m FROM Member m WHERE m.id > :#{#paramMember.id}", nativeQuery = false)
  public List<Member> selectJPQLById3(@Param(value = "paramMember") Member member);


  // SQL 쿼리 (파라미터 사용)
  @Query(value = "SELECT * FROM member WHERE member_id > ?1", nativeQuery = true)
  public List<Member> selectSQLById1(Long id);
  // 또는
  @Query(value = "SELECT * FROM member WHERE member_id > :id", nativeQuery = true)
  public List<Member> selectSQLById2(@Param(value = "id") Long id);

  @Query(value = "SELECT * FROM member WHERE member_id > ?1", nativeQuery = true)
  public List<Member> selectSQLById1(Long id);
}
```

위의 쿼리처럼 SELECT가 아닌 DML(INSERT, UPDATE, DELETE)을 사용할 때는 @Modifying, @Transactional 어노테이션을 추가적으로 사용해야 함
- @Modifying이 없으면 JPA가 기본적으로 SELECT 쿼리로 인식함
- @Transactional은 INSERT, UPDATE, DELETE 구문을 사용할 때 표기해줘야 함

```
// JPQL
@Modifying
@Transactional
@Query(value = "UPDATE Member SET nickname = :#{#member.nickname}, email = :#{#member.email} WHERE id = :#{#member.id}", nativeQuery = false)
public void updateJPQL(@Param(value = "member") Member member);
```