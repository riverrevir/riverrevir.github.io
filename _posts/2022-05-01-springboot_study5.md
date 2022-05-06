---
layout: post
title: 템플릿 상속과 질문 게시판 추가하기 - 게시판만들기(5)
comments: true
tags: Spring
---

<h4>템플릿 상속</h4>

매번 css 파일을 수정해서 추가하거나 귀찮은 부분을 해결 해주는 기능을 타임리프에서는 제공해주고 있다. 이는 템플릿 상속을 하여 기본 틀을 가지고 있으면 이어서 추가만 해주면 되는 편리한 기능을 가지고 있다.

```html
<!doctype html>
<html lang="ko">
<head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" type="text/css" th:href="@{/bootstrap.min.css}">
    <!-- sbb CSS -->
    <link rel="stylesheet" type="text/css" th:href="@{/style.css}">
    <title>Hello, sbb!</title>
</head>
<body>
<!-- 기본 템플릿 안에 삽입될 내용 Start -->
<th:block layout:fragment="content"></th:block>
<!-- 기본 템플릿 안에 삽입될 내용 End -->
</body>
</html>
```

기본적인 틀이 되는 layout.html을 추가하자. body안에 있는 타임리프 문법이 바로 개별적으로 구현해야 하는 영역이 된다. 이 부분만 작성하면 표준 HTML 문서로 사용할 수 있다.

이후에 html 파일의 작성은 간단하게 작성할 수 있다. 왜냐하면 부모의 기본 베이스를 작성해두었기 때문에 상속해서 쓰기만 하면된다. 부모의 body 부분을 수정하여 내가 원하는데로 사용하기만 하면된다.

```html
<html layout:decorate="~{layout}">
    <div layout:fragment="content" class="container my-3">
        //안에 내용
    </div>
</html>
```

부모 layout을 상속하기 위해 `layout:decorate`속성은 사용할 템플릿을 설정한다. 속성 값으로 `~{layout}`은 layout.html을 의미한다. 이후 div부분의 `layout:fragment="content"`이 부분은 부모 레이아웃에서 설정해둔 body부분을 보면 th:block 엘리먼트의 내용이 자식 템플릿의 div 엘리먼트로 교체된다.

<h4>질문 등록</h4>

```html
<html layout:decorate="~{layout}">
<div layout:fragment="content" class="container">
    <h5 class="my-3 border-bottom pb-2">질문등록</h5>
    <form th:action="@{/question/create}" method="post">
        <div class="mb-3">
            <label for="subject" class="form-label">제목</label>
            <input type="text" name="subject" class="form-control">
        </div>
        <div class="mb-3">
            <label for="content" class="form-label">내용</label>
            <textarea name="content" class="form-control" rows="10"></textarea>
        </div>
        <input type="submit" value="저장하기" class="btn btn-primary my-2">
    </form>
</div>
</html>
```

기존 메인 화면에서 하이퍼링크 이지만 부트스트랩에서의 버튼모양을 가지고 와서 질문을 작성하는 URL로 갈 수 있도록 하였고 컨트롤러에서 이 주소를 맵핑할 수 있도록 추가해주었다.

하지만 지금까지 코딩의 상황으로 보면 예외상황이 있을 수 있다. 무엇일까? `공백으로 질문을 제출`해도 제출이 된다는 점이다. 데이터베이스에 공백이 들어올 때 예외를 던져도 되고 폼에서 입력값을 체크하는 법이 있다. 우리는 폼에서 입력값을 확인하는 방법을 사용하겠다.

<h6>spring boot validation</h6>

`implementation 'org.springframework.boot:spring-boot-starter-validation'`이 라이브러리를 설치하면 입력 값을 검증할 수 있다.

[라이브러리 참조 사이트](https://beanvalidation.org/)

