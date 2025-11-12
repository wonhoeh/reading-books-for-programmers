# ✅ 사전 지식

## OAuth?
- 만들고 있는 서비스에서 사용자 정보를 취득하기 위한 방법
- 클라이언트는 인증 서버에서 토큰을 발급받고
- 토큰을 이용해 리소스 서버에서 사용자 정보(리소스 오너)를 요청할 수 있음

## 용어 정리

### 리소스 오너
- 서비스를 이용하는 사용자(User)
- 자신의 데이터를 갖고 있는 주체

### 리소스 서버
- 실제로 데이터(API)를 제공하는 서버
- 리소스 오너의 정보를 가지며 리소스 오너의 정보를 보호하는 주체
- 네이버, 구글, 페이스북이 리소스 서버에 해당

### 인증 서버
- 로그인 인증을 진행하고 액세스 토큰, 리프레시 토큰을 발급하는 역할
- 클라이언트 애플리케이션에게 리소스 오너 정보에 접근할 수 있는 토큰을 발급

### 클라이언트 애플리케이션
- 사용자 데이터를 사용하려고 하는 서비스
- 만들고 있는 서비스에 해당

<br>

## OAuth가 리소스 오너 정보를 취득하는 방법

### 권한 부여 코드 승인 타입
- 클라이언트가 리소스에 접근하는데 사용함
- 권한에 접근할 수 있는 코드와 리소스 오너에 대한 액세스 토큰을 발급받는 방식

### 1. 권한 요청
- 스프링 부트 서버가 특정 사용자 데이터에 접근하기 위해 구글 권한 서버에 요청을 보내는 것을 의미함
- 권한 요청을 위한 파라미터 예시
```
GET spring-authorization-server.example/authorize?
  client_id=66a36b4c2&
  redirect_uri=http://localhost:8080/myapp&
  response_type=code&
  scope=profile
```
- client_id
  - 인증 서버가 클라이언트에 할당한 고유 식별자
  - 이 값은 클라이언트 애플리케이션을 OAuth 서비스에 등록할 때 서비스에 생성함
- redirect_uri
  - 로그인 성공 시 이동해야 하는 URI
- response_type
  - 클라이언트가 제공받길 원하는 응답 타입
  - 인증 코드를 받을 때 code값을 포함해야함
- scope
  - 제공받고자 하는 리소스 오너의 정보 목록

### 2. 데이터 접근용 권한 부여
- 인증 서버에 요청을 처음 보내는 경우, 사용자에게 로그인 페이지로 이동시키고 사용자 동의를 받음
- 이후 인증 서버에서 동의 내용을 저장하고 있기 때문에 로그인만 진행함
- 로그인이 성공되면 권한 부여 서버는 데이터에 접근할 수 있게 인증 및 권한 부여를 수신함

### 3. 인증 코드 제공
- 사용자가 로그인에 성공하면 권한 요청 시에 파라미터로 보낸 redirect_uri로 이동시킴
- 파라미터에 인증 코드를 함께 제공함
```
GET http://localhost:8080/myapp?code=a1s2f3mcj2
```

### 4. 액세스 토큰 응답이란?
- 인증 코드를 받으면 액세스 토큰으로 교환해야 함
- 액세스 토큰은 로그인 세션에 대한 보안 자격을 증명하는 식별 코드를 의미함
- /token POST 요청을 보냄
```
POST spring-authorization-server.example.com/token
{
  "client_id": "66a36b4c2",
  "client_secret": "aabb11dd44",
  "redirect_uri": "http://localhost:8080/myapp",
  "grant_type": "authorization_code",
  "code": "a1b2c3d4e5f6g7h8"
}
```
- client_secret
  - OAuth 서비스에 등록할 때 제공받는 비밀키
- grant_type
  - 권한 유형을 확인하는데 사용함
  - 권한 서버는 요청 값을 기반으로 유효한 정보인지 확인하고 액세스 토큰을 응답함
```
{
  "access_token": "aasdffb",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "openid profile",
... 생략 ...
}  
```

### 5. 액세스 토큰으로 API 응답 & 반환
- 제공받은 액세스 토큰으로 리소스 오너의 정보를 가져올 수 있음
- 정보가 필요할 때마다 API 호출을 통해 정보를 가져오고 리소스 서버는 토큰이 유효한지 검사한 뒤에 응답함
```
GET spring-authorization-resource-server.example.com/userinfo
Header: Authorization: Bearer aasdffb
```


<br>

## 권한 부여 코드 승인 타입 흐름 정리 (카카오로 OTT 서비스 로그인)

