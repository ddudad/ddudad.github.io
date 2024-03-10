---
title: DI(Dependency Injection)란
excerpt: DI(Dependency Injection)를 알아보자
categories:
  - Spring
tags: 
permalink: /web/spring/DI란
toc: true
toc_sticky: true
date: 2024-02-02
last_modified_at: 2024-03-10
---
아래 내용은 김영한님의 강의를 들으며 이해가 안되던 부분들을 정리한 내용입니다.  더 자세한 내용이 궁금하다면 아래 강의를 들어보시는 것을 추천드립니다.  
[스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)  

---

[DI(Dependency Injection)을 사용하는 이유](https://ddudad.github.io/web/spring/DI%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%20%EC%9D%B4%EC%9C%A0) 에서 왜 DI가 필요한지 알아봤다.  
이번에는 DI에 대해서 자세히 알아보자.  

---
## DI(Dependency Injection)

dipendency injection으로 단어 뜻 그대로 해석하자면 의존 주입이라고 볼 수 있다.  

``` java
public class MemberService {
	//private MemberRepository memberRepository = new MemoryMemberRepository();
	private MemberRepository memberRepository = new JdbcMemberRepository();
}
```

위 코드처럼 ``new``를 통해 내부에서 객체를 직접 생성하는 것이 아니라 **외부에서 생성 후 주입시켜주는 방식**을 의미한다.  

---

## Spring에서 지원하는 DI 3가지 방법

+ Field Injection(필드 주입)
+ Setter Injection(Setter 주입)
+ Construct Inection(생성자 주입)

**참고**  
``@Autowired``가 붙어있으면 그 필드는 외부에서 Spring이 인스턴스를 주입해 준다고 보면 된다.  
``@Component`` 는 일단은 무시하면 된다.

### Field Injection(필드 주입)

이름 그대로 필드에서 바로 주입하는 방법이다.  
``` java
@Component 
public class OrderServiceImpl implements OrderService { 

	@Autowired 
	private MemberRepository memberRepository; 
	@Autowired 
	private DiscountPolicy discountPolicy; 
}
```

#### 단점
+ 외부에서 마음대로 변경이 불가능하기 때문에 테스트하기 힘들다.
+ Spring의 DI기능이 없다면 아무것도 할 수 없다.

### Setter Injection(Setter 주입)

``` java
@Component 
public class OrderServiceImpl implements OrderService { 

	private MemberRepository memberRepository; 
	private DiscountPolicy discountPolicy; 
	
	@Autowired 
	public void setMemberRepository(MemberRepository memberRepository) {
		this.memberRepository = memberRepository; 
	} 
	@Autowired 
	public void setDiscountPolicy(DiscountPolicy discountPolicy) {
		this.discountPolicy = discountPolicy; 
	} 
}
```

외부에서 setter함수를 통해 필요한 인스턴스를 주입하는 방식  

#### 단점
+ setter함수가 public으로 누구나 접근이 가능하기 때문에 주입이 끝난 곳에 또 주입하는 상황이 생길 수 있다.
### Constructor Injection(생성자 주입)

``` java
@Component 
public class OrderServiceImpl implements OrderService { 

	private final MemberRepository memberRepository; 
	private final DiscountPolicy discountPolicy;
	 
	@Autowired 
	public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) { 
		this.memberRepository = memberRepository; 
		this.discountPolicy = discountPolicy; 
		} 
}
```

생성자를 통해 주입하는 방식으로 제일 좋은 방식이다.

#### 장점
+ Spring 공식 문서에도 권장하는 방식
+ 생성될 때 1번 호출되기 때문에 인스턴스가 변경될 일이 없다.
+ 필드에 ``final``을 사용할 수 있어 혹여나 주입을 놓쳤을 경우 컴파일 과정에서 오류로 알아챌 수 있다.
+ ``Lombok`` 라이브러리와 ``@RequiredArgsConstructor``를 함께 활용할 경우 코드가 매우 깔끔해 질 수 있다.

#### Lombok 라이브러리 + @RequiredArgsConstructor 어노테이션 조합

``` java
@Component 
@RequiredArgsConstructor
@Getter @Setter
public class OrderServiceImpl implements OrderService { 

	private final MemberRepository memberRepository; 
	private final DiscountPolicy discountPolicy;
}
```

필드주입 급으로 깔끔해진 것을 볼 수 있다.

---

지금까지 Spring에서 사용할 수 있는 3가지 DI방법을 알아보았다.  
다음에는 Spring이 어떻게 DI를 지원하는지 동작 방식에 대해서 알아보겠다.