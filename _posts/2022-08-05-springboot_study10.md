---
layout: post
title: Spring Security CORS 해결
comments: true
tags: Spring
---

<h4>Spring Security</h4>

`CORS`는 에러라고 볼 수 없다. 서버는 8080포트로 구동되고 프론트는 3000 포트로 구동된다면 서로 다른 포트에 의해 CORS 위반 에러가 난다고 볼 수 있다. 단순히 보안상의 이유이므로 어떠한 누구의 잘못도 없으므로 에러라고 보기엔 애매하다.

CORS는 Spring Security에 앞서 처리되어야 한다. 가장 쉬운 방법으로는 `CorsFilter`를 이용하여 해결할 수 있다.

```java

protected void configure(HttpSecurity http) throws Exception {
        http
                .cors().configurationSource(corsConfigurationSource())
                .and()
                .csrf().disable()
                ~~~
}
@Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();

        configuration.setAllowedOriginPatterns(Arrays.asList("*"));
        configuration.setAllowedMethods(Arrays.asList("HEAD","POST","GET","DELETE","PUT"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
```

아래의 메서드를 사용하여 Cors를 해결 할 수 있다.