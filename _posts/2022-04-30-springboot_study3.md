---
layout: post
title: 파일 관리 및 템플릿(타임리프)에 대해서 - 게시판만들기(3)
comments: true
tags: Spring
---

하나의 패키지에서 자바파일을 관리하는 것은 바람직하지 않으므로, 도메인별로 패키지를 나누어 기능별로 작성하는게 중요하다.

주요 기능들은 `질문, 답변, 사용자`로써 도메인 별로 연결시켜 보자.

우선 `/question/list` URL에 대한 매핑을 설정해보도록 하겠다. 맵핑을 하기 위해서는 @RequestMapping("/question/list")를 이용하여 도메인을 연결시킬 수 있다.

<h4>템플릿</h4>

`thymeleaf`는 view template engine으로 컨트롤러에서 전달받은 데이터를 동적인 페이지를 만들 수 있으며 태그의 속성인 thymeleaf 명령어를 사용할 수 있다.

템플릿에 질문 목록을 뿌려주도록 만들어보자. 질문 목록을 가져오기 위해서는 question 리포지터리를 사용한다.

```java

package com.example.demo.question;

import java.util.List;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.ui.Model;

import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor
@Controller
public class QuestionController {

    private final QuestionRepository questionRepository;
    @RequestMapping("/question/list")
    public String list(Model model){
        List<Question> questionList = this.questionRepository.findAll();
        model.addAttribute("questionList",questionList);
        return "question_list";
    }
}

```

`@RequiredArgsConstructor` 애너테이션으로 questionRepository 속성을 포함하는 생성자를 생성하였다. 이 애너테이션은 lombok에서 Getter, Setter 메서드를 생성하는 것과 마찬가지로 자동으로 생성자를 생성한다. 즉 DI 규칙에 의해 questionRepository 객체가 자동으로 주입된다.

Question 리포지터의 findAll 메서드를 사용하여 질문 목록들을 리스트에 담아서 questionList에 저장하였다. Model 객체는 자바 클래스와 템플릿 간의 연결고리 역할을 해주며, Model 객체에 값을 담아두면 템플릿에서 사용할 수 있다. 이는 또한 스프링부트에서 자동으로 Model 객체를 생성해주므로 따로 생성할 필요가 없다.

템플릿 파일에서 Model 객체를 이용해보자.

`th:each="question : ${questionList}` th: 로시작하는 속성은 타임리프 템플릿 엔진이 사용하는 속성이다. 이 부분이 자바 코드와 연결된다.

우리는 QuestionController에서 questionList에 데이터를 담아서 Model 객체에 값을 저장하였다. th:each는 자바 코드의 for each와 같다고 생각하면 된다. 리스트에 담긴 목록들을 하나하나씩 접근할 수 있다.

<h6>타임리프의 속성</h6>

1. 분기문 속성
```java
th:if="${question != null}"
```
이런식으로 조건에 따라 표시하게 할 수 있다.
2. 반목문 속성
each는 반목문이다. `th:each`
3. 텍스트 속성
텍스트로 `값`을 출력해준다. `th:text`


