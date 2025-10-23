# ✅ 데이터베이스

## 데이터베이스 관리자, DBMS

- 데이터베이스를 관리하기 위한 소프트웨어
- MySQL, 오라클, PostgreSQL

### 관계형 DBMS

- 테이블 형태로 이루어진 데이터 저장소

### H2

- 자바로 작성되어 있는 RDBMS
- 인메모리 관계형 데이터베이스
- 애플리케이션 자체 내부에 데이터를 저장
- 애플리케이션을 다시 실행하면 데이터가 초기화
- 테스트 용도로 사용

## 데이터베이스 관련 용어

### 테이블

- 데이터를 구성하기 위한 가장 기본적인 단위
- 테이블은 행과 열로 구성

### 행

- 테이블의 가로로 배열된 데이터의 집합
- 회원 테이블이라면 한 명의 회원을 의미함
- 행은 반드시 고유한 식별자인 기본 키를 가져야함
- 행을 레코드라고 부르기도 함

### 열

- 행에 저장되는 데이터의 유형
- 각 요소에 대한 속성을 나타내며 무결성을 보장
- 이메일은 문자열, 나이는 숫자 유형

### 기본키

- 행을 구분할 수 있는 식별자
- 테이블에서 유일해야 하며 중복 값을 가질 수 없음 (NOT NULL + UNIQUE)
- 수정되면 안되고 유효한 값이어야 함, 즉 NULL이 될 수 없음

### 쿼리

- 데이터베이스에서 데이터를 조회, 삭제, 생성, 수정 처리를 하기 위해 사용
- SQL 이라는 데이터베이스 전용 언어를 사용하여 작성

---

# ✅ ORM

- 자바의 객체와 데이터베이스를 연결하는 프로그래밍 기법
- 자바 코드를 통해 데이터베이스의 값을 객체처럼 사용
- 자바 언어로만 데이터베이스를 다룰 수 있게 하는 도구를 의미

## ORM 장점과 단점

- 장점 1. SQL을 직접 작성하지 않고 데이터베이스에 접근할 수 있음
- 장점 2. 객체지향적으로 코드를 작성할 수 있음, 비즈니스 로직에 집중
- 장점 3. MySQL에서 PostgreSQL로 전환해도 추가로 드는 작업이 거의 없음, 데이터베이스 종속성이 줄어듬
- 장점 4. 매핑하는 정보가 명확해서 ERD에 대한 의존도를 낮추고 유지보수할 때 유리
- 단점 1. 프로젝트의 복잡성이 커질수록 사용 난이도가 올라감
- 단점 2. 복잡하고 무거운 쿼리는 ORM으로 해결이 불가능한 경우가 있음 

---

# ✅ JPA와 하이버네이트

- JPA는 자바에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스
- 실제 사용을 위해서는 ORM 프레임워크를 추가해야함, 대표적으로 하이버네이트를 사용
- 하이버네이트는 JPA 인터페이스를 구현한 구현체이자 자바용 ORM 프레임워크
- 내부적으로 JDBC API를 사용
- 자바 객체를 통해 데이터베이스 종류에 상관없이 데이터베이스를 자유자재로 사용

## JPA 컨셉

### 엔티티

- 데이터베이스의 테이블과 매핑되는 객체
- 엔티티의 본질은 자바 객체
- 데이터베이스의 테이블과 직접 연결되는 특징이 있음
- 엔티티는 객체지만 데이터베이스에 영향을 미치는 쿼리를 실행하는 객체

### 엔티티 매니저

- 엔티티를 관리해 데이터베이스와 애플리케이션 사이에서 객체를 생성, 수정, 삭제하는 역할
- 엔티티 매니저는 엔티티 매니저 팩토리에서 생성됨
- 사용자의 요청이 오면 팩토리에서 매니저를 생성해서 요청을 처리함
- 엔티티 매니저는 필요한 시점에 데이터베이스와 연결한 뒤에 쿼리
- 스프링 부트는 내부에서 엔티티 매니저 팩토리를 하나만 생성
- @PersistenceContext 또는 @Autowired 사용해서 엔티티 매니저를 사용

