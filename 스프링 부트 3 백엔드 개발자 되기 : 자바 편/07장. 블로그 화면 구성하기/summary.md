# ✅ 사전지식 : 타임리프

## 템플릿 엔진
- 스프링 서버에서 데이터를 받아 우리가 보는 웹 페이지에 데이터를 넣어 보여주는 도구
- 이름, 나이라는 키로 데이터를 템플릿 엔진에 넘겨주고 템플릿 엔진은 이를 받아 HTML에 값을 적용

간단한 템플릿 문법을 위한 예시
```
<h1 text=${이름}>
<p text=${나이}>
```

서버에서 보내준 데이터 예
```
{
    이름: "홍길동",
    나이: 11
}
```

## 타임리프 표현식과 문법

타임리프 표현식

| 표현식    | 설명                                 |
|--------|------------------------------------|
| ${...} | 변수의 값 표현식                          |
| #{...} | 속성 파일 값 표현식                        |
| @{...} | URL 표현식                            |
| *{...} | 선택한 변수의 표현식 th:object에서 선택한 객체 접근  |

타임리프 문법

| 표현식 | 설명 | 예제 |
|-|-|-|
| th:text | 텍스트를 표현할 때 사용 | th:text="${person.name}" |
| th:each | 컬렉션을 반복할 때 사용 | th:each="person : ${persons}" |
| th:if | 조건이 true일 때만 표시 | th:if="${person.age} >= 20" |
| th:unless | 조건이 false일 때만 표시 | th:unless="${person.age} >= 20" |
| th:href | 이동 경로 | th:href="@{/person(id=${person.id})}" |
| th:with | 변수값으로 지정 | th:with="name = ${person.name}" |
| th:object | 선택한 객체로 지정 | th:object="${person}" |

## 타임리프 의존성 추가
```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
```

## 타임리프 컨트롤러 예시
ExampleController.java
```
@Controller // 컨트롤러 명시
public class ExampleController {
    
    @GetMapping("/thymeleaf/example")
    public String thymeleafExample(Model model) { // 뷰로 데이터를 넘겨주는 모델 객체
        Person examplePerson = new Person();
        examplePerson.setId(1L);
        examplePerson.setName("홍길동");
        examplePerson.setAge(11);
        examplePerson.setHobbies(List.of("운동", "독서"));
        
        model.addAttribute("person", examplePerson); // Person 객체 저장
        model.addAttribute("today", LocalDate.now());
        
        return "example"; // example.html라는 뷰 조ㅗ히
    }
        @Setter
        @Getter
        class Person {
            private Long id;
            private String name;
            private int age;
            private List<String> hobbies;
        }
}
```

Model
- HTML 쪽으로 값을 넘겨주는 객체
- 따로 생성할 필요 없이 인자로 선언하면 스프링이 알아서 만들어줌
- addAttribute() 메서드로 모델에 값을 저장
- 반환하는 값은 뷰의 이름임 example.html 파일을 찾아서 이동

## 뷰 예시
example.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>타임리프 익히기</h1>
<!-- LocalDate를 yyyy-MM-dd 포맷으로 변경 -->
<p th:text="${#temporals.format(today, 'yyyy-MM-dd')}"></p>
<div th:object="${person}"> <!-- person을 선택한 객체로 지정 -->
    <p th:text="|이름 : *{name}|"></p>
    <p th:text="|나이 : *{age}|"></p>
    <p>취미</p>
    <ul th:each="hobby : *{hobbies}"> <!-- hobbies 개수만큼 반복 -->
        <li th:text="${hobby}"></li>
        <!-- 반복 대상이 운동이라면 '대표 취미'라는 표시 추가-->
        <span th:if="${hobby == '운동'}">(대표 취미)</span>
    </ul>
</div>
<!-- 1번 블로그 글을 보러 이동 -->
<a th:href="@{/api/articles/{id}(id=${person.id})}">글 보기</a>
</body>
</html>
```
#temporlas.format()
- LocalDate 타입인 오늘 날짜를 yyyy-MM-dd 형식의 String 타입으로 포매팅

th:object
- 모델에서 "person" 이라는 키를 가진 객체의 데이터를 지정함
- 하위 태그에서 *{...}를 사용해 부모 태그에 적용한 객체 값에 접근할 수 있음

th:text
- || 사용해서 '이름:' 문자열과 person 객체의 name값을 이어 붙임

th:each
- 객체의 hobbies 개수만큼 반복하는 반복자

th:if
- hobby의 값이 운동이면 (대표 취미) 라는 문자열을 추가로 표시함

---