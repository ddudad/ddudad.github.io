---
title: "@Builder default 값 지정 방법 2"
excerpt: "@Builder.Default"
categories:
  - gugumo
tags: 
permalink: /project/gugumo/Builder-default-value-2
toc: true
toc_sticky: true
date: 2024-05-12
last_modified_at: 2024-06-16
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

**2024/06/16 내용 추가**
## Builder 옵션 사용

`Builder` 어노테이션 옵션 중에서 `builderClassName, builderMethodName` 옵션이 있다.  

+ `builderClassName` : builder 패턴을 사용해서 반환되는 클래스의 이름을 변경할 수 있다.  
+ `builderMethodName` : builder 패턴을 사용할 때 기본으로 `build()`를 사용해서 시작하는데, 이 함수 명을 변경할 수 있다.  

``` java
@Builder(builderClassName = "UserJoin", builderMethodName = "userJoin")  
public Member(Long id) {  
    this.id = id;  
    this.role = MemberRole.ROLE_USER;  
    this.status = MemberStatus.active;  
}
```

위와 같은 코드처럼 `id`값은 외부에서 받아와서 설정하지만, `role, status` 는 특정 값을 직접 설정해서 `userJoin`을 통해 구현할 때는 기본 값을 설정할 수 있다.  

사용법
``` java
id = 1;
Member joinMember = Member.userJoin()  
        .id(id)
        .build();
```

`builderClassName, builderMethodName`을 통해서 builder를 여러 개 설정할 수 있다.  

---

처음 게시글을 작성할 때 더 좋은 방법이 있다면 추가하겠다고 했었는데 이제서야 추가했다.  
그래도 더 좋은 방법을 알게 되어서 다행이다.  





