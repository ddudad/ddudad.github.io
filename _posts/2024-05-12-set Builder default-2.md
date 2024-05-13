---
title: "@Builder default 값 지정 방법"
excerpt: "@Builder.Default"
categories:
  - gugumo
tags: 
permalink: /project/gugumo/Builder-default-value-2
toc: true
toc_sticky: true
date: 2024-05-12
last_modified_at: 2024-05-12
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

## 상황

이전 게시글에서 Builder패턴 중 기본 값을 지정하는 여러 가지 방법을 찾아봤는데, 기본 값을 상황에 따라 바꿀 수 없는 문제가 있었다. 이 글에서는 그것을 해결한 방법을 작성해 보겠다.  


---
## Builder 패턴 직접 구현

``` java 
public static UserBuilder createUserBuilder() {  
    return new UserBuilder();  
}

public static class UserBuilder {  
    private String username;  
    private String password;  
    private String nickname;  
  
    public UserBuilder username(String username) {  
        this.username = username;  
        return this;  
    }  
  
    public UserBuilder password(String password) {  
        this.password = password;  
        return this;  
    }  
  
    public UserBuilder nickname(String nickname) {  
        this.nickname = nickname;  
        return this;  
    }  
  
    public Member build() {  
        return new Member(this.username, this.password, this.nickname, "/default", MemberStatus.active, MemberRole.ROLE_USER);  
    }  
}
```

코드를 보면 `UserBuilder` 는 `username, password, nickname` 3개를 입력으로 받아 `Member` 인스턴스를 반환하게 되다.  

``` java
return new Member(this.username, this.password, this.nickname, "/default", MemberStatus.active, MemberRole.ROLE_USER);  
```

`build` 메소드에서는 `Member` 인스턴스를 반환하게 되는데, `username, password, nickname`은 입력 받은 값을 사용하고, 기본으로 지정할 값 `/dafault, MemberStatus.active, MemberRole.ROLE_USER` 는 직접 지정해줘서 그 값을 통해 `Member` 를 생성하였다.  

설명에서는 `User` 만 예시로 들었지만, 기본으로 지정하는 값을 바꿔줄 경우 `Admin, Manager` 등 여러 역할의 `Member`를 Builder패턴을 이용하여 생성할 수 있다.  

--- 


여러 자료를 찾아보고 구현한 방식이지만 더 좋은 방법이 있을 것이고 그 방법을 찾을 경우 수정할 예정이다.  





