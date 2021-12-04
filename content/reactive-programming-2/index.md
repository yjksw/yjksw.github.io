---
emoji: 🍀
title: "리액티브 시리즈 - 2. Spring WebFlux"
date: '2021-12-04 23:00:00'
author: 코다
tags: 스프링 웹 
categories: 스프링 웹
---

## 💡 Intro

- 앞에서 리액티브 프로그래밍에 대해서 다뤘었다.
- 한마디로 리액티브 프로그래밍에 대해서 정의하자면 **비동기/논블로킹 이벤트 드리븐 개발과 배압을 통해 적은 수의 쓰레드로 생상자가 소비자를 압도하지 못하며 확장성있는 개발이 가능하게 하는 프로그래밍 기법**이라고 할 수 있다.
- 리액티브 프로그래밍은 가용성(CPU utilization이라고 볼 수 있는 영역)과 응답성(오류가 생겨도 빠르게 응답 가능)을 향상시키므로 프로그램의 효율성과 성능을 높인다.
- 함수형 프로그래밍도 관련 중요 키워드이다. 리액티브 프로그래밍은 함수형 프로그래밍(선언형, 함수 조합, 등등)을 활용한다.
- Spring WebFlux는 Spring Framework5에서 추가된 모듈이다. 스프링이 리액티브 스택 웹 어플리케이션을 구축할 수 있도록 리액티브 스트림 API를 지원해 **논블로킹/비동기식**으로 동작할 수 있도록 한다.

<br> 

## 🌩 Spring WebFlux 란?

- 기존의 Spring Web MVC는 Servlet API, Servlet Container에서 동작하도록 되어 있으므로 전통적인 동기/블로킹 방식만 지원했다.
- 리액티브 스택 프레임워크인 Spring WebFlux는 **fully non-blocking backpressure**로 동작하는 **리액티브 스트림**을 지원한다.
- 스프링 프레임워크에서 spring-webmvc와 spring-webflux는 **공존**할 수 있으며 각 모듈은 optional이다. 즉, 둘다 사용할 수도 있고 하나의 모듈만 사용할 수도 있다.

<br> 

## 🌩 Spring WebFlux 탄생이유

한 문장으로 말하면 **1) 적은 수의 스레드**로 **2) 최대한의 동시성**을 핸들링하여 **3)적은 하드웨어 리소스**를 사용하기 위한 **4) 비동기 웹 스택**이 필요했기 때문이라고 할 수 있다. 

기존의 Servlet API에도 논블로킹 I/O를 할 수 있지만 해당 API를 사용하면 기본적으로 동기적(`Filter`, `Servlet`)이며 블로킹(`getParameter`, `getPart`)한 나머지 Servlet API를 사용하기 어려워진다. 따라서 완전히 논블로킹한 환경에서 동작할 수 있는 공통 API가 생겨나게 되었다. 

**또 다른 이유는 함수형 프로그래밍 때문이다.** 자바8에 람다 및 스트림 등의 문법이 추가되면서 자바로 함수형 API를 구현할 수 있는 환경이 갖추어지기 시작했다. 이는 비동기 로직을 지원하고 논블로킹 어플리케이션을 구현할 수 있는 통로가 되며 Spring에서도 그러한 특성을 지원하는 WebFlux를 탄생시키게 되었다. 

### 웹에서 비동기/논블로킹의 필요성

스프링은 웹 프레임워크이니 근본적인 개념보다 웹에 국한한여 생각해보자.

- 웹에서 병목을 유발하는 것은 외부 장치에 대한 I/O 작업인 경우가 많다. 즉, 요청당 하나의 스레드가 할당되어야 하지만 해당 스레드가 블로킹 되어 있는 상태인 경우가 훨씬 많다.
- 이때 수많은 요청이 들어오게 되어 thread pool에 있는 스레드 개수 이상의 요청이 들어오게 되면 스레드에 대한 대기로 인해 latency가 급격히 느려지는 Thread Pool Hell이 발생하게 된다.

    <p align="center"><img width="75%" src="https://user-images.githubusercontent.com/63405904/144712827-31460842-a91c-4d90-a53d-91eb797c6421.png"></p>
    
- 그렇다고 스레드를 추가로 생성하는 것은 오버헤드가 큰 작업이며 잦은 context switching으로 오히려 CPU utilization이 떨어져 비효율이 발생한다.
- 따라서 적은 수의 스레드로 동시성을 높이기 위해서는 스레드가 블로킹 되어 있지 않고 외부 I/O 작업이 일어날 때 필요한 다른 작업을 수행하도록 해야한다.
- 점점 더 MSA가 대두되면서 다수의 마이크로서비스로 분리되어 서비스간 메시지 통신을 하는 경우가 잦아졌다. 이런 외부 통신이 많아진 만큼 이 모든 요청을 동기/블로킹 방식으로 처리하면 동시성이 떨어지게 된다.
- 따라서 비동기/논블로킹 형식의 프로그래밍의 필요성이 더욱 강조되기 시작한다.

<br> 

## 🌩 Spring MVC vs. Spring WebFlux

