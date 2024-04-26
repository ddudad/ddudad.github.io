---
title: No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call 에러 해결
excerpt: gugumo 진행 중 에러 해결
categories:
  - gugumo
tags: 
permalink: /project/gugumo/테스트 코드의 중요성
toc: true
toc_sticky: true
date: 2024-04-18
last_modified_at: 2024-04-18
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

## 문제 발생

entity, repository 코드를 작성 후 테스트 코드를 실행할 때 다음과 같은 에러가 발생했다.

```
No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call
```

내용을 보니 스레드에서 이용 가능한 `EntityManager`가 없다고 하는 것 같다.  
구글에 검색해서 찾아보니 `@Transactional` 어노테이션을 작성하지 않아서 그런 것이었다.

---
## @Transactional

### Transaction

`@Transactional` 어노테이션을 알기 위해서는 Transaction의 개념을 알아야 한다.  
Transaction이란 여러 작업을 묶은 것으로 예시를 통해 알아보겠다.  

어떤 사람과 거래를 하고 있다고 하면  
+ 물건을 구매하기 위해 돈을 입금했다.
+ 판매자는 물건을 보낸다.
+ 구매자는 물건을 받았다.

Transaction 개념을 적용하지 않았을 경우
+ 구매자가 입금한 정보를 DB에 저장
+ 판매자가 보낸 물건 배송 정보를 DB에 저장
+ 구매자가 받았다는 정보를 DB에 저장

각각의 일이 끝날 때마다 바로 DB에 저장을 했을 것이다.  

근데 여기서 판매자가 사기를 치고 벽돌을 보냈다고 생각해보자.  
구매자는 입금을 취소하고 싶지만 이미 DB에 저장을 했기 때문에 취소하기 힘들 수 있다.

Transaction 개념을 적용한 경우
+ 구매자가 입금한 정보를 가지고 있는다. 
+ 판매자가 보낸 물건 배송 정보를 가지고 있는다.
+ 구매자가 받았다는 것을 가지고 있는다.
+ 가지고 있던 정보를 DB에 저장

동일하게 판매자가 사기를 쳐서 벽돌을 보냈다고 하면, 구매자는 입금 정보를 DB에 저장하지 않았기 때문에 작업을 취소할 수 있다.  

이처럼 Transaction은 포함하는 작업이 모두 성공할 시 작업 성공으로 보고 하나라도 실패하면 작업 실패로 본다.  

#### @Transactional 사용

``` java
@Repository  
@RequiredArgsConstructor  
@Transactional(readOnly = true)  
public class MemberRepository {  
  
    private final EntityManager em;  
  
    @Transactional  
    public void save(Member member) {  
        em.persist(member);  
    }
}
```

@Transactional은 기본 옵션이 `readOnly = false`이다.  
따라서 클래스 레벨에서는 `readOnly = true`를 주어 기본적으로 읽기 전용으로 만들고, 쓰는 기능이 필요할 경우 `@Transactional`옵션을 주는 방식으로 사용한다.  

클래스 레벨에서 `readOnly = true`로 설정하는 이유는 사용자도 모르게 값이 변경되는 것을 막기 위한 것이다. 

