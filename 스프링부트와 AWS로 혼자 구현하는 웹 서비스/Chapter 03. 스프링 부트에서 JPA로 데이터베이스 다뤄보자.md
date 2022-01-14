### 3.1 JPA소개

- 서로 지향하는 바가 다른 2개 영역(객체지향 프로그래밍 언어, 관계형 데이터베이스)을 중간에서 패러다임 일치시켜주기 위한 기술
- 즉, 개발자는 객체지향적으로 프로그래밍을 하고, JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행

**Spring Data JPA**

- JPA는 인터페이스로서 자바 표줌 명세서이며, 이를 사용하기 위해서는 구현체 필요
- Spring에서 JPA를 사용할 때는 이 구현체를 직접 다루지 않고, 좀 더 쉽게 사용하고자 추상화시킨 Spring Data JPA라는 모듈을 이용하여 JPA를 다룸
- JPA ← Hibernate ← Spring Data JPA
    
    : 구현체 교체의 용이성-Hibernate 외에 다른 구현체로 쉽게 교체하기 위함
    
    : 저장소 교체의 용이성-관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위함
    

### 3.2 프로젝트에 Spring Data Jpa 적용하기

build.gradle에 의존성 등록

```java
compile('org.springframework.boot:spring-boot-starter-data-jpa')
compile('com.h2database:h2')
```

- spring-boot-starter-data-jpa
    
    1. 스프링 부트용 Spring Data Jpa 추상화 라이브러리
    2. 스프링 부트 버전에 맞춰 자동으로 JPA 관련 라이브러리들의 버전을 관리
    
- h2
    
    1. 인메모리 관계형 데이터베이스
    2. 별도의 설치가 필요 없이 프로젝트 의존성망으로 관리 가능
    3. 메모리에서 실행되기 때문에 애플리케이션을 재시작할 때마다 초기화된다는 점을 이용하여 테스트 용도로 많이 사용
    4. 이 프로젝트에서 JPA 테스트, 로컬 환경에서의 구동에서 사용될 예정
    

domain 패키지 생성

이곳에서 도메인: 게시글, 댓글, 회원, 정산, 결제 등 소프트웨어에 대한 요구사항 혹은 문제 영역

```java
package com.jueun.springboot.domain.posts;

import com.jueun.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Getter
@NoArgsConstructor
@Entity
public class Posts extends BaseTimeEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(String title, String content, String author){
        this.title=title;
        this.content=content;
        this.author=author;
    }

    public void update(String title, String content){
        this.title=title;
        this.content=content;
    }
}

```

- 위의 Post 클래스는 실제 DB의 테이블과 매칭될 클래스이며 보통 Entity 클래스라고 함.
    
     JPA를 사용하면 DB 데이터에 작업할 경우 실제 쿼리를 날리기 보다 이 Entity 클래스의 수정을 통해 작업할 수 있음
    
- Entity 클래스에서는 절대 Setter 메소드를 만들지 않음
    
    기본적으로 생성자를 통해 최종값을 채운 후 사용
    
    대신, 해당 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼 수 있는 메소드 추가
    
- @Entity는 JPA의 어노테이션, @Getter, @NoArgsConstructor은 롬복의 어노테이션

**JPA에서 제공하는 어노테이션**

- @Entity
1. 테이블과 링크될 클래스임을 나타냄
2. 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍으로 테이블 이름을 매칭
ex) SalesManager.java -> sales_manager table
- @Id
1. 해당 테이블의 PK 필드를 나타냄
- @GeneratedValue
1. PK의 생성 규칙을 나타냄
2. 스프링 부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment가 됨
- @Column
1. 테이블의 칼럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 됨
2. 사용하는 이유는, 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용
3. 문자열의 경우 VARCHAR(255)가 기본값이지만, 사이즈를 500으로 늘리고 싶거나 타입을 TEXT로 변경하고 싶은 등의 경우에 사용됨

**롬복 라이브러리의 어노테이션**

- @NoArgConstructor
1. 기본 생성자 자동 추가
2. public Posts() {}와 같은 효과
- @Getter
1. 클래스 내 모든 필드의 Getter 메소드를 자동생성
- @Builder
1. 해당 클래스의 빌더 패턴 클래스를 생성
2. 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함

