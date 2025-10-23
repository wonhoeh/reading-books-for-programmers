# ✅ 사전 지식

## 인증과 인가

### 인증
- 사용자의 신원을 입증하는 과정
- 사용자가 사이트에 로그인을 할 때 누구인지 확인하는 과정

### 인가
- 사이트의 특정 부분에 접근할 수 있는지에 대한 권한을 확인하는 과정

## 스프링 시큐리티
- 스프링 기반의 애플리케이션(인증, 인가, 권한)을 담당하는 스프링 하위 프레임워크
- 설정이 쉽고, CSRF 공격, 세션 고정 공격을 방어함
- 요청 헤더 보안 처리
- 필터 기반으로 동작하는 스프링 시큐리티

## 주요 필터 소개

### UsernamePasswordAuthenticationFilter
- 아이디와 패스워드가 넘어오면 인증 요청을 위임하는 인증 관리자 역할

### FilterSecurityInterceptor
- 권한 부여 처리를 위임해 접근 제어 결정을 쉽게 하는 접근 결정 관리자 역할

---

# ✅ 회원 도메인 만들기

## 의존성 추가

build.gradle
```
dependencies {
    ... 생략 ...
    // 스프링 시큐리티를 사용하기 위한 스타터 추가
    implementation 'org.springframework.boot:spring-boot-starter-security'
    // 타임리프에서 스프링 시큐리티를 사용하기 위한 의존성 추가
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'
    // 스프링 시큐리티를 테스트하기 위한 의존성 추가
    testImplementation 'org.springframework.security:spring-security-test'
}   
```

## 엔티티 만들기

User.java
```
@Table(name = "users")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
public class User implementation UserDetails { // UserDetails를 상속받아 인증 객체로 사용

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", updatable = false)
    private Long id;
    
    @Column(name = "email", nullable = false, unique = true)
    private String email;
    
    @Column(name = "password")
    private String password;
    
    @Builder
    public User(String email, String password, String auth) {
        this.email = email;
        this.password = password;
    }
    
    @Override // 권한 반환
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("user"));
    }
    
    // 사용자의 id를 반환(고유한 값)
    @Override
    public String getUsername() {
        return email;
    }
    
    // 사용자의 패스워드를 반환
    @Override
    public String getPassword() {
        return password;
    }
    
    // 계정 만료 여부 반환
    @Override
    public boolean isAccountNonExpired() {
        // 만료되었는지 확인하는 로직
        return true; // true -> 만료 되지 않았음
    }
    
    // 계정 잠금 여부 반환
    @Override
    public boolean isAccountNonLocked() {
        // 계정 잠금되었는지 확인하는 로직
        return true; // true -> 잠금되지 않았음
    }
    
    // 패스워드의 만료 여부 반환
    @Override
    public boolean isCredentialsNonExpired() {
        // 패스워드가 만료되었는지 확인하는 로직
        return true; // true -> 만료되지 않았음
    }
    
    // 계정 사용 가능 여부 반환
    @Override
    public boolean isEnabled() {
        // 계정이 사용 가능한지 확인하는 로직
        return true; // true -> 사용 가능
    }
```

### UserDetails 클래스
- 스프링 시큐리티에서 사용자의 인증 정보를 담아 두는 인터페이스
- 해당 객체를 통해 인증 정보를 가져오므로 필수 오버라이드 메서드가 있음

| 메서드 | 반환 타입 | 설명 |
| - | - | - |
| getAuthorities() | Collection<? extends GrantedAuthority> | 사용자가 가지고 있는 권한의 목록을 반환합니다. |
| getUsername() | String | 사용자를 식별할 수 있는 사용자 이름을 반환합니다. 사용자 이름은 고유해야 합니다. |
| getPassword() | String | 사용자의 비밀번호를 반환합니다. 비밀번호는 암호화해서 저장해야 합니다. |
| isAccountNonExpired() | boolean | 계정이 만료되었는지 확인하는 메서드입니다. |
| isAccountNonLocked() | boolean | 계정이 잠금되었는지 확인하는 메서드입니다. |
| isCredentialsNonExpired() | boolean | 비밀번호가 만료되었는지 확인하는 메서드입니다. |
| isEnabled() | boolean | 계정이 사용 가능한지 확인하는 메서드입니다. |

