---
title: JPA 영속성 컨텍스트란
except: JPA 영속성 컨텍스트에 대해서 알아보자
categories: 
tags: 
permalink: 
toc: true
toc_sticky: true
date: 2024-03-17
last_modified_at: 2024-03-17
---
아래 내용은 김영한님의 강의를 들으며 중요하다고 생각되는 부분들을 정리한 내용입니다.  더 자세한 내용이 궁금하다면 아래 강의를 들어보시는 것을 추천드립니다.  
[자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)

---

## JPA 영속성 컨텍스트

ORM(Object Relational Mapping)은 객체와 데이터베이스 테이블의 매핑을 통해 엔티티 클래스의 정보를 데이터 테이블로 저장시키는 기술이다.  
JPA에서는 영속성 컨텍스트를 통해 엔티티 클래스를 관리하고, 데이터베이스 접근을 제어한다.(JPA에서 EntityManager를 통해 영속성 컨텍스트로 접근한다.)


### 엔티티 생명주기

+ 비영속  
  영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
  
  ``` java
  Member member = new Member();
  member.setId(1L);
  member.setName("memberA");
```

+ 영속  
  영속성 컨텍스트로 관리되는 상태
  
  ``` java
  EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
  EntityManager em = emf.createEntityManager();
  
  em.getTransaction().begin();
  em.persist(member);
```
  
  ``em.persist(member)``를 통해 영속성 컨텍스트에 추가했다.
  
    
  ``` java
  em.find(Member.class, 1L);
```
  
  ``em.find``를 통해 DB에서 가져온 엔티티도 영속성 컨텍스트로 관리된다.
  
+ 준영속  
  영속성 컨텍스트에 저장되었다가 **분리**된 상태  
  
  ``` java
  em.detach(member);
```
  
  단일 엔티티에 한해서 분리하는 것
  
+ 삭제  
  영속성 컨텍스트에서 삭제된 상태
  
  ``` java
  em.remove(member);
```
  
  영속성 컨텍스트 내 모든 엔티티를 삭제
  