```java
package com.jueun.springboot.domain.posts;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;

public interface PostsRepository extends JpaRepository<Posts,Long> {
    @Query("SELECT p FROM Posts p ORDER BY p.id DESC") //SpringDataJpa에서 제공하지 않는 메소드는 쿼리로 작성 가능
    List<Posts> findAllDesc();
}

//Dao 역할
//Entity 클래스와 기본 Entity Repository는 함께 위치해야 함
```

- Dao 역할
- @Repository를 추가할 필요는 없지만, Entity 클래스와 기본 Entity Repository는 함께 위치해야 함 → domain패키지에서 함꼐 관리

### 3.3 Spring Data JPA 테스트 코드 작성하기

```java
package com.jueun.springboot.web.domain.posts;

import com.jueun.springboot.domain.posts.Posts;
import com.jueun.springboot.domain.posts.PostsRepository;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.time.LocalDateTime;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PostRepositoryTest {
    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanup(){
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기(){
        //given
        String title="테스트 게시글";
        String content="테스트 본문";

        postsRepository.save(Posts.builder()
                                  .title(title)
                                  .content(content)
                                  .author("dlwndms0812@naver.com")
                                  .build());

        //when
        List<Posts> postsList=postsRepository.findAll();

        //then
        Posts posts=postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }

    @Test
    public void BaseTimeEntity_등록(){
        //given
        LocalDateTime now=LocalDateTime.of(2022,1,11,0,0,0);
        postsRepository.save(Posts.builder()
                                  .title("title")
                                  .content("content")
                                  .author("author")
                                  .build());

        //when
        List<Posts> postsList=postsRepository.findAll();

        //then
        Posts posts=postsList.get(0);

        System.out.println(">>>>>>>> createDate="+posts.getCreatedDate()+", modifiedDate="+posts.getModeifiedDate());
        assertThat(posts.getCreatedDate()).isAfter(now);
        assertThat(posts.getModeifiedDate()).isAfter(now);
    }
}

```

- @After
1. junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정
2. 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기위해 사용
3. 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로 남아있어 다음 테스트 실행시 테스트가 실패할 수 있음
- postRepository.save
1. 테이블 posts에 insert/update 쿼리를 실행
2. id 값이 있다면 update, 없다면 insert 쿼리가 실행
- postsRepository.findAll
1. 테이블 posts에 있는 모든 데이터를 조회해오는 메소드

### 3.4 등록/수정/조회 API 만들기

API를 만들기 위해 총 3개의 클래스가 필요함

1. Request 데이터를 받을 Dto
2. API 요청을 받을 Controller
3. 트랜잭션, 도메인 기능 간의 순서를 보장하는 Service

**Spring 웹 계층**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49e070d8-3069-45b1-9d4c-3b24ccf25f41/Untitled.png)

- Web Layer
    - 흔히 사용하는 컨트롤러(@Controller와 JSP/Freemarker 등의 뷰 템플릿 영역
    - 이외에도 필터(@Filter), 인터셉터, 컨트롤러 어드바이스(@ControllerAdvice) 등 외부 요청과 응답에 대한 전반적인 영역
- Service Layer
    - @Service에 사용되는 서비스 영역
    - 일반적으로 Controller와 Dao의 중간 영역에서 사용됨
    - @Transactional이 사용되어야 하는 영역
- Repositroy Layer
    - Database와 같이 데이터 저장소에 접근하는 영역
    - =Dao
- Dtos
    - Dto는 계층간의 데이터 교환을 위한 객체를 이야기 하며 Dtos는 이들의 영역을 이야기
    - ex) 뷰 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등
- Domain Model
    - 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것을 도메인 모델이라 함
    - ex) 택시 앱일시 배차, 탑승, 요금 등이 모두 도메인
    - @Entity가 사용된 영역 역시 도메인 모델
    - 다만, 무조건 데이터베이스의 테이블과 관계가 있어야하는 것은 아님
    - VO처럼 값 객체들도 이 영역에 해당하기 때문
    

**주문 취소 로직을 작성한다면?**

슈도 코드