## 리포지토리 만들기
- User 엔티티에 대한 리포지토리 생성

UserRepository.java
```
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email); // email로 사용자 정보를 가져옴
}
```

## 서비스 메서드 코드 작성하기
- 스프링 시큐리티에서 로그인을 진행할 때 사용자 정보를 가져오는 코드 작성
- 스프링 시큐리티에서 사용자의 정보를 가져오는 UserDetailsService 인터페이스를 구현
- loadUserByUsername()을 오버라이딩해서 사용자 정보를 가져오는 로직을 작성

UserDetailService.java
```
@RequiredArgsConstructor
@Service
// 스프링 시큐리티에서 사용자 정보를 가져오는 인터페이스
public class UserDetailService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    // 사용자 이름(email)으로 사용자의 정보를 가져오는 메서드
    @Override
    public User loadUserByUsername(String email) {
        return userRepository.findByEmail(email)
                .orElseThrow(() -> new IllegalArgumentException(email));
    }
}
```

---

# ✅ 시큐리티 설정하기
- 실제 인증 처리를 하는 시큐리티 설정 파일 WebSecurityConfig 클래스를 생성
- config 패키지 생성

WebSecurityConfig.java
```
package com.example.blogservice.config;

import com.example.blogservice.service.UserDetailService;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityCustomizer;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

import static org.springframework.boot.autoconfigure.security.servlet.PathRequest.toH2Console;

@RequiredArgsConstructor
@Configuration
public class WebSecurityConfig {

    private final UserDetailService userService;

    // 1️⃣ 스프링 시큐리티 기능 비활성화
    @Bean
    public WebSecurityCustomizer configure() {
        return (web -> web.ignoring()
                .requestMatchers(toH2Console())
                .requestMatchers("/static/**"));
    }

    // 2️⃣ 특정 HTTP 요청에 대한 웹 기반 보안 구성
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                .authorizeHttpRequests(authorize -> authorize // 3️⃣ 인증, 인가 설정
                        .requestMatchers("/login", "/signup", "/user").permitAll()
                        .anyRequest().authenticated()
                )
                .formLogin(form -> form         // 4️⃣ 폼 기반 로그인 설정
                        .loginPage("/login")
                        .defaultSuccessUrl("/articles", true))
                .logout(logout -> logout        // 5️⃣ 로그아웃 설정
                        .logoutSuccessUrl("/login")
                        .invalidateHttpSession(true))
                .csrf(csrf -> csrf.disable())   // 6️⃣ csrf 비활성화
                .build();
    }

    // 7️⃣ 인증 관리자 관련 설정
    @Bean
    public AuthenticationManager authenticationManager(HttpSecurity http,
                                                       BCryptPasswordEncoder bCryptPasswordEncoder,
                                                       UserDetailService userDetailService) throws Exception {
        AuthenticationManagerBuilder authenticationManagerBuilder =
                http.getSharedObject(AuthenticationManagerBuilder.class);

        authenticationManagerBuilder
                .userDetailsService(userService) // 8️⃣ 사용자 정보 서비스 설정
                .passwordEncoder(bCryptPasswordEncoder);

        return authenticationManagerBuilder.build();
    }

    // 9️⃣ 패스워드 인코더로 사용할 빈 등록
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

```

## 코드 설명
1️⃣ 스프링 시큐리티 기능 비활성화
- 모든 곳에 스프링 시큐리티 기능을 사용하지 않음
- 즉, 인증 인가를 모든 서비스에 적용하지 않음
- 정적 파일이 있는 static 하위 경로
- h2 데이터를 확인하는데 사용하는 h2-console 하위 url

2️⃣ 특정 HTTP 요청에 대한 웹 기반 보안 구성
- 인증, 인가 및 로그인, 로그아웃 관련 설정

