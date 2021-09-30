---
emoji: 🍀
title: 클래식한 스프링 웹 어플리케이션 구조
date: '2021-06-26 23:00:00'
author: 코다
tags: 스프링 웹
categories: 스프링 웹
---

다음 글은 [링크](Understanding Spring Web Application Architecture: The Classic Way)에 기술되어 있는 스프링 웹 어플리케이션 구조에 대한 글을 번역 한 내용이다. 

## 좋은 architecture를 위한 두 기둥

### The SoC(Separation of Concerns) 원칙

> A design principle for separating a computer program into distinct sections, which each section addresses a separate concern. <br> [출처]([https://en.wikipedia.org/wiki/Separation_of_concerns](https://en.wikipedia.org/wiki/Separation_of_concerns))

SoC에서 신경써야 할 부분은 두가지이다.

1. 고려해야 할 **concerns**가 무엇인지  
2. 어디서 해당 **concern**을 다루고 싶은지

SoC를 준수하게 된다면 각각의 layer와 해당 layer의 책임에 대해서 자연스럽게 정의할 수 있도록 도와준다. 

### The KISS(Keep It Simple Stupid) 원칙

> Most systems work best if they are kept simple rather than made complicated; therefore simplicity should be a key goal in design and unnecessary complexity should be avoided. <br> [출처]([http://en.wikipedia.org/wiki/KISS_principle](http://en.wikipedia.org/wiki/KISS_principle))

이 원칙은 각 레이어는 그만큼의 비용이 들고 복잡한 구조를 가진 어플리케이션은 그만큼의 높은 비용을 감수해야 한다는 것을 상기시켜준다. 

- 새로운 feature를 추가하는 경우 해당 정보를 여러 layer에 모두 전달해야 하기 때문에 과정이 오래 걸림
- 지나치게 복잡한 구조를 가지고 있기 때문에 아무도 제대로 이해하고 있지 않아서 해당 어플리케이션의 유지보수가 어려움

<br>

## 3 Layers가 가장 적당하다

웹 어플리케이션의 책임을 고려했을때 웹 어플리케이션 전체적으로 다음과 같은 concerns가 있다.

- 사용자의 입력을 받아서 적당한 응답을 반환
- 예외 처리를 하여 예외 상황시 적절한 에외 메세지를 반환
- 트랜잭션 관리 전략을 가짐
- 인가와 인증을 처리함
- 어플리케이션의 비지니스 로직을 작성
- 사용되는 데이터 저장소와 외부 리소스와의 커뮤니케이션 담당

위 역할을 감당하기 위해서 다음 3가지 layer로 충분하다. 

1. **The Web Layer**
    
    웹 어플리케이션의 최상단에 있는 layer이다. 사용자의 입력을 받아서 적정한 응답을 반환하는 역할을 맡는다. 이 레이어에서는 다른 레이어에서 발생한 예외들에 대한 핸들링을 처리해야한다. <br>
    
    현재 레이어는 해당 어플리케이션의 입구이기 때문에 인증 및 인가를 담당하여 허가되지 않은 사용자에 대한 1차 방어를 해야한다. 
    
2. **The Service Layer**
    
    웹 레이어 다음에 위치한 레이어이다. 이 레이어는 트랜잭션 단위를 구분하고, application과 infrastructure 서비스를 모두 포함한다. <br>
    
    여기서 **application services**는 서비스 레이어의 **public API**를 제공한다. 이 뜻은, 어플리케이션 서비스에서 처리하고하고자 하는 일들이 서비스 레이어의 `public` 메소드에서 처리된다는 것이다. 이 메소드 별로 트랜잭션 단위가 나뉘어지고 인증 및 인가 작업도 여기서 담당한다. <br>
    
    서비스 레이어에서 **infrastructure services**는 파일 시스템, 데이터베이스, 이메일 서버 등등의 외부 리소스와 커뮤니케이션하기 때문에 "plumbing code" (배관 코드)라고 불리기도 한다. 이 메소드들은 하나 이상의 어플리케이션 서비스 코드에서 주로 활용된다. 
    
3. **The Repository Layer**
    
    웹 어플리케이션의 최하위에 위치해있는 레이어이다. 데이터 저장소와의 연결을 담당한다. 
    
<br>

특정 레이어에 속한 컴포넌트는 동일 레이어, 혹은 더 하위 레이어의 컴포넌트를 활용할 수 있다. 

 <p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/135494819-559541d9-1b75-4f2b-a5fe-f03519508483.png"></p>

<br>

## Layer 나누어 설계하기

이제는 각 레이어에 대한 인터페이스를 설계해야하는데, 여기서 DTO(Domain Transfer Object)와 domain model이라는 키워드가 등장한다. 

- **DTO**
    
    단순한 데이터를 담는 객체로 각기 다른 프로세스와 어플리케이션의 레이어간 데이터 전달 때 사용되는 객체이다. 
    
- **Domain Model**
    
    도메인 모델에는 3가지 다른 역할의 객체들이 있다.
    
    1. **Domain Service**
        
        도메인과 관련된 operation 이지만 enity나 VO의 일부는 아닌 상태가 없는(stateless) 클래스
        
    2. **Entity**
        
        전체 라이프 사이클 동안 바뀌지 않고 그 indentity 자체로 정의되는 객체 **(?????)**
        
    3. **Value Object**
        
        어떤 것의 속성을 나타내고, 그 자체로의 identity나 lifecycle이 없는 객체이다. 주로 VO의 life cycle은 entity의 lifecycle에 종속되어 있다. 
        

각 레이어의 인터페이스에는 다음 것들이 포함되어 있어야 한다. 

- web layer는 DTO만을 다루어야 한다.
- service layer는 DTO를 메소드 인자로 받고 도메인 모델을 핸들링 할 수는 있지만 DTO만을 web layer에 다시 반환해야 한다.
- repository layer는 entity를 메소드 인자로 받고 entity를 반환해야 한다.

그렇다면 왜 VO가 아닌 DTO로 레이어간 소통을 해야하는지 궁금할 수 있다. 다음 두가지 이유로 VO를 직접 사용하는 것은 좋지 않다. 

1. 도메인 모델은 어플리케이션 내부 모델이다. 따라서 도메인 모델을 외부로 노출한다면 클라이언트는 해당 도메인 모델을 어떻게 다루어야하는지 인지해야한다. 하지만, 클라이언트는 해당 로직에 대해서 알 필요가 없다. 만일 DTO를 사용한다면 도메인 모델을 클라이언트에게 숨기고, 깔끔하고 쉬운 API를 제공할 수 있다. 
2. 도메인 모델을 외부에 노출한다면 도메인 모델을 수정할 때 해당 도메인 모델이 의존하는 것들을 함께 수정해야한다. 하지만 만일 DTO를 사용한다면도메인 모델을 수정하더라도 다른 것(other stuff)들이 DTO와 연결되어 있기 때문에 다른 것들을 수정하지 않아도 된다. 

 <p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/135494900-92d34444-4d4a-4312-af6a-a0d45bf62f75.png"></p>

<br>
<br>

**[참고링크]**

- [https://www.petrikainulainen.net/software-development/design/understanding-spring-web-application-architecture-the-classic-way/](https://www.petrikainulainen.net/software-development/design/understanding-spring-web-application-architecture-the-classic-way/)

```toc
```