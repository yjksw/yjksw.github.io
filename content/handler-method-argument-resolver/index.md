---
emoji: 🍀
title: HandlerMethodArgumentResolver 내부동작 원리 알아보기
date: '2021-06-25 23:00:00'
author: 코다
tags: 스프링 웹
categories: 스프링 웹
---

## INTRO

- `HandlerMethodArgumentResolver`는 Spring framework에서 제공하는 인터페이스로 request에서 메소드의 parameters를 해당하는 인자값으로 변환 혹은 바인딩 하는 resolver이다.

<br>

## 인터페이스 내용

`HandelrMethodArgumentResolver`에는 두가지 메소드가 있다. 

1. `supportsParameter()`
2. `resolveArgument()`

<br>

## 첫번째, Parameter가 해당 resolver를 지원하는 여부 확인

```java
supportsParameter(MethodParameter parameter)
```

**[참고]** 아래 설명은 `@RequestBody`가 붙은 인자의 경우를 보는 것이므로 그 구현체가 `RequestResponseBodyMethodProcess.java`의 예시로 설명한 것이다. <br>

- Parameter가 있는 수만큼 `InvocableHanderMethod#InvokeForRequest()` → `getMethodArgumentValues()` 안에서 for문을 돌면서 해당 parameter에 대한 argument resolve를 한다. 이때 resolve를 하기 위해서 현재 클래스가 가지고 있는 `resolvers` 가 해당 parameter 지원 하는지 여부를 확인한다.
- → 확인하는 로직은 `HandlerMethodArgumentResolverComposite` 안에 있는 resolver들의 배열을 돌면서(한번 찾으면 캐싱함) 해당 parameter를 지원하는 resolver를 찾아서 반환하고, null인지 여부를 체크해 boolean을 반환한다. 이때 `supportsParameter()` 메소드가 수행된다.
- → 다시 `InvocableHanderMethod#InvokeForRequest()` → `getMethodArgumentValues()` 의 반복문 안에서  `HandlerMethodArgumentResolver#resolveArgument()` 가 실행이 되면서(어노테이션에 따라서 구현체가 각기 따로 있음 ex. `RequestResponseBodyMethodProcess` 등등) controller 메서드에서 사용되는 인자를 배열에 저장한다.

<br>

## 두번째, 해당 parameter를 argument value로 변환 및 바인딩

```java
resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory)
```

Method parameter를 argument value로 변환 및 바인딩 하는 역할을 한다. 

- `ModelAndViewContainer` 는 request의 model에 접근할 수 있도록 한다.
- `WebDataBinderFactory`는 `WebDataBinder` 인스턴스를 추출해 데이터 바인딩이나 타입 변환을 지원한다.

성공 시 argument value를 리턴하고, 없다면 `null`을 리턴한다. 

만일 `@Valid` 어노테이션이 해당 인자에 붙어있다면 해당하는 validation도 이 메소드에서 처리한다. 

<br>

## Custom Resolver 구현

1. `HandlerMethodArgumentResolver` 인터페이스를 구현하는 custom resovler 클래스를 구현
2. `@Override` 한 `supportParameter()` 메소드에서 원하는 타입의 인자인 경우 `true`를 반환하도록 구현
3. `@Override`한 `resolveArgument()`에서 controller의 메서드 인자로 보면 값을 반환하도록 구현 
4. `WebMvcConfiguration` 를 구현하는 클래스의 `addArgumentResolver()`라는 메소드를 override하여 위 구현한 custom resolver를 추가해준다. 

<br>
<br>

**[참고링크]**

- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html)
- [https://enai.tistory.com/31](https://enai.tistory.com/31)

<br>

**[유용링크]**

- 스프링에서 요청을 처리하는 과정
    [https://m.blog.naver.com/PostView.nhn?blogId=duco777&logNo=220605479481&proxyReferer=https:%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=duco777&logNo=220605479481&proxyReferer=https:%2F%2Fwww.google.com%2F)


```toc
```
