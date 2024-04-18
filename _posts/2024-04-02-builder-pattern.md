---
title: Builder pattern
except: 빌더 패턴을 알아보자
categories:
  - 생성 패턴
tags: 
permalink: /Design Pattern/Creational/Builder pattern
toc: true
toc_sticky: true
date: 2024-04-13
last_modified_at: 2024-04-13
---
아래 내용은 디자인패턴에 대해서 공부한 내용을 정리한 것입니다.

---

## Builder 패턴

Bulider Pattern은 생성관련 패턴으로 어떤 클래스의 인스턴트를 생성할 때에는 여러 가지 방법이 있다.

### 생성자를 통한 생성

``` java
public class Member {
	private final String  name;
	private final int age;
	private final String email;

	public Member(String name, int age, String email) {
		this.name = name;
		this.age = age;
		this.email = email;
	}
}
```

``` java
Member member = new Member("이름", 10, "name.email.com")
```

생성자를 통해 인스턴스를 생성하는 방법이다.  

#### 단점
+ 생성자 생성할 때 매개변수의 순서가 중요한다.
+ 매개변수가 달라질 경우 그에 맞는 생성자를 구현해야 한다.
---
### setter 를 통한 생성

``` java
public class Member {
	private final String  name;
	private final int age;
	private final String email;

	public Member() {}

	public String setName(String name) {
		this.name = name;
	}

	public String setAge(int age) {
		this.age = age;
	}

	public String setEmail(String email) {
		this.email = email;
	}
}
```

``` java
Member member = new Member();
member.setName("이름");
member.setAge(10);
member.setEmail("name.email.com");
```

setter를 통한 인스턴스 생성 방법이라고 표현했지만, 인스턴스 생성은 기본 생성자를 통해 생성하고 setter를 통해 값을 넣는 방법이다.  

#### 단점
+ 필요한 매개변수가 있을 경우 setter를 추가적으로 작성해야 한다.
+ 어디서든 setter 호출을 통해 값을 변경할 수 있어 불변성을 보장하지 못한다.
---
### Builder 패턴을 통한 생성

``` java
public class Member {
	private final String  name;
	private final int age;
	private final String email;

	public Member(Builder builder) {
		this.name = builder.name;
		this.age = builder.age;
		this.email = builder.email;
	}

	public static class Builder {
		private String name;
		private int age;
		private String email;

		public Builder name(String name) {
			this.name = name;
			return this;
		}

		public Builder age(int age) {
			this.age = age;
			return this;
		}

		public Builder email(String email) {
			this.email = email;
			return this;
		}

		public Member build() {
			return new Member(this);
		}
	}
}
```

``` java
Member member = new Builder()  
        .name("name")  
        .age(10)  
        .email("email")  
        .build();
```

builder 패턴을 이용해서 인스턴스를 생성하는 방법이다.

#### 장점
+ 기본 생성자와 달리 순서에 영향을 받지 않는다.
+ 함수를 통해 생성하기 때문에 어떤 값이 들어가고 어떤 값이 빠졌는지 알아채기 쉽다.
+ 데이터의 불변성을 보장받을 수 있다.

#### 단점
+ setter와 마찬가지로 매개변수가 추가되면 코드를 작성해야 한다.

builder패턴을 사용할 때 특정 필드를 초기화하지 안을 경우 ``null``값이 들어가기 때문에 이 점을 유의하면서 사용해야겠다.


### 번외
lombok 라이브러리에서는 @Builder 어노테이션을 통해 Builder패턴을 지원한다.  

``` java
public class Member {
	private final String  name;
	private final int age;
	private final String email;

	@Builder
	public Member(String name, int age, String email) {
		this.name = name;
		this.age = age;
		this.email = email;
	}
}
```

---
