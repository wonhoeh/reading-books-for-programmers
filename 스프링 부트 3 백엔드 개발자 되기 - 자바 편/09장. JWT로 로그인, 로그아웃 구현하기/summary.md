# ✅ 사전 지식

## 토큰 기반 인증

- 토큰은 서버에서 클라이언트를 구분하기 위한 유일한 값
- 서버가 토큰을 생성해서 클라이언트에 제공
- 클라이언트는 여러 요청을 이 토큰과 함께 신청함
- 서버는 토큰만 보고 유효한 사용자인지 검증

## 토큰을 전달하고 인증 받는 과정

- [클라이언트 → 서버] 아이디와 비밀번호를 전달하면서 인증을 요청
- [서버 → 클라이언트] 유효한 사용자인지 검증하고 유효한 사용자면 토큰을 생성해서 응답
- [클라이언트 → 서버] 클라이언트는 서버에서 준 토큰을 저장하고 인증이 필요한 API 요청할 때 토큰을 함께 보냄
- [서버 → 클라이언트] 토큰이 유효한지 검증하고 유효하면 클라이언트가 요청한 내용을 처리

## 토큰 기반 인증의 특징

### 무상태성

- 인증 정보가 담겨 있는 토큰이 클라이언트에 저장되어 있음
- 클라이언트에서 사용자의 인증 상태를 유지하면서 요청을 처리해야함 이것을 상태를 관리한다고 함
- 서버는 인증 정보를 저장하거나 유지하지 않아도 되기 때문에 무상태로 검증을 할 수 있음

### 확장성

- 무상태성은 확장성에 영향을 줌
- 서버를 확장할 때 상태 관리를 신경 쓸 필요가 없으니 서버 확장에도 용이함

### 무결성

- 토큰 방식은 HMAC 기법이라고도 부름
- 토큰을 발급한 이후에 토큰 정보를 변경하는 행위를 할 수 없기 때문에 토큰의 무결성이 보장됨
- 토큰이 수정되면 유효하지 않은 토큰이라고 판단

## JWT

- 발급받은 JWT를 이용해 인증을 하려면 HTTP 요청 헤더에 Authorization 키 값에 Bearer + JWT 토큰값을 넣어 보내야함

### JWT 구조

`aaaaa.bbbbbb.cccccc  헤더.내용.서명`

| 부분 | 의미 | 설명 |
| --- | --- | --- |
| Header | 헤더 | 어떤 알고리즘을 썼는지 정보 |
| Payload | 내용(클레임) | 유저 정보, 만료 시간 등 |
| Signature | 서명 | 변조 여부 검증용 |

JWT 예시

```
Header:  { "alg": "HS256", "typ": "JWT" }
Payload: { "sub": "user123", "role": "admin" }

```

이걸 JWT 토큰값으로 만들면 이렇게 됨

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiJ1c2VyMTIzIiwicm9sZSI6ImFkbWluIn0
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

