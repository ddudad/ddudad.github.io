---
title: spring boot 프로젝트 ec2 자동 배포
excerpt: 
categories:
  - gugumo
tags: 
permalink: /project/gugumo/auto-publish-springboot-to-ec2
toc: true
toc_sticky: true
date: 2024-05-18
last_modified_at: 2024-05-18
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

## 기존 어플리케이션 배포 방법

기존에는 release를 하는 브랜치를 내 로컬 컴퓨터에 받아와서 직접 빌드한 다음에 생성된 jar파일을 ftp를 이용해서 배포 서버(ec2)로 옮긴 다음에 특정 명령어로 직접 실행을 시켜줬다.  

하지만 매번 코드를 받아서 빌드하고 배포작업을 거치는 것은 생산성 저하도 있고 매우 귀찮은 일이기에 배포 자동화를 도전해봤다.  

---
## 배포 자동화 과정

배포 자동화에는 다음과 같은 서비스를 이용한다.  

+ Github Action
+ AWS S3
+ AWS CodeDeploy
+ AWS ec2

### Github Action

어떤 브랜치에 push, pull request 를 요청하는 등의 특정 이벤트가 발생할 때 동작하는 기능들을 정의할 수 있으며 프로젝트를 빌드하는 CI 기능으로도 사용할 수 있다.  
### AWS S3

특정 브랜치의 코드 및 빌드 결과물을 올리는 저장소 역할을 한다.  
### CodeDeploy

소스코드를 운영환경에 자동 배포하는 역할을 수행하는 AWS Service로 ec2, ECS, Lambda 등 여러 배포 대상이 있다.  
### AWS ec2

소스코드가 실질적으로 돌아가는 공간


---

## AWS 배포 방법

배포 자동화는 [게시 글](https://velog.io/@juhyeon1114/%EC%8B%A4%EC%A0%84-Github-actions-AWS-Code-deploy%EB%A1%9C-Spring-boot-%EB%B0%B0%ED%8F%AC-%EC%9E%90%EB%8F%99%ED%99%94%ED%95%98%EA%B8%B0#-appspecyml-%EC%9E%91%EC%84%B1) 을 참고하여 완성했습니다.  이 글에서는 예외적으로 발생한 에러에 대해서 작성하겠습니다.  

--- 

## Missing credentials 에러

```
InstanceAgent::Plugins::CodeDeployPlugin::CommandPoller: Missing credentials - please check if this instance was started with an IAM instance profil
```

CodeDeploy 에서 배포가 실패하는 문제가 있었는데 에러를 확인해보니 다음과 같은 에러가 있었다. 에러 확인 방법은 [공식문서](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/troubleshooting-deployments.html#troubleshooting-agent-commandpoller-error) 를 참고하면 알 수 있습니다.  

에러 원인을 찾아보니 ec2가 동작하고 있는 상태에서 IAM, CodeDeploy를 설정하니 ec2에 제대로 적용이 안 되는 게 원인인 것 같았다.

### 해결법

+ CodeDeploy Agent 다시 실행
```
sudo service codedeploy-agent restart
```

+ ec2 재부팅
  ec2 console에서 사용하는 ec2 우클릭->인스턴스 재부팅
  
![1.png]({{site.url}}\assets\images\posts_img\auto-publish-springboot-to-ec2\1.png)


---

## build 결과가 2개인 이슈

ec2에서 build 결과를 확인해보니 2개의 jar파일이 있어 찾아보았다.  

spring boot gradle 플러그인 2.5 버전 부터 gradle 빌드 시 2개의 JAR 파일이 생성된다고 한다.  

+ {프로젝트 이름}-plain.jar
+ {프로젝트 이름}.jar

### plain archive

plain이 붙은 jar파일을 `plain archive` 라고 하며 gradle의 `jar` 를 통해 생성된다.  

plain archive는 프로젝트 실행에 필요한 의존성을 모두 포함하지 않으며 작성한 소스코드와 리소스 파일만 포함한다.  

모든 의존성을 포함한 것이 아니기 때문에 일반적인 실행 방법(`java -jar`)을 통해 실행이 불가능하다.  

### Executable Archive

plain이 없는 archive는 `executable archive` 라고 하며 gradle의 `bootJar` 를 통해 생성된다.  

execute archive는 모든 의존성 및 소스코드, 리소스를 포함한다.  

모든 의존성을 포함하기 때문에 `java -jar`를 통한 실행이 가능하다.  

```
소스코드와 리소스만 포함한 plain arhcive는 필요없기 때문에 빌드 과정에서 제외할 것이다.
```

---

## plain archive 생성되지 않도록 설정하기

`build.gradle` 파일에 다음 문구를 추가하면 plain archive 생성을 방지할 수 있다.  

```
jar {
	enabled = false
}
```

