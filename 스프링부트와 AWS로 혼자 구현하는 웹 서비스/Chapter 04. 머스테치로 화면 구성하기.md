### 4.1 서버 템플릿 엔진과 머스테치 소개

- 템플릿 엔진: 지정된 템플릿 양식과 데이터가 합쳐져 HTML 문서를 출력하는 소프트웨어
- 서버 템플릿 엔진: 서버에서 구동, ex) JSP...
- 클라이언트 템플릿 엔진: 브라우저에서 구동 ex) Vue.js, React.js...
- 머스테치: 수많은 언어를 지원하는 가장 심플한 템플릿 엔진

### 4.2 기본 페이지 만들기

```html
{{>layout/header}}

<h1>스프링부트로 시작하는 웹 서비스 Ver.2</h1>
<div class="col-md-12">
    <!--로그인 기능 영역-->
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
        {{#userName}}
            Logged in as: <span id="user">{{userName}}</span>
            <a href="/logout" class="btn btn-info active" role="button">Login</a>
        {{/userName}}
        {{^userName}}
            <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
            <a href="/oauth2/authorization/naver" class="btn btn-secondary active" role="button">Naver Login</a>
        {{/userName}}
        </div>
    </div>
    <br>
    <!--목록 출력 영역-->
    <table class="table table-horizontal table-bordered">
        <thead class="thead-strong">
        <tr>
            <th>게시글번호</th>
            <th>제목</th>
            <th>작성자</th>
            <th>최종수정일</th>
        </tr>
        </thead>
        <tbody id="tbody">
        {{#posts}}
            <tr>
                <td>{{id}}</td>
                <td><a href="/posts/update/{{id}}">{{title}}</a></td>
                <td>{{author}}</td>
                <td>{{modifiedDate}}</td>
            </tr>
        {{/posts}}
        </tbody>
    </table>
</div>

{{>layout/footer}}
```

- {{>layout/header}}
1. {{> }}는 현재 머스테치 파일을 기준으로 다른 파일을 가져옴

### 4.3 게시글 등록 화면 만들기

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body>
```

```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

<!--index.js 추가-->
<script src="/js/app/index.js"></script>
</body>
</html>

```

페이지 로딩 속도를 높이기 위해 css는 header에 js는 footer에 두고 있음

```html
{{>layout/header}}

<h1>게시글 등록</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
            </div>
            <div class="form-group">
                <label for="author"> 작성자 </label>
                <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
            </div>
            <div class="form-group">
                <label for="content"> 내용 </label>
                <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-save">등록</button>
    </div>
</div>

