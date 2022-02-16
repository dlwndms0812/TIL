- 스프링 시큐리티: 막강한 인증과 인가(혹은 권한 부여) 기능을 가진 프레임워크

### 5.3 구글 로그인 연동하기

```java
package com.jueun.springboot.domain.user;

import com.jueun.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role){
        this.name=name;
        this.email=email;
        this.picture=picture;
        this.role=role;
    }

    public User update(String name, String picture){
        this.name=name;
        this.picture=picture;
        return this;
    }

    public String getRoleKey(){
        return this.role.getKey();
    }

}

```

- @Enumerated(EnumType.STRING)
1. JPA로 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지를 결정함
2. 기본적으로는 int로 된 숫자가 저장됨
3. 숫자로 저장되면 데이터베이스로 확인할 때 그 값이 무슨 코드를 의미하는지 알 수가 없음
4. 그래서 문자열(EnumType.STRING)로 저장될 수 있도록 선언

```java
package com.jueun.springboot.domain.user;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum Role {
    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}
```

 스프링 시큐리티에서 권한 코드에 항상 ROLE_이 앞에 있어야함

```java
package com.jueun.springboot.domain.user;

import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User,Long> {
    Optional<User> findByEmail(String email);
}
```

- findByEmail
1. 소셜 로그인으로 반환되는 값 중 email을 통해 이미 생성된 사용자인지 처음 가입하는 사용자인지 판단하기 위한 메소드

```java
package com.jueun.springboot.config.auth;

import com.jueun.springboot.domain.user.Role;
import lombok.RequiredArgsConstructor;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception{
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                .authorizeRequests()
                .antMatchers("/","/css/**","/images/**", "/js/**", "/h2-console/**").permitAll()
                .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                .anyRequest().authenticated()
                .and()
                .logout()
                .logoutSuccessUrl("/")
                .and()
                .oauth2Login()
                .userInfoEndpoint()
                .userService(customOAuth2UserService);
    }
}
```

- @EnableWebSecurity
1. Spring Security 설정들을 활성화시켜줌
- csrf().disable().headers().frameOptions().disable()
1. h2-console 화면을 사용하기 위해 해당 옵션들을 disable함
- authorizeRequests
1. URL별 권한 관리를 설정하는 옵션의 시작점
2. authorizeRequests가 선언되어야만 antMatchers 옵션을 사용 가능
- antMatchers
1. 권한 관리 대상을 지정하는 옵션
2. URL, HTTP 메소드별로 관리가 가능
3. "/" 등 지정된 URL들은 permitAll() 옵션을 통해 전체 열람 권한을 줌
4. "/api/v1/**" 주소를 가진 API는 USER 권한을 가진 사람만 가능하도록 함
- anyRequest
1. 설정된 값들 이외 나머지 URL들을 나타냄
2. 여기서는 authenticated()을 추가하여 나머지 URL들은 모두 인증된 사용자들에게만 허용하게 함
3. 인증된 사용자 즉, 로그인한 사용자들을 이야기 함
- logout().logoutSuccessUrl("/")
1. 로그아웃 기능에 대한 여러 설정의 진입점
2. 로그아웃 성공 시 / 주소로 이동
- oauth2Login
1. OAuth2 로그인 기능에 대한 여러 설정의 진입점
- userInfoEndpoint
1. OAuth2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당함
- userService
1. 소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록
2. 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있음

/코드

```
package com.jueun.springboot.config.auth;

import com.jueun.springboot.config.auth.dto.OAuthAttributes;
import com.jueun.springboot.config.auth.dto.SessionUser;
import com.jueun.springboot.domain.user.User;
import com.jueun.springboot.domain.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserService;
import org.springframework.security.oauth2.core.OAuth2AuthenticationException;
import org.springframework.security.oauth2.core.user.DefaultOAuth2User;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Service;

import javax.servlet.http.HttpSession;
import java.util.Collections;

@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User>{
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes=OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);

        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(), attributes.getNameAttributeKey());
        }

        private User saveOrUpdate(OAuthAttributes attributes){
            User user=userRepository.findByEmail(attributes.getEmail())
                    .map(entity->entity.update(attributes.getName(),attributes.getPicture()))
                    .orElse(attributes.toEntity());
            return userRepository.save(user);
        }

    }

```

- registrationId
1. 현재 로그인 진행 중인 서비스를 구분하는 코드
2. 지금은 구글만 사용하는 불필요한 값이지만, 이후 네이버 로그인 연동 시에 네이버 로그인인지, 구글 로그인인지 구분하기 위해 사용
- userNameAttributeName
1. OAuth2 로그인 진행 시 키가 되는 필드값을 이야기. Primary Key와 같은 의미
2. 구글의 경우 기본적으로 코드를 지원하지만, 네이버 카카오 등은 기본 지원하지 않음. 구글의 기본 코드는 "sub".
3. 이후 네이버 로그인과 구글 로그인을 동시 지원할 때 사용됨
- OAuthAttributes
1. OAuth2UserService를 통해 가져온 OAuth2User의 arribute를 담을 클래스
2. 이후 네이버 등 다른 소셜 로그인도 이 클래스를 사용
- SessionUser
세션에 사용자 정보를 저장하기 위한 Dto 클래스

