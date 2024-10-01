---
title: 멀티 모듈 적용기
excerpt: 
categories:
  - tripot
tags: 
permalink: /project/tripot/multiModule
toc: true
toc_sticky: true
date: 2024-09-29
last_modified_at: 2024-09-29
---
아래 내용은 사이드 프로젝트(tripot)를 진행하면서 겪은 문제를 정리한 것입니다.  

---
새로운 프로젝트를 시작하며 멀티 모듈을 적용하고자 한다.

---
## 멀티모듈 이란

모듈은 독립적으로 배포될 수 있는 코드 단위를 말하며, 하나의 프로젝트에서 여러 모듈을 사용하는 것을 멀티모듈이라고 한다.  

멀티모듈의 장점
+ 코드의 중복을 피할 수 있다.
+ 유지보수가 쉽다.

---
## 멀티모듈을 굳이?

진행하는 프로젝트의 규모가 작기 때문에 멀티모듈을 사용하는 것이 맞는가? 라는 의문이 들긴 하지만 적용했을 때와 안 했을 때의 차이점을 느끼기 위해서 적용해 보기로 했다.  

---
## 멀티모듈 구조

멀티모듈 구조는 크게 3가지로 나눴다.  `api, domain, common` 3개로 나눴고, 추후에 `외부 api, batch` 서버를 추가할 예정이다.

### api 모듈

api와 관련된 모듈이다.  
+ controller
+ service

### domain 모듈
도메인 및 그와 관련된 것들을 모은 모듈이다.
+ domain
+ dto
+ repository

### common 모듈
공통으로 사용될 것들을 모은 모듈이다.
+ exception 

---
### 의존성 관리

멀티모듈을 사용하는 방법은 여러 글에서 소개하고 있기 때문에 자세히 작성하지는 않지만, 몇 가지 주의할 점만 간단히 작성하겠다.  

현재 모듈의 상태는 `api 모듈`을 통해서 웹서버가 동작하고, 엔티티나 DB에 관한 설정은 `domain모듈`에 작성이 되어있는 상태이기 때문에 `api 모듈`에서 `domain모듈` 관련 의존성을 알 수 있도록 따로 설정을 해줘야 한다.  
#### dependency 의존성 설정

`api 모듈`에서 `domain 모듈` 의 dependency를 의존하고 싶은 경우 `api 모듈`의 `build.gradle`에 다음 한 줄을 추가하면 된다.  

`implementation project(':{domain 모듈 이름}')`

#### entity 의존성 설정

`api 모듈` 이 실행될 때 빈, 설정정보, 엔티티 등을 스캔할 때 따로 설정한게 없다면 `api 모듈` 내에서만 스캔을 진행하게 된다. 따라서 스캔 범위를 지정해줘야 한다.  

``` java
@Configuration  
@EntityScan(basePackages = {"com.junior.domain"})  
@EnableJpaRepositories(basePackages = "com.junior.repository")  
public class ScanConfig {  
}
```

`@EntityScan`의 경우 `api 모듈` 외의 패키지에서 Entity로 등록할 범위를 지정할 때 사용하는 것으로 `basePackeges` 로 지정한 패키지 및 그 하위까지 전부 조회하게 된다.  

`@EnableJpaRepositories` 는 외부 패키지에서 JPA Repository 빈을 활성화하는 어노테이션이다.  

#### 설정 정보 의존성 설정

데이터베이스 연결 정보라던가 JPA 설정의 경우 `domain 모듈`의 `application.yml`에 저장이 되어있을 텐데 `api 모듈`은 이 설정 정보를 모르기 때문에 외부에서 참고할 수 있도록 설정을 해줘야 한다.  

``` java
@SpringBootApplication  
public class ModuleApiApplication {  
  
    public static void main(String[] args) {  
        System.setProperty("spring.config.name", "application-domain");  
        SpringApplication.run(ModuleApiApplication.class, args);  
    }  
}
```


위처럼 `System.setProperty("spring.config.name", "application-domain")`한 줄을 작성하면 외부 설정 정보를 참고할 수 있다.  

이 때 모듈마다 서로 다른 이름을 지정해줘야 원하는 설정 정보를 참조할 수 있으며 모듈마다 yml 파일 이름이 똑같을 경우 다른 설정 정보를 참조하는 버그가 발생할 수 있다고 한다.  

---
