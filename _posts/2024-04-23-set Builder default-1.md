---
title: "@Builder default 값 지정 방법 1"
excerpt: "@Builder.Default"
categories:
  - gugumo
tags: 
permalink: /project/gugumo/Builder-default-value-1
toc: true
toc_sticky: true
date: 2024-04-23
last_modified_at: 2024-05-12
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

## 상황

`Member` 클래스의 인스턴스를 `@Builder` 를 통해 생성할 때 특정 필드는 default값으로 초기화하는 게 좋을 것 같다는 생각이 들어서 여러 방법을 시도해 봤다.


---
## `@Builder.Default` 어노테이션

`@Builder` 어노테이션에서 `Default` 옵션을 사용하면 0이나 `null`이 아닌 원하는 값으로 초기화가 가능하다.  하지만 이 옵션을 사용하기 위해서는 클래스 레벨에서 `@Builder` 어노테이션을 사용해야 한다.  

``` java
@Builder  
@NoArgsConstructor(access = AccessLevel.PROTECTED)   
public class Member {  
  
    @Id @GeneratedValue  
    private Long id;  
    private String email;  
    private String password;  
    private String nickname;
	@Builder.Default
    private String profileImagePath = "/{default Path}";  
}
```

다음 코드와 같이 클래스 레벨에서 `@Builder`를 사용하고, 컴파일하려고 보니 다음과 같은 컴파일 에러가 발생했다.  

```
Lombok @Builder needs a proper constructor for this class
```

찾아보니 클래스 레벨에서 `@Builder`를 사용하려면 모든 필드를 포함하는 생성자가 있어야 한다는 소리였다.  

아니 이게 무슨 소리인가. 외부에서 생성자를 통해 생성하는 것을 막기 위해 최소한의 생성자(`NoArgsConstructor`) 만 생성했고(JPA에서 기본 생성자를 필요로 하기 때문), 그것도 `Protected`로 선언해서 외부에서 선언이 불가능하도록 구현했는데 모든 필드를 포함하는 생성자를 만들어야 한다니..  

그래서 꼼수를 부려보고자 몇 가지 방법을 생각해 봤다.  

### 일부 필드만 가지는 생성자에 `@Build` 어노테이션 사용하기

``` java
@Builder(builderMethodName = "createUser")  
public Member(String email, String password, String nickname) {  
    this.email = email;  
    this.password = password;  
    this.nickname = nickname;
    this.profileImagePath = "/{default path}";
}
```

혹시나 이렇게 작성하면 기본 값이 초기화가 될까 싶어서 돌려봤지만, `profileImagePath` 에는 `null` 로 초기화가 되었다.  

**실패**
### 모든 필드를 가지는 생성자를 `privated`로 생성하기

`@AllArgsConstructor(access = AccessLevel.PROTECTED)` 를 통해 접근은 `Protected`인 모든 필드를 가지는 생성자를 구현했다.  

``` java
@Builder  
@AllArgsConstructor(access = AccessLevel.PROTECTED)
@NoArgsConstructor(access = AccessLevel.PROTECTED)   
public class Member {  
  
    @Id @GeneratedValue  
    private Long id;  
    private String email;  
    private String password;  
    private String nickname;
	@Builder.Default
    private String profileImagePath = "/{default Path}";  
}
```

이렇게 하니 외부에서는 생성자를 통한 생성이 불가능하면서 Builder패턴을 사용했을 때 `profileImagePath` 필드에 원하는 default값이 들어가는 것을 확인할 수 있었다.  

**성공**

---

`Member` 클래스는 주로 `USER` 권한을 가지는 인스턴스를 생성하기 때문에 권한, 상태 유무 등 바뀌지 않는 값들을 default로 설정하고, `ADMIN`, `MANAGER` 등 다른 권한으로 바꾸는 경우는 `Update`메소드를 통해 바꾸려고 했었다.  

하지만 구현해보고 나니 `Member` 클래스가 `USER`를 위한 클래스라는 느낌이 강해졌고, `Member` 클래스가 복잡해지는 느낌을 받았기 때문에 일단은 외부에서 권한, 상태 등을 넣어주는 방식으로 사용해야 겠다.  