```java
@Transactional
public Order cancelOrder(int orderId){
    1) 데이터베이스로부터 주문정보(Orders), 결제정보(Billing), 배송정보(Delivery) 조회
    2) 배송 취소를 해야 하는지 확인
    3) if(배송 중이라면) {
        배송 취소로 변경
       }
    4) 각 테이블에 취소 상태 Update
}
```

실제 코드

```java
@Transactional
public Order cancelOrder(int orderId) {
    //1)
    OrdersDto order=ordersDao.selectOrders(orderId);
    BillingDto billing=billingDao.selectBilling(orderId);
    DeliveryDto delivery=deliveryDao.selectDelivery(orderId);

    //2)
    String deliveryStatus=delivery.getStatus();

    //3)
    if("IN_PROGRESS".equals(deliveryStatus)){
        delivery.setStatus("CANCEL");
        deliveryDao.update(delivery);
    }

    //4)
    order.setStatus("CANCEL");
    ordersDao.update(order);

    billing.setStatus("CANCEL");
    deliveryDao.update(billing);
 
    return order;
}
```

⇒ 모든 로직이 서비스 클래스 내부에서 처리됨 ⇒ 서비스 계층이 무의미 하며, 객체란 단순히 데이터 덩어리 역할만 하게 됨

도메인 모델에서 처리하는 경우

```java
@Transactional
public Order cancelOrder(int orderId){
    
    //1)
    Orders order=ordersRepository.findById(orderId);
    Billiing billing=billingRepository.findByOrderId(orderId);
    Delivery delivery=deliveryRepository.findByOrderId(orderId);

    //2-3)
    delivery.cancel();

    //4)
    order.cancel();
    billing.cancel();

    return order;
}
```

⇒ order, billing, delivery가 각자 본인의 취소 이벤트 처리를 하며, 서비스 메소드는 트랜잭션과 도메인 간의 순서만 보장

**등록, 수정, 삭제 기능**

```java
package com.jueun.springboot.web;

import com.jueun.springboot.domain.posts.PostsRepository;
import com.jueun.springboot.service.posts.PostsService;
import com.jueun.springboot.web.dto.PostsResponseDto;
import com.jueun.springboot.web.dto.PostsSaveRequestDto;
import com.jueun.springboot.web.dto.PostsUpdateRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
public class PostApiController {
    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto){
        return postsService.save(requestDto);
    }

    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto){
        return postsService.update(id,requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById(@PathVariable Long id){
        return postsService.findById(id);
    }
}
```

```java
package com.jueun.springboot.service.posts;

import com.jueun.springboot.domain.posts.Posts;
import com.jueun.springboot.domain.posts.PostsRepository;
import com.jueun.springboot.web.dto.PostsListResponseDto;
import com.jueun.springboot.web.dto.PostsResponseDto;
import com.jueun.springboot.web.dto.PostsSaveRequestDto;
import com.jueun.springboot.web.dto.PostsUpdateRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@RequiredArgsConstructor //final이 선언된 모든 필드를 인자값으로 하는 생성자를 대신 생성
//생성자를 직접 안쓰고 롬복 어노테이션을 쓴 이유: 해당 클래스의 의존성 관계가 변경될때마다 생성자 코드를 계속해서 수정하는 번거로움 해결
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto){
        return postsRepository.save(requestDto.toEntity()).getId();
    }

    @Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto){
        Posts posts=postsRepository.findById(id).orElseThrow(()->new IllegalArgumentException("해당 게시글이 없습니다. id="+id));
        posts.update(requestDto.getTitle(),requestDto.getContent());

        return id;
    }

    public PostsResponseDto findById(Long id){
        Posts entity=postsRepository.findById(id).orElseThrow(()->new IllegalArgumentException("해당 게시글이 없습니다. id="+id));

        return new PostsResponseDto(entity);
    }
}

```

스프링에서 Bean을 주입 받는 방식

1. @Autowired
2. setter
3. 생성자  →가장 권장, @RequiredArgsConstructor 사용

update 기능에서 데이터베이스에 쿼리를 날리는 부분이 없는 이유→ JPA의 영속성 컨텍스트 때문

**영속성 컨텍스트**: 엔티티를 영구 저장하는 환경

