---
layout: post
title: 스프링부트의 구조 및 엔티티 설계 - 게시판만들기(1)
comments: true
tags: Spring
---

`@Controller` 애너테이션은 컨트롤러의 기능을 수행한다는 의미로, 스프링부트 프레임워크가 이 코드를 컨트롤러로 인식하게 해준다.

```java
@Controller
public class HelloController {
	@RequestMapping("/hello")
	@ResponseBody
	public String hello() {
		return "Hello world";
	}
}
```

`@RequestMapping("/hello")` 애너테이션은 `http://localhost:8080/hello`의 URL 요청이 발생하면 hello 메서드가 실행된다. `/hello` URL과 hello 메서드를 매핑하는 역할을 한다.

<h6>설치파일</h6>

Node.js에서 서버 수정을 하고 서버를 재시작 해야만 수정사항이 적용되는 불편함을 해소하기 위해서 demon을 사용하지만, spring에서도 spring boot devtools을 설치하면 불편함을 해소할 수 있다. 이 프로그램을 사용하기 위해서는 `Gradle`로 설치해야한다.

build.gradle 파일에서 아래의 코드를 추가한다.

```java
developmentOnly 'org.springframework.boot:spring-boot-devtools'
```

<h4>스프링부트 프로젝트의 구조</h4>

`src/main/java` 디렉터리의 com.example.demo 패키지는 자바 파일을 작성하는 공간이다. 스프링부트의 컨트롤러, 폼과 DTO, DB처리를 위한 엔티티, 서비스 파일등이 있다. 이 패키지안에는 `프로젝트명 + Application.java`파일이 있다. 이는 시작을 담당하는 파일로써 스프링부트를 사용하기 위해서는 `@SpringBootApplication` 애너테이션이 적용되어 있어야 한다.

`src/main/resources` 디렉토리는 java 파일을 제외한 HTML, CSS, JS, 환경파일 등을 작성하는 공간이다. 이 안의 `templates` 디렉토리에는 HTML 파일 형태로 자바 객체와 연동되는 템플릿 파일을 저장한다. `static` 디렉토리에는 .css, .js, .jpg 등을 저장하는 공간이다. `application.properties 파일은 프로젝트의 환경, 데이터베이스 등의 설정을 파일에 저장한다.

`src/test/java` 디렉토리는 작성한 파일을 테스트하기 위한 테스트 코드를 작성하는 공간이다.

<h4>컨트롤러</h4>

```java
package com.example.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class MainController {
	@RequestMapping("/sbb")
	public void index() {
		System.out.println("index");
	}
}
```

`@Controller` 애너테이션을 적용하면 MainController 클래스는 스프링부트의 컨트롤러가 된다. `RequestMapping` 애너테이션은 요청된 URL과의 매핑을 담당한다. 즉 /sbb URL과 매핑되는 index 메서드를 MainController 클래스에서 찾아 실행한다.

<h4>엔티티</h4>

질문 엔티티의 구성
1. id - 질문의 고유 번호
2. subject - 질문의 제목
3. content - 질문의 내용
4. create_date - 질문을 작성한 일시

```java
package com.example.demo;

import java.time.LocalDateTime;

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
public class Question {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Integer id;

    @Column(length=200)
    private String subject;

    @Column(columnDefinition = "TEXT")
    private String content;

    private LocalDateTime createDate;
}

```

엔티티로 만들기 위해 `@Entity` 애너테이션을 적용하여 JPA가 엔티티로 인식하게 한다. lombok의 Getter, Setter를 자동으로 생성하기 위해 @Getter, @Setter 애너테이션을 적용한다.

엔티티의 속성으로 id, subject, content, createDate를 지정하였다. 각각의 내용에 대한 설명이다.

`@Id` 애너테이션은 id 속성을 기본 키로 지정해준다. 기본키는 동일한 값으로 저장할 수 없게하는 설정이다. 즉 고유한 번호를 가지게된다. 각각의 데이터를 구분하기 위해서 기본키를 설정해준다. 즉 데이터베이스에서 프라이머리키를 설정해준다 보면된다.

`@GeneratedValue` 애너테이션은 데이터를 저장할 때 1씩 자동으로 증가해서 저장시켜준다. strategy는 고유번호를 생성하는 옵션으로 generationtype.identity는 해당 컬럼만의 독립적인 시퀀스를 생성하여 번호를 증가시킬 때 사용한다. 보통 쌍으로 같이 많이 사용된다.

`Column` 애너테이션은 테이블의 컬럼명과 일치한다. length는 컬럼의 길이를 설정하고, columnDefinition은 컬럼의 속성을 정의할 때 사용한다. 글자수를 제한하지 않을 때 columnDefinition="TEXT"의 속성을 사용한다.

```
엔티티의 속성은 @Column 애너테이션을 사용하지 않더라도 테이블 컬럼으로 인식한다.
테이블 컬럼으로 인식하고 싶지 않은 경우에는 @Tranisent 애너테이션을 사용한다.
```

답변 엔티티 구성
1. id - 답변의 고유 번호
2. question - 질문
3. content - 답변의 내용
4. create_date - 답변을 작성한 일시

```java

package com.example.demo;

import javax.persistence.ManyToOne;
import java.time.LocalDateTime;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

import lombok.Getter;
import lombok.Setter;
import org.apache.tomcat.jni.Local;

@Getter
@Setter
@Entity
public class Answer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(columnDefinition = "TEXT")
    private String content;

    private LocalDateTime createDate;

    @ManyToOne
    private Question question;
}


```


@ManyToOne 애너테이션은 질문한개에 답변이 여러개 달릴 수 있는 n:1 관계를 설정해주는 애너테이션이다. 이는 부모 자식 관계를 갖는 구조에서 많이 사용한다. 실제 데이터베이스에서는 외래키로 작용한다.