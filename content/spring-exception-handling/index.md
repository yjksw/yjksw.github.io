---
emoji: 🖥
title: "스프링부트의 Exception handling"
date: '2021-06-19 23:00:00'
author: 코다
tags: 스프링부트 스프링 웹 
categories: 스프링부트 스프링 웹
---

## Spring Boot

Spring boot 자체에서 핸들링 되지 않은 error 에 대한 대비책을 마련해 두었다. 

- 먼저, Spring boot 자체에서 `/error` 에 대한 매핑을 찾아서 해당 URL에 대해서 동일한 이름을 가진 `error` 뷰를 매핑 한다. 해당 뷰는 `error.html` 을 반환한다. (해당 뷰는  Thymeleaf template인데, 만일 JSP를 사용한다면 `error.jsp`를 반환하도록 `InternalResourceViewResolver`에서 변경할 수 있다) 실질적인 매핑은 ViewResolver에서 담당한다.
- 만일 `/error`에 대해 그 어떠한 view-resolver도 매핑이 되어 있지 않다면 spring boot는 내부적으로 가지고 있는 대체 에러 페이지인 "Whitelabel Error Page"를 가지고 있다.
    
    이때 만일 RESTful request에 대한 응답이라면 Spring boot는 자체적인 JSON 형태로 "Whitelabel Error Page"의 응답을 받은 error 정보를 반환한다.
    
    ```java
    {"timestamp":"2018-04-11T05:56:03.845+0000","status":404,"error":"Not Found","message":"No message available","path":"/no-such-page"}
    ```
    
- Spring boot는 컨테이너에 대한 디폴트 error-page 또한 구축해놓았다. 만일 예외가 Spring MVC 밖에서 발생했더라도(ex. servlet Filter) 여전히 Spring Boot에 대비되어 있는 error page에 리포트 되어 반환된다.

<br>

## Exception Handling 동작원리

<br>

### HandlerExceptionResolver(interface)

`DispatcherServlet`의 application context에 선언되어 있는 Spring bean의 예외들은(Spring MVC system 내에서 발생한 경우들) `HandlerExceptionResolver`을 구현하여 intercept를 통해 핸들링 된다. 즉, Controller에서 에외를 핸들링하지 않는다. 

```java
public interface HandlerExceptionResolver {
    ModelAndView resolveException(HttpServletRequest request, 
            HttpServletResponse response, Object handler, Exception ex);
}
```

위 코드에서 `handler` 는 해당 예외가 발생한 controller를 말한다. 

MVC는 기본적은 3가지 resolvers를 생성한다. 

1. `ExceptionHandlerExceptionResolver` 
    
    예외들을 적합한 `@ExceptionHander` 어노테이션이 붙은 메소드에 매칭한다. handler(controller)와 controller-advice 두가지 모두 고려한다. 
    
2. `ResponseStatusExceptionResolver`
    
    처리되지 않은 예외들 중 `@ResponseStatus`가 붙은 것들을 확인한다.
    
3. `DefaultHandlerExceptionResolver` 
    
    기본 Spring Exceptions를 Http Status Codes로 변경하는 작업을 한다.(MVC 내부적인 부분) 
    

위 3가지 resolvers들은 체이닝 되어 위 순서대로 처리되는데 Spring이 내부적으로 해당 일을 처리하는 `HandlerExceptionResolverComposite` 빈을 생성하여 처리하도록 한다. 

<br>

### SimpleMappingExceptionResolver

Spring은 자체적으로 위 `HandlerExceptionResolver` 구현체인 `SimpleMappingExceptionResolver`를 제공하고 대부분의 어플리케이션에서 사용되고 있다. 해당 리졸버에서는 다음과 같은 것들을 결정하여 처리할 수 있다. 

- Exception 클래스를 뷰에 매핑
- 아무 곳에서도 처리되지 않은 예외에 대한 기본(default) error page를 설정
- Model에 추가되는 `exception` 속성의 이름을 설정한다.(View에서 추후 사용할 수 있도록) 기본값은 "exception" 이다. `@ExceptionHandler`에서 반환되는 뷰는 exception 자체에 대한 접근 권한이 없지만 `SimpleMappingExceptionResolver`에 정의되어 있는 뷰는 접근 권한이 있으므로 여기서 설정하도록 되어있다.

```java
@Configuration
@EnableWebMvc  // Optionally setup Spring MVC defaults (if you aren't using
               // Spring Boot & haven't specified @EnableWebMvc elsewhere)
public class MvcConfiguration extends WebMvcConfigurerAdapter {
  @Bean(name="simpleMappingExceptionResolver")
  public SimpleMappingExceptionResolver
                  createSimpleMappingExceptionResolver() {
    SimpleMappingExceptionResolver r =
                new SimpleMappingExceptionResolver();

    Properties mappings = new Properties();
    mappings.setProperty("DatabaseException", "databaseError");
    mappings.setProperty("InvalidCreditCardException", "creditCardError");

    r.setExceptionMappings(mappings);  // None by default
    r.setDefaultErrorView("error");    // No default
    r.setExceptionAttribute("ex");     // Default is "exception"
    r.setWarnLogCategory("example.MvcLogger");     // No default
    return r;
  }
  ...
}
```

위에서 `defaultErrorView` 속성으로 처리되지 않은 예외들에 대한 적합한 error 페이지를 설정할 수 있으므로 유용하다. 현재 대부분의 어플리케이션에 기본값은 java stack-trace를 노출하는 것인데 유저들은 stack-trace에 관심이 없다. 따라서 Spring boot는 해당 부분을 "white-label" 에러 페이지로 처리한다. 

<br>

### Extending SimpleMappingExceptionResolver

다음과 같은 이유로 `SimpleMappingExceptionResolver`를 확장하여 사용한다.

- Constructor를 사용하여 속성들 직접 지정하는 경우
    
    예를 들어서 exception 관련 로깅 및 로거를 설정
    
- default log message를 오버라이딩하여 변경하는 경우 (`buildLogMessage`를 변경)
    
    현재 기본 메세지는 *Handler execution resulted in exception* 이다. 
    
- 에러 view에 추가 정보를 설정하고 싶은 경우
    
    `doResolverException`을 오버라이딩 하여 설정 
    

```java
public class MyMappingExceptionResolver extends SimpleMappingExceptionResolver {
  public MyMappingExceptionResolver() {
    // Enable logging by providing the name of the logger to use
    setWarnLogCategory(MyMappingExceptionResolver.class.getName());
  }

  @Override
  public String buildLogMessage(Exception e, HttpServletRequest req) {
    return "MVC exception: " + e.getLocalizedMessage();
  }
    
  @Override
  protected ModelAndView doResolveException(HttpServletRequest req,
        HttpServletResponse resp, Object handler, Exception ex) {
    // Call super method to get the ModelAndView
    ModelAndView mav = super.doResolveException(req, resp, handler, ex);
        
    // Make the full URL available to the view - note ModelAndView uses
    // addObject() but Model uses addAttribute(). They work the same. 
    mav.addObject("url", request.getRequestURL());
    return mav;
  }
}
```

<br>
<br>

**[참고자료]**
- [https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#errors-and-rest](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#errors-and-rest)

```toc
```