JPA의 엔티티 매니저가 활성화된 상태로 트랜잭션 안에서 데이터베이스에서 데이터를 가져오면 이 데이터는 영속성 컨텍스트가 유지된 상태 → 이 상태에서 해당 데이터의 값을 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영 = **더티 체킹**

```java
package com.jueun.springboot.web.dto;

import com.jueun.springboot.domain.posts.Posts;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    private String title;
    private String content;
    private String author;
    @Builder
    public PostsSaveRequestDto(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public Posts toEntity(){
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}
```

**절대로 Entity 클래스를 Request/Response 클래스로 사용해서는 안됨**

```java
package com.jueun.springboot.web.dto;

import com.jueun.springboot.domain.posts.Posts;
import lombok.Getter;

@Getter
public class PostsResponseDto {

    private Long id;
    private String title;
    private String content;
    private String author;

    public PostsResponseDto(Posts entity){
        this.id= entity.getId();
        this.title= entity.getTitle();
        this.content= entity.getContent();
        this.author=entity.getAuthor();
    }
}
```

```java
package com.jueun.springboot.web.dto;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class PostsUpdateRequestDto {
    private String title;
    private String content;

    @Builder
    public PostsUpdateRequestDto(String title, String content){
        this.title=title;
        this.content=content;
    }
}
```

테스트코드

```java
package com.jueun.springboot.web;

import com.jueun.springboot.domain.posts.Posts;
import com.jueun.springboot.domain.posts.PostsRepository;
import com.jueun.springboot.web.dto.PostsSaveRequestDto;
import com.jueun.springboot.web.dto.PostsUpdateRequestDto;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;

import javax.xml.ws.Response;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {
    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @After
    public void tearDown() throws Exception{
        postsRepository.deleteAll();
    }

    @Test
    public void Posts_등록된다() throws Exception{
        //given
        String title="title";
        String content="content";
        PostsSaveRequestDto requestDto=PostsSaveRequestDto.builder()
                .title(title)
                .content(content)
                .author("author")
                .build();
        String url="http://localhost:"+port+"/api/v1/posts";

        //when
        ResponseEntity<Long> responseEntity=restTemplate.postForEntity(url,requestDto,Long.class);

        //then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Posts> all=postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }

    @Test
    public void Posts_수정된다() throws Exception{
        //given
        Posts savedPosts=postsRepository.save(Posts.builder()
                                                   .title("title")
                                                   .content("content")
                                                   .author("author")
                                                   .build());
        Long updateId= savedPosts.getId();
        String expectedTitle="title2";
        String expectedContent="content2";

        PostsUpdateRequestDto requestDto=PostsUpdateRequestDto.builder()
                .title(expectedTitle)
                .content(expectedContent)
                .build();

        String url="http://localhost:"+port+"/api/v1/posts/"+updateId;
        HttpEntity<PostsUpdateRequestDto> requestEntity=new HttpEntity<>(requestDto);

        //when
        ResponseEntity<Long> responseEntity=restTemplate.exchange(url, HttpMethod.PUT, requestEntity, Long.class);

        //then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);
        List<Posts> all=postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }
}
```

### 3.5 JPA Auditing으로 생성시간/수정시간 자동화하기

차후 유지보수에 있어서 중요하기에 엔티티에는 해당 데이터의 생성시간과 수정 시간을 포함

JPA Auditing을 사용

```java
package com.jueun.springboot.domain;

import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseTimeEntity {
    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime modeifiedDate;
}

```

BaseTimeEntity클래스는 모든 Entity의 상위 클래스가 되어 Entity들의 createdDate, modifiedDate를 자동으로 관리하는 역할

- @MappedSuperclass
1. JPA Entity 클래스들이 BaseTimeEntity을 상속할 경우 필드들(createdDate, modifiedDate)도 칼럼으로 인식하도록 함
- @EntityListeners(AuditingEntityListener.class)
1. BaseTimeEntity 클래스에 Auditing 기능을 포함시킴
- @CreatedDate
1. Entity가 생성되어 저장될 때 시간이 자동 저장
- @LastModifiedDate
1. 조회한 Entity의 값을 변경할 때 시간이 자동 저장