### 구체적으로 어떻게 더 좋은 걸까?

우선 동기/비동기, 블록/논블록에 대한 기본적인 이해가 있다는 것을 전제하에 설명한다. 비동기/논블록 방식이 적용되면 여러 외부 I/O 작업이나 API 호출이 필요할 때 각 경과시간의 합 만큼의 시간이 소요된다. 하지만 비동기/논블록 방식이 적용되는 리액티브라면 각 경과시간 중 최대시간 만큼의 시간이 소요된다. 

어떻게 이런 효과를 (적은 스레드를 가지고!!) 내는지 이해하기 위해서는 리액티브에 빠질 수 없는 키워드인 **event-driven**을 잘 이해해야한다. 일반적으로 event-driven이라고 한다면 다음 그림을 떠올리면 된다. 

<p align="center"><img width="75%" src="https://user-images.githubusercontent.com/63405904/144712817-0bea67a0-5be8-4f10-b403-19e09721a383.png"></p>

주 업무를 하는 주체는 Event loop, Events, Event Handlers이다. 사용자나 외부 요인에 의해 이벤트가 발생하면 이벤트 루프는 해당 이벤트를 받아서 관리하고 알맞은 핸들러에 넘기는 역할을 한다. 핸들러는 해당 이벤트를 처리한다. 

여기서 Event loop는 적은 스레드를 가지고 운용할 수 있다. 따라서 이전에는 각 이벤트에 대한 스레드가 각각 필요했다면 event-driven 형태에선 아무리 이벤트가 많이 발생하더라도 스레드 풀에 있는 적정량의 스레드(주로 CPU 코어 개수거나 두배)로 관리 및 처리할 수 있다. 

번외로 event-driven의 탄생이유에 대해서 말해보자면, 이전에는 예상 가능하게 순차적으로 프로그래밍 진행되곤 했다. 하지만 최근이 GUI가 발전하고 점점 더 사용자와의 인터랙션이 많아지면서 제어할 수 없는 유저 이벤트가 많아지면서 이런 방식이 생겨나고 많이 사용되게 되었다. 

### Spring WebFlux 구조

<p align="center"><img width="75%" src="https://user-images.githubusercontent.com/63405904/144713050-4d280534-1b6c-43c6-8bd1-fa37d4f7fa31.png"></p>

전반적인 Spring WebFlux의 구조를 보면서 리액티브 프로그래밍이 어떻게 적용되었는지 살펴보자. 

사용자 요청이 들어오면 Event loop를 통해서 event가 되어 관리가 된다. 이때 하나의 요청에 하나의 스레드가 배정되는 것이 아니라 적은 스레드로 이벤트 루프에서 관리할 수 있다. 이후 이벤트는 비동기/논블록으로 연산을 처리한다. 연산이 끝나면 콜백 함수로 처리하고 응답한다.

이렇게 Spring WebFlux는 더 효율적으로 I/O를 제어하여 좋은 성능을 낸다. 앞서 언급했던 점점 더 MSA 추세로 여러 서비스가 네트워크 호출을 해야하는 시기에는 더욱 효율적일 수 있다. 

주의해야할 점은 한 곳이라도 동기/블로킹이 되는 곳이 있다면 아무런 효용이 없다는 것이다. 결국 동기/블로킹 호출하는 API에서 병목이 일어나기 때문이다. 

현재 Spring WebFlux의 WebClient로 외부 API를 리액티브 방식으로 처리할 수 있지만 여전히 DB connection과 관련해서는 논블로킹 라이브러리가 많이 사용되고 있지 않다. (R2DBC, jasync sql 등등이 개발중이라고 한다.) 

<br> 

## 🌩 Spring WebFlux 무조건 좋을까?

당연한 이야기겠지만 Spring WebFlux가 무조건 좋지는 않다. (개발에 트레이드 오프는 항상 있으므로!) 그럼 언제 무엇을 쓰는 것이 좋을까? 정답은 없지만 스프링 공식문서에 Spring MVC와 WebFlux를 비교하고 설명한 포인트들을 짚어보자. 

아래 그림은 두개의 공통점과 차이점을 표현한 다이어그램이다. 

<p align="center"><img width="75%" src="https://user-images.githubusercontent.com/63405904/144713072-84bc054d-8073-4090-b2c9-3433ad45db2b.png"></p>

스프링 공식문서에서는 몇가지 상황에 어떠한 것을 제안하는지 적혀있다. 

