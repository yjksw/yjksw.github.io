---
emoji: 🖥
title: "@ModelAttribute vs. @RequestBody 더 깊이 파헤치기"
date: '2021-06-19 23:00:00'
author: 코다
tags: 스프링부트
categories: 스프링부트
---

## @RequestBody

request body를 method argument로 바꿀 때 `HttpMessageConverter`를 사용한다.

- `HttpMessageConverter` 는 두가지를 담당한다. 첫번째는 Http request message를 객체로 변환하는 것, 두번째는 객체를 Http response body로 변환하는 작업이다.

### 동작원리

`DispatcherServlet`에 의해서 호출되는 handler의 method parameters은 스프링의 `HandlerMethodArgumentResolver`에 의해 생성이 되고, handler의 return value는 `HandlerMethodReturnValueHandler`에 의해서 처리된다. `@ResponseBody`와 `@RequestBody`를 다루는 구현체는 `RequestResponseBodyMethodProcess`이다. 

- DispatcherServlet의 handle에서부터 Argument resolve 하는 과정

    `DispatcherServlet#handle()` → `AbstractHandlerMethodAdapter#handle()` → `RequestMappingHandlerAdapter#handleInternal()` → `RequestMappingHandlerAdapter#invokeHandlerMethod()` → `ServletInvocableHandlerMethod#invokeAndHandle()` → `InvocableHandlerMethod#invokeForRequest()` → `InvocableHandlerMethod #getMethodArgumentValues()` → *(Interface)*`HandlerMethodArgumentResolver#resolveArgument()` → *(Imp)*`RequestResponseBodyMethodProcessor#resolveArgument()`

- `RequestResponseBodyMethodProcessor#resolveArgument()` 내부에서 HttpMessageConverter를 사용해서 변환시키는 과정

    `RequestResponseBodyMethodProcessor#resolveArgument()` → (e*xtends  AbstractMessageConverterMethodArgumentResolver*)`RequestResponseBodyMethodProcessor#readWithMessageConverters()` → (*Imp HandlerMethodArgumentResolver*)`AbstractMessageConverterMethodArgumentResolver#readWithMessageConverters()` 에서 변환 로직을 시행한다. 

이때 HttpMessageConverter default 인스턴스들은 `WebMvcConfigurationSupport#addDefaultHttpMessageConverters()` 를 통해 등록된다. 

### HttpMessageConverter 로직

- `@RequestBody` 일때

    `HttpMessageConverter#canRead()` 로 converter가 해당 content의 인스턴스를 읽고 생성할 수 있는지 확인한다. 이후에 argument를 생성하여 반환한다. 

- `@ResponseBody` 일때

    `HttpMessageConverter#canWrite()` 를 통해서 `HttpMessageConverter`가 해당 반환값을 serialize 할 수 있는지 확인하고 response content를 생성하고, `Accept` 헤더가 있다면 해당 content-type에 매칭되는지도 확인한다. 

`MessagConverter`를 사용하는 `@RequestBody`는 값을 바인딩하는 것이 아니라, 해당 내용을 Java Object로 변환한다. 그렇기 때문에 Body가 존재하지 않은 `GET` 메서드에 `@RequestBody`를 적용하려고 하면 예외가 발생한다. 

### 4가지 Default MessageConverters

1. `ByteArrayHttpMessageConverter`

    `byte[]` 오브젝트 타입을 지원하여 들어오는 요청을 모두 바이트 배열로 받을 수 있다. 또한 리턴 타입이 `byte[]` 일 경우 `Content-type`이 `applcation/octet-stream`으로 설정된다. 

2. `StringHttpMessageConverter`

    `String` 오브젝트 타입을 지원하여 HTTP 본문을 그대로 `String`으로 가져오고, 그대로 리턴할 경우 `Content-type`은 `text/plain`으로 전달된다. 

3. `FormHttpMessageConverter`

    MultiValueMap<String, String>을 지원하는데, 지원하는 미디어 타입은 `application/x-www-form-urlencoded`이다. 하지만 form 데이터의 경우 `@ModelAttribute`가 더 유용하다. 

4. `SourceHttpMessageConverter`

    XML 문서를 Source 타입 객체로 변환하고 싶을 때 사용하지만 요즘에는 잘 쓰이지 않는다. 

### 자주 쓰이는 MessageConverters 3가지

1. `Jaxb2RootElementHttpMessageConverter`

    JAXB의 @XmlRootElement와 @XmlType이 붙은 클래스로 XML과 오브젝트 사이의 변환을 해준다. 

2. `MashallingHttpMessageConverter`

    스프링 OXM 추상화 `Mashaller`와 `Unmarshaller`를 이용해서 `XML`과 오브젝트 사이의 변환을 해준다. 

3. `MappingJacksonHttpMessageConverter`

    `Jackson`의 `ObjectMapper`를 사용해서 `JSON`과 오브젝트 사이의 변환을 해주고 지원하는 미디어타입은 `application/json`이다.