1. 사용자(Resource Owner) <br>
→ OTT 서비스(Client App)에 로그인 클릭

2. Client App <br>
→ 카카오 Authorization Server 로 로그인 요청 redirect

3. Authorization Server <br>
→ 사용자에게 "진짜 이 앱이 너의 정보 사용해도 돼?" 물어봄 (동의 화면)

4. 사용자 <br>
→ 동의함

5. Authorization Server <br>
→ Authorization Code 를 Client App 서버로 전달

6. Client App <br>
→ Authorization Code 를 이용하여 Access Token, Refresh Token 을 요청

7. Authorization Server <br>
→ Access Token & Refresh Token 발급

8. Client App <br>
→ Access Token 사용하여 Resource Server 에 사용자 정보 요청

9. Resource Server <br>
→ Access Token 이 유효하면 사용자 프로필 등을 응답

<br>

## 쿠키
- 방문한 웹 사이트의 서버에서 로컬 환경에 저장하는 작은 데이터를 의미
- 쿠키 값으로 이전에 방문한 적이 있는지 알 수 있음. 로그인 정보를 유지할 수 있음
- 키와 값으로 이루어짐
- 만료 기간, 도메인 등의 정보를 갖음
- 클라이언트가 정보를 요청하면 서버에서 정보를 쿠키의 헤더에 담아서 응답

---

# 스프링 시큐리티로 OAuth2 구현하고 적용하기
- 쿠키 관리 클래스 구현하기
- OAuth2에서 제공받은 인증 객체로 사용자 정보를 가져오는 역할을 하는 서비스 구현하기
- OAuth2 설정 파일 구현하기
- 테스트용도 뷰 구현하기


## 의존성 추가하기
build.gradle
```
dependencies {
... 생략 ...
  // OAuth2를 사용하기 위한 스타터 추가
  implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
}
```

## 쿠키 관리 클래스 구현하기
- OAuth2 인증 플로우를 구현할 때 사용할 유틸리티로 쿠키 관리 클래스를 구현하기
util/CookieUtil.java
```
package com.example.blogservice.util;

import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.util.SerializationUtils;

import java.util.Base64;

public class CookieUtil {
    // 요청값(이름, 값, 만료 기간)을 바탕으로 쿠키 추가
    public static void addCookie(HttpServletResponse response, String name, String value, int maxAge) {
        Cookie cookie = new Cookie(name, value);
        cookie.setPath("/");
        cookie.setMaxAge(maxAge);
        response.addCookie(cookie);
    }

    // 쿠키의 이름을 입력받아 쿠키 삭제
    public static void deleteCookie(HttpServletRequest request, HttpServletResponse response, String name) {
        Cookie[] cookies = request.getCookies();
        if (cookies == null) {
            return;
        }

        for (Cookie cookie : cookies) {
            if (name.equals(cookie.getName())) {
                cookie.setValue("");
                cookie.setPath("/");
                cookie.setMaxAge(0);
                response.addCookie(cookie);
            }
        }
    }

    // 객체를 직렬화해 쿠키의 값으로 변환
    public static String serialize(Object obj) {
        return Base64.getUrlEncoder()
                .encodeToString(SerializationUtils.serialize(obj));
    }

    // 쿠키를 역직렬화해 객체로 변환
    public static <T> T deserialize(Cookie cookie, Class<T> clazz) {
        return clazz.cast(
                SerializationUtils.deserialize(
                        Base64.getUrlDecoder().decode(cookie.getValue())
                )
        );
    }
}
```
- addCookie
  - 요청값(이름, 값, 만료 기간)을 바탕으로 HTTP 응답에 쿠키를 추가함
- deleteCookie
  - 쿠키 이름을 입력받아 쿠키를 삭제함
  - 실제로 삭제하는 방법은 없으므로 쿠키를 빈 값으로 바꾸고 만료 시간을 0으로 설정해서 처리함
- serialize
  - 객체를 직렬화해 쿠키의 값으로 들어갈 값으로 변환함
- deserialize
  - 객체를 역직렬화해 객체로 변환함

## OAuth2 서비스 구현하기
- 사용자 정보를 조회해 users 테이블에 사용자 정보가 있다면 리소스 서버에서 제공해주는 이름을 업데이트하고
- 없다면 users 테이블에 새 사용자를 생성해 데이터베이스에 저장하는 서비스를 구현함
domain/User.java
````
public class User implements UserDetails {

... 생략 ...

  // 사용자 이름
  @Column(name = "nickname", unique = true)
  private String nickname;
  
