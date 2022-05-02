---
layout: post
title: MVC 패턴 맛보고 스태틱 디렉터리 관리 - 게시판만들기(4)
comments: true
tags: Spring
---

엔티티 클래스는 컨트롤러에서 사용할 수 없게 설계하는게 좋다. 대신 사용할 수 있는 data transfer object(DTO) 클래스를 만들어야 한다. 이 부분을 서비스 부분에서 컨트롤러와 리포지터리의 중간 다리 역할을 해준다.

이제는 Service 클래스를 추가하여 `controller -> service -> repository` 구조로 데이터를 전달할 것이다.

메인화면에 이제 질문리스트들을 뿌려주게 된다. 우리는 이 질문리스트를 누르면 상세한 정보를 볼 수 있게 할 것이고, 여기에서 답변을 등록할 수 있게 해줄 것이다. 질문을 클릭하면 `/question/detail/id`의 URL로 이동시켜보자.

```java
@RequestMapping(value="/question/detail/{id}")
    public String detail(Model model,@PathVariable("id") Integer id){
        Question question = this.questionService.getQuestion(id);
        model.addAttribute("question",question);
        return "question_detail";
    }
```

@PathVariable 애너테이션은 변화하는 값을 얻어야 할때 사용한다. 매개변수와 경로에서 사용한 값의 이름은 동일해야 한다.

<h6>답변등록</h6>

```java
package com.example.demo.answer;

import com.example.demo.question.Question;
import com.example.demo.question.QuestionService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@RequestMapping("/answer")
@RequiredArgsConstructor
@Controller
public class AnswerController {
    private final QuestionService questionService;

    @PostMapping("/create/{id}")
    public String createAnswer(Model model,@PathVariable("id") Integer id,@RequestParam String content){
        Question question = this.questionService.getQuestion(id);
        return String.format("redirect:/question/detail/%s",id);
    }
}

```

POST 요청을 받을 때는 @PostMapping 애너테이션을 이용한다. 템플릿에서 작성된 내용을 얻기 위해 @RequestParam 을 추가하여 템플릿의 `name` 속성명을 가져와 사용할 수 있다.

<h4>스태틱 디렉터리</h4>

스타일 시트인 .css는 스태틱 디렉터리에 저장하고 사용한다.