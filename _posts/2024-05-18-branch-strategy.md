---
title: github 브랜치 전략
excerpt: github 브랜치 전략
categories:
  - gugumo
tags: 
permalink: /project/gugumo/branch-strategy
toc: true
toc_sticky: true
date: 2024-05-18
last_modified_at: 2024-05-18
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

## 현재 사용하고 있는 개발 방법

`develop` : 코드가 통합되는 브랜치
`branch1` : 팀원1이 개발하는 브랜치
`branch2` : 내가 사용하는 브랜치

개발자는 `branch1` , `branch2` 에서 개발을 하고 `develop` 에 Merge를 한 다음 다시 개인 브랜치로 fetch를 해서 개발했는데, 엄청 비효율적이고 잘못 덮어쓰기가 있을 수 있기 때문에 브랜치 전략을 이용하기로 했다. 

---

## 브랜치 전략이란

깃허브의 브랜치를 목적 별로 이용하여 코드를 좀 더 전략적으로 관리할 수 있는 기법이다.  

전통적으로 크게 2가지 방법이 있는 듯 하다.  
+ Git Flow
+ GitHub Flow

---

## Git Flow

![1.png]({{site.url}}\assets\images\posts_img\branch-strategy\1.png)

`feature` : 새로운 기능을 개발하는 브랜치
`develop` : 새로운 기능이 합쳐지는 브랜치
`release` : 다음 출시에 사용될 코드가 올라가 있는 브랜치
`hot fix` : `master` 브랜치에서 버그가 생길 경우 사용하는 브랜치
`master`   : 현재 출시되어 배포되고 있는 브랜치

Git Flow 개발 흐름

1. `develop` 브랜치에서  `feature` 브랜치를 생성하고 새로운 기능을 개발 후 `develop` 브랜치로 병합(Merge)한다. (일반적으로 Pull Request를 통해 팀원들의 리뷰를 받은 후 Merge 진행)
2. `develop` 브랜치가 목표한 개발이 완료가 된 경우 `release` 브랜치로 병합 후 배포 준비(이 때 발생한 버그는 `release` 브랜치에서 바로 반영)
3. `release` 브랜치가 목표한 개발을 완료한 경우 `master` 브랜치로 병합하여 제품 출시

Git Flow 전략은 복잡한 시스템이나 체계적인 관리가 필요한 경우에 사용하는 것으로 규모가 작거나 빠른 배포가 필요한 경우에는 적합하지 않다.  

---

## GitHub Flow

![2.png]({{site.url}}\assets\images\posts_img\branch-strategy\2.png)

`master` : 현재 출시되어 배포되고 있는 브랜치
`feature` : 새로운 기능을 개발하는 브랜치

Git Flow에 비하면 간단한 전략이다.  

GitHub Flow 개발 흐름
1. `feature` 브랜치를 통해 기능을 개발한다.
2. 기능 개발이 완료된 경우 `master` 브랜치로 Pull Request를 통해 팀원들의 리뷰를 받고 병합(Merge) 한다.  

GitHub Flow 전략은 Git Flow 전략보다 간단하기 때문에 빠르게 기능을 테스트하고 배포가 가능하나 충분한 테스트와  검증이 없기 때문에 버그의 위험성이 있다. 주로 작은 프로젝트에 적합한 전략이다.  