  // 생성자에 nickname 추가
  @Builder
  public User(String email, String password, String nickname) {
    this.email = email;
    this.password = password;
    this.nickname = nickname;
  }
  
  ... 생략 ...
  
  // 사용자 이름 변경
  public User update(String nickname) {
    this.nickname = nickname;
    
    return this;
  }
}
````

- config/oauth/OAuth2UserCustomService.java
- loadUser() 리소스 서버에서 보내주는 사용자 정보를 불러오는 메서드로 사용자를 조회하고
- 사용자가 users 테이블에 사용자 정보가 있다면 이름을 업데이트하고
- 없다면 saveOrUpdate() 메서드를 실행해 users 테이블에 회원 데이터를 추가함
````
package com.example.blogservice.config.oauth;

import com.example.blogservice.domain.User;
import com.example.blogservice.domain.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest;
import org.springframework.security.oauth2.core.OAuth2AuthenticationException;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Service;

import java.util.Map;

@RequiredArgsConstructor
@Service
public class OAuth2UserCustomService extends DefaultOAuth2UserService {
    private final UserRepository userRepository;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        // 요청을 바탕으로 유저 정보를 담은 객체 반환
        OAuth2User user = super.loadUser(userRequest);
        saveOrUpdate(user);
        return user;
    }

    // 유저가 있으면 업데이트, 없으면 유저 생성
    private User saveOrUpdate(OAuth2User oAuth2User) {
        Map<String, Object> attributes = oAuth2User.getAttributes();
        String email = (String) attributes.get("email");
        String name = (String) attributes.get("name");
        User user = userRepository.findByEmail(email)
                .map(entity -> entity.update(name))
                .orElse(User.builder()
                        .email(email)
                        .nickname(name)
                        .build());
        return userRepository.save(user);
    }


}
````

## OAuth2 설정 파일 구현하기
/config/WebOAuthSecurityConfig.java
```
package com.example.blogservice.config;

import com.example.blogservice.config.jwt.TokenProvider;
import com.example.blogservice.config.oauth.OAuth2AuthorizationRequestBasedOnCookieRepository;
import com.example.blogservice.config.oauth.OAuth2SuccessHandler;
import com.example.blogservice.config.oauth.OAuth2UserCustomService;
import com.example.blogservice.domain.repository.RefreshTokenRepository;
import com.example.blogservice.service.UserService;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpStatus;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityCustomizer;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.HttpStatusEntryPoint;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import static org.springframework.boot.autoconfigure.security.servlet.PathRequest.toH2Console;

@RequiredArgsConstructor
@Configuration
public class WebOAuthSecurityConfig {
    private final OAuth2UserCustomService oAuth2UserCustomService;
    private final TokenProvider tokenProvider;
    private final RefreshTokenRepository refreshTokenRepository;
    private final UserService userService;

    @Bean
    public WebSecurityCustomizer configure() { // 스프링 시큐리티 기능 비활성화
        return (web) -> web.ignoring()
                .requestMatchers(toH2Console())
                .requestMatchers("/img/**", "/css/**", "/js/**");
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        // 1. 토큰 방식으로 인증을 하기 때문에 기존에 사용하던 폼로그인, 세션 비활성화
        return http
                .csrf(AbstractHttpConfigurer::disable)
                .httpBasic(AbstractHttpConfigurer::disable)
                .formLogin(AbstractHttpConfigurer::disable)
                .logout(AbstractHttpConfigurer::disable)
                .sessionManagement(management -> management.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                // 2. 헤더를 확인할 커스텀 필터 추가
                .addFilterBefore(tokenAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)
                // 3. 토큰 재발급 URL을 인증 없이 접근 가능하도록 설정. 나머지 API URL은 인증 필요
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/api/token").permitAll()
                        .requestMatchers("/api/**").authenticated()
                        .anyRequest().permitAll())
                .oauth2Login(oauth2 -> oauth2
                        .loginPage("/login")
                        // 4. Authorization 요청과 관련된 상태 저장
                        .authorizationEndpoint(authorizationEndpoint ->
                                authorizationEndpoint.authorizationRequestRepository(oAuth2AuthorizationRequestBasedOnCookieRepository()))
                        // 5. 인증 성공 시 실행할 핸들러
                        .successHandler(oAuth2SuccessHandler())
                        .userInfoEndpoint(userInfoEndpoint -> userInfoEndpoint.userService(oAuth2UserCustomService))
                )
                .logout(logout -> logout.logoutSuccessUrl("/login"))
                // 6. /api로 시작하는 url인 경우 401 상태 코드를 반환하도록 예외 처리
                .exceptionHandling(exceptionHandling -> exceptionHandling
                        .defaultAuthenticationEntryPointFor(
                                new HttpStatusEntryPoint(HttpStatus.UNAUTHORIZED),
                                request -> request.getRequestURI().startsWith("/api/")
                        ))
                .build();
    }

    @Bean
    public OAuth2SuccessHandler oAuth2SuccessHandler() {
        return new OAuth2SuccessHandler(tokenProvider, refreshTokenRepository, oAuth2AuthorizationRequestBasedOnCookieRepository(), userService);
    }

    @Bean
    public TokenAuthenticationFilter tokenAuthenticationFilter() {
        return new TokenAuthenticationFilter(tokenProvider);
    }

    @Bean
    public OAuth2AuthorizationRequestBasedOnCookieRepository oAuth2AuthorizationRequestBasedOnCookieRepository() {
        return new OAuth2AuthorizationRequestBasedOnCookieRepository();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```