- 참고링크: [https://joont92.github.io/spring/MessageConverter/](https://joont92.github.io/spring/MessageConverter/)

<br>

## @ModelAttribute

흔히 `@ModelAttribute`를 들어온 요청에 대해서 method parameter를 매핑시키는 용도로만 알고 있는데, 이외에 return value를 지정된 model attribute로 바인딩하고 웹 뷰에 표현하는 작업도 담당한다. <br>

따라서 `@ModelAttribute`는 두 가지 level에서 사용되는데 **1) method parameter**와 **2) method level**이다. <br>

일반적으로 `@ModelAttribute`는 form data에 사용이 되는데, 이때 value 속성의 값을 함께 넘겨준다. <br>

```html
<form:form method="POST" action="/spring-mvc-basics/addEmployee" 
  modelAttribute="employee">
    <form:label path="name">Name</form:label>
    <form:input path="name" />
    
    <form:label path="id">Id</form:label>
    <form:input path="id" />
    
    <input type="submit" value="Submit" />
</form:form>
```

하지만 만일 `value` 속성이 함께 제공되지 않았다면 default로 Method level에 적용되는 `@ModelAttribute`에게는 반환 타입이, Method-argument에는 parameter 타입이  지정된다. 

### At Method Level

Method level에서 `@ModelAttribute`이 추가되어 있다면 해당 메서드는 Model에 하나 또는 여러개의 속성(attribute)을 추가한다는 것을 의미한다. `@RequestMapping` 어노테이션이 붙어있는 메서드와 마찬가지로 같은 argument(Model mode)을 제공하는것은 동일하지만, 들어오는 request에 직접적으로 매핑되지는 않는다. 

```java
//하나의 model attribute를 추가할 때
@ModelAttribute
public void addAccount(@RequestParam String number) {
	return accountManager.findAccount(number);
}

//하나 이상의 model attributes를 추가할 때 
@ModelAttribute
public void addAttributes(Model model) {
	model.addAttribute("msg", "Welcome to the Netherlands");
	model.addAttribute("name", "This is name");
}
```

일반적으로 Spring MVC는 위 메서드를 request handler를 호출하기 이전에 우선적으로 호출한다.  <br>

**!!! `@ModelAttribute` 메서드들이 controller에서 `@RequestMapping`으로 지정된 메서드들 보다 먼저 실행된다는 것이다. (같은 controller에 있는 경우)** <br>

만일 global하게 들어오는 모든 요청에 대해서 특정 model attribute을 추가하고 싶다면, 해당 controller를 `@ControllerAdvice`을 지정하는 것이 좋다. <br>

일반적으로 method level의 `@ModelAttribute`의 목적은 하나 또는 하나 이상의 model attributes를 추가하기 위해서이다. Controller는 여러개의 `@MethodAttribute` 메소드들을 가지고 있을 수 있는데, 그 어떤 요청이 들어오면 같은 controller 안에 있는 그 모든 메소드들이 실행된다. 전역적으로 실행하고 싶다면 `@ControllerAdvice` 어노테이션을 추가하면 된다. <br>

### At Method Argument

만일 method argument 레벨에서 `@ModelAttribute`가 사용된다면 model 에서 인자가 추출되어야 한다는 것을 의미한다. 

> "An @ModelAttribute on a method argument indicates the argument should be retrieved from the model. If not present in the model, the argument should be instantiated first and then added to the model."

즉, Method Argument 레벨에서 `@ModelAttribute`를 사용하면, 우선 model에 해당 attribute가 있는지 확인하여 반환하고, 없다면 **1) arguments를 초기화** **2)model에 추가**한다. 

```java
@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute Pet pet) {
	...
}
```

위와 같이 Method Argument에서 `@ModelAttribute`가 사용되었을 경우 인자인 Pet instance의 출처는 다음 4가지 중 하나일 수 있다. 

1. `@SessionAttributes` 에 의해 이미 존재하는 model attribute
2. `@ModelAttribute` 메소드에 의해 이미 존재하는 model attribute
3. URI template와 type converter에 의해 추출된 instance
4. default constructor에 의해 초기화된 instance

<br>
<br>

**[참고자료]**

- [https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/remoting.html#rest-message-conversion](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/remoting.html#rest-message-conversion)
- [https://mangkyu.tistory.com/72](https://mangkyu.tistory.com/72)
- [https://stackoverflow.com/questions/29517613/how-exactly-works-requestbody-annotation-and-how-it-is-related-to-the-httpmessa](https://stackoverflow.com/questions/29517613/how-exactly-works-requestbody-annotation-and-how-it-is-related-to-the-httpmessa)
- [https://stackoverflow.com/questions/17362177/how-does-the-modelattribute-annotation-work-why-cant-i-got-the-value](https://stackoverflow.com/questions/17362177/how-does-the-modelattribute-annotation-work-why-cant-i-got-the-value)
- [https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-ann-modelattrib-methods](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-ann-modelattrib-methods)

<br>
<br>

**[MORE]**
- HttpMessageConverters 내부동작원리

    [https://www.baeldung.com/spring-httpmessageconverter-rest](https://www.baeldung.com/spring-httpmessageconverter-rest)

- 언제 무엇? 장단점
- URI template와 type converter에 의해 추출된 instance 동작원리

```toc
```