{{>layout/footer}}
```

```jsx
var main = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });

        $('#btn-update').on('click', function () {
            _this.update();
        });

        $('#btn-delete').on('click', function () {
            _this.delete();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    update : function () {
        var data = {
            title: $('#title').val(),
            content: $('#content').val()
        };

        var id = $('#id').val();

        $.ajax({
            type: 'PUT',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    delete : function () {
        var id = $('#id').val();

        $.ajax({
            type: 'DELETE',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8'
        }).done(function() {
            alert('글이 삭제되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }

};

main.init();
```

- window.location.href=’/’
1. 글 등록이 성공하면 메인페이지(/)로 이동

### 4.4 전체 조회 화면 만들기

```html
{{>layout/header}}

<h1>스프링부트로 시작하는 웹 서비스 Ver.2</h1>
<div class="col-md-12">
    <!--로그인 기능 영역-->
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
        {{#userName}}
            Logged in as: <span id="user">{{userName}}</span>
            <a href="/logout" class="btn btn-info active" role="button">Login</a>
        {{/userName}}
        {{^userName}}
            <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
            <a href="/oauth2/authorization/naver" class="btn btn-secondary active" role="button">Naver Login</a>
        {{/userName}}
        </div>
    </div>
    <br>
    <!--목록 출력 영역-->
    <table class="table table-horizontal table-bordered">
        <thead class="thead-strong">
        <tr>
            <th>게시글번호</th>
            <th>제목</th>
            <th>작성자</th>
            <th>최종수정일</th>
        </tr>
        </thead>
        <tbody id="tbody">
        {{#posts}}
            <tr>
                <td>{{id}}</td>
                <td><a href="/posts/update/{{id}}">{{title}}</a></td>
                <td>{{author}}</td>
                <td>{{modifiedDate}}</td>
            </tr>
        {{/posts}}
        </tbody>
    </table>
</div>

{{>layout/footer}}
```

- {{#posts}}
1. posts라는 List를 순회함
2. Java의 for문과 동일

- {{id}} 등의 {{변수명}}
1. List에서 뽑아낸 객체의 필드를 사용

```html
package com.jueun.springboot.web.dto;

import com.jueun.springboot.domain.posts.Posts;
import lombok.Getter;
import java.time.LocalDateTime;

@Getter
public class PostsListResponseDto {
    private Long id;
    private String title;
    private String author;
    private LocalDateTime modifiedDate;

    public PostsListResponseDto(Posts entity){
        this.id=entity.getId();
        this.title=entity.getTitle();
        this.author=entity.getAuthor();
        this.modifiedDate=entity.getModeifiedDate();
    }
}
```

```java
package com.jueun.springboot.web;

import com.jueun.springboot.config.auth.LoginUser;
import com.jueun.springboot.config.auth.dto.SessionUser;
import com.jueun.springboot.service.posts.PostsService;
import com.jueun.springboot.web.dto.PostsResponseDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@RequiredArgsConstructor
@Controller
public class IndexController {
    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user){
        model.addAttribute("posts",postsService.findAllDesc());

        if(user!=null){
            model.addAttribute("userName",user.getName());
        }
        return "index";
    }

    @GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model){
        PostsResponseDto dto=postsService.findById(id);
        model.addAttribute("post",dto);

        return "posts-update";
    }
}
```

- Model
1. 서버 템플릿 엔진에서 사용할 수 있는 객체를 저장 가능
2. 여기서는 postsService.findAllDesc()로 가져온 결과를 posts로 index.mustache에 전달함

### 4.5 게시글 수정, 삭제 화면 만들기

```html
{{>layout/header}}

<h1>게시글 수정</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">글 번호</label>
                <input type="text" class="form-control" id="id" value="{{post.id}}" readonly>
            </div>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" value="{{post.title}}">
            </div>
            <div class="form-group">
                <label for="author"> 작성자 </label>
                <input type="text" class="form-control" id="author" value="{{post.author}}" readonly>
            </div>
            <div class="form-group">
                <label for="content"> 내용 </label>
                <textarea class="form-control" id="content">{{post.content}}</textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-update">수정 완료</button>
        <button type="button" class="btn btn-danger" id="btn-delete">삭제</button>
    </div>
</div>

{{>layout/footer}}
```

- {{post.id}}
1. 머스테치는 객체의 필드 접근 시 점으로 구분
2. 즉, post 클래스의 id에 대한 접근은 post.id로 사용할 수 있음

- readonly
1. input 태그에 읽기 가능만 허용하는 속성
2. id와 author은 수정할 수 없도록 읽기만 허용하도록 추가

```jsx
var main = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });

        $('#btn-update').on('click', function () {
            _this.update();
        });

        $('#btn-delete').on('click', function () {
            _this.delete();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    update : function () {
        var data = {
            title: $('#title').val(),
            content: $('#content').val()
        };

        var id = $('#id').val();

        $.ajax({
            type: 'PUT',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    delete : function () {
        var id = $('#id').val();

        $.ajax({
            type: 'DELETE',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8'
        }).done(function() {
            alert('글이 삭제되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }

};

main.init();
```

- $(’#btn-update’).on(’click’)
1. btn-update란 id를 가진 HTML 엘리먼트에 click 이벤트가 발생할 때 update function을 실행하도록 이벤트를 등록

- update:function()
1. 신규로 추가될 update function

- type: ‘PUT’
1. 여러 HTTP Method 중 PUT 메소드를 선택

- url:’/api/v1/posts/’+id
1. 어느 게시글을 수정할지 URL Path로 구분하기 위해 Path에 id를 추가

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

    @Transactional
    public void delete(Long id){
        Posts posts=postsRepository.findById(id)
                .orElseThrow(() ->new IllegalArgumentException("해당 사용자가 없습니다. id+="+id));
        postsRepository.delete(posts);
    }

    @Transactional(readOnly = true)
    public PostsResponseDto findById(Long id){
        Posts entity=postsRepository.findById(id).orElseThrow(()->new IllegalArgumentException("해당 게시글이 없습니다. id="+id));

        return new PostsResponseDto(entity);
    }

    @Transactional(readOnly = true)
    public List<PostsListResponseDto> findAllDesc(){
        return postsRepository.findAllDesc().stream()
                .map(PostsListResponseDto::new)
                .collect(Collectors.toList());
    }

}

```

- postsRepository.delete(posts)

1. JpaRepository에서 이미 delete 메소드를 지원하고 있으니 이를 활용
2. 엔티티를 파라미터로 삭제할 수도 있고, deleteById 메소드를 이용하명 id로 삭제할 수도 있음
3. 존재하는 Posts인지 확인을 위해 엔티티 조회 후 그대로 삭제
