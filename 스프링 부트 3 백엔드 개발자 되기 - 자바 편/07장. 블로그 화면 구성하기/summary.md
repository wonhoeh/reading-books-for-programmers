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

### 타임리프 표현식

| 표현식    | 설명                                 |
|--------|------------------------------------|
| ${...} | 변수의 값 표현식                          |
| #{...} | 속성 파일 값 표현식                        |
| @{...} | URL 표현식                            |
| *{...} | 선택한 변수의 표현식 th:object에서 선택한 객체 접근  |

### 타임리프 문법

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

### Model
- HTML 쪽으로 값을 넘겨주는 객체
- 따로 생성할 필요 없이 인자로 선언하면 스프링이 알아서 만들어줌
- addAttribute() 메서드로 모델에 값을 저장
- 반환하는 값은 뷰의 이름임 example.html 파일을 찾아서 이동

## HTML 뷰 예시
### example.html
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
#temporals.format()
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

# ✅ 블로그 글 목록 뷰 구현하기

## 컨트롤러 메서드
### ArticleListViewResponse.java
- 뷰에게 데이터를 전달하기 위한 객체
```
@Getter
public class ArticleListViewResponse {
    
    private final Long id;
    private final String title;
    private final String content;
    
    public ArticleListViewResponse(Article article) {
        this.id = article.getId();
        this.title = article.getTitle();
        this.content = article.getContent();
    }
}
```

### BlogViewController.java
- /articles GET 요청을 처리할 코드 작성
```
@RequiredArgsConstructor
@Controller
public class BlogViewController {
    
    private final BlogService blogService;
    
    @GetMapping("/articles")
    public String getArticles(Model model) {
        List<ArticleListViewResponse> articles = blogService.findAll().stream()
                .map(ArticleListViewResponse::new)
                .toList();
        model.addAttribute("articles", articles); // 블로그 글 리스트 저장
        
        return "articleList"; // articleList.html 뷰 조회
    }
}   
```

## HTML 뷰
### articleList.html
- 모델에 전달한 블로그 글 리스트 개수만큼 반복해 글 정보를 보여주도록 코드 작성
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>블로그 글 목록</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css">
</head>
<body>
<div class="p-5 mb-5 text-center</> bg-light">
  <h1 class="mb-3">My Blog</h1>
  <h4 class="mb-3">블로그에 오신 것을 환영합니다.</h4>
</div>

<div class="container">
  <div class="row-6" th:each="item : ${articles}"> <!-- article 개수만큼 반복 -->
    <div class="card-header" th:text="${item.id}"> <!-- item id 출력 -->
    </div>
    <div class="card-body">
      <h5 class="card-title" th:text="${item.title}"></h5>
      <p class="card-text" th:text="${item.content}"></p>
      <a href="#" class="btn btn-primary">보러 가기</a>
    </div>
  </div>
</div>
</body>
</html>
```

---

# ✅ 블로그 글 뷰 구현하기
- [보러 가기] 버튼을 누르면 블로그 글이 보이도록 블로그 글 뷰를 구현
- 엔티티에 생성 시간, 수정 시간을 추가
- 컨트롤러 메서드를 만든 다음 HTML 뷰를 만들고 확인하는 과정으로 개발

## 엔티티에 생성, 수정 시간 추가하기
### Article.java
- 엔티티에 생성 시간과 수정 시간을 추가
- @CreatedDate를 사용하면 엔티티가 생성될 때 생성 시간을 created_at 컬럼에 저장함
- @LastModifiedDate를 사용하면 엔티티가 수정될 때 마지막으로 수정된 시간을 updated_at 컬럼에 저장함
```
public class Article {

    ... 생략 ...
    
