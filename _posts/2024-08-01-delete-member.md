---
title: 회원 탈퇴 관리 방법
excerpt: 
categories:
  - gugumo
tags: 
permalink: /project/gugumo/deleteMember
toc: true
toc_sticky: true
date: 2024-08-01
last_modified_at: 2024-08-01
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

회원관리에서 탈퇴는 어떻게 처리하는지 알아보겠다.  

---

### 회원 탈퇴시 고려해야할 점

처음 회원 탈퇴를 진행할 때 처음에는 Member의 상태를 `active, inactive, delete` 로 지정할 수 있게 구현하고, 탈퇴를 진행할 경우 상태를 `delete`로 업데이트하는 방식을 사용하였다.  

하지만 다른 컬럼의 데이터(이메일 정보, 닉네임 등)가 남아있다보니  회원 관련 기능을 개발할 때마다 상태를 체크하는 기능이 들어가야하는 문제가 있어 수정할 필요가 있었다.  

처음에는 데이터베이스에서 완전 삭제하는 방식으로 구현을 했는데, 회원이다보니 다른 테이블이 회원 정보를 FK로 가지고 있거나 하는 등 여러 테이블과 관계가 있기 때문에 회원 데이터 하나만 삭제하는 것은 사실상 불가능하다.  

그렇다고 관련된 데이터를 모두 삭제하는 것 또한 무리가 있다. 일단 요청이 엄청 많아지는 문제도 있고, 매우 비효율적이기 때문이다.  

---

### 해결책

매우 간단하면서 쉽게 해결할 수 있는 방법이 있는데, 모든 컬럼을 `null`로 바꾸는 것이다.  

어떤 기능이든 개발할 때 `null`예외처리는 꼭 해주기 때문에 다른 테이블을 건드는 일 없이 깔끔하게 회원 데이터를 삭제할 수 있다.  

``` java
public void deleteMember() {  
  
    this.status = MemberStatus.delete;  
  
    this.nickname = null;  
    this.profileImagePath = null;  
    this.role = null;  
    this.isAgreeTermsUse = null;  
    this.isAgreeCollectingUsingPersonalInformation = null;  
    this.isAgreeMarketing = null;  
    this.favoriteSports = null;  
    this.isEmailLogin = null;  
    this.username = null;  
    this.password = null;  
    this.isKakaoLogin = null;  
    this.kakaoId = null;  
    this.kakaoNickname = null;  
}
```

`Memer` 클래스에 다음과 같이 모든 데이터를 `null`로 처리하는 함수를 하나 작성해 두고 혹시 몰라서 `status`만 `delete`로 남겨두었다.  

이러면 동일한 정보로 다시 가입을 하더라도 중복되는 이슈 없이 새롭게 가입할 수 있다.  