3️⃣ 인증, 인가 설정
- 특정 경로에 대한 액세스 설정
- requestMatcher(): 특정 요청과 일치하는 url에 대한 액세스를 설정
- permitAll(): 누구나 접근이 가능하게 설정합니다. "/login", "/signup", "/user"로 요청이 오면 인증/인가 없이도 접근할 수 있음
- anyRequest(): 위에서 설정한 url 이외의 요청에 대해서 설정
- authenticated(): 별도의 인가는 팔요하지 않지만 인증이 접근할 수 있음


4️⃣ 폼 기반 로그인 설정
- loginPage(): 로그인 페이지 경로를 설정
- defaultSuccessUrl(): 로그인이 만료되었을 때 이동할 경로를 설정

5️⃣ 로그아웃 설정
- logoutSuccessUrl(): 로그아웃이 완료되었을때 이동할 경로를 설정
- invalidateHttpSession(): 로그아웃 이후에 세션을 전체 삭제할지 여부를 설정

6️⃣ csrf 설정
- CSRF 설정

7️⃣ 인증 관리자 관련 설정
- 사용자 정보를 가져올 서비스를 재정의하거나, 인증 방법(LDAP, JDBC) 기반 인증 등을 설정할 때 사용

8️⃣ 사용자 정보 서비스 설정
- userDetailsService(): 사용자 정보를 가져올 서비스를 설정합니다. 이때 설정하는 서비스 클래스는 반드시 UserDetailsService를 상속받은 클래스여야 함

9️⃣ 패스워드 인코더 빈 등록
- 패스워드 인코더를 빈으로 등록함

---

# ✅ 회원 가입 구현하기
- 회원 정보를 추가하는 서비스 메서드를 작성하고 컨트롤러를 구현

## 서비스 메서드 코드 작성하기
- 사용자 정보를 담고 있는 객체 생성 (AddUserRequest.java)

AddUserRequest.java
```
@Getter
@Setter // Setter 없으면 폼에서 입력받은 데이터가 전달되지 않음
public class AddUserRequest {
    private String email;
    private String password;
}
```

UserService.java
```
@RequiredArgsConstructor
@Service
public class UserService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;
    
    public Long save(AddUserRequest dto) {
        return userRepository.save(User.builder()
                .email(dto.getEmail())
                .password(bCryptPasswordEncoder.encode(dto.getPassword()))
                .build().getId();
    }
}
```

## 컨트롤러 작성하기
- 회원 가입 폼에서 회원 가입 요청을 받으면
- 서비스 메서드를 사용해 사용자를 저장한 뒤
- 로그인 페이지로 이동하는 signup() 메서드 작성

UserApiController.java
```
@RequiredArgsConstructor
@Controller
public class UserApiController {
    
    private final UserService userService;
    
    @PostMapping("/user")
    public String signup(AddUserRequest request) {
        userService.save(request); // 회원 가입 메서드 호출
        return "redirect:/login";  // 회원 가입이 완료된 이후에 로그인 페이지로 이동
    }
}
```

---

# ✅ 회원 가입, 로그인 뷰 작성하기
- 사용자가 회원 가입, 로그인 경로에 접근하면
- 회원 가입, 로그인 화면으로 연결해주는 컨트롤러를 생성
- 사용자가 실제로 볼 수 있는 화면 작성

## 뷰 컨트롤러 구현하기
- 로그인, 회원 가입 경로로 접근하면 뷰 파일을 연ㄹ결하는 컨트롤러 생성
- /login 경로로 접근하면 login() 메서드가 login.html을 반환

UserViewController.java
```
@Controller
public class UserViewController {
    
    @GetMapping("/login")
    public String login() {
        return "login";
    }
    
    @GetMapping("/signup")
    public String signup() {
        return "signup";
    }
}
```

## 뷰 작성하기

login.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>로그인</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.1/dist/css/bootstrap.min.css">

    <style>
        .gradient-custom {
            background: linear-gradient(to right, rgba(106, 17, 203, 1), rgba(37, 117, 252, 1))
        }
    </style>