```

### 검증 절차

- 클라이언트가 보낸 JWT 문자열을 .로 분리: header_b64.payload_b64.signature_b64
- header_b64와 payload_b64를 Base64Url 디코딩하면 각각 JSON이 나온다. (이때 비밀키 필요 없음)
  → 이건 단순 디코딩이지 복호화가 아님.
- 서버는 header_b64 + "." + payload_b64와 서버가 가지고 있는 secretKey를 이용해 동일한 알고리즘(HMACSHA256 등)으로 서명(signature') 을 생성한다.
- 생성한 서명 signature'를 Base64Url로 인코딩한 값과 토큰의 signature_b64를 비교한다.
    - → 같으면 토큰이 발행된 이후 변조되지 않았다는 뜻(서명 무결성 통과).
    - → 다르면 위조/변조이므로 거부.
- 서명이 통과하면 Payload의 클레임들을 검사:
- exp 만료시간 체크 (현재시간 < exp)
- nbf, iat, aud, iss 등 필요한 표준 클레임 검증
- 추가 보안(선택): DB 체크
- sub(user id)로 사용자 존재 여부 확인
- 토큰의 jti나 토큰 버전(tokenVersion) 등으로 리프레시 토큰 회전/블랙리스트 검사
- 사용자 계정 상태(탈퇴/차단/패스워드 변경 등) 확인 — 필요시 토큰 무효 처리

### Base64 인코딩

- 앞의 두 부분(Header, Payload)은 실제 JSON
- 그냥 JSON을 문자열로 넣으면 깨질 수 있으니 Base64Url 인코딩을 적용해서 사람이 못 알아보는 문자열로 바꾼 것
- 디코딩해서 원래 내용을 볼 수 있음
- JSON을 Base64로 인코딩해서 변환한 것

### 서명

- 마지막 `ccccccc` 부분은 해싱 알고리즘이 적용된 곳
- 서버만 아는 secretKey로 Header+Payload를 해싱(HMAC)해서 만든 서명 값
- 토큰 내용을 조작하면 이 서명값이 달라져서 검증 실패됨

### 클레임

- JWT Payload 안에 들어가는 정보(클레임)는 크게 3종류가 있음
- 등록 클레임, 공개 클레임, 비공개 클레임

### 1. 등록 클레임

- JWT에서 자주 사용하는 핵심 필드들이고 표준 스펙에 포함된 것

| 이름 | 설명 |
| --- | --- |
| iss | 토큰 발급자(issuer) |
| sub | 토큰 제목 |
| aud | 토큰 대상자(audience) |
| exp | 토큰의 만료 시간. 시간은 NumericDate형식으로 하며 항상 현재 시간 이후로 설정합니다. |
| nbf | 토큰의 활성 날짜와 비슷한 개념으로 nbf는 Not Before를 의미합니다. NumericDate 형식으로 날짜를 지정하며, 이 날짜가 지나기 전까지는 토큰이 처리되지 않습니다. |
| iat | 토큰이 발급된 시간으로 iat은 issued at을 의미합니다. |
| jti | JWT의 고유 식별자로서 주로 일회용 토큰에 사용합니다. |

### 2. 공개 클레임

- 공개 클레임은 충돌을 방지할 수 있는 이름을 가져야 함. 보통 URI로 구성

### 3. 비공개 클레임

- 애플리케이션에서 필요에 따라 임의로 정의하는 클레임
- 서비스가 자체적으로 설정한 커스텀 데이터
- 클라이언트와 서버 간의 통신에 사용됨

JWT 예

```
{
    "iss": "jwtservice@gmail.com", // 등록 클레임
    "iat": 1622370878,             // 등록 클레임
    "exp": 1622372678,             // 등록 클레임
    "sub": "123",                  // 등록 클레임 (사용자 ID)
    "<https://example.com/jwt_claims/is_admin>": true,  // 공개 클레임
    "role": "ADMIN",               // 비공개 클레임 (권한)
    "nickname": "호이"             // 비공개 클레임 (서비스 정보)
}

```

## 리프레시 토큰

- 발급된 토큰은 탈취한 사람이 요청한 것인지 확인할 수 없기 때문에 액세스 토큰 만료시간을 설정해야함
- 리프레시 토큰은 액세스 토큰과 별개의 토큰
- 액세스 토큰이 만료되었을 때 새로운 액세스 토큰을 발급하기 위해 사용

## 리프레시 토큰 전달 과정

1. [클라이언트 → 서버] 인증을 요청함
2. [서버 → 클라이언트] 인증 정보가 유효한지 확인하고 액세스 토큰과 리프레시 토큰을 만들고 전달
3. [서버 → 데이터베이스] 리프레시 토큰은 DB에도 저장해둠
4. [클라이언트 → 서버] 인증을 필요로 하는 API를 호출할 때 토큰과 함께 요청
5. [서버 → 클라이언트] 액세스 토큰이 유효한지 검사한 뒤에 요청한 내용을 처리
6. [클라이언트 → 서버] 액세스 토큰이 만료된 뒤에 API 요청
7. [서버 → 클라이언트] 토큰이 만료되었다는 에러를 전달
8. [클라이언트 → 서버] 리프레시 토큰으로 새로운 액세스 토큰 발급을 요청
9. [서버 → 데이터베이스] DB에서 리프레시 토큰을 조회해 유효한 토큰인지 확인
10. [서버 → 클라이언트] 유효한 리프레시 토큰이면 새로운 엑세스 토큰을 발급

---

# ✅ JWT 서비스 구현

- JWT 생성하고 검증하는 서비스를 구현
- 의존성과 토큰 제공자 추가
- 리프레시 토큰 도메인과 토큰 필터를 구현

## 의존성 추가

### build.gradle

- JWT를 사용하기 위한 라이브러리 추가
- XML 문서와 자바 객체 간 매핑을 자동화하는 jax-api 추가

```
dependencies {
    ... 생략 ...
    testAnnotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.projectlombok:lombok'
    implementation 'io.jsonwebtoken:jjwt:0.9.1' // 자바 JWT 라이브러리
    implementation 'javax.xml.bing:jaxb-api:2.3.1 // XML 문서와 Java 객체 간 매핑을 자동화
}