    @CreatedDate // 엔티티가 생성될 때 생성 시간 저장
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @LastModifiedDate // 엔티티가 수정될 때 수정 시간 저장
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    ... 생략 ...
}
```

### resources/data.sql
- 스프링 부트 서버를 실행할 때 데이터 추가
```
INSERT INTO article(title, content, created_at, updated_at) VALUES ('제목 1', '내용 1', NOW(), NOW())
INSERT INTO article(title, content, created_at, updated_at) VALUES ('제목 2', '내용 2', NOW(), NOW())
INSERT INTO article(title, content, created_at, updated_at) VALUES ('제목 3', '내용 3', NOW(), NOW())
```

### XXXApplication.java
- @EnableJpaAuditing created_at, updated_at을 자동으로 업데이트하기 위해 추가
```
@EnableJpaAuditing // created_at, updated_at 자동 업데이트
@SpringBootApplication
public class XXXApplication {
    public static void main(String[] args) {
        SpringApplication.run(XXXApplication.class, args);
    }
}
```

## 컨트롤러 메서드

### ArticleViewResponse.java
- 뷰에서 사용할 DTO 생성
```
@NoArgsConstructor
@Getter
public class ArticleViewResponse {
    
    private Long id;
    private String title;
    private String content;
    private LocalDateTime createdAt;
    
    public ArticleViewResponse(Article article) {
        this.id = article.getId();
        this.title = article.getTitle();
        this.content = article.getContent();
        this.createdAt = article.getCreatedAt();
    }
}
```

### BlogViewController.java
- 블로그 글을 반환할 컨트롤러 매서드 
```
@RequiredArgsConstructor
@Controller
public class BlogViewController {
    
    ... 생략 ...
    
    @GetMapping("/articles/{id}")
    public String getArticle(@PathVariable Long id, Model model) {
        Article article = blogService.findById(id);
        model.addAttribute("article", new ArticleViewResponse(article));
        
        return "article";
    }
}
```

## HTML 뷰
### article.html
- 글 상세 화면
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>블로그 글</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css">
</head>
<body>
<div class="p-5 mb-5 text-center</> bg-light">
  <h1 class="mb-3">My Blog</h1>
  <h4 class="mb-3">블로그에 오신 것을 환영합니다.</h4>
</div>
<div class="container mt-5">
  <div class="row">
      <div class="col-lg-8">
          <article>
              <header class="mb-4">
                  <h1 class="fw-bolder mb-1" th:text="${article.title}"></h1>
                  <div class="text-muted fst-italic mb-2" th:text="|Posted on ${#temporals.format(article.createdAt, 'yyyy-MM-dd HH:mm')}|"></div>
              </header>
              <section class="mb-5">
                  <p class="fs-5 mb-4" th:text="${article.content}"></p>
              </section>
              <button type="button" class="btn btn-primary btn-sm">수정</button>
              <button type="button" class="btn btn-primary btn-sm">삭제</button>
          </article>
      </div>
  </div>
</div>
</body>
</html>
```

### articleList.html
- 글 리스트 화면
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>블로그 글 목록</title>
    <link rel="stylesheet" href="http://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css">
</head>
<body>
<div class="p-5 mb-5 text-center</> bg-light">
  <h1 class="mb-3">My Blog</h1>
  <h4 class="mb-3">블로그에 오신 것을 환영합니다.</h4>
</div>

<div class="container">
  <div class="row-6" th:each="item : ${articles}"> <!-- article 개수만큼 반복 -->
    <div class="card-header" th:text="${item.id}"> <!-- item id 출력 -->
    </div>
    <div class="card-body">
      <h5 class="card-title" th:text="${item.title}"></h5>
      <p class="card-text" th:text="${item.content}"></p>
      <!-- th:href 수정: 글 리스트에서 글 상세 화면으로 이동 -->  
      <a th:href="@{/articles/{id}(id=${item.id})}" class="btn btn-primary">보러 가기</a>
    </div>
  </div>
</div>
</body>
</html>
```

---

# ✅ 삭제 기능 추가하기

## 삭제 기능 코드 작성
### resources/static/article.js
- delete-btn 엘리먼트에 클릭 이벤트가 발생하면 fetch() 메서드 실행
- fetch() 메서드는 /api/articles/ DELETE 요청을 보내는 역할
- then() 메서드는 fetch()가 잘 완료되면 연이어 실행되는 메서드
- location.replace()는 삭제가 완료되며 현재 페이지에서 /articles 경로로 이동시켜주는 역할
```
// 삭제 기능
const deleteButton = document.getElementById('delete-btn');

