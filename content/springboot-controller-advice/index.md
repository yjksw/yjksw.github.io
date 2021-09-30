---
emoji: 🖥
title: "@ControllerAdvice 알아보기"
date: '2021-06-19 23:00:00'
author: 코다
tags: 스프링부트
categories: 스프링부트
---

- `@ControllerAdvice`를 통해서 어플리케이션 전역적으로 exception을 핸들링 할 수 있다. 다르게 표현하면 `@RequestMapping` 메서드에서 던져지는 exceptions들의 interceptor라고 할 수 있다. (shared across multiple @Controller classes)
- 주로 `@ControllerAdvice`에서 전역적으로 처리하고 싶은 어노테이션은 `@ExceptionHandler`, `@InitBinder`, `@ModelAttribute` 등이 있다.
- `ResponseEntityExceptionHandler` 클래스가 `@ControllerAdvice` 어노테이션에서 전역적인 exception handling 을 구현할 수 있도록 하는 base class이다. 해당 클래스에서 Spring MVC 내부에서 발생한 예외들을 처리할 수 있는 메서드들을 제공한다. (`DefaultHandlerExceptionResolver`는 `ModelAndView`를 반환하는 반면 `ResponseEntityExceptionHandler`는 `ResponseEntity`를 반환한다)

## 여러 @ControllerAdvice 간의 우선순위

@ControllerAdvice 클래스들은 Bean으로 등록이 되도록 하는데, 해당 빈들은 `Ordered` 인터페이스를 구현하여 orderable 한 속성을 부여하거나, `@Order`/ `@Priority`를 사용해서 우선순위를 부여할 수 있다. (여기서 `Ordered` semantic이 `@Order`/ `@Priority` 에 우선순위를 가진다) 

- 예외를 처리하는 경우
    
    매칭이 되는 exception handler method가 있는 가장 처음 매칭되는 advice의 `@ExceptionHandler`가 실행된다. 
    
- model attribute와 data binding 초기화 경우
    
    `@ModelAttribute`와 `@InitBinder` 메소드가 `ControllerAdvice`의 우선순위 순서에 따라서 실행된다.  
    
- `@ControlerAdvice`의 우선순위에 따른 ExceptionHandler 선언 팁

    <p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/135494123-1441631a-7277-4b6f-b82e-30610bcf4b5e.png"></p>
    

기본적으로 `@ControllerAdvice`는 모든 controller에 전역적으로 적용이 되기 때문에, 더 구체적인 controller에 적용하기 위해서는 selectors를 사용해야한다. (`annotations()` , `basePackageClasses()` , `basePackages()`)만일 여러 selectors가 있으면 OR 로 적용이되면 이 selectors 체크는 runtime에 실행이 되므로, 만일 너무 많은 selectors를 사용하면 런타임 퍼포먼스 효율이 떨어지게 된다.  

<br>
<br>

**[참고자료]**

- [https://zetcode.com/springboot/controlleradvice/](https://zetcode.com/springboot/controlleradvice/)
- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html)

```toc
```