- filterChain() 메서드
  - 토큰 방식으로 인증을 하므로 기존 폼 로그인, 세션 기능을 비활성화함


- addFilterBefore() 헤더값 확인용 커스텀 필터 추가
  - UsernamePasswordAuthenticationFilter(로그인 처리 필터) 전에 TokenAuthenticationFilter를 실행시키겠다는 의미
  - 요청 헤더에서 토큰을 가져와서 유효한 지 검사하고
  - SecurityContextHolder에 Authentication 저장
    - 컨트롤 계층에서 @AuthenticationPrincipal 등을 통해 사용자 정보를 가져오기 위함


- authorizeHttpRequests() 메서드 URL 인증 설정
  - 토큰 재발급 URL은 인증 없이 접근하도록 설정하고 나머지 API들은 모두 인증을 해야 접근하도록 설정


- oauth2Login() 저장소와 핸들러 설정
  - OAuth2에 필요한 정보를 세션이 아닌 쿠키에 저장해서 쓸 수 있도록 인증 요청과 관련된 상태를 저장할 저장소를 설정함
  - 인증 성공 시 실행할 핸들러도 설정


- exceptionHandling() 메서드 예외 처리 설정
  - /api로 시작하는 url인 경우 인증 실패 시 401 상태 코드(Unauthorized) 반환


### 인증 요청과 관련된 상태를 저장할 저장소 구현하기
- 인증 서버에 다녀오는 사이에 상태를 잃지 않기 위해 쿠키에 저장함
- 권한 인증 흐름에서 클라이언트의 요청을 유지하는데 사용하는 AuthorizationRepository 클래스를 구현해서
- 쿠키를 사용해 OAuth의 정보를 가져오고 저장하는 로직 작성
/config/oauth/OAuth2AuthorizationRequestBasedOnCookieRepository.java
````
package com.example.blogservice.config.oauth;

import com.example.blogservice.util.CookieUtil;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.oauth2.client.web.AuthorizationRequestRepository;
import org.springframework.security.oauth2.core.endpoint.OAuth2AuthorizationRequest;
import org.springframework.web.util.WebUtils;

public class OAuth2AuthorizationRequestBasedOnCookieRepository implements AuthorizationRequestRepository<OAuth2AuthorizationRequest> {

    public final static String OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME = "oauth2_auth_request";
    private final static int COOKIE_EXPIRE_SECONDS = 18000;

    @Override
    public OAuth2AuthorizationRequest removeAuthorizationRequest(HttpServletRequest request,
                                                                 HttpServletResponse response) {
        return this.loadAuthorizationRequest(request);
    }

    @Override
    public OAuth2AuthorizationRequest loadAuthorizationRequest(HttpServletRequest request) {
        Cookie cookie = WebUtils.getCookie(request, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME);
        return CookieUtil.deserialize(cookie, OAuth2AuthorizationRequest.class);
    }

    @Override
    public void saveAuthorizationRequest(OAuth2AuthorizationRequest authorizationRequest,
                                         HttpServletRequest request,
                                         HttpServletResponse response) {
        if (authorizationRequest == null) {
            removeAuthorizationRequestCookies(request, response);
            return;
        }
        CookieUtil.addCookie(response, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME,
                CookieUtil.serialize(authorizationRequest), COOKIE_EXPIRE_SECONDS);
    }

    public void removeAuthorizationRequestCookies(HttpServletRequest request, HttpServletResponse response) {
        CookieUtil.deleteCookie(request, response, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME);
    }
}
````