if (deleteButton) {
    deleteButton.addEventListener('click', event => {
        let id = document.getElementById('article-id').value;
        fetch(`/api/articles/${id}`, {
            method: 'DELETE'
        })
        .then(() => {
            alert('삭제가 완료되었습니다.');
            location.replace('/articles');
        });
    });
}
```


### article.html
- [삭제] 버튼 엘리먼트에 delete-btn 아이디 값 추가
- article.js가 화면에서 동작하도록 임포트
```
<!DOCTYPE html>
... 생략 ...

<div class="container mt-5">
  <div class="row">
      <div class="col-lg-8">
          <article>
              <!-- 블로그 글 id 추가 -->
              <input type="hidden" id="article-id" th:value="${article.id}">
              <header class="mb-4">
                  <h1 class="fw-bolder mb-1" th:text="${article.title}"></h1>
                  <div class="text-muted fst-italic mb-2" th:text="|Posted on ${#temporals.format(article.createdAt, 'yyyy-MM-dd HH:mm')}|"></div>
              </header>
              <section class="mb-5">
                  <p class="fs-5 mb-4" th:text="${article.content}"></p>
              </section>
              <button type="button" class="btn btn-primary btn-sm">수정</button>
              <!-- [삭제] 버튼에 id 추가 -->
              <button type="button" id="delete-btn" class="btn btn-primary btn-sm">삭제</button>
          </article>
      </div>
  </div>
</div>
<!-- article.js 임포트 -->
<script src="/js/article.js"></script>
</body>
```

---

# ✅ 수정/생성 기능 추가하기

## 수정/생성 뷰 컨트롤러 작성하기


### BlogViewController.java
- 글 수정과 생성은 같은 화면에서 이루어짐
- 생성할 때는 URL에 별도 쿼리 파라미터가 없음
- 수정할 때는 글의 id를 쿼리 파라미터에 추가해 요청함
```
@RequiredArgsConstructor
@Controller
public class BlogViewController {
    ... 생략 ...
    
    @GetMapping("/new-article")
    // id 키를 가진 쿼리 파라미터의 값을 id 변수에 매핑(id는 없을 수도 있음)
    public String newArticle(@RequestParam(required=false) Long id, Model model) {
        if (id == null) { // id가 없으면 생성
            model.addAttribute("article", new ArticleViewResponse());
        } else {
            Article article = blogService.findById(id);
            model.addAttribute("article", new ArticleViewResponse(article));
        }
        
        return "newArticle";
    }
}        
```

## 수정/생성 뷰 만들기

### newArticle.html
- 수정할 때는 id가 필요하므로 input 엘리먼트의 타입을 hidden으로 설정해 엘리먼트를 숨김
- th:value로 글의 id를 저장
- th:if로 id가 있을 때 [수정] 버튼, 없을 때는 [등록] 버튼이 나타나도록 함
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>블로그 글</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css">
</head>
<body>
<div class="p-5 mb-5 text-center</> bg-light">
    <h1 class="mb-3">My Blog</h1>
    <h4 class="mb-3">블로그에 오신 것을 환영합니다.</h4>
</div>
<div class="container mt-5">
    <div class="row">
        <div class="col-lg-8">
            <article>
                <!-- 아이디 정보 저장 -->
                <input type="hidden" id="article-id" th:value="${article.id}">
                
                <header class="mb-4">
                    <input type="text" class="form-control" placeholder="제목" 
                           id="title" th:value="${article.title}">
                </header>
                <section class="mb-5">
                    <textarea class="form-control h-25" rows="10" placeholder="내용"
                              id="content" th:text="${article.content}"></textarea>
                </section>
                <!-- id가 있을 때는 [수정] 버튼을, 없을 때는 [등록] 버튼이 보이게 함 -->
                <button th:if="${article.id} != null" type="button" 
                        id="modify-btn" class="btn btn-primary btn-sm">수정</button>
                <button th:if="${article.id} == null" type="button" 
                        id="create-btn" class="btn btn-primary btn-sm">등록</button>
            </article>
        </div>
    </div>
</div>

<script src="/js/article.js"></script>
</body>
</html>
```

