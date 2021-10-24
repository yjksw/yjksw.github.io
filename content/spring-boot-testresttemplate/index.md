---
emoji: 🖥
title: Springboot의 TestRestTemplate 알아보기
date: '2021-10-24 22:00:00'
author: 코다
tags: 스프링부트 테스트
categories: 스프링부트 테스트
---

> 다음은 [TestRestTemplate 링크]([https://www.baeldung.com/spring-boot-testresttemplate](https://www.baeldung.com/spring-boot-testresttemplate)) 를 번역하면서 공부한 글입니다. 🙌

<br> 

기존에 RestTemplate을 활용하여 통합테스트를 많이 했을 것이다. 스프링부트에는 굉장히 비슷하게 동작하는 TestRestTemplate이 있다. 
두가지 모두 통합테스트에서 유용하며 HTTP API를 다룰 수 있다. TestRestTemplate의 예시를 한번 들여다보자. 

```java
TestRestTemplate testRestTemplate = new TestRestTemplate();
ResponseEntity<String> response = testRestTemplate.
  getForEntity(FOO_RESOURCE_URL + "/1", String.class);
 
assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
```

RestTemplate과 거의 유사한 형테를 지니고 있다. 하지만 TempRestTemplate은 RestTemplate을 확장하지 않으며 몇가지 다른 기능을 제공한다. 

<br>

## 🌩 TestRestTemplate은 무엇이 다를까?

### 1. Auth Credentials을 설정할 수 있는 생성자를 제공한다.

TestRestTemplate을 생성할 때 기본 authentication을 설정하여 생성할 수 있다. 그러면 해당 인스턴스를 활용한 모든 요청이 해당 credential이 적용된 채로 수행된다. 

```java
TestRestTemplate testRestTemplate
 = new TestRestTemplate("user", "passwd");
ResponseEntity<String> response = testRestTemplate.
  getForEntity(URL_SECURED_BY_AUTHENTICATION, String.class);
 
assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
```
<br>

### 2. HttpClientOption을 제공하는 생성자를 제공한다.

기타 다른 Http 옵션등을 설정할 수 있다. Enum을 제공하며 `ENABLE_COOKIES`, `ENABLE_REDIRECTS`, `SSL` 이 있다. 

```java
TestRestTemplate testRestTemplate = new TestRestTemplate("user", 
  "passwd", TestRestTemplate.HttpClientOption.ENABLE_COOKIES);
ResponseEntity<String> response = testRestTemplate.
  getForEntity(URL_SECURED_BY_AUTHENTICATION, String.class);
 
assertThat(response.getStatusCode(), equalTo(HttpStatus.OK))
```
<br>

### 3. 새로운 메소드

생성자 뿐 아니라 `withBasicAuth()` 라는 메소드를 통해서 authentication을 추가할 수도 있다. 

```java
TestRestTemplate testRestTemplate = new TestRestTemplate();
ResponseEntity<String> response = testRestTemplate.withBasicAuth(
  "user", "passwd").getForEntity(URL_SECURED_BY_AUTHENTICATION, 
  String.class);
 
assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
```

<br>

## 🌩 RestTemplate과 TestRestTemplate 둘다 사용하기

TestRestTemplate은 RestTemplate의 wrapper로 활용될 수 있다. 예를 들어 이미 restTemplate으로 구현된 레거시 코드가 있다면 wrapper로 TestRestTemplate으로 전환해 손쉽게 사용이 가능하다. 

```java
RestTemplateBuilder restTemplateBuilder = new RestTemplateBuilder();
restTemplateBuilder.configure(restTemplate);
TestRestTemplate testRestTemplate = new TestRestTemplate(restTemplateBuilder);
ResponseEntity<String> response = testRestTemplate.getForEntity(
  FOO_RESOURCE_URL + "/1", String.class);
 
assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
```
<br>

## 🌩 결론

TestRestTemplate은 단순히 RestTemplate의 확장버전이 아니다. 오히려 더 간단하게 통합테스트를 할 수 있고 유용하게 authentication을 설정할 수 있는 대체안이다. Apache Http 클라이언트를 커스텀할 수 있으며 RestTemplate의 wrapper 클래스로도 활용될 수 있다.

```toc
```