```
@PersistenceContext
EntitiyManager em; // 프록시 엔티티 매니저, 필요할 때 진짜 엔티티 매니저 호출

```

- 스프링 부트는 기본적으로 빈을 하나만 생성해서 공유하므로 동시성 문제가 있음
- 그래서 실제 엔티티 매니저가 아닌 프록시(가짜) 엔티티 매니저를 사용하고
- 필요할 때 데이터베이스 트랜잭션과 관련된 실제 엔티티 매니저를 호출해서 동시성 문제를 해결

## 영속성 컨텍스트

- 엔티티 매니저는 엔티티를 영속성 컨텍스트에 저장
- 엔티티를 관리하는 가상의 공간
- 1차 캐시, 쓰기 지연, 변경 감지, 지연 로딩 이라는 특징이 있음

### 1차 캐시

- 캐시의 키는 엔티티의 @Id 에 달린 기본키
- 캐시의 값은 엔티티
- 1차 캐시에서 데이터를 조회하고 값이 있으면 반환
- 값이 없으면 데이터베이스에서 조회해 1차 캐시에 저장한 다음 반환
- 데이터베이스를 거치지 않아도 되므로 매우 빠르게 데이터 조회가 가능

### 쓰기 지연

- 트랜잭션을 커밋하기 전까지는 데이터베이스에 실제로 질의문을 보내지 않음
- 쿼리를 모았다가 트랜잭션을 커밋하면 쿼리를 한번에 실행
- 데이터베이스 시스템의 부담을 줄임1

### 변경 감지

- 트랜잭션을 커밋하면 1차 캐시에 저장되어 있는 엔티티의 값과 현재 엔티티의 값을 비교
- 변경된 값이 있으면 변경 사항을 감지해 변경된 값을 데이터베이스에 자동으로 반영
- 데이터베이스 시스템의 부담을 줄임2

### 지연 로딩

- 쿼리로 요청한 데이터를 애플리케이션에 바로 로딩하지 않음
- 필요할 때 쿼리를 날려 데이터를 조회하는 것을 의미
- 데이터베이스 시스템의 부담을 줄임3
- 데이터베이스의 접근을 최소화해 성능을 높임!

## 엔티티의 상태

- 분리 상태(detached)
- 관리 상태(managed)
- 비영속 상태(transient)
- 삭제된 상태(removed)

```
public class EntityManagerTest {

    @Autowired
    EntitiyManager em;

    public void example() {
        // 1. 엔티티 매니저가 엔티티를 관리하지 않는 상태(비영속 상태)
        Member member = new Member(1L, "홍길동");

        // 2. 엔티티가 관리 상태가 됩니다(관리 상태)
        em.persist(member);

        // 3. 엔티티 객체가 분리된 상태가 됩니다(분리 상태)
        em.detach(member);

        // 4. 엔티티 객체가 삭제된 상태가 됩니다(삭제 상태)
        em.remove(member);

    }
}

```

1. 엔티티를 처음 만들면 엔티티는 비영속 상태
2. persist() 메서드를 사용해 엔티티를 관리 상태로 만듬, 영속성 컨텍스트에서 상태가 관리됨
3. 영속성 컨텍스트에서 관리하고 싶지 않다면 detach() 메서드를 사용해 분리 상태로 만듬
4. 객체가 필요 없다면 remove() 메서드를 사용해 영속성 컨텍스트와 데이터베이스에서 삭제

---

# ✅ 스프링 데이터와 스프링 데이터 JPA

