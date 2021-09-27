---
emoji: 🖥
title: "@Controller vs. @RestController"
date: '2021-04-28 23:00:00'
author: 코다
tags: 스프링부트
categories: 스프링부트
---

제일 핵심이 되는 차이점은 HTTPResponse Body의 생성 방식이다. 

## @Controller

본래 Spring MVC 컨트롤러의 주 역할은 View를 반환하는 것이다. 

아래 사진을 보면, 클라이언트가 URL을 통해서 Dispatcher Servlet 에 요청을 보내면 적절한 Handler를 매핍하고 컨트롤러에서 해당 View를 `ViewResolver`를 통해서 반환한다.  

<p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/134930324-a380f854-7718-4ef1-8161-1fe94cdd2e30.png"></p>

하지만 컨트롤러는 항상 View를 반환하는 것이 아니라 data도 반환해야한다. 이때는 @ResponseBody 어노테이션을 통해서 Json 형태로 클라이언트에 데이터를 반환할 수 있도록 해야 한다. 이때는 View를 반환할 때 사용하는 ViewResolver를 반환하는 것이 아니라 HttpMessageConverter 을 사용해서 데이터를 반환한다. 

<p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/134930518-240c33bd-f309-47d1-b836-2ad737a5c201.png"></p>

따라서 Data를 반환하는 응답에 대해서는 `@ResponseBody`를 붙여주어야하는데 매번 그러기가 번거로우니 Controller에 `@ResponseBody`가 자동으로 붙어있는 `@RestController`가 등장하게 된 것이다.

```toc
```