```

## 토큰 제공자 추가

### application.yml

- JWT 토큰을 만들려면 이슈 발급자와 비밀키를 필수로 설정해야함

```
spring:
    jpa:
        show-sql: true
        properties:
            hibernate:
                format_sql: true
        defer-datasource-initialization: true
    datasource:
        url: jdbc:h2:mem:testdb
        username: sa
        password:
    h2:
        console:
            enabled: true
jwt:
    issuer: example@gmail.com
    secret_key: study-springboot

```

<br>

### config/jwt/JwtProperties.java

- 해당 값들을 변수로 접근하는 데 사용할 JwtProperties 클래스를 생성

```
@Setter
@Getter
@Component
@ConfigurationProperties("jwt") // 자바 클래스에 프로퍼티값을 가져와서 사용하는 애너테이션
public class JwtProperties {
    private String issuer;
    private String secretKey;
}

```

<br>

### config/jwt/TokenProvider.java

- 토큰을 생성하고 올바른 토큰인지 유효성 검사
- 토큰에서 필요한 정보를 가져오는 클래스

```
@RequiredArgsConstructor
@Service
public class TokenProvider {

    private final JwtProperties jwtProperties;

    public String generateToken(User user, Duration expiredAt) {
        Date now = new Date();
        return makeToken(new Date(now.getTime() + expiredAt.toMillis()), user);
    }

    // 1. JWT 토큰 생성 메서드
    private String makeToken(Date expiry, User user) {
        Date now = new Date();

        return Jwts.builder()
                .setHeaderParam(Header.TYPE, Header.JWT_TYPE) // 헤더 typ: JWT
                // 내용 iss: example@gmail.com(properties 파일에서 설정한 값)
                .setIssuer(jwtProperties.getIssuer())
                .setIssuedAt(now)       // 내용 iat: 현재 시간
                .setExpiration(expiry)  // 내용 exp: expiry 멤버 변숫값
                .setSubject(user.getEmail()) // 내용 sub: 유저의 이메일 (토큰 제목)
                .claim("id", user.getId())   // 클레임 id: 유저 ID
                // 서명: 비밀값과 함께 해시값을 HS256 방식으로 암호화
                .signWith(SignatureAlgorithm.HS256, jwtProperties.getSecretKey())
                .compact();
    }

    // 2. JWT 토큰 유효성 검증 메서드
    public boolean validToken(String token) {
        try {
            Jwts.parser()
                    .setSigningKey(jwtProperties.getSecretKey()) // 비밀값으로 복호화
                    .parseClaimJws(token);
            return true;
        } catch (Exception e) { // 복호화 과정에서 에러가 나면 유효하지 않은 토큰
            return false;
        }
    }

    // 3. 토큰 기반으로 인증 정보를 가져오는 메서드
    public Authentication getAuthentication(String token) {
        Claims claims = getClaims(token);
        Set<SimpleGrantedAuthority> authorities
            = Collections.singleton(new SimpleGrantedAuthority("ROLE_USER"));

        return new UsernamePasswordAuthenticationToken(new org.springframework.
                security.core.userdetails.User(claims.getSubject(), "", authorities), token, authorities);
    }

    // 4. 토큰 기반으로 유저 ID를 가져오는 메서드
    public Long getUserId(String token) {
        Claims claims = getClaims(token);
        return claims.get("id", Long.class);
    }

