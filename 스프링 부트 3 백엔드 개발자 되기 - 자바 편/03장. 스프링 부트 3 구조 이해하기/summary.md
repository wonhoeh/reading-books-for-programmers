# ✅ 스프링 부트 3 구조 살펴보기

## 스프링 부트의 구조
| 계층 : 각자의 역할과 책임이 있는 어떤 소프트웨어의 구성 요소

### 프레젠테이션 계층
- HTTP 요청을 받고 이 요청을 비즈니스 계층으로 전송하는 역할
- 컨트롤러가 프레젠테이션 계층의 역할
- 컨트롤러는 스프링 부트 내에 여러 개가 있을 수 있음

### 비즈니스 계층
- 모든 비즈니스 로직을 처리함
- 비즈니스 로직: 서비스를 만들기 위한 로직
- 주문 서비스
  - 주문 개수, 가격 등의 데이터를 처리하기 위한 로직

### 퍼시스턴스 계층
- 모든 데이터베이스 관련 로직을 처리함
- 데이터베이스에 접근하는 DAO 객체를 사용
- DAO: 데이터베이스 계층과 상호작용하기 위한 객체
- 리포지토리가 퍼시스턴스 계층의 역할

| 계층은 개념의 영역이고 컨트롤러, 서비스, 리포지토리는 실제 구현을 위한 영역

## 디렉터리 설명

### main
- 실제 코드를 작성하는 공간
- 소스 코드나 리소스 파일은 모두 이 폴더 안에 있음

### test
- 프로젝트의 소스코드를 테스트할 목적의 코드나 리소스 파일이 있음

### build.gradle
- 빌드를 설정하는 파일
- 의존성이나 플러그인 설정

### setting.gradle
- 빌드할 프로젝트의 정보를 설정하는 파일

## main 디렉터리 구성하기

### java
- 소스 코드가 들어 있는 디렉터리

### resources
- template
  - HTML 뷰 관련 파일을 넣음
- static
  - JS, CSS, 이미지와 같은 정적 파일을 넣음
- application.yml
  - 스프링 부트 설정을 할 수 있는 파일
  - 데이터베이스의 설정 정보, 로깅 설정 정보


---

# ✅ 스프링 부트 3 프로젝트 발전시키기

## build.gradle에 의존성 추가하기
```
dependencies {
... 생략 ...
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2' // 인메모리 데이터베이스
    compileOnly 'org.projectlombok:lombok' // 롬복
    annotationProcessor 'org.projectlombok:lombok'
}
```
- 스프링 데이터 JPA
- 로컬 환경과 테스트 환경에서 사용할 인메모리 데이터베이스 H2
- 반복 메서드 작성 작업을 줄여주는 라이브러리인 롬복

## 프레젠테이션, 서비스, 퍼시스턴스 계층

프레젠테이션 계층
```
@RestController
public class TestController {
    @Autowired // TestService 빈 주입
    TestService testService;
    
    @GetMapping("/test")
    public List<Member> getAllMembers() {
        List<Member> members = testService.getAllMembers();
        return members;
    }
}
```

서비스 계층
```
@Service
public class TestService {
  @Autowired
  MemberRepository memberRepository; // 빈 주입
  
  public List<Member> getAllMembers() {
    return memberRepository.findAll(); // 멤버 목록 얻기
  }
} 
```

DAO
```
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Getter
@Entity
public class Member {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "id", updateable = false)
  private Long id; // DB 테이블의 'id' 컬럼과 매칭
  
  @Column(name = "name", nullable = false)
  private String name; // DB 테이블의 'name' 컬럼과 매칭
}
```

퍼시스턴스 계층
```
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

## 작동 확인하기

### 더미 데이터 입력하기
resources/data.sql
```
INSERT INTO member (id, name) VALUES (1, 'name 1')
INSERT INTO member (id, name) VALUES (2, 'name 2')
INSERT INTO member (id, name) VALUES (3, 'name 3')
```

application.yml
```
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
    
    defer-datasource-initialization: true
```
- show-sql, format_sql
  - 쿼리의 실행 구문을 모두 보여주는 옵션
- defer-datasource-initialization
  - 애플리케이션을 실행할 때 테이블을 생성하고 data.sql 파일에 있는 쿼리를 실행하는 옵션

---

# ✅ 스프링 부트 요청-응답 과정 한 방에

1. [GET] localhost:8080/test 요청
2. DispatcherServlet에서 URL을 분석하고 컨트롤러를 찾음
3. TestController에 /test GET 요청을 처리할 수 있는 메서드를 갖고 있으니, 얘한테 전달함
4. TestController -> TestService -> TestRespository로 전달하면서 TestRespository에서 getAllMembers() 메서드 수행
5. 뷰 리졸버가 템플릿 엔진을 사용해 HTML 문서를 만들거나 JSON, XL 등의 데이터를 생성함
6. 그 결과 members를 리턴해서 포스트맨에서 응답값을 볼 수 있게 됨

--- 

# ✅ 핵심 요약
- ***프레젠테이션 계층***은 HTTP 요청을 받고 비즈니스 계층으로 전송함
- ***비즈니스 계층***은 모든 비즈니스 로직을 처리함. 퍼시스턴스 계층에서 제공하는 서비스를 사용할 수도 있고, 권한을 부여하거나 유효성 검사를 하기도 함
- ***퍼시스턴스 계층***은 모든 스토리지 관련 로직을 처리함. 이 과정에서 데이터베이스에 접근하기 위한 객체인 DAO를 사용할 수도 있음