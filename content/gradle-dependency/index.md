---
emoji: 🏡
title: Gradle 의존성 주입 시 implementation vs. compile
date: '2021-03-02 23:00:00'
author: 코다
tags: 빌드
categories: 빌드
---

웹 UI/DB 를 적용한 온라인 체스 게임을 구현하는 중 리뷰어가 다음과 같은 질문을 했다. 처음 웹 개발을 해보는 것이라서 우선 돌아가기 위해 인터넷과 크루들이 추가한 `dependency` 를 우선 가져와 추가했었는데 리뷰어의 질문을 받고 해당 개념을 찾아보았다. 

[참고 링크]([https://tomgregory.com/gradle-implementation-vs-compile-dependencies/#:~:text=The compile dependency configuration is,the same functionality as compile.&text=You should always use implementation,as compile is now deprecated](https://tomgregory.com/gradle-implementation-vs-compile-dependencies/#:~:text=The%20compile%20dependency%20configuration%20is,the%20same%20functionality%20as%20compile.&text=You%20should%20always%20use%20implementation,as%20compile%20is%20now%20deprecated).)

<p align="center"><img width="90%" src="https://user-images.githubusercontent.com/63405904/134770516-da835d13-928a-4cf9-95fc-61b73542173c.png"></p> 


결론부터 말해서 다음을 기억하면 될 것 같다. 

- `compile` 은 Gradle 7.0 부터 depracated 되므로 대부분의 상황에서 `implementation` 을 사용하도록 한다.

`Compile`과 `implementation`은 거의 같은 가능을 하기 때문에 서로가 대체 되어도 상관없다. 

### 그렇다면 implementation 의존성 주입은 무엇일까?

Java 프로젝트가 실행이 될 때 2개의 classpath가 존재한다. 

1. Complie classpath 
2. Runtime classpath 

Gradle dependency를 추가할 때 위 두가지 경우에 필요한 의존성들이 나누어지고 둘다 필요한 경우도 있다. 따라서 각각 필요한 경우에 추가할 수 있는 경우들과 두가지 경우 모두 필요할 경우 추가할 수 있는 키워드가 따로 있다. 

1. compileOnly : compile classpath에서 필요한 경우
2. runtimeOnly : runtime classpath에서 필요한 경우
3. implementation : 위 두가지에 모두 필요한 경우

위 경우들을 나누어서 의존성을 추가했을 때, 각각의 경우에 dependencies와 classpaths의 간결함으로 컴파일 시간이 단축되고 프로그램 복잡도를 낮출 수 있다.

```toc
```