---
emoji: 🖥
title: Dispatcher Servlet 알아보기
date: '2021-06-03 23:00:00'
author: 코다
tags: 웹 서블릿
categories: 웹
---

## Servlet 개념 및 구조

Servlet은 **웹 서버**를 구현한 **자바**의 프로그램이며 **interface**이다. 서블릿이 하는 일은 다음과 같다. Servlet은 웹 클라이언트로부터 요청을 받아서 응답을 반환한다. 

Servlet 인터페이스는 servlet을 초기화하고, 서비스를 요청하고, servlet을 서버에서 제거하는 메소드를 제공한다. (이걸 life-cycle 메소드라고 말한다) 

1. `init` 메소드를 통해서 서블릿이 구축된다. 
2. 클라이언트에서 호출된 `service` 메소드가 수행된다. 
3. 수행된 서블릿이 `service`에서 제거되고 `destroy` 메소드를 통해서 소멸된다. 

추가로 Servlet 초기세팅 정보를 `ServletConfig` 에 담아서 반환하는 `getServletConfig()` 와, Servlet 정보를 반환하는 `getServletInfo()` 메소드도 존재한다. 

### HttpServlet 구조

- `GenericServlet`을 확장하고 `Servlet` 인터페이스를 구현한다.
- 웹 환경에 최적화되어 있어서 HTTP 메소드를 지원한다. 즉, HttpServlet 에서는 `service()` 를 override 할 이유가 거의 없다. 왜냐햐면 이미 정의되어 있는 Http 요청들을 수행하도록 되어 있기 때문이다.

<p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/135119857-ab8335e6-7bbf-409f-9213-76beb4941e03.png"></p>

### Servlet 동작방식

1. 브라우저에서 URL을 입력해서 request를 Servlet Container 로 전송
2. Servlet Container에서 `HttpServletRequest`, `HttpServletResponse` 객체 생성
3. `xml`에 기입된 정보를 통해서 해당 URL과 매칭되는 서블릿을 검색
4. 해당 서블릿의 service를 호출하여 처리
5. 해당 서블릿의 service 내의 메소드에서 처리하고 동적 페이지를 생성하여 `HttpServletResponse` 에 응답을 담아서 전송
6. 모든 처리가 끝난 후 `HttpServletRequest`, `HttpServletResponse` 소멸

<br>

## Servlet Container

서블릿을 관리하기 위해서는 서블릿 컨테이너가 필요하다. <br>

톰캣(Tomcat)과 같이 클라이언트의 요청을 받아서 정의된 서블릿을 수행하고, 응답을 할 수 있도록 웹 서버와 소켓 통신을 관리한다. <br>

본래 클라이언트와 웹서버만 통신했을 때에는 정적 페이지만 전송할 수 이기때문에 비효율적이다. 따라서 동적으로 웹 페이지를 서버에서 만들어서 전송할 수 있도록 하는 것이 servlet container이다. 웹 서버에서 클라이언트와 서블릿이 소통할 수 있도록하는 일부분이다. <br>

웹 서버는 http 요청을 받은 경우 servlet container로 해당 요청을 포워딩한다. 이때 부터 서블릿 컨테이너가 요청을 받아서 생명주기에 맞게 핸들링 한다. <br>

### 역할

1. 통신 지원 (웹 서버 ↔  소켓)

    서버와 소켓 통신을 하기 위해서는 소켓 생성 및 listen, accept 등의 일을 해야하지만 해당 과정은 복잡하고 동일하게 반복해야하는 부분이다. 따라서 서블릿 컨테이너는 해당 일과 관련된 기능을 API로 제공**(?)** 해줘서 관리하게 해준다. 즉, 비지니스 로직에만 집중할 수 있도록 지원한다. 

2. 서블릿 생명주기 관리

    서블릿의 탄생 및 죽음을 관리한다. 

    즉, 요청이 들어왔을 때, 해당되는 서블릿 클래스를 인스턴스화(`HttpServletRequest`, `HttpServletResponse` 등등) 하고, 서비스 메소드 호출, 처리 후 GC를 진행한다.

3. 멀티 스레드 지원 및 관리

    하나의 서블릿 요청 당 하나의 자바 스레드가 생성된다. 서비스 메소드 실행 후 스레드는 소멸된다. 그리고 서버에서 다중 스레드를 관리해준다. (하나의 소켓 당 하나의 스레드가 할당되기 때문에? 그렇다면 소켓 관리를 톰캣이 한다는 것이기 때문에)