- 엔티티를 직접 관리하는 건 신경 써야 할 부분이 많음
- 스프링 데이터는 데이터베이스 사용 기능을 클래스 레벨에서 추상화함. 비즈니스 로직에 집중하기 위해
- CRUD 포함한 여러 메서드 제공
- 페이징 처리 기능과 메서드 이름으로 자동으로 쿼리 빌딩하는 기능

## 스프링 데이터 JPA

- 스프링 데이터 인터페이스인 PaginAndSortingRepository를 상속받아 JpaRepsitory 인터페이스를 만듬
- 리포지토리 역할을 하는 인터페이스로 테이블 조회, 수정, 생성, 삭제같은 작업을 간단히 함
- <엔티티 이름, 엔티티 기본키의 타입>을 입력하면 기본 CRUD 메서드 사용 가능

```
public interface MemberRepository extends JpaRepository<Member, Long> {
}

```

## 스프링 데이터 JPA에서 제공하는 메서드

```
@Service
public class MemberService {

    @Autowired
    MemberRepository memberRepository;

    public void test() {
        // 1. 생성(Create)
        memberRepository.save(new Member(1L, "A"));

        // 2. 조회(Read)
        Optional<Member> member = memberRepository.findById(1L); // 단건 조회
        List<Member> members = memberRepository.findAll();       // 전체 조회

        // 3. 삭제(Delete)
        memberRepository.deleteById(1L);
    }
}

```

1. save() 메서드를 호출해 데이터 객체를 저장할 수 있음
2. findById() 메서드에 id를 저장해 엔티티를 하나 조회 <br>
   findAll() 메서드는 전체 엔티티를 조회
3. deleteById() 메서드에 id를 지정하면 엔티티를 삭제

---

# ✅ 예제 코드 살펴보기

```
@Entity // 1. 엔티티로 지정
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 2. 기본 생성자
@AllArgsConstructor
public class Member {

    @Id // 3. id 필드를 기본키로 지정
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 4. 기본키 자동으로 1씩 증가
    @Column(name = "id", updateable = false)
    private Long id;

    @Column(name = "name", nullable = false) // 5. name이라는 not null 컬럼과 매핑
    private String name;
}

```

### @Entity

- @Entity는 Member 객체를 JPA가 관리하는 엔티티로 지정함
- Member 클래스와 실제 데이터베이스의 테이블을 매핑
- name 속성을 사용하면 name의 값을 가진 테이블 이름과 매핑
- 지정하지 하지 않으면 클래스 이름과 같은 테이블 이름과 매핑

    ```
    @Entity(name = "member_list") // 'member_list' 이름을 가진 테이블과 매핑
    public class Article {
    ... 생략 ...
    }
    
    ```


### @NoArgsConstructor(access = AccessLevel.PROTECTED)

- 엔티티는 반드시 기본 생성자를 있어야 함
- 접근 제어자는 public 또는 protecte여야 함
- protected이 더 안전하기 때문에 protected 기본 생성자를 생성함

### @Id

- Long 타입의 id 필드를 테이블의 기본 키로(primary key)로 지정

### @GeneratedValue(strategy = GenerationType.IDENTITY)

- 기본 키의 생성 방식을 결정함
- AUTO: 선택한 데이터베이스 방언에 따라 방식을 자동으로 선택(기본값)
- IDENETITY: 기본 키 생성을 데이터베이스 위임(=AUTO_INCREMENT)
- SEQUENCE: 데이터베이스 시퀀스를 사용해서 기본키를 할당, 주로 오라클에서 사용
- TABLE: 키 생성 테이블 사용

### Column()

- 데이터베이스의 컬럼과 필드를 매핑
- name: 필드와 매핑할 컬럼 이름. 설정하지 않으면 필드 이름으로 지정함
- nullable: 컬럼의 null 허용 여부. 설정하지 않으면 true(nullable)
- unique: 컬럼의 유일한 값(unique)여부. 설정하지 않으면 false(non-unique)
- columnDefinition: 컬럼 정보 설정. default 값을 줄 수 있음