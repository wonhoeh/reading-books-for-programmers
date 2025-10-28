# ✅ 사전 지식

## OAuth 용어 정리

### 리소스 오너
- 서비스를 이용하는 사용자 (User)
- 자신의 데이터를 갖고 있는 주체

### 리소스 서버
- 실제로 데이터(API)를 제공하는 서버
- 리소스 오너의 정보를 가지며 리소스 오너의 정보를 보호하는 주체
- 네이버, 구글, 페이스북이 리소스 서버에 해당

### 인증 서버
- 로그인 인증을 진행하고 액세스 토큰, 리프레시 토큰을 발급하는 역할
- 클라이언트에게 리소스 오너의 정보에 접근할 수 있는 토큰을 발급

### 클라이언트 애플리케이션
- 사용자 데이터를 사용하려고 하는 서비스
- 만들고 있는 서비스에 해당

<br>

## 권한 부여 코드 승인 타입(authorization code grant type)
- 클라이언트가 리소스에 접근하는데 사용
- 권한에 접근할 수 있는 코드와 리소스 오너에 대한 액세스 토큰을 발급받는 방식

<br>

## 실제 흐름 예시 (카카오 로그인으로 OTT 서비스 로그인)

1. 사용자(Resource Owner)
→ OTT 서비스(Client App)에 로그인 클릭

2. Client App
→ 카카오 Authorization Server 로 로그인 요청 redirect

3. Authorization Server
→ 사용자에게 "진짜 이 앱이 너의 정보 사용해도 돼?" 물어봄 (동의 화면)

4. 사용자
→ 동의함

5. Authorization Server
→ Authorization Code 를 Client App 서버로 전달

6. Client App
→ Authorization Code 를 이용하여 Access Token, Refresh Token 을 요청

7. Authorization Server
→ Access Token & Refresh Token 발급

8. Client App
→ Access Token 사용하여 Resource Server 에 사용자 정보 요청

9. Resource Server
→ Access Token 이 유효하면 사용자 프로필 등을 응답

## 권한 요청
- Client App이 Authorization Server(카카오나 구글)에 사용자 데이터에 접근하기 위해 요청을 보내는 것
- 권한 요청을 위한 파라미터 예시
```
Get spring-authorization-server.example/authorize?
    client_id=66a36b4c2&
    redirect_uri=http://localhost8080/myapp&
    response_type=code&
    scope=profile
```
- client_id
  - Client App에 할당된 고유 식별자
  - OAuth 서비스에 등록할 때 서비스에서 생성하는 값
- redirect_url
  - 로그인 성공시 이동해야하는 URI
- response_type
  - 클라이언트가 제공받길 원하는 응답 타입
  - 인증 코드를 받을 때 code값을 포함해야 함
- scope
  - 제공받고자 하는 리소스 오너의 정보 목록

## 액세스 토큰 응답
- 인증 코드를 받으면 액세스 토큰으로 교환
- /token POST 요청 예
```
POST spring-authorization-server.example.com/token
{
    "client_id": "66a36b4c2",
    "client-secret": "aabb11dd44",
    "redirect_uri": "http://localhost:8080/myapp",
    "grant_type": "authorization_code",
    "code": "a1b2c3d4e5f6g7h8"
}
```
- client_secret
  - Oauth 서비스에 등록할 때 제공받는 비밀키
- grant_type
  - 권한 유형을 확인하는 데 사용
  - authorization_code로 설정해야함
  - 요청 값을 기반으로 유효한 정보인지 확인하고 유효한 정보면 액세스 토큰을 응답
- 액세스 토큰 응답 값의 예
```
{
    "access_token": "aasdffb",
    "token_type": "Bearer",
    "expires_in": 3600,
    "scope": "openid profile",
    ... 생략 ...
}
```

### 액세스 토큰으로 API 응답&반환
- 제공받은 액세스 토큰으로 리소스 오너의 정보를 가져오기
- 리소스 서버는 토큰이 유효한지 검사한 뒤 응답
```
GET spring-authorization-resource-server.example.com/userinfo
Header: Authorization: Bearer aasdffb
```

<br>

## 쿠키
- 방문한 웹 사이트의 서버에서 로컬 환경에 저장하는 작은 데이터를 의미
- 쿠키 값으로 이전에 방문한 적이 있는지 알 수 있음. 로그인 정보를 유지할 수 있음
- 키와 값으로 이루어짐
- 만료 기간, 도메인 등의 정보를 갖음
- 클라이언트가 정보를 요청하면 서버에서 정보를 쿠키의 헤더에 담아서 응답

---



