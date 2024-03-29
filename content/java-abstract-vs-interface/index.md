---
emoji: ✊
title: 추상클래스와 인터페이스 더 이상 헷갈리지 않기
date: '2021-11-15 23:00:00'
author: 코다
tags: 자바
categories: 자바
---

## 💡 Intro

- 인터페이스와 추상클래스의 차이를 명확하게 구분해보자.
- 언제 무엇을 쓰는 것이 좋은지 나름의 정의를 내려본다.
- 상속의 위험성에 대해서 고민해본다.

<br>

## 🌩 추상클래스

- 추상 클래스는 "미완성 설계도" 이다.
- 공통부분을 우선 정의한 미완성 설계도를 만들고 각기 다른 상황에 대해서 추가로 구현할 수 있다.
    - 완성되지 않은 abstract 메소드를 포함하고 있다.
- 추상클래스는 abstract 메소드가 있다는 것을 제외하고는 일반클래스와 동일하다.
- **상속은 자손 클래스를 만드는데 조상 클래스를 사용하는 것**
- **추상화는 자손 클래스의 공통부분을 뽑아내서 조상 클래스를 만드는 것**

### 상속

- 추상클래스를 `extends` 하는 명령어가 상속에서 사용되기 때문에 두 개념이 혼용되어서 사용되기도 한다. 엄밀히 말하면 두 개념이 겹칠수도 있지만 완전히 동일한 것은 아니다.
- 상속이란 기존의 클래스를 재사용하여 새로운 클래스를 작성하는 것
    - 적은 양의 코드로 새로운 클래스를 작성할 수 있고 공통부분을 관리할 수 있다는 장점이 있다.
- **상속은 is-a 관계로 표현될 수 있다.**
- `**final`로 정의된 클래스가 아니라면 모두 상속이 가능하다. 추상클래스는 그 일부분이다.**
- 상속을 할 때 주의할 점 ‼️
    - 상속은 조상 클래스의 캡슐화가 깨지는 위험이 있다.
    - 또한 조상 클래스와 자손 클래스간의 강한 결합이기 때문에 조상 클래스 메서드에 변화가 생기면 자손 클래스에 아무런 변화가 없음에도 다르게 동작할 수 있다.
    - 따라서 상속보다는 **조합**을 사용하도록 추천한다. (Deck는 Card를 가지고 있다 와 같은 경우에 조합 사용 has-a 관계)
    - 일반 클래스는 `final`을 추가하여 상속을 막거나 미완성 설계도인 abstract 클래스를 정의하도록 추천한다.
        - 또한 추상클래스의 부모 메소드를 수정하지 않는 것이 좋다.

### 왜 자바는 단일 상속일까? (single inheritance)

- 다중상속을 하게 되면 복합적인 기능을 가진 클래스를 손쉽게 구현할 수 있지만 클래스 간 관계가 굉장히 복잡해진다.
- 여러 다른 클래스로 상속받은 멤버의 이름이나 메서드가 같은 경우 구별도 어렵다.

<br>

## 🌩 인터페이스

- 인터페이스는 "기본 설계도"이다. (추상클래스는 "미완성 설계도")
- 인터페이스는 면대면이 만나는 부분이라는 뜻을 가지고 있으며 2개의 구성요소가 상호작용할 수 있도록 접속 경계를 연결하는 부분이다. (플러그와 같은 역할)
- 인터페이스끼리 상속받을 수 있으며 다중상속이 가능하다.
- 인터페이스는 `implements` 를 사용해서 구현하며 다중구현이 가능하다.
- 인터페이스는 구현 메서드나 멤버 필드가 없다.
    - default 메서드가 가능하도록 jdk1.8부터 추가가 되었고, static 변수만 가능하다.
- 인터페이스는 해당 인터페이스를 구현하는 모든 클래스가 특정 메소드를 반드시 구현하도록 강제하는 역할을 한다 → 따라서 명세의 역할을 한다고도 한다.

### 추상클래스와의 차이점

- 우선 구현메서드 존재 여부, 필드 존재 여부, 다중 상속 및 구현에 대한 차이점이 존재한다
- 추상클래스는 공통기능에 대한 재사용과 정제의 역할을 한다. (정제란 불완전한 행동을 완전하게 만드는 것을 의미)
- 인터페이스는 구현체가 특정 메소드가 반드시 존재하도록 하는 역할을 하여 특정 기능을 반드시 제공한다는 것을 보장하는 역할을 한다.

### 동일 메서드를 가진 인터페이스

- 동일한 메서드 명과 시그니처를 가진 메서드가 두 개의 인터페이스에 있을때는 어떻게 할까?
    - 이때는 컴파일 오류로 충돌을 해결해야만 컴파일이 가능하다.

<br>

## 🛋 느낀 점

- 추상클래스와 인터페이스는 역할 자체가 다르다.
    - 추상 클래스는 정제의 역할을 인터페이스는 기능 구현 강제의 역할을 한다.
    - 인터페이스를 통해서는 중복을 해결할 수 없다. 어떤 두 요소가 연결되기 위해 사전에 정의한 기능들이 모두 구현이 된다는 것을 보장할 뿐이다.
    - 상속은 부모 클래스의 캡슐화가 깨지기 때문에 조합을 이용하는 것이 더 적절하다.


```toc
``` 