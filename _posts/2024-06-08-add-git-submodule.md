---
title: git submodule 사용법
excerpt: 
categories:
  - gugumo
tags: 
permalink: /project/gugumo/addGitSubmodule
toc: true
toc_sticky: true
date: 2024-06-08
last_modified_at: 2024-06-08
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

프로젝트를 진행하면서 jwt 암호화 키, sns 로그인 키 등 외부로 노출되면 안되는 키들을 저장해야 하는 경우가 있다.  

이 때 사용할 수 있는 방법이 몇가지 있다.  

+ submodule
+ git ignore
+ git secret
+ 설정 파일 암호화
+ 외부 서비스

git ignore, git secret, submodule 중에서 하나를 선택하려고 했다.  

외부 서비스는 비용적인 문제가 있었고, 설정 파일 암호화는 매번 암호화하고 복호화하는 과정을 거쳐야 하기 때문에 선택지에서 제외했다.  

git ignore는 설정파일에 변동이 있을 경우 서버에 직접 업로드를 해야 하기 때문에 실수할 확률이 적지 않아서 제외했다.  

남은 것은 git secret, submodule 이었는데, submodule을 이용할 경우 설정 파일도 형상관리를 할 수 있겠다 싶어서 submodule을 적용하기로 했다.  

---
### submodule이란

다음 두 리포지토리가 있다고 가정하자.
+ public으로 공개된 리포지토리(open)
+ private으로 비공개된 리포지토리(close)

open 리포지토리에 노출되어서는 안되는 정보들이 있을 때 그 파일들은 open리포지토리에서 제외하고 close리포지토리에 추가한 후에 open리포지토리에서 close리포지토리를 참조하는 형식으로 사용하는 것이다.  

결론적으로 비밀스러운 내용들은 private 리포지토리에서 관리하고 그 외 내용들은 public 리포지토리에서 관리할 수 있다.  

---
### submodule 적용법

+ 비밀스러운 정보들은 private 리포지토리에 업로드한 후 public 리포지토리에서 파일 삭제
   ![]({{site.url}}\assets/images/posts_img/add-git-submodule/1.png)

+ public 프로젝트 최상위 경로에서 private 리포지토리를 submodule로 저장한다.
   + `git submodule add {private 리포지토리 주소} {원하는 경로}
	 
	 필자의 경우 config라는 이름으로 프로젝트 root 경로에서 진행하였다.  
		 `git submodule add https://github.com/gugumo-service/gugumo_properties.git config`
		![]({{site.url}}\assets/images/posts_img/add-git-submodule/3.png)
		 프로젝트에서 확인할 경우 폴더로 생셩된 것을 확인할 수 있을 것이다.
		 
	 submodule을 적용할 경우 `.gitmodules` 파일이 생기고 몇가지 정보가 저장된다.  
	   ![]({{site.url}}\assets/images/posts_img/add-git-submodule/2.png)
	  

+ config 아래의 application.yml 파일 복사하기
```
task copyGitSubmodule(type: Copy) {  
    from './config'  
    include '*.yml'  
    into './src/main/resources'  
}
```


build.gradle 아래에 다음 내용을 추가해준다.

---
### public 프로젝트 clone 후 submodule 적용법

submodule을 적용했다고 해서 public 프로젝트를 clone해도 자동으로 submodule 내용을 가져오지 않기에 다음 작업을 거쳐야 한다.  

+ `git clone {public 리포지토리 주소}`
+ `git submodule init`
+ `git submodule update`

이 작업을 거치면 `.gitmodules`에 저장된 경로, 리포지토리를 가져오며 private 리포지토리에 권한이 없을 경우 가져오지 못하게 된다.  

---
### submodule 특이점

submodule은 리포지토리의 HEAD를 참조하는 것이 아닌 commit을 참조하고 있기 떄문에 submodule 내용이 업데이트되어도 같은 commit정보만 가져오게 된다.  

이럴 때에는 다음 명령어를 통해 참조하는 commit을 업데이트할 수 있다.  
`git submodule update --remote`  

업데이트 이후에는 public 리포지토리에서 `commit, pull` 작업을 진행해야 다음에 public 리포지토리를 받고 submodule을 적용할 때 최신 내역으로 받을 수 있다.  


프로젝트를 진행 도중에 submodule에 저장한 내용이 바뀔 경우 해당 경로로 가서 commit, push를 진행해야 한다.  만일 그렇지 않을 경우 다른 사용자는 바뀌기 이전의 정보를 가져다가 사용하는 일이 발생할 수도 있다.  

---

### 트러블 슈팅 1

*fatal: 'src/main/resources' already exists in the index* 

submodule 적용을 이미 있는 폴더에 하는 경우 발생하는 에러로 다음 명령어를 통해 해결할 수 있다.  

1. 기존 폴더 삭제
2. `git ls-files --stage [지우고자하는 폴더경로]`
3. `git rm -r --cached [지우고자하는 폴더경로]`

기존 폴더를 삭제한 후에 캐시가 남아있기 때문에 2, 3번 명령어로 캐시까지 지워주면 된다.  

---

### 트러블 슈팅 2

```
repository '[https://github.com/gugumo-service/gugumo_properties.git/](https://github.com/gugumo-service/gugumo_properties.git/)' not found
```

private 리포지토리를 찾지 못하는 이슈

github action은 `ubuntu` 의 `cli` 환경에서 동작하게 되는데, github는 `cli`상에서 `ssh key` 를 통한 접근만을 허용한다. 따라서 `ssh key`를 등록해야 접근할 수 있다.  

```
    steps:
    - name: Checkout master branch
      uses: actions/checkout@v3
      with:
        token: ${{secrets.GUGUMO_PROPERTIES}}
        submodules: true
        ref: master
```

`ssh key` 생성 및 등록하는 방법은 구글에 검색하면 쉽게 찾을 수 있다.  

---
### 트러블 슈팅 3

```
Error starting ApplicationContext. To display the condition evaluation report re-run your application with 'debug' enabled.
```

github action을 통한 cicd를 사용 중이었는데 ec2에서 다음과 같은 에러가 발생하면서 spring이 동작하지 않는 이슈가 발생했다.  
위의 에러와 함께 데이터베이스도 local에서만 실행하는 h2로 동작하는 것을 확인했다.  


github action에서 빌드할 때 submodule을 복사하는 작업을 먼저 진행하도록 해서 해결

```
 - name: 프로젝트 빌드(테스트 코드 제외)
      run: |
        ./gradlew copyGitSubmodule
        ./gradlew clean build --exclude-task test
```

기존에는 `./gradlew copyGitSubmodule` 코드가 없었고, 빌드하면 알아서 되겠지 했는데 이렇게 수정하니 해결이 됐다.  
왜 따로 작업을 진행해야 제대로 되는지는 아직 모르겠다.  

---

### 특이점

public 리포지토리에 push할 시점에 등록했던 submodule 의 commit 정보만을 가져온다.  
한번 push한 코드는 properties 리포지토리를 아무리 바꿔도 최신 버전으로 가져오지를 않는다.  

매번 새로운 브랜치에서 작업을 진행하기 때문에 크게 문제가 되지는 않을 것 같지만 유의하면서 개발해야겠다.  