```java
package com.jueun.springboot.config.auth.dto;

import com.jueun.springboot.domain.user.Role;
import com.jueun.springboot.domain.user.User;
import lombok.Builder;
import lombok.Getter;

import java.util.Map;

@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture){
        this.attributes=attributes;
        this.nameAttributeKey=nameAttributeKey;
        this.name=name;
        this.email=email;
        this.picture=picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes){
        if("naver".equals(registrationId)){
            return ofNaver("id",attributes);
        }
        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes){
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes){
        Map<String, Object> response=(Map<String,Object>) attributes.get("response");

        return OAuthAttributes.builder()
                .name((String)response.get("name"))
                .email((String)response.get("email"))
                .picture((String)response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity(){
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}
```

- of()
1. OAuth2User에서 반환하는 사용자 정보는 Map이기 떄문에 값 하나하나를 반환함

- toEntitiy()
1. User 엔티티를 생성함
2. OAuthAttributes에서 엔티티를 생성하는 시점은 처음 가입할 때
3. 가입할 때의 기본 권한을 GUEST로 주기 위해서 role 빌더 값에는 Role.GUEST를 사용
4. OAuthAttributes 클래스 생성이 끝났으면 같은 패키지에 SessionUser 클래스를 생성

```java
package com.jueun.springboot.config.auth.dto;

import com.jueun.springboot.domain.user.User;
import lombok.Getter;

import java.io.Serializable;

@Getter
public class SessionUser implements Serializable {
    private String name;
    private String email;
    private String picture;

    public SessionUser(User user){
        this.name=user.getName();
        this.email=user.getEmail();
        this.picture=user.getPicture();
    }
}
```

```html
h1>스프링부트로 시작하는 웹 서비스 Ver.2</h1>
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
```

- {{#userName}}
1. 머스테치는 다른 언어와 같은 if문을 제고앟지 않음
2. true/false만 판단
3. 따라서 항상 최종값만 넘겨주어야 함
4. userName이 있다면 userName을 노출

- a href=”/logout”
1. 스프링 시큐리티에서 기본적으로 제공하는 로그아웃 URL
2. 즉, 개발자가 별도로 저 URL에 해당하는 컨트롤러를 만들 필요 없음

- {{^userName}}
1. 머스테치에서 해당 값이 존재하지 않는 경우에는 ^ 사용
2. userName이 없다면 로그인 버튼을 노출시키도록 구성

- a href=”/oauth2/authorization/google”
1. 스프링 시큐리티에서 기본적으로 제공하는 로그인 URL
2. 로그아웃 URL과 마찬가지로 개발자가 별도의 컨트롤러를 생성할 필요 없음

```java
@GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user){
        model.addAttribute("posts",postsService.findAllDesc());

        if(user!=null){
            model.addAttribute("userName",user.getName());
        }
        return "index";
    }
```

- (SessionUser) httpSession.getAttriubute(”user”)
1. 앞서 작성된 CustromOAuth2UserService에서 로그인 성공 시 세션에 SessionUser을 저장하도록 구성
2. 즉, 로그인 성공 시 httpSession.getAtribute(”user”)에서 값을 가져올 수 있음

- if(user≠null)
1. 세션에 저장된 값이 있을때만 model에 userName으로 등록
2. 세션에 저장된 값이 없으면 model에 아무런 값이 없는 상태이니 로그인 버튼이 보이게 됨

### 5.4 어노테이션 기반으로 개선하기

같은 코드가 계속해서 반복되는 부분을 메소드 인자로 세션값을 바로 받을 수 있도록 하여 해결하기

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser{
}
```

- Target(ElementType.PARAMETER)
1. 이 어노테이션이 생성될 수 있는 위치 지정
2. PARAMETER로 지정했으니 메소드의 파라미터로 선언된 객체에서만 사용할 수 있음

- @interface
1. 이 파일을 어노테이션 클래스로 지정함
2. LoginUser라는 이름을 가진 어노테이션 생성

```java
package com.jueun.springboot.config.auth;

import com.jueun.springboot.config.auth.dto.SessionUser;
import lombok.RequiredArgsConstructor;
import org.springframework.core.MethodParameter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

import javax.servlet.http.HttpSession;

@RequiredArgsConstructor
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver{
    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter){
        boolean isLoginUserAnnotation=parameter.getParameterAnnotation(LoginUser.class)!=null;
        boolean isUserClass= SessionUser.class.equals(parameter.getParameterType());
        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception{
        return httpSession.getAttribute("user");
    }
}
```

- HandlerMethodArgumentResolver
1. 조건에 맞는 경우 메소드가 있다면 HanderMethodArgumentResolver의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있음

- supportsParameter()
1. 컨트롤러 메서드의 특정 파라미터를 지원하는지 판단
2. 여기서는 파라미터에 @LoginUser 어노테이션이 붙어있고, 파라미터 클래스 타입이 SessionUser.class인 경우 true를 반환
- resloveArgument()
1. 파라미터에 전달할 객체를 생성
2. 여기서는 세션에서 객체를 가져옴
