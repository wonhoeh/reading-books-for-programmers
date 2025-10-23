# ✅ 스프링과 스프링 부트
## 스프링의 등장
- 서버 성능, 안정성, 보안을 매우 높은 수준으로 제공하는 도구
- 엔터프라이즈 애플리케이션을 위한 개발 환경을 제공, 기능 개발에 집중하게 됨
## 스프링 부트 - 스프링을 더 쉽게!
- 설정이 복잡한 스프링 프레임워크를 더 쉽고 빠르게 이용할 수 있도록 만들어주는 도구
- starter(의존성 세트)를 사용해 간편하게 의존성 사용
## 스프링 부트의 주요 특징
- 톰캣이 내장되어 있어서 웹 애플리케이션 서버를 따로 설치하지 않아도 실행 가능
- 빌드 구성을 단순화하는 부트 스타터를 제공
- XML 설정을 하지 않고 자바 코드로 모두 작성
- JAR를 이용해서 자바 옵션만으로도 배포 가능
- 애플리케이션의 모니터링 및 관리 도구인 스프링 액츄에이터를 제공
## 스프링 vs 스프링 부트로 개발할 때 차이점

|                |스프링|스프링 부트|
|----------------|--|---------|
| 목적             |엔터프라이즈 애플리케이션 개발을 더 쉽게 만들기| 스프링 개발을 더 빠르고 쉽게 하기|
| 설정 파일          |개발자가 수동으로 구성|자동 구성
| XML            |일부 파일은 XML로 직접 생성하고 관리|사용하지 않음
| 인메모리 데이터베이스 지원 |지원하지 않음|인메모리 데이터베이스 자동 설정 지원
| 서버(WAS)        |프로젝트를 띄우는 서버를 별도로 수동 설정|내장형 서버를 제공해 별도의 설정이 필요 없음
- WAS
    - 웹 애플리케이션을 실행하기 위한 장치
    - 스프링 애플리케이션은 일반적으로 톰캣에 배포됨 
---
# ✅ 스프링 콘셉트 공부하기
스프링 프레임워크가 돌아가는 원리를 이해하기 위해!
## ⚡ 제어의 역전과 의존성 주입
스프링은 모든 기능의 기반을 제어의 역전과 의존성 주입에 두고 있음
### 1. 제어의 역전(IOC:Inversion of Control)
다른 객체를 직접 생성하거나 제어하는 것이 아니라 외부에서 관리하는 객체를 가져와 사용하는 것을 말함
```
public class A {
  private B b; // 코드에서 객체를 생성하지 않고, 어디선가 받아온 객체를 b에 할당
}
```
  
### 2. 의존성 주입(DI:Dependency Injection)
#### a. '의존한다'의 의미
- 'A가 B에 의존한다' 는 'A는 B 없이 제대로 동작할 수 없다' 라는 뜻
```
public class A {
  private B b;
  
  public A() {
    this.b = new B(); // A는 B 없이는 동작 불가
  }
  
  public void doSomething() {
    b.run(); // B의 기능을 써야 A의 기능이 완성됨
  }
}
```
#### b. 주입(Injection)이 필요한 이유
- A가 new B() 코드를 갖고 있으면 다음과 같은 문제가 있음
  - B에 코드 변경이 있으면 A도 코드를 변경해야함! 결합도(coupling) ↑ 
  - 유지보수성이 떨어짐
- 그래서 A가 B를 직접 만들지 말고 '누군가' 대신 넣어주면 문제가 해결됨
- 누군가 객체를 넣어주는 걸 의존성 주입(Dependency Injection) 이라고 함
#### c. 스프링의 역할

```
@Component
public class B {}

@Component
public class A {
  private final B b;
  
  @Autowired
  public A(B b) { // 스프링이 대신 넣어줌
    this.b = b;
  }
}
```
- 스프링 컨테이너가 '누군가'의 역할을 함
- A는 여전히 B에 의존하지만
- 의존성 주입 덕분에 A가 B를 직접 관리하지 않아도 된다

#### d. 정리

|개념|의미|
|-|-|
|의존(Dependency)|A가 B 없이는 일을 못함
|의존성 주입(DI)|A가 B를 직접 만들지 않고, 외부(스프링)가 대신 넣어줌
|결과|코드의 결합도↓, 유연성↑, 테스트 용이성↑

## ⚡ 빈과 스프링 컨테이너
### 1. 스프링 컨테이너
- 빈이 생성되고 소멸되기까지의 생명주기를 관리함
- @Autowired를 사용해 빈을 주입받을 수 있게 DI를 지원함
### 2. 빈(Bean)
- 스프링의 객체 : 스프링 컨테이너가 생성하고 관리하는 객체
- 위의 코드에서 B가 빈임
- @Component를 붙이면 클래스가 빈으로 등록됨
- MyBean -> myBean (소문자변환해서 저장)

## ⚡ 관점 지향 프로그래밍(AOP)
- 프로그래밍에 대한 관심을 핵심 관점과 부가 관점으로 나누어서 모듈화하는 것
- 핵심 관점: 계좌 이체, 고객 관리 로직
- 부가 관점: 로깅, 데이터베이스 연결