4. 선언적 보안 관리

    서블릿 컨테이너를 사용할 경우, 보안 내용을 xml에 기록하므로 서블릿이나 자바에 구현하지 않아도 된다. 

### Servlet 생명주기

간단하게 서블릿의 생명주기는 `init() → service() → destroy()` 로 진행된다.

- 여기서 `init()`은 요청이 왔을 때, 서블릿이 메모리에 있는지 확인하고 없는 경우 `init()` 을 실행하고 처음에 한번만 실행한다. 서블릿 요청별로 새로운 스레드가 생성되기 때문에 해당 스레드간 공통적인 부분은 여기에 구현하는 것이 좋다.
- 서블릿 컨테이너가 서블릿 종료 요청(***종료 요청 시점은? 우선은 톰캣 종료 시점***)을 할 때 호출되는 `destroy()` 도 마찬가지로 마지막에 한번만 실행된다.

### Servlet 과 Jvm

Servlet을 사용하면 jvm에서 각각의 요청들을 각각의 자바 스레드에서 사용할 수 있도록 해준다. jvm에서 각각의 servlet은 하나의 자바 클래스이고, servlet에 요청이 들어오면 그것을 Jvm에서 처리해서 반환한다. 

<br>

## Dispatcher Servlet

디스패쳐 서블릿을 이해하려면 우선 front controller 패턴에 대한 이해가 있어야 한다. Front controller 패턴([https://www.geeksforgeeks.org/front-controller-design-pattern/](https://www.geeksforgeeks.org/front-controller-design-pattern/)) 에 대해서 간단하게 설명하자면, front-controller는 들어오는 요청에 대해서 하나의 핸들러가 처리를 담당하고 그것을 처리할 수 있는 적합한 핸들러에게 dispatch 즉, 보내는 역할을 하도록 하는 디자인 패턴이다. 

### Dispatcher servlet processing

- 여러 Web-context가 존재할 수 있기 때문에 우선 해당 DispatcherServlet에 해당되는 WebApplicatonContext 을 DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE 를 키로 우선 검색한다. 
- `DispatcherServlet`은 `HandlerAdapter` 의 구현체를 `getHandle()` 을 통해서 구현체를 가져온다. 그래서 `handle()` 메서드를 통해서 해당 요청에 대해 처리할 것을 진행한다.
- **HandlerExceptionResolver -**  WebapplicationContext 안에 선언되어 있는, 요청 처리 중 발생한 exceptions 들을 처리하는 resolver

### HandlerAdapter Interfaces

HandlerAdapter 인터페이스는 controller, servlets, HttpRequests를 관리하여 사용하게 한다. <br>

HandlerAdapter 구현체는 디스패처의 `getHandler()` 메서드를 통해서 `HandlerExecutionChain` 에 들어간다. 여기에 들어간 각각의 구현체들이 `handle()` 메서드를 통해서 `HttpServletRequest` 요청을 처리한다. 

- **Mapping**

    HandlerMapping 인터페이스는 컨트롤러와 밀접하게 연관이 되어 있다. 때문에 컨트롤러에 어떠한 annotation이 붙어있는지에 따라서 HandlerMapping을 다르게 동작한다. 

    `SimpleControllerHandlerAdapter` 의 경우에는 `@Controller` 어노테이션이 붙어있지 않은 컨트롤러의 경우에 동작할 수 있고, `RequestMappingHandler` 은 `@RequestMapping` 어노테이션이 붙은 메서드의 경우에만 적용할 수 있다. 

    `RequestMappingHandler`의 경우를 자세히 들여다보자. @RequestMapping 어노테이션은 해당 `WebApplicationContext` 내에서 handler가 가능한 지점을 알려준다. 그렇기 때문에 `@RequestMapping` 어노테이션에 서술되어 있는 path는 `HandlerMapping` 인터페이스에 의해서 관리된다. (URL 구조는 `DispatcherServlet`과 관련이 깊으며, servlet mapping에 직접 사용된다) 

- **HTTP Request Handling**

    DispatcherServlet 의 핵심적인 책임 중 하나는 들어오는 HttpRequest를 알맞은  handler에 보내는(dispatch) 것이다. (여기서 `@Controller`, `@RestController` 어노테이션에 관련이 있다) 

- **ViewResolver Interface**

    `ViewResolver` 는 `DispatcherServlet`에서 `ApplicationContext`에 대한 환경설정을 담당한다. <br>

    `ViewResolver`는 `dispatcher`에 의해서 제공되는 view의 종류와 어디서 해당 view가 제공되는지 파악한다.  <br>

    `ViewResolver`가 실제적으로 수행하는 일은 다음과 같다. 

    1. prefix 설정 : 적절한 view를 찾기위한 default URL 경로를 설정
    2. suffix 설정 : default view type을 설정하는 접미사 설정
    3. view class 설정 : 적절한 view 클래스를 resolver에 설정해서 렌더링이 필요한 기술을 제공받을 수 있도록 한다. (JSTL or Tiles) 

- **LocaleResolver Interface**

    디스패처로 session, request, cookie information 등을 LocaleResolver로 커스텀 할 수 있다. 

    - `CookieLocaleResolver`는 쿠키를 통해 stateless한 어플리케이션의 property를 설정할 수 있도록 한다.
    - `SessionLocalResolver`로 stateful한 어플리케이션의 session-specific  configuration을 설정할 수 있다.

    ```java
    @Bean 
    public CookieLocaleResolver cookieResolver() {
    	CookieLocaleResolver localeResolver
    		= new CookieLocaleResolver();
    	localeResolver.setDefaultLocale(Locale.ENGLISH);
    	localeResolver.setCookieName("cookiename");
    	localeResolver.setCookieMaxAge(3600);
    	return localeResolver;
    }
    ```

- **ThemeResolver & MultipartResolver 생략**

- **HandlerExceptionResolver**

    스프링의 `HandlerExceptionResolver`는 전체 웹 어플리케이션에 대해서 균일한 에러 핸들링을 가능하게 해준다. 어플리케이션 전역적으로 적용이 되는 커스텀 에러 핸들링을 구현하기 위해서는 **@ControllerAdvice 어노테이션을 추가해서 구현하도록 한다.** 

    ```java
    @ControllerAdvice
    public class ExampleGlobalExceptionHandler {
    	 
    	@ExceptionHandler
    	@ResponseBody
    	public String handleExampleException(Exception e) {
    		//...
    	}
    }
    ```

    이 경우 `@ExceptionHandler` 어노테이션이 추가된 클래스는 디스패쳐의 영역 안에 있는 모든 컨트롤러에 적용될 수 있다. 

### Spring MVC 와 DispatcherServlet 동작방식

<p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/135120169-742f3adf-be12-4f14-8d35-6c18c4115e06.png"></p>

<br>
<br>

### 파생 키워드 및 주제

**⇒ 서블릿 컨테이너의 웹 서버와 통신 지원 방식 (Tomcat 내부구현 확인 필요)** 

**⇒ Front Controller desgin pattern (**[https://www.geeksforgeeks.org/front-controller-design-pattern/](https://www.geeksforgeeks.org/front-controller-design-pattern/))

**⇒ SimpleControllerHandlerAdapter 와 annotation이 붙지 않은 controller의 동작 방식** 

**⇒ SpringController 깊이 알기** ([https://www.baeldung.com/spring-controllers](https://www.baeldung.com/spring-controllers))

**⇒ Spring의 error handling 깊이 알기** ([https://www.baeldung.com/exception-handling-for-rest-with-spring](https://www.baeldung.com/exception-handling-for-rest-with-spring))

<br>

### 의문

- ~~Spring MVC 의 DispatcherServlet의 동작방식에서 **HandlerMapping**과 **HandlerAdapter**의 차이~~

<br>

### 참고링크

- [https://mangkyu.tistory.com/14](https://mangkyu.tistory.com/14)
- [https://dzone.com/articles/what-servlet-container](https://dzone.com/articles/what-servlet-container)
- [https://www.baeldung.com/spring-dispatcherservlet](https://www.baeldung.com/spring-dispatcherservlet)
- [https://velog.io/@ehdrms2034/스프링-MVC-Dispatcher-Servlet을-직접-구현해보자](https://velog.io/@ehdrms2034/%EC%8A%A4%ED%94%84%EB%A7%81-MVC-Dispatcher-Servlet%EC%9D%84-%EC%A7%81%EC%A0%91-%EA%B5%AC%ED%98%84%ED%95%B4%EB%B3%B4%EC%9E%90)

```toc
```