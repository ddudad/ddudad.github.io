---
title: 싱글톤 패턴이란
excerpt: 싱글톤 패턴에 대해서 알아보자
categories:
  - Spring
tags: 
permalink: /web/spring/싱글톤 패턴이란
toc: true
toc_sticky: true
date: 2024-02-05
last_modified_at: 2024-03-16
---
아래 내용은 김영한님의 강의를 들으며 중요하다고 생각되는 부분들을 정리한 내용입니다.  더 자세한 내용이 궁금하다면 아래 강의를 들어보시는 것을 추천드립니다.  
[스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)  

---

## 싱글톤 패턴

### 싱글톤 패턴을 사용하는 이유

![1.png]({{site.url}}\assets\images\posts_img\singleton\1.png)

우리의 웹 애플리케이션 서버가 있다고 할 때 사용자는 특정 기능을 하기 위해서 서버에 요청을 보낼 것이다.(그 요청을 실행할 수 있는 class는 ``memberService``라고 하자)  

이 때 싱글톤이 보장되지 않는다면 우리의 서버는 요청이 들어올 때마다 ``memberService``의 인스턴스를 생성하고 필요한 기능을 실행한 후 인스턴스는 소멸될 것이다. 이러한 방법은 사용자가 적다면 아무런 문제가 없겠지만 사용자가 매우 많아 요청이 많아진다면 어떻게 될까?  

동시에 수만건의 요청이 들어온다고 하면 인스턴스또한 수만개가 생성이 될 것이고, 이는 하드웨어에 부담이 될 것이다.(메모리에 생성하고 GC를 통해 지우고.. 등등)  

싱글톤이란 특정 클래스의 인스턴스가 하나만 생성되고 그 인스턴스를 계속해서 사용하는 것을 의미힌다.

### 싱글톤 패턴 예제

``` java
public class SingletonService { 
	private static final SingletonService instance = new SingletonService(); 
	
	public static SingletonService getInstance() { return instance; } 
	
	private SingletonService() { 
	} 
```

싱글톤을 코드로 구현하면 위 코드처럼 볼 수 있는데, 여기서 봐야 할 부분은 3가지가 있다.  
+  ``instance``를 ``static``으로 선언했기 때문에 클래스 영역에서 메모리가 생성
+ ``getInstance``를 통해 ``static``으로 생성한 ``instance``를 반환
+ 기본 생성자가 ``private``이기 때문에 외부에서 기본 생성자를 사용할 수 없는 점

``SingletonService``는 기본생성자를 통해 생성할 수 없고 ``getInstance``를 통해 인스턴스를 받아와서 사용해야 한다.  

![2.png]({{site.url}}\assets\images\posts_img\singleton\2.png)

싱글톤을 적용한 모습이다.  
사용자들은 요청을 하고 서버는 동일한 ``memberService``인스턴스를 반환하면서 같은 인스턴스가 수만 개 생성되는 문제를 해결한 것을 볼 수 있다.

### 싱글톤 패턴 단점

하지만 싱글톤을 사용하는 데에는 몇가지 문제점이 있다.  

+ 싱글톤을 유지하기 위해 들어가는 코드가 많다.
+ 의존관계상 클라이언트가 구체 클래스에 의존하게 된다.(DIP 위반)
	+  클라이언트는 구체 클래스의 ``getInstance``를 직접 호출해서 사용하고 있으니 의존하고 있다.
+ 클라이언트가 구체 클래스에 의존하기 때문에 OCP원칙을 위반할 가능성이 높다.


**참고할 점**
``memberService``는 ``static``으로 생성된 인스턴스이기 때문에 클라이언트의 상태를 저장하는 필드가 있을 경우 모든 클라이언트가 그 상태를 공유하는 문제가 발생한다.  

만약 ``memberService``에 클라이언트의 ID를 저장하는 필드가 있고 조회하는 기능이 있을 때, 클라이언트A의 ID를 ``memberService``에 저장했다고 하자.  
클라이언트B가 ID를 조회하는 기능을 실행할 경우 클라이언트A의 ID가 조회될 것이다.  

싱글톤을 사용할 때에는 이러한 문제를 예방하기 위해서 클라이언트가 직접 값을 변경할 수 있는 필드를 사용하지 않고, 공유되지 않는 지역변수, 파라미터를 통해 받는 방법, ThreadLocal 등을 사용한다.

몇가지 이유가 더 있지만, DIP, OCP를 위반한다는 점에서 좋지 않다는 것은 분명하다.

---

싱글톤이 필요한 이유와 싱글톤을 적용했을 때 나타나는 단점들에 대해서 알아봤다.  
스프링에서는 싱글톤의 단점을 해결하면서 싱글톤의 장점을 얻을 수 있는데 어떻게 그럴 수 있는지 다음에 알아보겠다.
  