    private Claims getClaims(String token) {
        return Jwts.parser() // 클레임 조회
                .setSigningKey(jwtProperties.getSecretKey())
                .parseClaimsJws(token)
                .getBody();
    }
}
```

<br>

#### JWT 토큰 생성 메서드
```
private String makeToken(Date expiry, User user) {
    Date now = new Date();

    return Jwts.builder()
            .setHeaderParam(Header.TYPE, Header.JWT_TYPE) // 헤더 typ: JWT
            // 내용 iss: example@gmail.com(properties 파일에서 설정한 값)
            .setIssuer(jwtProperties.getIssuer())
            .setIssuedAt(now)       // 내용 iat: 현재 시간
            .setExpiration(expiry)  // 내용 exp: expiry 멤버 변숫값
            .setSubject(user.getEmail()) // 내용 sub: 유저의 이메일 (토큰 제목)
            .claim("id", user.getId())   // 클레임 id: 유저 ID
            // 서명: 비밀값과 함께 해시값을 HS256 방식으로 암호화
            .signWith(SignatureAlgorithm.HS256, jwtProperties.getSecretKey())
            .compact();
}
```
- `인자 expiry: 만료시간, user: 유저 정보`
- 헤더, 내용, 서명을 지정함

<br>

#### JWT 토큰 유효성 검증 메서드
```
public boolean validToken(String token) {
    try {
        Jwts.parser()
                .setSigningKey(jwtProperties.getSecretKey()) // 비밀값으로 복호화
                .parseClaimJws(token);
        return true;
    } catch (Exception e) { // 복호화 과정에서 에러가 나면 유효하지 않은 토큰
        return false;
    }
}
```
- JwtProperties에 선언한 비밀값과 함께 토큰 복호화를 진행
- 복호화 과정에서 유효하지 않은 토큰이면 false를 반환

<br>

#### 토큰 기반으로 인증 정보를 가져오는 메서드 
```
public Authentication getAuthentication(String token) {
    Claims claims = getClaims(token);
    Set<SimpleGrantedAuthority> authorities
        = Collections.singleton(new SimpleGrantedAuthority("ROLE_USER"));

    return new UsernamePasswordAuthenticationToken(
            new org.springframework.security.core.userdetails.User(claims.getSubject(), "", authorities), // principal에 저장
            token,        // credentials에 저장
            authorities); // authorities에 저장 
}
```
- `Authentication` → 인증 정보를 담은 객체
  - `Principal` 사용자 정보 객체 (UserDetails)
  - `Credentials` 인증에 사용된 값 (JWT 토큰)
  - `Authorities` 사용자가 가진 권한 목록 (ROLE_USER)

<br>

- `new SimpleGrantedAuthority("ROLE_USER)`
  - 스프링 시큐리티에서 권한을 나타내는 객체
  - 문자열 "ROLE_USER" → 사용자 권한 이름
  - SimpleGrantedAuthority → 이 문자열을 Spring Security가 이해할 수 있는 권한 객체로 감싸는 역할

<br>

- `UsernamePasswordAuthenticationToken` is-a `Authentication`
  - `new org.springframework.security.core.userdetails.User(claims.getSubject(), "", authorities)` → principal에 저장
  - `token` → credentials에 저장
  - `authorities` → authorities에 저장

<br>

#### 토큰 기반으로 유저 ID를 가져오는 메서드
```
public Long getUserId(String token) {
    Claims claims = getClaims(token);
    return claims.get("id", Long.class);
}
```
- getClaim() 메서드로 클레임 정보를 반환받고 클레임에서 id 키로 지정된 값을 가져와 반환

<br>

#### 클레임 정보 반환

```
private Claims getClaims(String token) {
    return Jwts.parser() // 클레임 조회
            .setSigningKey(jwtProperties.getSecretKey())
            .parseClaimsJws(token)
            .getBody();
}
```
- `Jwts.parser()`
  - 토큰을 읽고(파싱하고) 검증하는 역할
  - Header.Payload.Signature로 분리
  - Base64Url 디코딩해서 JSON 형태로 복원
  - setSigningKey(secretKey)로 서명을 검증해서 토큰 위조 여부 확인
  - Payload 안의 클레임(사용자 정보, 만료 시간 등)을 꺼내주는 과정

<br>

### test/config/jwt/JwtFactory.java
- JWT 토큰 서비스를 테스트하는데 사용할 모킹(mocking)용 객체

```
@Getter
public class JwtFactory {
  private String subject = "test@gmail.com";
  private Date issuedAt = new Date();
  private Date expiration = new Date(new Date().getTime() + Duration.ofDays(14).toMillis());
  private Map<String, Object> claims = emptyMap();
  
  // 빌더 패턴을 사용해 설정이 필요한 데이터만 선택 설정
  @Builder
  public JwtFactory(String subject, Date issuedAt, Date expiration,
                    Map<String, Object> claims) {
    this.subject = subject != null ? subject : this.subject;
    this.issuedAt = issuedAt != null ? issuedAt : this.issuedAt;
    this.expiration = expiration != null ? expiration : this.expiration;
    this.claims = claims != null ? claims : this.claims;
  }
  
  public static JwtFactory withDefaultValues() {
    return JwtFactory.builder.build();
  }
  
  // jjwt 라이브러리를 사용해 JWT 토큰 생성
  public String createToken(JwtProperties jwtProperties) {
    return Jwts.builder()
            .setSubject(subject)
            .setHeaderParam(Header.TYPE, Header.JWT_TYPE);
            .setIssuer(jwtProperties.getIssuer())
            .setIssuedAt(issuedAt)
            .setExpiration(expiration)
            .addClaims(claims)
            .signWith(SignatureAlgorithm.HS256, jwtProperties.getSecretKey())
            .compact();
  }
}
```

### test/config/jwt/TokenProvider.java

```
@SpringBootTest
public class TokenProviderTest {

    @Autowired
    private TokenProvider tokenProvider;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private JwtProperties jwtProperties;

    // generateToken() 검증 테스트
    @DisplayName("generateToken(): 유저 정보와 만료 기간을 전달해 토큰을 만들 수 있다.")
    @Test
    void generateToken() {
        // given: 토큰에 유저 정보를 추가하기 위한 테스트 유저 생성
        User testUser = userRepository.save(User.builder()
                .email("user@gmail.com")
                .password("test")
                .build());

        // when: 토큰 제공자의 generateToken() 메서드를 호출해 토큰을 생성
        String token = tokenProvider.generateToken(testUser, Duration.ofDays(14));

        // then: 토큰을 복호화하고 토큰을 만들 때 클레임으로 넣은 id값이 given절의 id와 같은지 확인
        Long userId = Jwts.parser()
                .setSigningKey(jwtProperties.getSecretKey())
                .parseClaimsJws(token)
                .getBody()
                .get("id", Long.class);

        assertThat(userId).isEqualTo(testUser.getUserId());
    }

    // validToken() 검증 테스트
    @DisplayName("validToken(): 만료된 토큰인 때에 유효성 검증에 실패한다.")
    @Test
    void validToken_invalidToken() {
        // given:
        String token = JwtFactory.builder()
                .expiration(new Date(new Date().getTime() - Duration.ofDays(7).toMillis()))
                .build()
                .createToken(jwtProperties);

        // when
        boolean result = tokenProvider.validToken(token);

        // then
        assertThat(result).isFalse();
    }

    @DisplayName("validToken(): 유효한 토큰인 때에 유효성 검증에 성공한다.")
    @Test
    void validToken_validToken() {
        // given
        String token = JwtFactory.withDefaultValues().createToken(jwtProperties);

        // when
        boolean result = tokenProvider.validToken(token);

        // then
        assertThat(result).isTrue();
    }

    // getAuthentication() 검증 테스트
    @DisplayName("getAuthentication(): 토큰 기반으로 인증 정보를 가져올 수 있다.")
    @Test
    void getAuthentication() {
        // given
        String userEmail = "user@email.com";
        String token = JwtFactory.builder()
                .subject(userEmail)
                .build()
                .createToken(jwtProperties);

        // when
        Authentication authentication = tokenProvider.getAuthentication(token);

        // then
        assertThat(((UserDetails) authentication.getPrincipal()).getUsername()).isEqualTo(userEmail);
    }

    // getUserId() 검증 테스트
    @DisplayName("getUserId(): 토큰으로 유저 ID를 가져올 수 있다.")
    @Test
    void getUserId() {
        // given
        Long userId = 1L;
        String token = JwtFactory.builder()
                .claims(Map.of("id", userId))
                .build()
                .createToken(jwtProperties);

        // when
        Long userIdByToken = tokenProvider.getUserId(token);

        // then
        assertThat(userIdByToken).isEqualTo(userId);
    }
} 
```

### generateToken() 검증 테스트
```
@DisplayName("generateToken(): 유저 정보와 만료 기간을 전달해 토큰을 만들 수 있다.")
@Test
void generateToken() {
  // given
  User testUser = userRepository.save(User.builder()
           .email("user@gmail.com")
           .password("test")
           .build());
  // when
  String token = tokenProvider.generateToken(testUser, Duration.ofDays(14));
  
  // then
  Long userId = Jwts.parser()
          .setSigningKey(jwtProperties.getSecretKey())
          .parseClaimsJws(token)
          .getBody()
          .get("id", Long.class);
  assertThat(userId).isEqualTo(testUser.getId());
}
```
- given: 토큰에 유저 정보를 추가하기 위한 `testUser` 생성
- when: `TokenProvider.generateToken()` 메서드로 토큰 생성
- then: 토큰을 복호화해서 꺼낸 클레임의 userId와 testUser의 userId가 같은지 확인


### invalidToken() 검증 테스트
```
@DisplayName("validToken(): 만료된 토큰인 때에 유효성 검증에 실패한다.")
@Test
void validToken_invalidToken() {
  // given
  String token = JwtFactory.builder()
          .expiration(new Date(new Date().getTime() - Duration.ofDays(7).toMillis()))
          .build()
          .createToken(jwtProperties);
  
  // when
  boolean result = tokenProvider.validToken(token);
  
  // then
  assertThat(result).isFalse();
}
```
- given: 현재 시간으로부터 7일 전의 시간으로 유효 기간이 만료된 토큰을 생성
- when: `TokenProvider.validToken()` 메서드로 유효한 토큰인지 검증
- then: 반환값이 false인 것을 확인, 즉 유효하지 않은 토큰임


### validToken() 검증 테스트
```
@DisplayName("validToken(): 유효한 토큰인 때에 유효성 검증에 성공한다.")
@Test
void validToken_validToken() {
  // given
  String token = JwtFactory.withDefaultValues()
            .createToken(jwtProperties);
  
  // when
  boolean result = tokenProvider.validToken(token);
  
  // then
  assertThat(result).isTrue();
}
```
- given: 현재 시간으로부터 14일 뒤로 만료되지 않은 토큰을 생성
- when: `TokenProvider.validToken()` 메서드로 유효한 토큰인지 검증
- then: 반환값이 true인 것을 확인, 즉 유효한 토큰임


### getAuthentication() 검증 메서드
```
@DisplayName("getAuthentication(): 토큰 기반으로 인증 정보를 가져올 수 있다.");
@Test
void getAuthentication() {
  // given
  String userEmail = "user@email.com";
  String token = JwtFactory.builder()
          .subject(userEmail)
          .build()
          .createToken(jwtProperties);
  
  // when
  Authentication authentication = tokenProvider.getAuthentication(token);
  
  // then
  assertThat((UserDetails) authentication.getPrincipail()).getUsername()).isEqualTo(userEmail);
}
```
- given: 토큰의 제목(subject)를 "user@email.com" 값을 사용해서 토큰 생성
- when: `TokenProvider.getAuthentication()` 메서드로 인증 객체를 반환
- then: 반환받은 인증 객체의 유저 이름을 가져와 given절에서 설정한 subject값이 같은지 확인


### getUserId() 검증 테스트
```
@DisplayName("getUserId(): 토큰으로 유저 ID를 가져올 수 있다.")
@Test
void getUserId() {
    // given
    Long userId = 1L;
    String token = JwtFactory.builder()
            .claims(Map.of("id", userId))
            .build()
            .createToken(jwtProperties);

    // when
    Long userIdByToken = tokenProvider.getUserId(token);

    // then
    assertThat(userIdByToken).isEqualTo(userId);
}
```
- given: id가 1인 클레임을 추가해서 토큰 생성
- when: `TokenProvider.getUserId()` 메서드로 유저 ID 반환
- then: 반환받은 유저 ID가 given절에서 설정한 유저 ID가 같은지 확인

---

# ✅ 리프레시 토큰 도메인 구현하기

## domain/RefreshToken.java
- 리프레시 토큰은 데이터베이스에 저장하는 정보이므로 엔티티를 생성
```
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
public class RefreshToken {
  
  @Id
  @GenerateValue(stragy = GenerationType.IDENTITY)
  @Column(name = "id", updatable = false)
  private Long id;
  
  @Column(name = "user_id", nullable = false, unique = true)
  private Long userId;
  
  @Column(name = "refresh_token", nullable = false)
  private String refreshToken;
  
  public RefreshToken(Long userId, String refreshToken) {
    this.userId = userId;
    this.refreshToken = refreshToken;
  }
  
  public RefreshToken update(String newRefreshToken) {
    this.refreshToken = newRefreshToken;
    return this;
  }
}
```

## RefreshTokenRepository.java
```
public interface RefreshTokenRepository extends JpaRepository<RefreshToken, Long> {
  Optional<RefreshToken> findByUserId(Long userId);
  Optional<RefreshToken> findByRefreshToken(String refreshToken);
}
```

## TokenAuthenticationFilter.java
- 액세스 토큰값이 담긴 Authorization 헤더값을 가져온 뒤 액세스 토큰이 유효하다면 인증 정보를 설정
  - 클라이언트가 보낸 요청 헤더에서 JWT 토큰을 꺼냄
  - 토큰이 유효한지 검증
  - 유효하면 인증 객체(Authentication)을 SecurityContext에 저장
  - Security Context는 SecurityContextHolder에 저장됨
```
@RequiredArgsConstrctor
public class TokenAuthenticationFilter extends OncePerRequestFilter {
  private final TokenProvider tokenProvider;
  private final static String HEADER_AUTHORIZATION = "Authorization";
  private final static String TOKEN_PREFIX = "Bearer ";
  
  @Override
  protected void doFilterInternal(
              HttpServletRequest request,
              HttpServletResponse response,
              FilterChain filterChain) throws ServletException, IOException {
      
    // 요청 헤더의 Authorization 키의 값 조회
    String authorizationHeader = request.getHeader(HEADER_AUTHORIZATION);
    // 가져온 값에서 접두사 제거
    String token = getAccessToken(authorizationHeader);
    // 가져온 토큰이 유효한지 확인하고, 유효한 때는 인증 정보를 설정
    if (tokenProvider.validToken(token)) {
      Authentication authentication = tokenProvider.getAuthentication(token);
      SecurityContextHolder.getContext().setAuthentication(authentication);
    }
    
    filterChain.doFilter(request, response);
  }
  
  private String getAccessToken(String authorizationHeader) {
    if (authorizationHeader != null && authorizationHeader.startsWith(TOKEN_PREFIX)) {
      return authorizationHeader.substring(TOKEN_PREFIX.length());
    }
    return null;
  }
}  
```
### SecurityContextHolder
- SecurityContext를 스레드 단위로 저장
  - 스레드 A → A 유저 정보만 저장
- 요청마다 인증 정보를 분리해서 안전하게 보관하기 위함
```
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
```
- 컨트롤러에서 인증 정보를 꺼낼 때 어디서든 꺼내 쓸 수 있는 이유가 SecurityContextHolder가 관리하기 때문

---

# 토큰 API 구현하기
- `리프레시 토큰`을 전달받아 검증하고 유효한 `리프레시 토큰`이면 새로운 `엑세스 토큰`을 생성하는 API

## UserService.java
- 유저 ID로 유저를 검색해서 전달하는 findById() 메서드 추가

```
@RequiredArgsConstructor
@Service
public class UserService {
  ... 생략 ...
  public User findById(Long userId) {
    return userRepository.findById(userId)
            .orElseThrow(() -> new IllegalArgumentException("Unexpected user"));
  }
}
```

## RefreshTokenService.java
- 전달받은 `리프레시 토큰`으로 `리프레시 토큰 객체`를 검색해서 전달하는 `findByRefreshToken()` 메서드 구현

```
@RequiredArgsConstructor
@Service
public class RefreshTokenService {
  private final RefreshTokenRepository refreshTokenRepository;
  
  public RefreshToken findByRefreshToken(String refreshToken) {
    return refreshTokenRepository.findByRefreshToken(refreshToken)
              .orElseThrow(() -> new IllegalArgumentException("Unexpected token"));
  }
}
```

## TokenService.java
```
@RequiredArgsConstructor
@Service
public class TokenService {
  
  private final TokenProvider tokenProvider;
  private final RefreshTokenService refreshTokenService;
  private final UserService userService;
  
  public String createNewAccessToken(String refrshToken) {
    // 토큰 유효성 검사에 실패하면 예외 발생
    if (!tokenProvider.validToken(refreshToken)) {
      throw new IllegalArgumentException("Unexpected token");
    }
    
    Long userId = refreshTokenService.findByRefreshToken(refreshToken).getUserId();
    User user = userService.findById(userId);
    return tokenProvider.generateToken(user, Duration.ofHours(2));
  }
}
```

### createNewAccessToken()
- 전달받은 리프레시 토큰으로 토큰 유효성 검사를 진행
- 유효한 토큰인 경우, 리프레시 토큰으로 사용자 ID를 찾음
- `TokenProvider.generateToken()` 메서드로 새로운 엑세스 토큰을 발행