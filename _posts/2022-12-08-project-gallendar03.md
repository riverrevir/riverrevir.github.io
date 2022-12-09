---
layout: post
title: Project gallendar - Members refactor
comments: true
tags: project
---

<h4> Members entity 리팩토링 </h4>

```java
@Entity
@Getter
@NoArgsConstructor
```

우선 프로젝트에서 사용한 애너테이션이다. `@Setter`는 객체가 언제든지 변경될 수 있는 위험이 있어 안정성을 보장할 수 없습니다. 따라서 의미있는 메서드를 사용하여 데이터의 값이 변경이 필요로 할 때 사용할 수 있도록 설계하였습니다. 

`@NoArgsConstructor` 은 파라미터가 없는 생성자를 자동으로 생성해주는 애너테이션 입니다. `(access = AccessLevel.PROTECTED)`로 변경한다면 의미 없는 객체의 생성을 막을 수 있습니다. 

```java
Members members=new Members(); // error
```

의미가 담긴 객체를 생성하기 위해서는 `@Builder` 를 사용할 수 있습니다. 

```java
    @Builder
    public Members(String id, String email, String password, MemberRole role) {
        this.id = id;
        this.email = email;
        this.password = password;
        this.role = role;
    }
```

<h4> Members Controller 리팩토링 </h4>

```java
@GetMapping
    public List<MemberSearchResponse> searchMemberById(@RequestParam(value = "separatorKey") @NotBlank String id) {
        return memberSearchService.MemberSearchById(id);
    }
```


**RequestParm vs PathVariable**

리팩토링 하기 전에는 RequestParam과 PathVariable의 차이를 단순히 ?와 /로 구분하는 정도로 알고 있었지 이를 어떨 때 마다 구분을 해서 사용하는지에 대한 이해도가 깊지 않았습니다. 하지만 이번 리팩토링을 하면서 명확하게 구분하고자 공부를 하였습니다.

우선 공부한 내용은 RequestParam은 자원 안에 정렬된 값을 뽑으려고 할 때 사용한다면 PathVariable은 자원에 대한 응답을 줄 때 사용하는 것으로 이해하였다.
```
/members/id
/members/id?sex=man
```

 **@RequestParam(value = "separatorKey")**

원래는 value 값에 id라는 값을 명시하였습니다. 하지만 이는 API 문서를 보았을 때 어떤 id 값인지, 역할을 하는게 뭔지에 대한 명확성이 없었습니다. 

해당 separatorKey값에 따라 결과를 도출 해준다의 의미가 좀 더 명확한 것 같아 수정하였습니다.

**응답 개선**

기존에는 어글리한 response를 클라이언트에 주었습니다. 이는 어떠한 상황에서의 오류인지 클라이언트 입장에서 제대로 알 수 없었습니다. 

팀에 따라 응답을 어떻게 줄지에 대한 룰을 정하고 그에 따라 하면될 것 같습니다. 필자는 전역으로 공통 Exception을 처리를 하였습니다.

```java
public enum ErrorCode {
    /* 400 BAD_REQUEST : 잘못된 요청 */
    INVALID_REFRESH_TOKEN(BAD_REQUEST, "리프레시 토큰이 유효하지 않습니다"),
    MISMATCH_REFRESH_TOKEN(BAD_REQUEST, "리프레시 토큰의 유저 정보가 일치하지 않습니다"),

    /* 401 UNAUTHORIZED : 인증되지 않은 사용자 */
    INVALID_AUTH_TOKEN(UNAUTHORIZED, "권한 정보가 없는 토큰입니다"),
    UNAUTHORIZED_MEMBER(UNAUTHORIZED, "현재 내 계정 정보가 존재하지 않습니다"),

    /* 404 NOT_FOUND : Resource 를 찾을 수 없음 */
    MEMBER_NOT_FOUND(NOT_FOUND, "해당 유저 정보를 찾을 수 없습니다"),
    REFRESH_TOKEN_NOT_FOUND(NOT_FOUND, "로그아웃 된 사용자입니다"),

    /* 409 CONFLICT : Resource 의 현재 상태와 충돌. 보통 중복된 데이터 존재 */
    DUPLICATE_RESOURCE(CONFLICT, "데이터가 이미 존재합니다"),
    ;
    private final HttpStatus httpStatus;
    private final String detail;
}
```

`ResponseStatusException`과 비슷해 보이지만, 전역으로 공통 Exception을 처리하는 것과 가장 큰 차이점은 커스텀을 할 수 있다는 차이가 있습니다. 따라서 재사용성이 높은 에러 응답 코드를 관리할 수 있습니다.

`@ControllerAdvice`는 프로젝트 전역에서 발생하는 모든 예외를 잡아줍니다. `@ExceptionHandler` 는 발생한 특정 예외를 잡아서 하나의 메소드에서 공통 처리해줄 수 있게 해줍니다.

잘못된 요청이 클라이언트에서 왔을 시 좀 더 정리된 응답을 주면 클라이언트 입장에서도 편할 것이다. `@Valid`의 예외는 `MethodArgumentNotValidException`을 발생시켜 GlobalExceptionHandler에 이런식으로 추가해주었다. 그렇다면 내가 설정한 예외처리를 응답결과로 보내줄 수 있다.

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        log.error("handleValidException throw Exception : {}", INVALID_INPUT_VALUE);
        return ErrorResponse.toResponseEntity(INVALID_INPUT_VALUE);
    }
```


<h4>회고</h4>

유효한 요청인지에 대한 검증은 클라이언트 측에서도 하겠지만, 백엔드에서도 해야함을 느껴서 검증부분을 추가하였습니다. 또한 응답 개선을 통해 모호한 서버에러를 주는 것 보다 명확한 에러에 대한 정보를 준다면 오류를 찾기도 한층 수월해질 것입니다. 아직 객체지향적에 대한 패턴의 공부를 하지 못하여 제대로 된 리팩토링을 하진 못하였지만, 리팩토링을 하면서 많은 것들에 대한 생각을 할 수 있어서 좋았습니다.