### article.js
- modify-btn인 엘리먼트를 찾고, 그 엘리먼트에서 클릭 이벤트가 발생하면
- id가 title, content인 엘리먼트의 값을 가져와 fetch() 메서드를 통해 수정 API로 /api/articles/ PUT 요청을 보냄
- 요청을 보낼 때 headers에 요청 형식을 지정하고 body에 HTML에 입력한 데이터를 JSON 형식으로 바꿔 보냄
- 요청이 완료되면 then() 메서드로 마무리 작업
```
// 수정 기능
// id가 modify-btn인 엘리먼트 조회
const modifyButton = documnet.getElementById('modify-btn');

if (modifyButton) {
    // 클릭 이벤트가 감지되면 수정 API 요청
    modifyButton.addEventListener('click', event => {
        let params = new URLSearchParams(location.search);
        let id = params.get('id');
        
        fetch(`/api/articles/${id}`, {
            method: 'PUT',
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringfy({
                title: document.getElementById('title').value,
                content: document.getElementById('content').value
            })
        })
        .then(() => {
            alert('수정이 완료되었습니다.');
            location.replace(`/articles/${id}`);
        });
    });
}
```



### article.html
- [수정] 버튼에 id값과 클릭 이벤트를 추가
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>블로그 글</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css">
</head>
<body>
<div class="p-5 mb-5 text-center</> bg-light">
  <h1 class="mb-3">My Blog</h1>
  <h4 class="mb-3">블로그에 오신 것을 환영합니다.</h4>
</div>
<div class="container mt-5">
  <div class="row">
      <div class="col-lg-8">
          <article>
              <input type="hidden" id="article-id" th:value="${article.id}">
              <header class="mb-4">
                  <h1 class="fw-bolder mb-1" th:text="${article.title}"></h1>
                  <div class="text-muted fst-italic mb-2" th:text="|Posted on ${#temporals.format(article.createdAt, 'yyyy-MM-dd HH:mm')}|"></div>
              </header>
              <section class="mb-5">
                  <p class="fs-5 mb-4" th:text="${article.content}"></p>
              </section>
              <!-- 수정 버튼에 id, th:onclick 추가 -->
              <button type="button" id="modify-btn"
                      th:onclick="|location.href='@{/new-article?id={articleId}(articleId=${article.id})}'|"
                      class="btn btn-primary btn-sm">수정</button>
              <button type="button" id="delete-btn" class="btn btn-primary btn-sm">삭제</button>
          </article>
      </div>
  </div>
</div>

<script src="/js/article.js"></script>
</body>
</html>
```

---

# ✅ 생성 기능 마무리하기

## 생성 기능 작성하기
### article.js
- [등록] 버튼을 누르면 입력 칸에 있는 데이터를 가져와 게시글 생성 API에 글 생성 관련 요청을 보내주는 코드 추가
```
... 생략 ... 

// 등록 기능
// id가 create-btn인 엘리먼트
const createButton = document.getElementById("create-btn");

if (createButton) {
    // 클릭 이벤트가 감지되면 생성 API 요청
    createButton.addEventListener("click", (event) => {
        fetch("/api/articles", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                title: document.getElementById("title").value,
                content: document.getElementById("content").value,
            }),
        }).then(() => {
            alert("등록 완료되었습니다.");
            location.replace("/articles");
        });
    });
}
```

### articleList.html
- id가 create-btndls [생성] 버튼을 추가
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>블로그 글 목록</title>
    <link rel="stylesheet" href="http://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css">
</head>
<body>
<div class="p-5 mb-5 text-center</> bg-light">
  <h1 class="mb-3">My Blog</h1>
  <h4 class="mb-3">블로그에 오신 것을 환영합니다.</h4>
</div>

<div class="container">
  <!-- 글 등록 button 추가 -->
  <button type="button" id="create-btn" 
          th:onclick="|location.href='@{/new-article}'|" 
          class="btn btn-primary btn-sm mb-3">글 등록</button>
          
  <div class="row-6" th:each="item : ${articles}"> <!-- article 개수만큼 반복 -->
    ... 생략 ...
  </div>
</div>
</body>
```