## ⚡ 이식 가능한 서비스 추상화(PSA)
- 어느 기술을 사용하던 일관된 방식으로 처리하도록 하는 것
- JPA, MyBatis, JDBC 어느 기술이던 일관된 방식으로 접근할 수 있게 인터페이스를 지원
---
# ✅ 스프링 부트 3 둘러보기
## 첫 번째 스프링 부트 3 예제 만들기
```
@RestController
public class TestController {
  @GetMapping("/test")
  public String test() {
    return "Hello World!";
  }
}
```
- http://localhost:8080/test
  - localhost: 현재 사용 중인 컴퓨터
  - 8080: 포트 번호
  - test: 경로

## 스프링 부트 스타터 살펴보기
- 스프링 부트 스타터는 의존성이 모여 있는 그룹
- spring-boot-starter-{작업유형} 이라는 명명규칙이 있음

|스타터|설명|
|-|-|
|spring-boot-starter-web|Spring MVC를 사용해서 RESTful 웹 서비스를 개발할 때 필요한 의존성 모음
|spring-boot-starter-test|스프링 애플리케이션을 테스트하기 위해 필요한 의존성 모음
|spring-boot-starter-validation|유효성 검사를 위해 필요한 의존성 모음
|spring-boot-starter-actuator|모니터링을 위해 애플리케이션에서 제공하는 다양한 정보를 제공하기 쉽게 하는 의존성 모음
|spring-boot-starter-data-jpa|ORM을 사용하기 위한 인터페이스의 모음인 JPA를 더 쉽게 사용하기 위한 의존성 모음

### 스프링 부트가 의존성을 가져오는 방법
- 스프링 부트는 현재 버전에 맞는 라이브러리를 알아서 관리함
- 어떤 의존성을 사용하는지 버전별 확인이 필요하면 스프링 공식 문서를 참고

## 자동 구성
- 스프링 부트는 서버를 시작할 때 구성 파일을 읽어와서 설정함
- 자동 설정은 META-INF에 있는 spring.factories 파일에 담겨 있음
- 스프링 부트를 시작할 때 이 파일에 설정되어 있는 클래스는 모두 불러옴
- 이후 프로젝트에서 사용할 것들만 자동으로 구성해 등록함
- 자동 구성이 없다면 개발자가 특정 기술을 사용할 때 마다 직접 설정해야함

## 스프링 부트 3 와 자바 버전
- 스프링 부트 3 이전과 이후는 사용할 수 있는 자바 버전 범위가 다름
- 스프링 부트 2는 자바 8 버전 이상을 사용
- 스프링 부트 3은 자바 17 버전 이상을 사용
- 패키지 네임스페이스가 javax.* 에서 jakarta.* 로 변경
- 시작 시간과 메모리 오버 헤드를 줄일 GraalVM 기반의 스프링 네이티브를 공식 지원
---
# ✅ 스프링 부트 3 코드 이해하기
## @SpringBootApplication 이해하기
```
@SpringBootApplication
public class SpringBootDeveloperApplication {
  public static void main(String[] args) {
    SpringApplication.run(SpringBootDeveloperApplication.class, args);
  }
}
```

- 스프링 부트의 시작점
- 스프링 부트 사용에 필요한 기본 설정을 해줌
- 첫 번째 인수: 애플리케이션의 메인 클래스
- 두 번째 인수: 커맨드 라인의 인수들

```
@Target(Element.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documeted
@Inherited
@SpringBootConfiguration // 스프링 부트 관련 설정
@ComponentScan(excludeFilters = {
@Filter(type = FilterType.CUSTOM,
  // 사용자가 등록한 빈을 읽고 등록
  classes = TypeExcludeFilter.class),
  @Filter(type = FilterType.CUSTOM,
  classes = AutoConfigurationExcludeFilter.class)
})
@EnableAutoConfiguration // 자동으로 등록된 빈을 읽고 등록
public @interface SpringBootApplication {
... 생략 ...
}
```

### @SpringBootConfiguration
- 스프링 부트 관련 설정을 나타내는 애너테이션

### @ComponentScan
- 사용자가 등록한 빈을 읽고 등록하는 애너테이션
- @Component 라는 애너테이션을 가진 클래스들을 찾아 빈으로 등록하는 역할
- @Component 를 감싼 애너테이션 목록

|애너테이션명|설명|
|-|-|
|@Configuration|설정 파일 등록
|@Repository|ORM 매핑
|@Controller, @RestController|라우터
|@Service|비즈니스 로직

### @EnableAutoConfiguration
- 스프링 부트에서 자동 구성을 활성화하는 애너테이션
- spring.factories 에 있는 클래스들이 모두 이 애너테이션을 사용할 때 자동 설정됨



## 테스트 컨트롤러 살펴보기
```
@RestController
public class TestController {
  // /test GET 요청이 오면 test() 메서드 실행
  @GetMapping("/test") 
  public String test() {
    return "Hello World!";
  }
}
```
- @RestController는 라우터 역할을 하는 애너테이션
  - 라우터란 HTTP 요청과 메서드를 연결하는 장치를 의미
- @RestController = @Controller + @ResponseBody
  - @Controller 내부로 들어가면 @Component 애너테이션이 있음
- 그래서 @ComponentScan을 통해 @RestController가 빈으로 등록됨