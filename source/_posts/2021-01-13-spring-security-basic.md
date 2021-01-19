---
layout: post
title:  "Spring Security 의 기본 사용방법에 대한 고찰"
date:   2021-01-13 11:38:54 +0900
categories: spring
---

스프링에서 제공해주는 인증, 권한의 기능을 활용하기 위해서 Spring Security 라이브러리를 적용했다. 처음으로 적용해본 경험으로는 다른 스프링 라이브러리와 다르게 조금은 침투적인 성향이 있는거 같다. 회원이라는 개념을 가진 객체를 다루기 위해서는 Spring Security 가 제공하는 추상 객체를 구현하여야 한다. 이부분이 처음에는 조금 어렵게 다가왔지만 조금만 이해하면 손쉽게 회원 관리를 할 수 있을 것 같다.

### 의존성 추가

Spring Security 를 사용하기 위한 의존성을 추가한다. 스프링 부트에서 버전 관리를 해주는 의존성이기 때문에 버전설정을 하지 않아도 된다.

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

```

### Config 파일 구현

Spring Security 를 사용하기 위해서 기본적인 설정을 적용한다.

```java

@Configuration
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final MemberService memberService;

    /* 패스워드를 암호화시킬 인코더 객체를 생성하고 이를 빈으로 등록. */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /* 
        HTTP 통신에 대한 기본적인 설정을 한다. 
        .csrf().disable() 
        : CSRF(Cross Site Request Forgery) 공격에 대비하기 위한 설정.

        .authorizeRequests().anyRequest().authenticated() 
        : 각 URL 별로 권한 설정이 가능한데 이 케이스는 모든 요청에 대해서 기본적인 권한처리를 한다는 의미.
    
        .httpBasic()
        : HTTP 통신 시 가장 기본이 되는 설정으로 등록.
    */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .csrf().disable()
        .authorizeRequests().anyRequest().authenticated()
        .and()
        .httpBasic();
    }

    /* 회원 객체를 위해 사용하는 Service와, 패스워드를 위한 인코더를 등록한다. */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(memberService).passwordEncoder(passwordEncoder());
    }
}

```

주의해서 봐야할 사항은 우선 WebSecurityConfigurerAdapter 인터페이스 객체를 구현해야한다. 그리고 이 객체가 가지고 있는 추상 메서드들을 필요조건에 따라서 맞추어 코드를 작성한다. 아래 코드는 http 설정을 하는 방법을 예시로 가져왔다.

```java

@Override
protected void configure(HttpSecurity http) throws Exception {
    http
          .authorizeRequests()
            .antMatchers("/login", "/signup", "/user").permitAll() // 누구나 접근 허용
            .antMatchers("/").hasRole("USER") // USER, ADMIN만 접근 가능
            .antMatchers("/admin").hasRole("ADMIN") // ADMIN만 접근 가능
            .anyRequest().authenticated() // 나머지 요청들은 권한의 종류에 상관 없이 권한이 있어야 접근 가능
        .and() 
          .formLogin()
            .loginPage("/login") // 로그인 페이지 링크
            .defaultSuccessUrl("/") // 로그인 성공 후 리다이렉트 주소
        .and()
          .logout()
            .logoutSuccessUrl("/login") // 로그아웃 성공시 리다이렉트 주소
	    .invalidateHttpSession(true) // 세션 날리기
    ;
}

```

### 회원 도메인 객체 생성

```java

@Slf4j
@Getter
@Setter
@ToString(exclude = "cells")
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Member implements UserDetails {
    @Id
    private String email;

    private String password;

    private String auth;

    /* 
        이 함수에서 반환된 결과가 이 멤버가 가지고 있는 권한 정보이다. auth 값에 "USER, ADMIN" 이렇게 들어있다면 USER, ADMIN 두 권한을 가지는 것이다.
    */
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Set<GrantedAuthority> roles = new HashSet<>();
        for(String role : auth.split(",")) {
            roles.add(new SimpleGrantedAuthority(role));
        }

        return roles;
    }

    /* 회원 객체에서 ID에 해당하는 값을 반환시킨다 */
    @Override
    public String getUsername() {
        return email;
    }

    /* 계정의 만료 여부를 반환하는 함수 */
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    /* 계정의 잠금 여부를 반환하는 함수 */
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    /* 패스워드의 만료 여부를 반환하는 함수 */
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    /* 계정 사용 가능 여부를 반환하는 함수 */
    @Override
    public boolean isEnabled() {
        return true;
    }
}

```

여기서는 회원으로 사용할 도메인 객체에 Spring Security 에서 제공하는 추상 클래스 UserDetails 를 구현한다.
이 객체는 꽤 많은 추상 메서드들을 구현해야 하는데 주석에 각 추상 메서드의 기능을 적어두었다. 구현해야하는 코드가 많아서 조금은 마음에 안들지만 이해하면 그렇게 어렵지 않다.

### Service 구현

```java

@Slf4j
@Service
public class MemberService implements UserDetailsService {
    private MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Override
    public Member loadUserByUsername(String email) throws UsernameNotFoundException {
        return memberRepository.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException(email));
    }
}

```

UserDetailsService 인터페이스를 구현하고, loadUserByUsername() 함수를 구현한다. 주의할 점은 반환형이 우리가 사용하는 회원 객체로 바꾸어 준다.