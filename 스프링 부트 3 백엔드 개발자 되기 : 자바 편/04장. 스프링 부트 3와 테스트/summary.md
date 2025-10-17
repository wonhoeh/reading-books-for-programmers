# ✅ 테스트 코드 개념 익히기

## 테스트 코드 패턴
given - when - then 패턴
- given: 실행을 준비하는 단계
- when: 테스트를 진행하는 단계
- then: 테스트 결과를 검증하는 단계

예시 코드
```
@DisplayName("새로운 메뉴를 저장한다.")
@Test
public void saveMenuTest() {
    //given: 메뉴를 저장하기 위한 준비 과정
    final String name = "아메리카노";
    final int price = 2000;
    
    final Menu americano = new Menu(name, price);
    
    // when: 실제로 메뉴를 저장
    final long savedId = menuService.save(americano);
    
    // then: 메뉴가 잘 추가되었는지 검증
    final Menu savedMenu = menuService.findById(savedId).get();
    aseertThat(savedMenu.getName()).isEqualTo(name);
    aseertThat(savedMenu.getPrice()).isEqualTo(price);
}
```

---

# ✅ 스프링 부트 3와 테스트

## spring-boot-starter-test 스타터 테스트 도구 목록
- JUnit
  - 자바 프로그래밍 언어용 단위 테스트 프레임워크
- Spring Test & Spring Boot Test
  - 스프링 부트 애플리케이션을 위한 통합 테스트 지원
- AssertJ
  - 검증문인 어설션을 작성하는 데 사용되는 라이브러리
- Hamcrest
  - 표현식을 보다 이해하기 쉽게 만드는데 사용되는 Matcher 라이브러리
- Mockito
  - 테스트에 사용할 가짜 객체인 목 객체를 쉽게 만들고, 관리하고, 검증할 수 있게 지원하는 테스트 프레임워크
- JSONassert
  - JSON용 어설션 라이브러리
- JsonPath
  - JSON 데이터에서 특정 데이터를 선택하고 검색하기 위한 라이브러리

## JUnit
- 자바 언어를 위한 단위 테스트 프레임워크
- 작은 단위로 검증하는 것을 의미하는데, 보통 메서드 단위로 테스트함
- @Test로 메서드를 호출할 때 마다 새 인스턴스를 생성해서 독립 테스트가 가능함

## JUnit으로 단위 테스트 코드 만들기
```
public class JUnitTest {
  @DisplayName("1 + 2는 3이다") // 테스트 이름
  @Test // 테스트 메서드
  public void junitTest() {
    int a = 1;
    int b = 2;
    int sum = 3;
    
    Aseertions.assertEquals(a + b, sum); // 값이 같은지 확인
  }
}
```
- @DisplayName()
  - 테스트 이름을 명시함
- @Test
  - 애너테이션이 붙은 메서드는 테스트를 수행하는 메서드가 됨
  - 테스트끼리 영향을 주지 않기 위해 실행 객체를 만들고 끝나면 실행 객체를 삭제함

### 자주 사용하는 JUnit 애너테이션
```
import org.junit.jupiter.api.*;

public class JUnitCycleTest {
  @BeforeAll // 전체 테스트를 실행하기 전에 1회 실행하므로 메서드는 static으로 선언
  static void beforeAll() {
    System.out.println("@BeforeAll");
  }
  
  @BeforeEach // 테스트 케이스를 시작하기 전마다 실행
  public void beforeEach() {
    System.out.println("@BeforeEach");
  }
  
  @Test
  public void test1() {
    System.out.println("test1");
  }
  
  @Test
  public void test2() {
    System.out.println("test2");
  }
  
  @Test
  public void test3() {
    System.out.println("test3");
  }
  
  @AfterAll // 전체 테스트를 종료하기 전에 1회 실행하므로 메서드는 static으로 선언
  static void afterAll() {
    System.out.println("@AfterAll");
  }
  
  @AfterEach // 테스트 케이스를 종료하기 전마다 실행
  public void afterEach() {
    System.out.println("@AfterEach");
  }
}
```

#### @BeforeAll
- 전체 테스트를 시작하기 전에 처음으로 한 번만 실행
- 데이터베이스 연결, 테스트 환경 초기화할 때 사용
- 전체 테스트 실행 주기에서 한 번만 호출되어야 하기 때문에 static으로 선언해야함

#### @BeforeEach
- 테스트 케이스를 시작하기 전에 매번 실행
- 테스트에서 사용하는 객체를 초기화, 테스트에 필요한 값 넣을 때 사용

#### @AfterAll
- 전체 테스트를 마치고 종료하기 전에 한 번만 실행
- 데이터베이스 연결 종료, 사용한 자원 해제할 때 사용
- static 선언해야함

#### @AfterEach
- 각 테스트 케이스를 종료하기 전 매번 실행
- 특정 데이터 삭제해야 하는 경우 사용