</head>
<body class="gradient-custom">
<section class="d-flex vh-100">
    <div class="container-fluid row justify-content-center align-content-center">
        <div class="card bg-dark" style="border-radius: 1rem;">
            <div class="card-body p-5 text-center">
                <h2 class="text-white">LOGIN</h2>
                <p class="text-white-50 mt-2 mb-5">서비스를 사용하려면 로그인을 해주세요!</p>

                <div class = "mb-2">
                    <form action="/login" method="POST">
                        <input type="hidden" th:name="${_csrf?.parameterName}" th:value="${_csrf?.token}" />
                        <div class="mb-3">
                            <label class="form-label text-white">Email address</label>
                            <input type="email" class="form-control" name="username">
                        </div>
                        <div class="mb-3">
                            <label class="form-label text-white">Password</label>
                            <input type="password" class="form-control" name="password">
                        </div>
                        <button type="submit" class="btn btn-primary">Submit</button>
                    </form>

                    <button type="button" class="btn btn-secondary mt-3" onclick="location.href='/signup'">회원가입</button>
                </div>
            </div>
        </div>
    </div>
</section>
</body>
</html>
```

signup.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>회원 가입</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.1/dist/css/bootstrap.min.css">

    <style>
        .gradient-custom {
            background: linear-gradient(to right, rgba(254, 238, 229, 1), rgba(229, 193, 197, 1))
        }
    </style>
</head>
<body class="gradient-custom">
<section class="d-flex vh-100">
    <div class="container-fluid row justify-content-center align-content-center">
        <div class="card bg-dark" style="border-radius: 1rem;">
            <div class="card-body p-5 text-center">
                <h2 class="text-white">SIGN UP</h2>
                <p class="text-white-50 mt-2 mb-5">서비스 사용을 위한 회원 가입</p>

                <div class = "mb-2">
                    <form th:action="@{/user}" method="POST">
                        <!-- 토큰을 추가하여 CSRF 공격 방지 -->
                        <input type="hidden" th:name="${_csrf?.parameterName}" th:value="${_csrf?.token}" />
                        <div class="mb-3">
                            <label class="form-label text-white">Email address</label>
                            <input type="email" class="form-control" name="email">
                        </div>
                        <div class="mb-3">
                            <label class="form-label text-white">Password</label>
                            <input type="password" class="form-control" name="password">
                        </div>

                        <button type="submit" class="btn btn-primary">Submit</button>
                    </form>
                </div>
            </div>
        </div>
    </div>
</section>
</body>
</html>
```

---

# ✅ 로그아웃 구현하기

## 로그아웃 메서드 추가하기
- UserApiController에 logout() 메서드 추가
- /logout GET 요청을 하면 로그아웃을 담당하는 SecurityContextLogoutHandler의 logout() 메서드를 호출
UserApiController.java
```
public class UserApiController {

    ... 생략 ...

    @GetMapping("/logout")
    public String logout(HttpServletRequest request, HttpServletResponse response) {
        new SecurityContextLogoutHandler()
            .logout(request, response,
                    SecurityContextHolder.getContext().getAuthentication());
        return "redirect:/login";
    }
}                                       
```

## 로그아웃 뷰 추가하기
- articleList.html에 [로그아웃] 버튼을 추가

articleList.html
```
... 생략 ...
            <div class="card-body">
                <h5 class="card-title" th:text="${item.title}"></h5>
                <p class="card-text" th:text="${item.content}"></p>
                <a th:href="@{/articles/{id}(id=${item.id})}" class="btn btn-primary">보러 가기</a>
            </div>
        </div>
        <br>
    </div>

    <button type="button" class="btn btn-secondary" onclick="location.href='/logout'">로그아웃</button>
</div>

<script src="/js/article.js"></script>
```

---

# ✅ 실행 테스트하기

## 테스트를 위한 환경 변수 추가
- application.yml에 datasource 환경 변수 추가

application.yml
```
spring:
    jpa:
        show-sql: true
        properties:
            hibernate:
                format_sql: true
        defer-datasource-initialization: true
    datasource: # 데이터베이스 정보 추가
        url: jdbc:h2:mem:testdb
        username: sa
        password:
    h2: # H2 콘솔 활성화
        console:
            enabled: true   
```

