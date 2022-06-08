---
layout: post
title: JWT 토큰 구현 - 토이프로젝트(3)
comments: true
tags: Spring
---

`Spring boot + MySQL + JPA + Gradle`

`Git flow : feature/jwt`


JSON 웹 토큰(JSON Web Token, JWT)은 선택적 서명 및 선택적 암호화를 사용하여 데이터를 만들기 위한 인터넷 표준이다. 구조는 간단하게 설명하면, `헤더, 페이로드, 서명`으로 이루어진다. 이후 프로젝트를 진행하면서 로직에 대한 그림을 깃허브에 첨부하도록 하겠다. 우선 구현을 하면서 어떠한 문제점이 있었는지, 이 소스가 어떤 역할을 하는지 간략히 알아보고 넘어가도록 하자.

<h4>auth API 구현</h4>

**build.gradle**

`implementation group: 'io.jsonwebtoken', name: 'jjwt', version: '0.2'` 설치를 해준다.

**application.yml 설정**
```java
jwt:
    #비밀키 노출 조심해야합니다.
    secret: jwtscret
```

시크릿키를 작성합니다.

**JwtTokenUtil**
```java
@Service
public class JwtTokenUtil implements Serializable {
    private static final long serialVersionUID = -2550185165626007488L;

    public static final long JWT_TOKEN_VALIDITY = 5 * 60 * 60;

    @Value("${spring.jwt.secret}")
    private String secret;

    //retrieve username from jwt token
    // jwt token으로부터 username을 획득한다.
    public String getUsernameFromToken(String token) {
        return getClaimFromToken(token, Claims::getSubject);
    }

    //retrieve expiration date from jwt token
    // jwt token으로부터 만료일자를 알려준다.
    public Date getExpirationDateFromToken(String token) {
        return getClaimFromToken(token, Claims::getExpiration);
    }

    public <T> T getClaimFromToken(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = getAllClaimsFromToken(token);
        return claimsResolver.apply(claims);
    }

    //for retrieveing any information from token we will need the secret key
    private Claims getAllClaimsFromToken(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }

    //check if the token has expired
    // 토큰이 만료되었는지 확인한다.
    private Boolean isTokenExpired(String token) {
        final Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }

    //generate token for user
    // 유저를 위한 토큰을 발급해준다.
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return doGenerateToken(claims, userDetails.getUsername());
    }

    //while creating the token -
    //1. Define  claims of the token, like Issuer, Expiration, Subject, and the ID
    //2. Sign the JWT using the HS512 algorithm and secret key.
    //3. According to JWS Compact Serialization(https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-41#section-3.1)
    //   compaction of the JWT to a URL-safe string
    private String doGenerateToken(Map<String, Object> claims, String subject) {

        return Jwts.builder().setClaims(claims).setSubject(subject).setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + JWT_TOKEN_VALIDITY * 1000))
                .signWith(SignatureAlgorithm.HS512, secret).compact();
    }

    //validate token
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
}
```

**UserDetailService**
```java
@Service
public class JwtUserDetailService implements UserDetailsService {
    @Autowired
    UserService userService;

    @Override
    public UserDetails loadUserByUsername(String UserId) throws UsernameNotFoundException {
        Optional<User> userOptional = userService.findUserByUserId(UserId);
        User user = userOptional.orElseThrow(() -> new UsernameNotFoundException("user name not found!"));
        return new org.springframework.security.core.userdetails.User(user.getUserId(), user.getPassword(), new ArrayList<>());
    }

}

```

**JwtAuthenticationController**
```java
@RestController
@CrossOrigin
public class JwtAuthenticationController {
    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private JwtTokenUtil jwtTokenUtil;

    @Autowired
    private JwtUserDetailService jwtUserDetailService;

    @RequestMapping(value = "/api/user/auth", method = RequestMethod.POST)
    public ResponseEntity<?> createAuthenticationToken(@RequestBody JwtRequest authenticationRequest) throws Exception {
        authenticate(authenticationRequest.getUserid(), authenticationRequest.getPassword());
        final UserDetails userDetails = jwtUserDetailService.loadUserByUsername(authenticationRequest.getUserid());
        final String token = jwtTokenUtil.generateToken(userDetails);
        System.out.println(token);
        ApiResponse apiResponse = new ApiResponse();
        apiResponse.setData(token);
        return new ResponseEntity<>(apiResponse, HttpStatus.OK);
    }

    private void authenticate(String userid, String password) {
        try {
            authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(userid, password));
        } catch (DisabledException ex) {
            throw new DisabledException("USER_DISABLED", ex);
        } catch (BadCredentialsException ex) {
            throw new BadCredentialsException("INVALID_CREDENTIALS", ex);
        }
    }
}
```