### AsesrtJ로 검증문 가독성 높이기
```
Assertions.assertEquals(a + b, sum);
```
- 기댓값과 비교값이 잘 구분 안됨 Assertions 예

```
assertThat(a + b).isEqualTo(sum);
```
- 위의 코드 보다 가독성이 더 좋음 AssertJ 예

AssertJ의 다양한 메서드

| 메서드 이름            | 설명               |
|-------------------|------------------|
| isEqualTo(A)      | A 값과 같은지 검증      
| isNotEqualTo(A)   | A 값과 다른지 검증      
| contains(A)       | A 값을 포함하는지 검증    
| doesNotContain(A) | A 값을 포함하지 않는지 검증 
| startsWith(A)     | 접두사가 A인지 검증      
| endsWith(A)       | 접미사가 A인지 검증      
| isEmpty()         | 비어있는 값인지 검증      
| isNotEmpty()      | 비어있지 않은 값인지 검증   
| isPositive()      | 양수인지 검증          
| isNegative()      | 음수인지 검증          
| isGreaterThan(1)  | 1보다 큰 값인지 검증     
| isLessThan(1)     | 1보다 작은 값인지 검증    

---

# ✅ 제대로 테스트 코드 작성하기

```
@SpringBootTest // 테스트용 애플리케이션 컨텍스트 생성
@AutoConfigureMockMvc // MockMvc 생성
class TestControllerTest {
  
  @Autowired
  protected MockMvc mockMvc;
  
  @Autowired
  private WebApplicationContext context;
  
  @Autowired
  private MemberRepository memberRepository;
  
  @BeforeEach // 테스트 실행 전 실행하는 메서드
  public void mockMvcSetUp() {
    this.mockMvc = MockMvcBuilders.webAppContextSetup(context)
                  .build();
                  
  @AfterEach // 테스트 실행 후 실행하는 메서드
  public void cleanUp() {
    memberRepository.deleteAll();
  }
```
#### @SpringBootTest
- @SpringBootApplication이 있는 클래스를 찾고
- 그 클래스에 포함되어 있는 빈을 찾은 다음 테스트용 애플리케이션 컨텍스트를 만듬

#### @AutoConfigureMockMvc
- MockMvc를 생성하고 자동으로 구성함
- MockMvc는 컨트롤러를 테스트할 때 사용되는 클래스

```
@SpringBootTest // 테스트용 애플리케이션 컨텍스트 생성
@AutoConfigureMockMvc // MockMvc 생성
class TestControllerTest {
  
  ... 생략 ...
  
  @DisplayName("getAllMembers: 아티클 조회에 성공한다.")
  @Test
  public void getAllMembers() throws Exception {
    // given
    final String url = "/test";
    Member savedMember = memberRepository.save(new Member(1L, "홍길동"));
    
    // when
    final ResultActions result = mockMvc.perform(get(url)
            .accept(MediaType.APPLCATION_JSON));
            
    // then
    result
          .andExpect(status().isOk())
          // 응답의 0번째 값이 DB에서 저장한 값과 같은지 확인
          .andExpect(jsonPath("$[0].id").value(savedMember.getId()))
          .andExpect(jsonPath("$[0].name").value(savedMember.getName()));
    
  }
}        
```

#### perform()
- 요청을 전송하는 역할
- 결과로 ResultActions 객체를 받음
- andExpect() 메서드를 제공함

#### accept()
- 요청을 보낼 때 무슨 타입으로 응답을 받을지 결정하는 메서드
- JSON, XML 등 다양한 타입이 있음

#### andExpect()
- 응답을 검증하는 역할
- HTTP 응답 코드를 확인할 수 있음

#### jsonPath("$[0].$(필드명)")
- JSON 응답값의 값을 가져오는 역할을 하는 메서드
- 0번째 배열에 들어있는 객체의 id, name 값을 가져오고 저장된 값과 확인함

HTTP 주요 응답 코드

| 코드                        | 매핑 메서드                  | 설명                               |
|---------------------------|-------------------------|----------------------------------|
| 200 OK                    | isOk()                  | HTTP 응답 코드가 200 OK인지 검증          
| 201 Created               | isCreated()             | HTTP 응답 코드가 201 Created인지 검증     
| 400 Bad Request           | isBadRequest()          | HTTP 응답 코드가 400 Bad Request인지 검증 
| 403 Forbidden             | isForbidden()           | HTTP 응답 코드가 403 Forbidden인지 검증          
| 404 Not Found             | isNotFound()            | HTTP 응답 코드가 404 Not Found인지 검증          
| 400 번대 응답 코드              | is4xxClientError()      | HTTP 응답 코드가 400번대 응답 코드인지 검증          
| 500 Internal Server Error | isInternalServerError() | HTTP 응답 코드가 500 Internal Server Error 인지 검증          
| 500번대 응답 코드               | is5xxServerError()      | HTTP 응답 코드가 500번대 응답 코드인지 검증          