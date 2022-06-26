---
layout: post
title: Spring Security JWT 예외처리 ExceptionHandler - 토이프로젝트(4)
comments: true
tags: Spring
---


이번 프로젝트를 진행하면서 기존의 예외처리는 ExceptionHandler로 예외처리를 하였다. 하지만 JWT 토큰 검증 부분에서의 예외처리를 `SignatureException`를 이용하려 하였으나 실패하였다. 이유를 알아보자. 


<h4>처리 순서</h4>

우리가 만든 ExceptionHandler는 Spring에서 처리해주는 부분이라고 볼 수 있다. 하지만 JWT 토큰의 인증 유무에 따라 서버에서 기능을 제공해주는 부분은 `Spring Security에서 먼저 필터링`하기 때문에 우리가 만든 예외를 거치지 못하는 것이였다. 여러 가지 처리 방법이 있지만, 나는 `JwtAuthenticationEntryPoint` 즉 토큰에서 발생하는 모든 에러는 이 부분에서 응답해주는 것을 깨달았다. 

```java
@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint, Serializable {
    private static final long serialVersionUID = -7858869558953243875L;

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        response.setContentType("application/json");
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.getOutputStream().println("{ \"error\": \"" + authException.getMessage() + "\" }");
    }
}
```


기존의 JWT 예제를 보면 응답하는 부분이 미흡하다고 생각하였고, ExceptionHandler에서 예외를 보기 쉽게 처리를 하려 하였으나 실패하여서 `AuthenticationEntryPoint`에서 제공하는 예외처리를 사용하도록 하겠다. 물론 예외처리 부분을 부분적으로 나눠서 어떤 에러인지 명확하게 표현하는 방법도 중요하다. 하지만 굳이 사용자에게 토큰의 유효성이 만료되었다던지, 서명이 잘못되었다는지 이러한 예외상황을 굳이 알려줄 필요는 없다고 판단하였고, 각자가 원하는 응답을 해주면 될 것 같다.

[Github 소스코드 확인](https://github.com/riverrevir/devstudy-today/commit/c7efdb0a1860f409d01a65464701de7717a44297) 좀 더 자세한 프로젝트의 소스코드는 여기서 확인할 수 있습니다. 


<h4>회고</h4>

Spring Security에서의 처리 순서에 대해 생각하게 되었고, JWT 토큰 관련 문서를 좀 더 정확하게 읽어보면서 이해를 하는 것이 필요할 것 같다. 추가로 웹페이지에서 JWT 토큰 처리 방식에 대해 내가 몇일간 삽질한 내용들을 정리하여 올릴 예정이다.