**JwtRequest**
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class JwtRequest implements Serializable {
    private static final long serialVersionUID = 263011851808996064L;
    private String userid;
    private String password;
}
```

**JwtResponse**
```java
@Getter
@AllArgsConstructor
public class JwtResponse implements Serializable {
    private static final long serialVersionUID = -868765630229614668L;
    private final String jwttoken;
}
```

**JwtRequestFilter**
```java
@Slf4j
@Component
public class JwtRequestFilter extends OncePerRequestFilter {
    @Autowired
    private JwtUserDetailService jwtUserDetailService;

    @Autowired
    private JwtTokenUtil jwtTokenUtil;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        final String requestTokenHeader = request.getHeader("Authorization");
        String userid = null;
        String jwtToken = null;
        // JWT 토큰은 "Beare token"에 있다. Bearer단어를 제거하고 토큰만 받는다.
        if (requestTokenHeader != null && requestTokenHeader.startsWith("Bearer ")) {
            jwtToken = requestTokenHeader.substring(7);
            try {
                userid = jwtTokenUtil.getUsernameFromToken(jwtToken);
            } catch (IllegalArgumentException ex) {
                logger.error("Unable to get JWT token", ex);
            } catch (Exception ex) {
                logger.error("token valid error:" + ex.getMessage(), ex);
                throw new RuntimeException("11 Username from token error");
            }
        } else {
            logger.warn("JWT token does not begin with Bearer String");
        }
// 토큰을 가져오면 검증을 한다.
        if (userid != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.jwtUserDetailService.loadUserByUsername(userid);

            // 토큰이 유효한 경우 수동으로 인증을 설정하도록 스프링 시큐리티를 구성한다.
            if (jwtTokenUtil.validateToken(jwtToken, userDetails)) {
                UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken =
                        new UsernamePasswordAuthenticationToken(
                                userDetails,
                                null,
                                userDetails.getAuthorities()
                        );
                usernamePasswordAuthenticationToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request)
                );

                // 컨텍스트에 인증을 설정 한 후 현재 사용자가 인증되도록 지정한다.
                // 그래서 Spring Security 설정이 성공적으로 넘어간다.
                SecurityContextHolder.getContext().setAuthentication(usernamePasswordAuthenticationToken);
            }
        }
        filterChain.doFilter(request, response);
    }

}
```

**JwtAuthenticationEntryPoint**
```java
@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint, Serializable {
    private static final long serialVersionUID = 4858836244833364014L;

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized");
    }
}
```

**SecurityConfig**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Autowired
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;

    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private JwtRequestFilter jwtRequestFilter;

    @Autowired
    public void configGlobal(AuthenticationManagerBuilder auth) throws Exception {
        // 일치하는 자격증명을 위해 사용자를 로드할 위치를 알수 있도록
        // AuthenticationManager를 구성한다.
        // BCryptPasswordEncoder를 이용
        auth.userDetailsService(userDetailsService)
                .passwordEncoder(passwordEncoder());
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .cors().disable()
                .csrf().disable()
                // 이 요청은 인증을 하지 않는다.
                .authorizeRequests().antMatchers(
                        "/authenticate",
                        "/api/user/login",
                        "/api/user/auth",
                        "/api/user/register")
                .permitAll()
                // 다른 모든 요청은 인증을 한다.
                .anyRequest().authenticated().and()
                // 상태없는 세션을 이용하며, 세션에 사용자의 상태를 저장하지 않는다.
                .exceptionHandling().authenticationEntryPoint(jwtAuthenticationEntryPoint).and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
                .formLogin().disable()
                .headers().frameOptions().disable();
        // 모든 요청에 토큰을 검증하는 필터를 추가한다.
        http.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);
    }


}
```

**ApiResponse**

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class ApiResponse {
    private int status = 200;
    private String message = "ok";
    private String code = "200";
    private Object data = "no data";
}
```

자세한 동작과정은 게시글을 계속해서 수정하면서 추가해보도록 하겠다. 예제를 잘 따라 왔다면, JWT 토큰 부여도 간단하게 성공하였을 것이다. 자세한 개발흐름은 깃허브를 참고해주시거나 이슈나 댓글 달아주시면 답변해드리겠습니다.

<h4>회고</h4>

이걸 작성하면서 여러 가지 오류에 휘말려서 진행에 힘든점도 있었지만, 계속해서 오류를 접할 때 마다 이슈로 정리하여 다른 사람들도 나와 같은 오류가 발생하였을 때 시간낭비를 하지 않도록 하기 위해서 블로그나 깃허브에 정리하고 있습니다.

[에러 발생 및 해결방안](https://github.com/riverrevir/devstudy-today/issues/17)
