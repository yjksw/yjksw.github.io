---
emoji: 🍀
title: Bean Scope 종류 알아보기 
date: '2021-05-30 23:00:00'
author: 코다
tags: 스프링 웹
categories: 스프링 웹
---

스프링 프레임워크에서 사용되는 Bean scope에 6가지 종류가 있다. 일반적으로 많이 쓰이는 scope은 싱글톤이다. 

## Singleton scope

스프링 빈이 singleton scope을 가지고 있다면, 컨테이너가 빈의 단 하나의 인스턴스를 해당 빈이 필요할 때마다 캐싱된 빈을 리턴한다. 빈 객체를 수정하면 해당 빈을 참조하고 있는 모든 곳에 반영이 된다. 싱글톤 스콥은 스프링의 기본값이다. 

## Prototype scope

 프로토타입 스콥은 빈 요청이 있을때마다 매번 다른 인스턴스를 컨테이너로부터 반환한다. 설정방법은 다음과 같다. 

```java
@Bean
@Scope("prototype")
public Persion personPrototype() {
	return new Person();
}
```

## Web Aware Scopes

앞에 두 개의 범위를 제외하고 4개의 범위가 더 존재한다. 하지만 조건이 있는데, web-aware application 맥락에서만 적용이 될 수 있는 범위이다. 

### Request Scope

Request scope은 하나의 HTTP request 당 하나의 빈을 생성한다. 

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator requestScopedBean() {
	return new HelloMessageGenerator();
}
```

이 scope을 사용할 때는 proxyMode 속성이 필수로 필요하다. 왜냐하면 웹 어플리케이션 context가 시작하는 그 순간에는 활성화 되어 있는 request 가 없기 때문에 proxy 객체가 필요하다. 스프링이 시작되었을 때 프록시 객체가 주입된다. 그리고 해당 빈이 필요한 request가 왔을 때 대상 bean을 초기화하고 주입한다. 

더 간단하게는 아래와 같이 표현이 가능하다. 

```java
@Bean
@RequestScope
public HelloMessageGenerator requestScopedBean() {
	return new HelloMessageGenerator();
}
```

이제 컨트롤러에서 requestScopedBean을 정의해서 사용할 수 있다. 

```java
@Controller
public class ScopesController {
	@Resource(name = "requestScopedBean")
	HelloMessageGenerator requestScopedBean;
	
	@RequestMapping("/scopes/request")
	public String getRequestScopeMessage(final Model model) {
		model.addAttribute("previousMessage", requestScopedBean.getMessage()); //이 메세지가 항상 null이 된다.
		requestScopedBean.setMessage("Good Morning!");
		model.addAttribute("currentMessage", requestScopedBean.getMessage());
		return "scopeExample";
	}
}
```

### Session Scope

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator sessionScopedBean() {
    return new HelloMessageGenerator();
}
```

세션의 생명주기와 같은 범위로 bean이 생성된다. 

### Application Scope

`Application scope`은 `ServletContext`의 생명주기와 동일하게 빈 생명주기가 결정된다. 그렇기 때문에 어떻게 보면 `singleton scope` 과 굉장히 비슷하지만 bean 의 관점에서 보면 중요한 차이가 있다.

만일 `application scope`로 되어 있다면, 동일한 bean 인스턴스가 동일한 `ServletContext` 공유하는 여러 servlet-based 어플리케이션에서 활용된다. 하지만 만일, `singleton scope`이라면 하나의 어플리케이션에서 싱글톤 인스턴스로 관리된다. 

### WebSocket Scope

`WebSocket scope`을 사용하면 Bean들이 WebSocket session 속성에 저장이 된다. 따라서 동일한 WebSocket session에 빈 요청이 있을 때, 같은 bean 인스턴스를 반환한다. 

참고자료: [https://www.baeldung.com/spring-bean-scopes](https://www.baeldung.com/spring-bean-scopes)

[https://docs.spring.io/spring-framework/docs/3.0.0.M3/reference/html/ch04s04.html#beans-factory-scopes-prototype](https://docs.spring.io/spring-framework/docs/3.0.0.M3/reference/html/ch04s04.html#beans-factory-scopes-prototype)

```toc
```