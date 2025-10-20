# ✅ 사전 지식

## API
- 클라이언트의 요청을 서버에 전달하고
- 서버의 결과물을 클라이언트에게 돌려주는 역할

## REST API
- 자원을 이름으로 구분해 자원의 상태를 주고 받는 API 방식
- URL 설계 방식
- REST하게 디자인한 API를 RESTful API 라고 부름

- REST API의 특징
  - 서버/클라이언트 구조
  - 무상태
  - 캐시 처리 가능
  - 계층화
  - 인터페이스 일관성


- REST API 장점
  - URL만 보고도 무슨 행동을 하는 API인지 명확히 알 수 있음
  - 상태가 없기 때문에 클라이언트와 서버의 역할이 명확하게 분리됨
  - HTTP 표준을 사용하는 모든 플랫폼에서 사용 가능

- REST API 단점
  - HTTP 메서드 개수 제한이 있음
  - 설계를 하기 위해 공식적으로 제공되는 표준 규약이 없음

## REST API 사용하는 방법
- 규칙1. URL에는 동사를 쓰지 말고, 자원을 표시해야함
- 자원은 가져오는 데이터를 의미함
```
# RESTful API 적합한 예시 - 1번 글/학생을 가져온다는 의미가 명확함
/students/1
/articles/1

# 부적합한 예시 - 동사가 있음
/articles/show/1
/show/articles/1
```

- 규칙2. 동사는 HTTP 메서드로
- POST, GET, PUT, DELETE
- 추가, 조회, 수정, 삭제

---

# ✅ 블로그 개발을 위한 엔티티 구성하기

## 의존성

build.gradle
```
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-web'
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
  runtimeOnly 'com.h2database:h2'
  compileOnly 'org.projectlombok:lombok'
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

## 엔티티
Articles.java
```
@Getter
@NoArgsConstructor(accessLevel = AccessLevel.PROTECTED) 
@Entity // 엔티티로 지정
public class Article {
    
    @Id // id 필드를 기본키로 지정
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 기본 키 자동으로 1씩 증가
    @Column(name = "id", updatable = false)
    private Long id;
    
    @Column(name = "title", nullable = false) // 'title' 이라는 not null 컬럼과 매핑
    private String title;
    
    @Column(name = "content", nullable = false)
    private String content;
    
    @Builder // 빌더 패턴으로 객체 생성
    public Article(String title, String content) {
      this.title = title;
      this.content = content;
    }
```

빌더 패턴 예시
```
Article.builder()
  .title("abc")
  .content("def")
  .build();
```

## 리포지터리
BlogRepository.java
```
public interface BlogRepository extends JpaRepository<Article, Long> {
}
```
- JpaRepository 클래스를 상속받을 때 엔티티 Article과 엔티티의 PK 타입 Long을 인수로 넣음

---

# ✅ 블로그 글 작성을 위한 API 구현하기

## 서비스 메서드 코드 작성하기
- 서비스 계층에서 요청을 받을 객체 생성 (AddArticleRequest)
- BlogService에서 블로그 글 추가 메서드 save() 구현
- DTO: 계층끼리 데이터를 교환하기 위해 사용하는 객체
  - 전달자 역할이므로 별도의 비즈니스 로직을 포함하지 않음
- DAO: 데이터베이스와 연결되고 데이터를 조회하고 수정하는데 사용하는 객체

<br>

AddArticleRequest.java
```
@NoArgsConstructor // 기본 생성자 추가
@AllArgsConstructor // 모든 필드 값을 파라미터로 받는 생성자 추가
@Getter
public class AddArticleRequest {

  private String title;
  private String content;
  
  public Article toEntity() { // 생성자를 사용해 객체 생성
    return Article.builder()
            .title(title)
            .content(content)
            .build();
  }
}
```

<br>

BlogService.java
```
@RequiredArgsConstructor // final이 붙거나 @NotNull이 붙은 필드의 생성자 추가
@Service // 빈으로 등록
public class BlogService {

  private BlogRepository blogRepository;
  
