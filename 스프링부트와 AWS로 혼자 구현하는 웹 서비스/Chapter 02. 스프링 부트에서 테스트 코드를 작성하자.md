### 2.1 테스트 코드 소개

- TDD: 테스트가 주도하는 개발. 테스트 코드를 먼저 작성하는 것 부터 시작

![다운로드.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6ab00428-a503-4bda-8580-4a7b5557d976/다운로드.png)

**레드 그린 사이클**

Red: 항상 실패하는 테스트를 먼저 작성하고

Green: 테스트가 통과하는 프로덕션 코드를 작성하고

Refactor: 테스트가 통과하면 프로덕션 코드를 리팩토링

- 단위 테스트: TDD의 첫 번째 단계인 기능 단위의 테스트 코드 작성
- 테스트 코드 작성으로 얻는 이점
    1. 단위 테스트는 개발 단계 초기에 문제를 발견하게 도와줌
    2. 단위 테스트는 개발자가 나중에 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 기존 기능이 올바르게 작동하는지 확인 가능
    3. 단위 테스트는 기능에 대한 불확실성 감소 가능
    4. 단위 테스트는 시스템에 대한 실제 문서 제공. 즉, 단위 테스트 자체가 문서로 사용할 수 있음

### 2.2 Hello Controller 테스트 코드 작성하기

```java
package com.jueun.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication 
public class Application { //메인 클래스
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```

- @SpringBootApplication
    
    이것으로 인해 스프링 부트의 자동 설정, 스프링 Bean 읽기/생성 자동 실행
    
    @SpringBootApplication이 있는 위치부터 설정을 읽기에 이 클래스(Application.java)는 항상 프로젝트의 최상단에 위치
    

- SpringApplication.run
    
    SpringApplication.run으로 인해 내장 WAS(웹 애플리케이션 서버)를 실행
     -> 항상 서버에 톰캣을 설치할 필요가 없게 되고, 스프링 부트로 만들어진 Jar 파일(실행 가능한 Java 패키징 파일)로 실행
    내장 WAS를 권장하는 이유: 언제 어디서나 같은 환경에서 스프링 부트를 배포할 수 있기 때문
    

```java
package com.jueun.springboot.web;

import com.jueun.springboot.web.dto.HelloResponseDto;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/hello") // /hello로 요청이 오면 문자열 hello를 반환하는 기능
    public String hello(){
        return "hello";
    }

    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name,
                                     @RequestParam("amount") int amount){
        return new HelloResponseDto(name, amount);
    }
}

```

- @RestController
1. 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 줌
2. 이전의 @ResponseBody를 각 메소드마다 선언했던 것을 한번에 사용할 수 있게 함
- @GetMapping
1. HTTP Method인 Get의 요청을 받을 수 있는 API를 만들어 줌
- @RequestParam
1. 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션
2. 여기서는 외부에서 name(@RequestParam("name))이란 이름으로 넘긴 파라미터를 메소드 파라미터 name(String name)에 저장하게 됨

```java
package com.jueun.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception{
        String hello="hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }

    @Test
    public void helloDto가_리턴된다() throws Exception{
        String name="hello";
        int amount=1000;

        mvc.perform(
                get("/hello/dto")
                .param("name",name)
                .param("amount",String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name",is(name)))
                .andExpect(jsonPath("$.amount", is(amount)));

    }
}
```

- @RunWith(SpringRunner.class)
1. 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행
2. 여기서는 SpringRunner라는 실행자 사용
3. 즉, 스프링 부트 테스트와 JUnit 사이에 연결자 역할
- @WebMvcTest
1. 여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션
2. 선언할 경우 @Controller, @ControllerAdvice 등을 사용할 수 있음
3. 단, @Service, @Component, @Repository 등은 사용할 수 없음
- @Autowired
1.  스프링이 관리하는 빈(Bean)을 주입 받음

- private MockMvc mvc
1. 웹 API를 테스트할 때 사용
2. 스프링 MVC 테스트의 시작점
3. 이 클래스를 통해 HTTP GET, POST 등에 대한 API 테스트를 할 수 있음
- mvc.perform(get("/hello"))
1. MockMvc를 통해 /hello 주소로 HTTP GET 요청을 함
2. 체이닝이 지원되어 아래와 같이 여러 검증 기능을 이어서 선언할 수 있음
- .andExcept(status().isOk())
1. mvc.perform의 결과를 검증
2. HTTP Header의 Status를 검증
3. 200, 404, 500 등의 상태를 검증
4. 여기선 Ok, 즉 200인지 아닌지 검증
- .andExpect(content().string(hello))
1. mvc.perform의 결과를 검증
2. 응답 본문의 내용을 검증
3. Controller에서 "hello"를 리턴하기 때문에 이 값이 맞는지 검증
- param
1. API를 테스트할 때 사용될 요청 파라미터를 설정
2. 단, 값은 String만 허용
3. 그래서 숫자/날짜 등의 데이터도 등록할 때는 문자열로 변경
- jsonPath
1. JSON의 응답값을 필드별로 검증할 수 있는 메소드
2. $를 기준으로 필드명을 명시

### 2.4 Hello Controller 코드를 롬복으로 전환하기

테스트 코드가 코드를 지켜주기 때문에 롬복으로 전환 가능

```java
package com.jueun.springboot.web.dto;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public class HelloResponseDto {
    private final String name;
    private final int amount;
}

```

- @Getter
1. 선언된 모든 필드의 get 메소드를 생성
- @RequiredArgsConstructor
1. 선언된 모든 final 필드가 포함된 생성자를 생성
2. final이 없는 필드는 생성자에 포함X

```java
package com.jueun.springboot.web.dto;

import org.junit.Test;
import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDtoTest {
    @Test
    public void 롬복_기능_테스트(){
        //given
        String name="test";
        int amount=1000;

        //when
        HelloResponseDto dto=new HelloResponseDto(name, amount);

        //then
        assertThat(dto.getName()).isEqualTo(name);
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}

```

- assertThat
1. assertj라는 테스트 검증 라이브러리의 검증 메소드
2. 검증하고 싶은 대상을 메소드 인자로 받음
3. 메소드 체이닝이 지원되어 isEqualTo와 같이 메소드를 이어서 사용할 수 있음
- isEqualTo
1. assertj의 동등 비교 메소드
2. assertThat에 있는 값과 isEqualTo의 값을 비교해서 같을 때만 성공
- Junit와 비교하여 assertj의 장점
1. 추가적으로 라이브러리가 필요하지 않음
2. 자동완성이 좀 더 확실히 지원됨