- 우선 Spring MVC로 어플리케이션이 정상동작하면 굳이 바꿀필요는 없다. 명령형 프로그래밍은 개발하기도, 이해하기도, 디버깅하기도 더 좋다. 즉, 생산성이 더 좋다는 것이다. 리액티브 개념은 이제 막 발전중이기 때문에 기존의 명령형에 비해 라이브러리도 부족하다.
- 만일 자바8의 람다나 코틀린을 사용하는 가벼운 함수형 웹 프레임워크에 관심이 있다면 WebFlux는 좋은 선택이다. WebFlux는 작은 어플리케이션이나 복잡하지 않은 요구사항을 구현한 마이크로서비스에 적합하다.
- MSA에서는 각 어플리케이션이 Spring MVC나 Spring WebFlux를 혼합해서 사용하고 있을 수 있다. 어노테이션 기반의 프로그래밍 모델은 위 두 프레임워크를 재사용하기도 편하게 해준다.
- 어느 어플리케이션에 무엇을 쓸지 헷갈린다면 가장 간단한 방법은 의존성을 체크해보는 것이다. 만일 플로킹 persistence API의 일종인 JPA, JDBC 등을 사용하거나 블로킹 네트워크 API를 사용하고 있다면 Spring MVC가 더 적합하다. 물론 Reactor나 RxJava등을 통해 블로킹 작업을 별도의 스레드에서 처리하도록 하는 것이 가능하지만 여전히 논블로킹 웹 스택의 장점을 온전히 활용하지 못하는 경우다.
- 만일 지금 Spring MVC 어플리케이션을 쓰고 있고 외부 API를 호출해야 한다면 리액티브 `webClient`를 활용해보는 것을 추천한다. 각 요청에 대한 latency가 향상되며 그 장점이 극대화된다.
- 스프링 공식 문서에 이런 내용도 있어서 놀랐다. 공식문서에 따르면 만일 팀에서 적용하고자 한다면 논블로킹이나 선언형 프로그래밍으로 전환하기 위해서는 매우 가파른 러닝커브가 존재한다는 것을 염두해두라고 한다.
    
    우선 가상 효율적으로 전환하는 방식은 현재 구조에서 reactive한 `webClient` 부터 적용해보는 것이다. 그리고 나서 점진적으로 적용을 시작하고 변화로부터 얻는 효용을 계산해보기를 추천한다. 공식문서에서 말하기를 "예상하건데 어플리케이션 전반적인 측면에서 논블로킹 선언형으로의 전환은 불필요할 것이다" 라고 언급한다. 따라서 만일 전환으로 인한 분명한 효용이 눈에 보이지 않거든 우선 논블로킹 I/O가 어떻게 동작하는지부터 공부하기를 추천한다. 
    
<p align="center"><img width="75%" src="https://user-images.githubusercontent.com/63405904/144713083-097806d8-8c1a-4a39-a480-9704274c4bc4.png"></p>

다음 [링크](https://dzone.com/articles/raw-performance-numbers-spring-boot-2-webflux-vs-s)에서는 Springboot와 webFlux의 성능을 측정해 보았다. 초반에 성능이 비슷한 구간이 분명히 있다. 만일 지금 환경이 그 구간이라면 전환은 불필요하다. 오히려 단점이 될 수 있는 것이 기존의 방식은 매우 직관적으고 코드를 작성하고, 디버깅하고, 이해하기 쉽기 때문에 생상성 측면에서 훨씬 뛰어나다. 

<br> 

## 🛋 느낀 점

- 먼저, 스프링 공식문서는 매우 친절하다!!
- 리액티브에 대해서 나름 깊이(힘들게) 공부하고 난 뒤에 WebFlux에 대해서 다시보니 좀 이해가 되는 것 같다.
- 그래서 여기서 러닝커브가 높으므로 이것이 생산성을 떨어뜨릴 수 있으므로 반드시 꼭 필요한 효용성이 눈에 보일 때 적용하라고 한 것이 무엇보다 많이 와닿았다.
- 점점 더 요청이 많아지고 Thread pool의 스레드가 부족하니 나온 해결책이라는 배경을 알게되니 굉장히 흥미로웠다. 불편함을 찾고 문제를 해결하는 것이 멋있다고 느껴졌다.
- 나는 기술로 불편을 해결한 적이 있나 하는 고민을 요즘 많이 하게 된다. 비생산적이고 비효율적인 환경이나 루틴을 문제의식 없이 받아드리기 보다 적극적으로 해결해보자는 생각이 든다.
- 번외로 [Reactive Manifesto](https://www.reactivemanifesto.org/) 내용과 내장되어 있는 용어집도 정리해보고 싶다 🙌
- 처음에 정말 이해가 안갔는데.... 일단 계속 또 보고 또 보고 또 보면 결국 이해가 되는구나 ...💦

<br> 
<br> 

**[참고자료]**

- [https://heeyeah.github.io/spring/2020-02-29-web-flux/](https://heeyeah.github.io/spring/2020-02-29-web-flux/)
- [https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)
- [https://www.baeldung.com/spring-webflux](https://www.baeldung.com/spring-webflux)
- [https://alwayspr.tistory.com/44](https://alwayspr.tistory.com/44)
- [https://deepakpol.wordpress.com/2015/09/29/event-driven-and-reactive-architecture/](https://deepakpol.wordpress.com/2015/09/29/event-driven-and-reactive-architecture/)
- [https://dzone.com/articles/raw-performance-numbers-spring-boot-2-webflux-vs-s](https://dzone.com/articles/raw-performance-numbers-spring-boot-2-webflux-vs-s)
- [https://juneyr.dev/reactive-programming](https://juneyr.dev/reactive-programming)

```toc
```