  // 블로그 글 추가 메서드
  public Article save(AddArticleRequest request) {
    return blogRepository.save(request.toEntity());
  }
}
```
- @RequiredArgsConstructor는 빈을 생성자로 생성하는 롬복에서 지원하는 애너테이션
- final, @NotNull이 붙은 필드로 생성자를 만들어 줌

## 컨트롤러 메서드 코드 작성하기
BlogApiController.java
```
@RequiredArgsConstructor
@RestController // HTTP Response Body에 객체 데이터를 JSON 형식으로 반환하는 컨트롤러
public class BlogApiController {
  
  private BlogService blogService;
  
  // HTTP 메서드가 POST일 때 전달받은 URL와 동일하면 메서드로 매핑
  @PostMapping("/api/articles")
  // 요청 본문 값 매핑
  public ResponseEntity<Article> addArticle(@RequestBody AddArticleRequest request) {
    
    Article savedArticle = blogService.save(request);
    // 요청한 자원이 성공적으로 생성되었으며 저장된 블로그 글 정보를 응답 객체에 담아 전송
    return ResponseEntity.status(HttpStatus.CREATED)
            .body(savedArticle);
  }
}
```
- @RestController: HTTP 응답으로 객체 데이터를 JSON 형식으로 반환
- @PostMapping(): HTTP 메서드가 POST일 때 요청받은 URL와 동일한 메서드와 매핑
- 응답 코드
  - 200 OK: 요청이 성공적으로 수행되었음
  - 201 Created: 요청이 성공적으로 수행되었고, 새로운 리소스가 생성되었음
  - 400 Bad Request: 요청 값이 잘못되어 요청에 실패했음
  - 403 Forbidden: 권한이 없어 요청에 실패했음
  - 404 Not Found: 요청 값으로 찾은 리소스가 없어 요청에 실패했음
  - 500 Internal Server Error: 서버 상에 문제가 있어 요청에 실패했음

## 테스트 코드 작성하기
BlogApiControllerTest.java
```
@SpringBootTest // 테스트용 애플리케이션 컨텍스트
@AutoConfigureMockMvc // MockMvc 생성
class BlogApiControllerTest {

  @Autowired
  protected MockMvc mockMvc;
  
  @Autowired
  protected ObjectMapper objectMapper; // 직렬화, 역직렬화를 위한 클래스
  
  @Autowired
  private WebApplicationContext context;
  
  @Autowired
  BlogRepository blogRepository;
  
  @BeforeEach // 테스트 실행 전 실행하는 메서드
  public void mockMvcSetUp() {
    this.mockMvc = MockMvcBuilders.webAppContextSetUp(context)
                    .build();
    blogRepository.deleteAll();
  }
  
  @DisplayName("addArticle: 블로그 글 추가에 성공한다.")
  @Test
  public void addArticle() throws Exception {
    //given
    final String url = "/api/articles";
    final String title = "title";
    final String content = "content";
    final AddArticleRequest userRequest = new AddArticleRequest(title, content);
    
    // 객체 JSON으로 직렬화
    final String requestBody = objectMapper.writeValueAsString(userRequet);
    
    // when
    // 설정한 내용을 바탕으로 요청 전송
    ResultActions result = mockMvc.perform(post(url)
            .contentType(MediaType.APPLICATION_JSON_VALUE)
            .content(requestBody);
    
    // then
    result.andExpect(status().isCreated());
    
    List<Article> articles = blogRepository.findAll();
    
    assertThat(articles.size()).isEqualTo(1);  // 크기가 1인지 검증
    assertThat(articles.get(0).getTitle()).isEqualTo(title);
    aseertThat(articles.get(0).getContent()).isEqualTo(content);
  } 
}
```

- ObjectMapper
  - 자바 객체를 JSON 데이터로 변환하는 직렬화 또는 반대인 역직렬화를 할 때 사용
- writeValueString()
  - 객체를 JSON으로 직렬화
- MockMvc
  - HTTP 메서드, URL, 요청 본문, 요청 타입을 설정
  - contentType()에 요청을 보낼 때 JSON, XML 등 다양한 타입 중 하나를 골라 요청을 보냄
- assertThat 메서드
  - assertThat(articles.size()).isEqualTo(1);
  - assertThat(articles.size()).isGreaterThan(2);
  - assertThat(articles.size()).isLessThan(5);
  - assertThat(articles.size()).isZero();
  - assertThat(articles.title()).isEqualTo("제목");
  - assertThat(articles.title()).isNotEmpty();
  - assertThat(articles.title()).contains("제");

---