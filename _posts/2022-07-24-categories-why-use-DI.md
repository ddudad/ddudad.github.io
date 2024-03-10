---
title: DI(Dependency Injection)을 사용하는 이유
excerpt: DI(Dependency Injection)은 왜 사용할까
categories:
  - Spring
tags: 
permalink: /web/spring/객체지향 프로그래밍이란
toc: true
toc_sticky: true
date: 2024-02-01
last_modified_at: 2024-03-10
---

# 객체 지향 프로그래밍
---
객체 지향 프로그래밍이란 컴퓨터 프로그램을 **객체들의 모임**으로 파악하고자 하는 것이다. 객체는 메시지를 주고받고, 데이터를 처리할 수 있다. 이러한 객체 지향 프로그래밍을 잘 사용할 경우 프로그램을 **유연**하고 **변경이 용이**하게 만들 수 있다.

### 객체 지향 특징
+ 추상화
+ 캡슐화
+ 상속
+ **다형성**

### 좋은 객체 지향 설계의 5가지 원칙
+ SRP(single responsibility principle) : 단일 책임 원칙
+ OCP (Open/closed principle) : 개방-폐쇄 원칙
+ LSP(Liskov substitution principle) : 리스코프 치환 원칙 
+ ISP(Interface segregation principle) : 인터페이스 분리 원칙 
+ DIP(Dependency inversion principle) : 의존관계 역전 원칙 

## 자바 언어의 다형성
---
다형성이란 어떤 기능 (ex. 로그인 검증 방법) 을 변경할 때 쉽게 변경할 수 있도록 하는 것이다.  
**오버라이딩**을 생각하면 된다.

### 오버라이딩 예시

![DI-1-1.png]({{site.url}}\assets\images\why_use_DI\DI-1-1.jpg)
만약 ``MemberService``클래스에 직접적으로 ``MemoryMemberRepository, JdbcMemberRepository`` 중 하나가 구현되어 있다면 ``MemberService``안의 ``Repository``코드를 다 수정해야 변경이 가능할 것이다.  

하지만 ``MemberService`` 클래스가 ``MemberRepository`` 인터페이스를 통해 ``save``기능을 호출할 경우 오버라이딩을 통해 실제 구현체인 ``MemoryMemberRepository, JdbcMemberRepository`` 둘 중 하나의 ``save``함수가 실행될 것이다.  

코드를 통해 예시를 들면 다음과 같다.  

![DI-1-2.png]({{site.url}}\assets\images\why_use_DI\DI-1-2.jpg)

``MemberService`` 안에 ``MemberRepository`` 인터페이스를 선언하고, 그 구현체를 바꿔주는 것만으로 ``Repository``를 바꿔서 사용할 수 있게 된다.

하지만 여기에는 2가지 문제점이 존재하는데, OCP, DIP원칙을 위배한다는 것이다.  
+ DIP 위반 : ``MemberService``는 ``MemberRepository``를 의존하지만 ``new``를 통해 구현체를 생성하기 때문에 구현체에도 의존을 하고 있다.
+ OCP 위반 : 구현체를 변경하기 위해서는 ``MemberRepository``의 코드 변경이 불가피하다.

따라서 다형성을 사용하는 것은 의존성과 코드 변경을 최소화로 한 것일 뿐, 의존성과 코드 변경은 불가피하다.  

스프링에서는 이를 해결하기 위해 나온 것이 ``DI(Dependency Inejction)``이다. ``DI``를 사용할 경우 다형성, OCP, DIP를 지키는 객체지향 설계가 가능해진다.