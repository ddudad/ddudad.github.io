---
title: Spring이 어떻게 의존성을 주입하는가-자동편
excerpt: Spring에서 Bean을 어떻게 등록하는지 알아보자
categories:
  - Spring
tags: 
permalink: /web/spring/Spring이 어떻게 의존성을 주입하는가 자동편
toc: true
toc_sticky: true
date: 2024-02-03
last_modified_at: 2024-03-10
---
아래 내용은 김영한님의 강의를 들으며 이해가 안되던 부분들을 정리한 내용입니다.  더 자세한 내용이 궁금하다면 아래 강의를 들어보시는 것을 추천드립니다.  
[스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)  

---

[DI(Dependency Injection)란](https://ddudad.github.io/web/spring/DI%EB%9E%80)에서 DI에 대해서 알아봤다.  
``@Autowired``어노테이션을 통해 의존성을 주입받을 수 있다고 했는데 Spring에서 어떤 방식으로 의존성을 주입하는지 알아보자.  

의존성 주입 방식에는 크게 수동, 자동 2가지 방식이 있는데 수동방식에 대해서 알아보겠다.

---
## 스프링 빈의 라이프사이클

의존성 주입 방식을 알아보기 전에 스프링 빈의 라이프사이클을 알아보자.  
```
스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸 전 콜백 -> 스프링 종료
``` 
다음과 같은 과정을 거치는데, 우리가 알아볼 것은 ``스프링 빈 생성 -> 의존관계 주입`` 부분이다.  

---
## @SpringBootApplication

``` java
@SpringBootApplication  
public class JpashopApplication {  
  
    public static void main(String[] args) {  
       SpringApplication.run(JpashopApplication.class, args);  
    }  
  
}
```

Spring 프로젝트를 실행시키는 함수를 보면 ``@SpringBootApplication`` 어노테이션이 있는 것을 볼 수 있고, 그 안을 확인해 보면

``` java 
@Target({ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@SpringBootConfiguration  
@EnableAutoConfiguration  
@ComponentScan(  
    excludeFilters = {@Filter(  
    type = FilterType.CUSTOM,  
    classes = {TypeExcludeFilter.class}  
), @Filter(  
    type = FilterType.CUSTOM,  
    classes = {AutoConfigurationExcludeFilter.class}  
)}  
)  
public @interface SpringBootApplication {  
    @AliasFor(annotation = EnableAutoConfiguration.class)  
    Class<?>[] exclude() default {};  
  
    @AliasFor(annotation = EnableAutoConfiguration.class)  
    String[] excludeName() default {};  
  
    @AliasFor(annotation = ComponentScan.class, attribute = "basePackages")  
    String[] scanBasePackages() default {};  
  
    @AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")  
    Class<?>[] scanBasePackageClasses() default {};  
  
    @AliasFor(annotation = ComponentScan.class, attribute = "nameGenerator")  
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;  
    @AliasFor(annotation = Configuration.class)  
    boolean proxyBeanMethods() default true;  
}
```

다음과 같이 구성되어 있고 ``@ComponentScan``이 있는 것을 확인할 수 있는데, Spring의 DI는 이 어노테이션으로 부터 시작된다.  



---

## @ComponentScan

``@ComponentScan``는 스캔할 수 있는 어노테이션이 붙은 클래스의 인스턴스를 생성한 다음 스프링 컨테이너에 추가하는 역할을 한다.  

``@ComponentScan``이 스캔할 수 있는 컴포넌트는 다음과 같다.
+ ``@Controller``
+ ``@Repository``
+ ``@Service``
+ ``@Configuration`` : 수동으로 빈을 등록할 때 사용하는 방식으로 따로 수동편에서 자세히 다루겠다.  

Spring이 컴포넌트를 스캔하고 자동으로 의존성을 주입하는 과정
1. 
   ![1.png]({{site.url}}\assets\images\posts_img\how-regist-spring-bean\1.png)
   빈 이름과 빈 객체를 스프링 빈 저장소에 저장하는 과정이 있는데 이 때 빈 이름은 클래스 명에서 맨 앞 글자만 소문자로 바꾼 이름이 들어가게 되고, 빈 객체는 싱글톤 방식으로 저장이 된다. (프록시 패턴을 통해 싱글톤으로 저장하며 싱글톤으로 저장하는 이유는 싱글톤 관련 글에서 알아보겠다.)
   
2. 
   ![2.png]({{site.url}}\assets\images\posts_img\how-regist-spring-bean\2.png)
   ``@Autowired``가 붙은 곳에서 필요한 인스턴스를 주입하며, 생성자에 파라미터가 많아도 다 찾아서 주입한다.

---

큰 범위에서 동작 방식을 설명한 것이며 세부적인 동작 방식 및 여러 옵션, 사용법은 추후 자세히 설명하겠다.