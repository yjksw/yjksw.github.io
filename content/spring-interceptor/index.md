---
emoji: 🍀
title: 초간단 Interceptor 알아보기 
date: '2021-06-23 23:00:00'
author: 코다
tags: 스프링 웹
categories: 스프링 웹
---

## 개요

Handler interceptors는 어떤 요청들에 대한 특정 기능을 적용하고 싶을 때 사용이 되는데, 특히 어떤 조건 및 원칙들을 검증하는데 많이 사용된다. 

<br>

## Interceptor 구성

Interceptor를 구현하기 위해서는 HandlerInterceptor를 구현해야 한다. 해당 인터페이스에는 interceptor가 실행되는 3가지 경우에 대한 메소드가 정의되어 있다. 

1. handler가 실행되기 이전
2. handler가 실행된 이후
3. 전체 요청 처리가 모두 수행된 이후

이것들 중 handler 실행 이전에 수행되는 메소드인 `preHandle()` 은 boolean 값을 반환한다. `postHandle()`과 `afterCompletion()`은 void를 반환한다. 

위 세가지 메소드 모두 공통된 인자로 Servlet에 의해서 생성된 `HttpServletRequest`, `HttpServletResponse`, `handler`(Object 타입)을 받는다. 따라서 void 반환타입인 경우 HttpServletResponse에 후처리를 할 수 있다. (`postHandle()`의 경우에는 `ModelAndView`를 `afterCompletion()`은 `Exception`을 `@Nullable` 속성으로 받는다)  

### preHandle() 동작원리

`DispatcherServlet`은 `interceptors`와 `handler`를 execution chain으로 실행한다. (마지막에 handler가 실행이 되는 형태) 

이 `preHandle()` 메소드를 통해서 이 execution chain 중단 여부를 결정할 수 있다. 만일 체이닝 되어 있는 interceptor가 true를 반환한다면 그 다음 interceptor 혹은 handler를 실행한다. 만일 false를 반환한다면 체이닝 된 interceptor 혹은 handler 실행을 멈추고 **DispatcherServlet은 해당 interceptor가 알아서 response에 필요한 처리를 했다고 간주한다.** 

<br>
<br>

**[참고링크]**

- [https://docs.spring.io/spring-framework/docs/3.0.x/reference/mvc.html#mvc-handlermapping-interceptor](https://docs.spring.io/spring-framework/docs/3.0.x/reference/mvc.html#mvc-handlermapping-interceptor)

```toc
```