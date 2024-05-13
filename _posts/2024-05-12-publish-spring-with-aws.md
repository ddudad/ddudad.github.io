---
title: aws를 통한 spring 배포 방법 및 postgresql 설정 방법
excerpt: aws 배포 방법
categories:
  - gugumo
tags: 
permalink: /project/gugumo/publish-spring-with-aws
toc: true
toc_sticky: true
date: 2024-05-12
last_modified_at: 2024-05-12
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

## 어플리케이션 배포

우리의 프로젝트를 외부에서 접속하고 이용하기 위해서는 배포라는 작업을 해주어야 한다.  
내 컴퓨터를 직접 서버로 사용하고, 특정 포트를 허용하여 내 컴퓨터에 접속이 가능하도록 설정할 수도 있지만, 설정이 힘들기도 하고 컴퓨터 인터넷 성능이 좋지 않다면 서버도 불안정하기 때문에 보통은 aws나 cafe24 같은 서비스를 이용한다.   

---
## AWS 선택 이유

+ 다양한 참고 자료
+ 보안
+ 비용(사용한 리소스만 지불)

---

## AWS 배포 방법

aws를 이용한 배포는 [개발자 유미](https://youtu.be/fBzR6DqP5jk?si=xv3UQvstUv3CXIyw) 님의 유튜브를 보고 참고했습니다.  어려운 내용은 없으니 쉽게 참고할 수 있습니다.  

--- 

## postgresql 설치 및 적용

아래 작업은 ubuntu 22.04에서 진행했습니다.

1. postgresql 설치

``` 
sudo apt update
sudo apt install postgresql -y
```

	두 명령어를 통해 postgresql을 설치할 수 있다.  

```
psql --version
```

	그 후 postgresql 버전을 확인할 수 있다.

2.  테이블 생성

```
sudo -u postgres psql
```

명령어를 통해 postgresql 로 진입할 수 있으며 

```
CREATE DATABASE "테이블 이름"
```
+ JPA를 이용했기 때문에 필드를 따로 생성하지 않았지만 그렇지 않은 경우 필드를 생성해줘야 한다.

```
\l
```

명령어를 통해 테이블 리스트를 확인할 수 있다.

![2.png]({{site.url}}\assets\images\posts_img\publish-spring-project\1.png)


3. postgres 유저 설정
   
```
ALTER USER postgres PASSWORD '{password}';
```

	postgres 유저의 비밀번호 설정

## spring 설정

build.gradle
```
implementation 'org.postgresql:postgresql:42.6.0'
```


application.yml 
``` 
spring:
	datasource:
		url: jdbc:postgresql://localhost:5432/{table name}
		username: postgres
		password: {password}
		driver-class-name: org.postgresql.Driver
```

설정 후 실행하면 동작하는 것을 확인할 수 있다.

---

## 프로젝트 빌드

우측 gradle 메뉴를 클릭하면 아래와 같은 창을 볼 수 있다.  

![2.png]({{site.url}}\assets\images\posts_img\publish-spring-project\2.png)


`Tasks/build/bootJar` 를 더블클릭해서 Jar파일 빌드
+ 저장 위치는 `{project 경로}/build/libs` 에 저장된다.  

ftp 기능을 이용해 aws ec2 컴퓨터에 파일을 옮긴 후 실행하면 외부에서 접속할 수 있게 된다.

외부 접속 주소
```
{ec2 외부 IP}:8080
```