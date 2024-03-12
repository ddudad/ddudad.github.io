---
title: Spring이 어떻게 의존성을 주입하는가-수동편
excerpt: Spring에서 Bean을 어떻게 등록하는지 알아보자
categories:
  - Spring
tags: 
permalink: /web/spring/Spring이 어떻게 의존성을 주입하는가 수동편
toc: true
toc_sticky: true
date: 2024-02-04
last_modified_at: 2024-03-10
---
아래 내용은 김영한님의 강의를 들으며 중요하다고 생각되는 부분들을 정리한 내용입니다.  더 자세한 내용이 궁금하다면 아래 강의를 들어보시는 것을 추천드립니다.  
[스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)  

---

[DI(Dependency Injection)란](https://ddudad.github.io/web/spring/DI%EB%9E%80)에서 DI에 대해서 알아봤다.  
``@Autowired``어노테이션을 통해 의존성을 주입받을 수 있다고 했는데 Spring에서 어떤 방식으로 의존성을 주입하는지 알아보자.  

의존성 주입 방식에는 크게 수동, 자동 2가지 방식이 있는데 수동방식에 대해서 알아보겠다.

---
## 스프링 빈의 라이프사이클

의존성 주입 방식을 알아보기 전에 스프링 빈의 라이프사이클을 알아보자.  
```
스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸 전 콜백 -> 스프링 종료
``` 
다음과 같은 과정을 거치는데, 우리가 알아볼 것은 ``스프링 빈 생성 -> 의존관계 주입`` 부분이다.  

---
## Config 관련 클래스

``` java
@Configuration  
public class AppConfig {  
  
    @Bean  
    public MemberService memberService() {  
        return new MemberServiceImpl(memberRepository());  
    }  
  
    @Bean  
    public OrderService orderService() {  
        return new OrderServiceImpl(memberRepository(), discountPolicy());  
    }  
}
```

스프링 빈 자동에서는 ``@Component``로 스프링 빈에 등록할 클래스를 지정해서 클래스의 인스턴스를 자동으로 등록했다면 수동에서는 위와 같은 자바 코드를 통해 인스턴스를 생성 후 등록해줘야 한다.  

## @Configuration

``@Configuration``은 스프링 빈으로 등록되는 인스턴스가 싱글톤으로 유지될 수 있도록 하는 어노테이션이다.  

``` java
@Bean  
    public MemberService memberService() {  
        return new MemberServiceImpl(memberRepository());  
    }  
```

코드를 보면 ``new``를 통해 새로운 인스턴스를 생성하기 때문에 ``memberService()``가 호출될 때마다 서로 다른 인스턴스를 생성하게  되는데 이러면 싱글톤으로 유지할 수 없기 때문에 ``@Configuration``을 사용한다.  

## @Bean

``@Bean``이 붙은 메서드는 스프링 빈에 등록하겠다는 의미이다.  
``@Bean``이 없다면 ``memberService()``함수가 호출될 때마다 새로운 인스턴스를 반환하겠지만 ``@Bean``이 있을 경우 그 인스턴스를 스프링 빈으로 등록하게 된다.


## 등록한 빈 사용하는 방법

``` java 
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);  
  
MemberService memberService = applicationContext.getBean("memberService", MemberService.class);

Long memberId = 1L;  
Member member = new Member(memberId, "memberA", Grade.VIP);  
memberService.join(member);
```

스프링에서는 ``ApplicationContext``라는 인터페이스를 지원하며 그 구현체인 ``AnnotationConfigApplicationContext``의 함수인 ``getBean``을 통해 스프링 빈에 등록된 인스턴스를 가져오고 사용할 수 있다.


---

## 어떤 상황에서 수동등록을 사용하는가

보통은 자동으로 빈을 등록해서 사용하는 것이 좋다.  
왜냐하면 수동으로 등록하는 것은 등록하려는 빈이 많이질 수록 Config클래스에 작성할 코드가 많아지고 관리하기 어려워지기 떄문이다.  

하지만 외부 라이브러리의 경우 코드에 ``@Component``같은 어노테이션을 사용할 수 없기 때문에 수동으로 등록해서 사용하는 편이다.  

---

큰 범위에서 동작 방식을 설명한 것이며 세부적인 동작 방식 및 여러 옵션, 사용법은 추후 자세히 설명하겠다.