---
layout: post
title: 스프링 시큐리티 + 회원가입 구현 - 토이프로젝트(1)
comments: true
tags: Spring
---

`Spring boot + Gradle + MySQL`

[Github](https://github.com/riverrevir/devstudy-today) 


<h4>스프링 시큐리티</h4>
스프링 기반 애플리케이션의 인증과 권한을 담당하는 스프링의 하위 프레임워크이다.

```
인증(authenticate) - 로그인을 의미한다.

권한(authorize) - 인증된 사용자가 어떤 것을 할 수 있는지 의미한다.
```

스프링 시큐리티를 설치하기 위해 build에 추가하여 설치하자.

```java
implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity5'
```

스프링 시큐리티를 환경설정하는 파일을 하나 작성한다.

```java
package tody.devstudy;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        http.authorizeRequests().antMatchers("/**").permitAll();
    }
}

```
@Configuration 애너테이션은 스프링의 환경설정 파일임을 의미한다. @EnableWebSecurity 모든 요청 url이 스프링 시큐리티의 제어를 받도록 만드는 애너테이션이다. 이는 내부적으로 SpringSecurityFilterChain 이 동작하여 URL 필터가 적용된다.

`http.authorizeRequests().antMatchers("/**").permitAll();` 은 모든 인증되지 않은 요청을 허락한다는 의미이다. 로그인을 하지 않더라도 모든 페이지에 접근할 수 있다.

<h4>회원가입 구현</h4>

**User.java**


```java
package today.devstudy.user;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@Entity

public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true)
    private String username;

    private String password;

    private String sex;

    @Column(unique = true)
    private String email;

}
```

@Entity JPA 엔티티임을 나타낸다. 

@Id 기본 키에 해당한다.

**Repository Interface**

```java
package today.devstudy.user;

import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User,Long>{

}

```

데이터베이스에 접근하기 위해서 레포지토리를 작성해줍니다. 추후에 JPA에서 제공하는 손쉬운 문법들을 적용할 예정입니다.


**UserService**

```java
package today.devstudy.user;

import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.security.crypto.password.PasswordEncoder;
import lombok.RequiredArgsConstructor;
import org.springframework.ui.Model;

@RequiredArgsConstructor
@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    public User create(String username, String email, String password,String sex){

            User user = new User();
            user.setUsername(username);
            user.setEmail(email);
            user.setSex(sex);
            user.setPassword(passwordEncoder.encode(password));
            this.userRepository.save(user);
            return user;
    }
}
```

컨트롤러에서 PostMapping을 통해 들어온 데이터들을 데이터베이스에 넣기 위해 create 메서드를 작성하였다. 이는 회원가입이 되었을 때 데이터베이스에 회원목록들을 순서대로 넣어주게 될 것이다.

**UserController**

```java
package today.devstudy.user;

import javax.validation.Valid;

import org.springframework.dao.DataIntegrityViolationException;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

import lombok.RequiredArgsConstructor;

import java.util.ArrayList;
import java.util.List;

@RequiredArgsConstructor
@Controller
@RequestMapping("/api/user")
public class UserController {
    private final UserService userService;

    List<String> gender;

    @ModelAttribute
    public void genderLoad(){
        gender = new ArrayList<String>();
        gender.add("남");
        gender.add("여");
    }

    /**
     * register(회원가입)
     * @param model
     * @param userCreateForm
     * @return
     */
    @GetMapping("/register")
    public String register(Model model, UserCreateForm userCreateForm){
        model.addAttribute("sex",gender);
        return "register_form";
    }

    @PostMapping("/register")
    public String register(@Valid UserCreateForm userCreateForm,BindingResult bindingResult){
        if(bindingResult.hasErrors()){
            return "register_form";
        }
        if(!userCreateForm.getPassword1().equals(userCreateForm.getPassword2())){
            bindingResult.rejectValue("password2","passwordInCorrect","패스워드가 일치하지 않습니다.");
            return "register_form";
        }
        try {
            userService.create(userCreateForm.getUsername(), userCreateForm.getEmail(), userCreateForm.getPassword1(), userCreateForm.getSex());
        }catch(DataIntegrityViolationException e){
            e.printStackTrace();
            bindingResult.reject("registerFailed","이미 등록된 사용자입니다.");
            return "register_form";
        }catch(Exception e){
            e.printStackTrace();
            bindingResult.reject("registerFailed",e.getMessage());
            return "register_form";
        }
        return "redirect:/";
    }
}

```

클라이언트에서 요청이 들어오면 어떤 동작을 해야할지 결정해준다고 보면된다. 대부분의 흐름은 기존 회원가입과 비슷하므로 큰 설명은 하지 않겠다. 회원가입 부분의 소스코드의 커밋내용들을 보려면 feature/register 커밋내용이 날라가서 깃허브의 master init 부분을 보시면 됩니다. 

앞으로 프로젝트를 진행하면서 이슈들을 깃허브에서 관리할 것이며 하나의 기능을 만들 때 마다 글로 정리하면서 다시한번 정리하려고 합니다. 

