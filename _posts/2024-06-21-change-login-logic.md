---
title: spring 로그인 로직 변경
excerpt: 
categories:
  - gugumo
tags: 
permalink: /project/gugumo/changeLoginLogic
toc: true
toc_sticky: true
date: 2024-06-21
last_modified_at: 2024-06-21
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

[개발자 유미](https://www.youtube.com/watch?v=3Ff7UHGG3t8&list=PLJkjrxxiBSFCcOjy0AAVGNtIa08VLk1EJ&index=7) 님의 영상을 보고 로그인 로직을 구현했었는데, 그 방법을 변경하려고 한다.  
이 게시글에서는 어떤 방식으로 변경했는지, 왜 변경했는지 기록하려고 한다.  

---

### 기존 로그인 로직

기존에는 spring security를 이용해서 로그인 로직을 구현했었다.  

spring security의 `AbstractAuthenticationProcessingFilter` 추상 클래스를 재정의하고 로그인 성공에 따른 handler를 구현해서 로그인하였다.  

#### spring security 로그인 로직

1. AbstractAuthenticationProcessingFilter 를 재정의한 클래스에서 로그인 정보를 받아온다.
2. AuthenticationManager에서 적당한 인증 방법을 찾아서 USerDetailService로 유저 조회를 한다.
3. 조회 여부에  따라 성공 실패 처리를 진행한다.  

2번 과정에서 AuthenticationManager는 인터페이스고 그 구현체로 ProviderManager가 있는데, 정의된 많은 로그인 방법 중에서 사용자 요청에 맞는 구현체를 찾아 로그인을 요청하게 된다.  

prividerManager는 AuthenticationProvider를 여러 개 가지고 있고, AuthenticationProvider는 authenticate와 supports 2개의 함수를 가지고 있다.  authenticate는 실질적으로 인증을 담당하는 부분이고, supports는 사용자의 요청을 처리할 수 있는가에 대한 true, false를 반환하는 함수이다.  

간단히 말하면 사용자의 요청이 있을 때 AuthenticationManager의 구현체인 ProviderManager는 자기가 가지고 있는 많은 AuthenticationProvider 중 사용할 수 있는 인증 방법을 사용하게 된다.  

**provider 패턴**을 찾아보시면 더욱 이해하기 쉽습니다.

#### 로그인 로직을 변경하는 이유

+ spring security의 로직을 따르다 보니 이 로직을 모르는 경우 흐름을 파악하기가 힘들다.  

sprint security를 이용해서 `HTTP_METHOD, CONTENT_TYPE` 을 원하는 타입만 받을 수 있다는 장점도 있지만, 이러한 장점 보다는 로직을 한 눈에 파악할 수 있는 것이 더 좋다고 생각해서 다시 구현하려고 한다.  

**필자의 현재 상태에서 정리한 것이기 때문에 완전하지 않은 정보입니다.**

---

### 새로운 로그인 로직

Controller, Service를 통해 로그인 방식을 직접 구현하려고 한다.  

**Controller.java**
``` java 
@PostMapping("/api/v1/emailLogin")  
@ResponseStatus(HttpStatus.OK)  
public ApiResponse<String> emailLogin(@RequestBody EmailLoginRequestDto emailLoginRequestDto) {  
  
    String token = memberService.emailLogin(emailLoginRequestDto);  
  
    return ApiResponse.createSuccess("Bearer " + token);  
}
```

**EmailLoginRequestDto.java**
``` java
@Getter  
public class EmailLoginRequestDto {  
    private String username;  
    private String password;  
}
```

**memberService.emailLogin.java**
``` java
public String emailLogin(EmailLoginRequestDto emailLoginRequestDto) {  
  
    Member findMember = memberRepository.findByUsername(emailLoginRequestDto.getUsername())  
            .orElseThrow(() ->  
                    new UserNotFoundException("회원이 없습니다.")  
            );  
  
    if(!passwordEncoder.matches(emailLoginRequestDto.getPassword(), findMember.getPassword())) {  
        throw new UserNotFoundException("회원이 없습니다.");  
    }  
  
    EmailLoginProcessDto emailLoginDto = EmailLoginProcessDto.builder()  
            .id(findMember.getId())  
            .username(findMember.getUsername())  
            .role(findMember.getRole().name())  
            .requestTimeMs(System.currentTimeMillis())  
            .build();  
  
    return jwtUtil.createJwt(emailLoginDto);  
}
```

복잡한 로직 없이 간단한 코드로 구현하였기에 누가 봐도 흐름을 한 눈에 파악할 수 있다는 장점이 있다.  

여기서 중요한 점은 JWT 토큰을 반환할 때 앞에 `Bearer ` 를 추가해줬다는 것이다.  

**JwtFilter.java**
``` java
if (authorization == null || !authorization.startsWith("Bearer ")) {  
    filterChain.doFilter(request, response);  
    return;  
}
```

위 코드를 보면 `Bearer `  로 시작하지 않을 경우 더 이상 진행하지 않고 `return`하는 것을 볼 수 있다. 이는 HTTP 이증 방식을 따른 것으로 이 토큰의 소유자에게 권한을 부여하라는 의미를 가지고 있다. 자세한 내용은 JWT 관련 글에서 다루겠다.  


