스프링 부트는 JSP를 사용할 수 있지만 보통 JSP 대신에 Thymelaf나 FreeMarker, Mustache등을 이용

Thymelaf

- JSP와 유사하게 ${}을 별도의 처리 없이 이용 가능
- Model에 담긴 객체를 화면에서 JavaScript로 처리하기 편리
- 연산이나 포맷과 관련된 기능을 추가적인 개발 없이 지원
- 개발 도구를 이용할 때 .html 파일로 생성하는데 문제가 없고 별도의 확장자를 이용하지 않음

### 3.1 Thymeleaf를 사용하는 프로젝트 생성

Thymeleaf를 이용하는 프로젝트는 변경 후에 만들어진 결과를 보관(캐싱)하지 않도록 설정해두는 것이 편리

```java
spring.thymeleaf.cache=false
```

Thymeleaf는 HTML은 그대로 두고 필요한 동작이나 값을 추가하는 방식

### 3.2 Thymeleaf의 기본 사용법

```java
package com.example.ex3.dto;

import lombok.Builder;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Builder(toBuilder = true)
public class SampleDTO {
    private Long sno;
    private String first;
    private String last;
    private LocalDateTime regTime;
}
```

@Data

- Getter/Setter, toString(), equals(), hashCode()를 자동으로 생성

```java
@Controller
@RequestMapping("/sample")
@Log4j2
public class SampleController {
    @GetMapping("/ex1")
    public void ex1(){
        log.info("ex1.......");
    }

    @GetMapping({"/ex2"})
    public void exModel(Model model){
        List<SampleDTO> list= IntStream.rangeClosed(1,20).asLongStream().mapToObj(i->{
            SampleDTO dto=SampleDTO.builder().sno(i)
                    .first("First.."+i)
                    .last("Last.."+i)
                    .regTime(LocalDateTime.now())
                    .build();
            return dto;
        }).collect(Collectors.toList());
        model.addAttribute("list",list);
    }
}
```

@GetMapping

- value의 속성값을 {}로 처리하면 하나 이상의 URL을 지정할 수 있음

**3.2.1 반복문 처리**

th:each를 사용하여 반복 처리

```html
th:each="변수 : ${목록}"
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    <li th:each="dto, state :${list}" th:if="${dto.sno%5==0}">
        [[${dto}]]
    </li>
</ul>
</body>
</html>
```

[[ ]]

- 인라인 표현식으로, 별도의 태그 속성으로 지정하지 않고 사용하고자 할 때 사용

state

- 상태 객체
- 순번이나 인덱스 번호, 홀수, 짝수 등을 지정할 수 있음

th:if~unless

- 제어문
- 제어문은 삼항 연산자로도 사용 가능

**3.2.3 inline 속성**

```html
@GetMapping({"/exInline"})
    public String exInline(RedirectAttributes redirectAttributes){
        log.info("exInline......");
        SampleDTO dto=SampleDTO.builder()
                .sno(100L)
                .first("First....100")
                .last("Last...100")
                .regTime(LocalDateTime.now())
                .build();
        redirectAttributes.addFlashAttribute("result","success");
        redirectAttributes.addFlashAttribute("dto",dto);

        return "redirect:/sample/ex3";
    }

    @GetMapping("/ex3")
    public void ex3(){
        log.info("ex3");
    }
```

exInline()은 내부적으로 RedirectAttributes를 이용하여 /ex3으로 result와 dto라는 이름의 데이터를 심어서 전달

```html
<!DOCTYPE html>
<html land="en" xmlns:th="http://www.thymleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

</head>
<body>
<h1 th:text="${result}"></h1>
<h1 th:text="${dto}"></h1>
<script th:inline="javascript">
    var msg=[[${result}]]
    var dto=[[${dto}]]
</script>
</body>
</html>
```

th:inline의 속성 값이 javascript이기에 브라우저에서 경로는 /sample/exInline으로 호출

→ 리다이렉트 되기 때문에 /sample/ex3을 다시 호출

th:block

- 별도의 태그가 필요하지 않음
- 실제화면에서는 html로 처리되지 않기 때문에 루프 등을 별도로 처리하는 용도로 많이 사용

**3.2.4 링크 처리**

```java
@GetMapping({"/ex2","exLink"})
    public void exModel(Model model){
        List<SampleDTO> list= IntStream.rangeClosed(1,20).asLongStream().mapToObj(i->{
            SampleDTO dto=SampleDTO.builder().sno(i)
                    .first("First.."+i)
                    .last("Last.."+i)
                    .regTime(LocalDateTime.now())
                    .build();
            return dto;
        }).collect(Collectors.toList());
        model.addAttribute("list",list);
    }
```

```java
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    <li th:each="dto : ${list}" >
        <a th:href="@{/sample/exView}">[[${dto}]]</a>
    </li>
</ul>
</body>
</html>
```

@{ } 로 구성된 링크 처리

### 3.3 Thymeleaf의 기본 객체와 LocalDateTime

기본 객체

- 문자나 숫자, 웹에서 사용되는 파라미터, request, response, session등 다양함
- 이를 이용하여 JSP와 유사하거나 좀 더 편리하게 코드 작성 가능

### 3.4 Thymeleaf의 레이아웃

Thtmeleaf의 레이아웃 기능

- JSP의 include와 같이 특정 부분을 외부 혹은 내부에서 가져와서 포함하는 형태
- 특정한 부분을 파라미터로 전달해서 내용에 포함하는 형태

**3.4.1 include 방식의 처리**

th:insert

- 특정한 부분을 다른 내용으로 변경할 수 있음
- 기존 내용의 바깥쪽 태그는 그대로 유지하면서 추가하는 방식

th:replace

- 특정한 부분을 다른 내용으로 변경할 수 있음
- 기존의 내용을 완전히 대체하는 방식

파라미터 방식의 처리

- JSP와 달리 Thymeleaf를 이용하면 특정한 태그를 파라미터처럼 전달해서 다른 th:fragment에서 사용할 수 있음

this: #ulFirst

- this: 현재 페이지를 의미할 때 사용하는데 생략이 가능함
- #ulFirst: CSS의 id 선택자를 의미

**3.4.2 레이아웃 템플릿 만들기**

파라미터로 필요한 영역을 전달해서 처리 가능

= 레이아웃 전체를 하나의 페이지로 구성→필요한 부분을 파라미터로 전달하는 방식으로 공통의 레이아웃 사용 가능
