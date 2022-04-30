---
layout: post
title: JPA의 기본문법을 익혀보자 - 게시판만들기(2)
comments: true
tags: Spring
---

<h4>리포지터리</h4>

데이터 처리를 위해서는 데이터베이스와 연동하는 JPA 리포지터리가 필요하다. 즉 데이터베이스의 테이블에 접근할 수 있는 메서드들을 사용하기 위한 인터페이스이다.

흔히 데이터를 수정삭제삽입만들기 CRUD작업을 많이 할 것이다. 이 때 사용하는 것이 리포지터리다.

```java
package com.example.demo;

import org.springframework.data.jpa.repository.JpaRepository;

public interface QuestionRepository extends JpaRepository<Question,Integer>{

}

```

리포지터리로 만들기 위해서는 JpaRepository 인터페이스를 상속받아야 한다. `<Question,Integer>` 대상이 되는 엔티티의 타입과 해당 엔티티의 PK의 속성 타입을 지정해야 한다. Question의 pk는 우리가 자동으로 게시글에 넘버링을 한 Integer형식이다.

<h4>테스트해보기</h4>

파일 목록에 test라는 파일안에 파일이름Tests라는 파일이 있다. 여기에서 우리는 데이터를 삽입해볼 것이다. 

@SpringBootTest 애너테이션은 스프링부트 테스트 클래스임을 의미한다. @Autowired 애너테이션은 스프링의 DI(Dependency Injection) 기능으로 객체를 스프링이 자동으로 생성해준다. 즉 스프링이 객체를 대신 생성하여 주입한다고 보면된다.

실제로는 Autowired를 사용하는 것보다는 Setter 또는 생성자를 이용해서 객체를 주입한다. 이는 순환참조의 문제가 있다고한다. 이에 대해서는 나중에 자세히 알아보도록 하겠다.

@Test 애너테이션은 테스트 메서드임을 의미한다.

`JUnit`은 테스트코드를 작성하고 이 코드를 실행하기 위해 사용하는 자바의 테스트 프레임워크이다.

<h6>findAll</h6>

```java
List<Question> all = this.questionRepository.findAll();
```

Question 테이블에 저장된 모든 데이터를 조회하기 위해서는 findAll 메서드를 사용하면된다. 리스트에 담아서 출력할 수 있다.

<h6>findById</h6>

```java
Optional<Question> oq=this.questionRepository.findById(1);

		if(oq.isPresent()){
			Question q=oq.get();
			System.out.println(q.getSubject());
		}
```

id 값으로 데이터를 조회하기 위해서는 findById 메서드를 이용한다. findById의 리턴값은 `Optional`이기 때문에 isPresent로 null인지 아닌지 확인안 후에 get으로 실제 객체 값을 얻어야 한다.

주로 `orElseThrow()`를 통해 값이 없을 경우 예외를 던져주거나 `orElse`, `orElseGet`을 통해 null일 경우 값을 지정해줄 수 있다.

`findBy + 엔티티의 속성명`과 같은 리포지터리 메서드를 작성하면 findById와 같이 사용이 가능하다. 왜 사용이 가능할까? 우리는 메서드를 구현하지도 않았는데 저런 형식의 메서드를 선언만 하여 사용이 가능한 이유는 JpaRepository를 상속한 객체가 `DI에 의해 스프링이 자동으로 객체를 생성하고 리포지터리 객체의 메서드가 실행될 때 JPA가 해당 메서드명을 분석하여 쿼리를 만들고 실행한다.`

두 개의 속성을 `And` 조건으로도 리포지터리에 메서드를 추가하여 값을 가져올 수 있다. 이는 `findBy + 엔티티의 속성명+ And + 엔티티의 속성명`으로 검색하여 값을 받을 수 있다.

`Like`검색은 `findBy + 엔티티 명 + Like`를 붙여서 해당 데이터가 포함되어 있는지 검사할 수 있다.

<li>test% - test로 시작하는 문자열</li>
<li>%test - test로 끝나는 문자열</li>
<li>%test% - test를 포함하는 문자열</li>

[더 많은 조합 문서 바로가기](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)



