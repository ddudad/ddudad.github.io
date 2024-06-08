---
title: postgresql 사용 방법
excerpt: 
categories:
  - gugumo
tags: 
permalink: /project/gugumo/howToEditDatabase
toc: true
toc_sticky: true
date: 2024-06-08
last_modified_at: 2024-06-08
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

로컬에서는 `applicaiton.yml`에서 `jpa.hibernate.ddl-auto` 설정을 `create` 로 했기에 데이터가 다 삭제되긴 했지만 엔티티가 변경되거나 추가되는 경우에 따로 무언가를 작업할 필요가 없었다.  

하지만 운영하는 서버의 경우 데이터가 날아가면 큰일이기 때문에 `none`으로 설정하고 있는데, 이렇게 하면 데이터베이스의 변경이 있을 경우 직접 수정해줘야 한다.  

그래서 postgresql 사용법에 대해 정리하고자 한다.  

---

### DB 모드 접속
```
sudo -u postgres psql
```

### 데이터베이스 리스트 확인
```
\l
```

### 특정 데이터베이스 선택
```
\c {데이터베이스 이름}
```

### 컬럼 추가
```
ALTER TABLE {테이블명} ADD COLUMN {컬럼명} {데이터타입} {제약조건};

ex)
ALTER TABLE TEST ADD COLUMN tel varchar(11) NOT NULL;
```

### 컬럼명 변경
```
ALTER TABLE {테이블명} RENAME COLUMN {컬럼명} TO {변경할컬럼명}; 
```

### 주의할 점

1. 데이터베이스는 모든 이름을 스네이크식으로 변경하기 때문에 그에 맞춰서 추가해줘야 한다.
	만약 변수명을 `isCheck` 같이 했을 경우 `is_check`로 등록을 해야된다.
2. 테이블을 추가할 때 테이블 뿐만 아니라 시퀀스도 추가해줘야 한다.

--- 
### 테이블 추가할 시 약간의 편법

로컬에서 `jpa.hibernate.ddl-auto` 옵션을 `create`로 하고 `jpa.properties.hibernate.format_sql` 옵션을 `true`로 해주면 JPA가 자동으로 해주는 쿼리문을 볼 수 있는데, 이 쿼리문을 복사해서 추가하면 테이블을 추가할 수 있다.  

