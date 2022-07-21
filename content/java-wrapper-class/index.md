---
emoji: ✊
title: "[Java] Wrapper Classes in Java" 
date: '2020-08-08 23:00:00'
author: 코다
tags: 자바
categories: 자바
---

다음은 Java에 존재하는 아주 특이한 클래스인 Wrapper class에 대한 내용이다. JAVA wrapper class에 대해서 설명한 한 <a href=https://www.baeldung.com/java-wrapper-classes>사이트</a>를 **번역**하는 겸 공부한 내용을 정리해 작성해놓았다.  

<br>

## 1. 개요

Wrapper class(감싸는 클래스) 이름이 설명하듯이 wrapper class는 자바의 Primitive types들을 객체로 감싸는 역할을 하는 클래스이다. <br>

다음과 같은 자바의 primitive 타입들은 모두 각자의 wrapper 클래스가 있다. <br>

- boolean, byte, short, char, int, long, float, double <br>
- Boolean, Byte, Short, Character, Integer, Long, Float, Double<br>

이것들은 모두 java.lang 패키지에 정의되어 있으므로 따로 import 하지 않아도 사용할 수 있다. <br>

<br>

## 2. Wrapper Classes

"Wrapper 클래스의 목적은 무엇입니까?"는 자바 관련 인터뷰에서 흔하게 물어보는 질문 중 하나이다. <br>

그 목적은 간단하게 말해서, generic classes는 객체와 호환되서 작동되지 primitive 타입과는 호환이 되지 않기 때문이다. 따라서, 우리가 generic classe들을 사용하고 싶다면 우리는 primitive type들을 객체로 만들 수 있도록 하는 방법이 필요한 것이다. <br>

예를 들어서, 많은 유용한 기능들을 제공하는 Java Collection Framwork는 객체와만 작동하도록 되어 있다. Java 버전 5 이하, 즉 거의 15년 전 자바를 사용할때는 나중에 소개할 autoboxing 기능이 없었기 때문에 현재 사용되는 것처럼 간단히 ``add(5)`` 와 같이 collection of Integers를 사용할 수 없었다. <br>

당시에는 primitive 타입의 값들은 직접 각 wrapper classes로 변환되어 새롭게 저장되어야 사용될 수 있었다. 지금은 autoboxing이라는 기능이 추가되면서, `ÀrrayList.add(101)`와 같은 문법을 그대로 사용할 수 있다. 자바 내부적으로 primitive value들을 Integer등과 같은 wrapper class 객체로 변환해서 사용될 수 있도록 하기 때문이다. <br>

<br>

## 3. Primitive에서 Wrapper Class로 변환

그럼 이 시점에 나올 만한 질문은 "어떻게 primitive value를 해당 타입 wrapper class로 변환할 수 있는가? "이다. 예를 들어, int 를 Integer로 변환하고, char를 Character로 변환하는 방법은 무엇인가 이다. <br>

간단히 2가지 방법이 있다. 생성자 constructor를 사용하던지, static factory method를 사용해서 primitive value를 wrapper class로 변환할 수 있다. <br>

하지만 Java 9에 들어서는 Integer나 Long에 대한 생성자는 중요도가 떨어지면 지원되지 않는 경우가 많기 때문에 factory method를 사용하는 것을 높이 권장한다. 다음 코드를 참고해보자. <br>

```java
Integer object = new Integer(1); //생성자 사용

Integer another Object = Integer.valueOf(1); //factory method 사용
```

위의 코드에 `valueOf()`메소드는 int를 Integer 객체로 변환해여 반환한다. 캐싱된 값을 반환하기 때문에 매우 효율적이고, -128 에서 127까지의 값은 항상 캐싱하며 이 범위 박의 값들도 캐싱할 수 있는 장점을 가지고 있다. <br>

위의 방법관 같이 boolean을 Boolean으로, byte 를 Byte, chat를 Character로, long을 Long으로, float을 Float로, double을 Double로 변환할 수 있다. 이와 다르게 String을 Integer로 변환하고 싶을 때는 String이 wrapper class가 아니기 때문에 `parseInt()` 를 사용해야 한다. <br>

반대로 wrapper 객체를 다시 Primitive type으로 변환하고 싶을 때는 각자에 맞는 `intValue()` 나 `doubleValue()` 와 같은 메소드를 사용해야 한다. <br>

```java
int val = object.intValue();
```

<br>

## 4. Autoboxing 과 Unboxing

이전 파트에서는 어떻게 primitive 값을 객체로 매뉴얼하게 바꿀 수 있는지에 대해서 다뤘었다. 하지만 이전에 언급했듯이 Java 5 이후에는 자동으로 변환해주는 autoboxing, unboxing이라는 속성이 등장했다.<br>

"Boxing"이라는 뜻이 primitive value를 해당 Wrapper class로 변환해준다는 의미이다. 자동적으로 변환되기 때문에 다음 단어에 auto를 덧붙여서 autoboxing이라고 불린다. <br>

비슷한 경우로 wrapper 객체가 unwrapped 되어 다시 primitive value가 되는 개념이 unboxing이다. <br>

즉, autoboxing과 unboxing은 실전에서 사용될 때 사용자가 primitive value를 특정 메소드에서 사용할 때 wrapper object이나 해당 primitive을 기대한 객체의 타입으로 바꾸어 준다는 뜻이다. <br>

```java
List<Integer> list = new ArrayList<>();
list.add(1); //autoboxing

Integer val = 2; //autoboxing
```

위의 예시에서는 자바가 자동적으로 primitive value인 int를 wrapper로 변환해주었다. 사실, 내부적으로는 valueOf() 메소드를 사용해서 변환한 것이다. 예를 들어, 다음 두 코드는 동일하게 동작하는 것이라고 여겨진다. <br>

```java
Integer value = 3;

Integer value = Integer.valueOf(3);
```

autoboxing은 코드를 한결 간결하게 만들어주고, 가독성을 높여주지만 때로는 **사용하지 말아야 할 때**가 있다. 예를 들어 **반복문 안**에서는 사용하지 않도록 한다. <br>

Autoboxing과 비슷하게 unboxing 또한 객체를 어떤 메소드에 전달했을 때 자동으로 기대되는 primitive로 변화해주는 역할을 한다. <br>

```java
Integer object = new Integer(1);
int val1 = getSquareValue(object); //unboxing
int val2 = object; //unboxing

public static int getSquareValue(int i){
  return i*i;
}
```

즉, 어떠한 메소드가 primitive value나 wrapper object을 기대할 때 두 가지 경우 모두를 전달할 수 있다는 것이다. 자바가 내부적으로 전달받은 변수에 대해서 맞는 primitive나 wrapper 클래스로 변환해 줄 것이기 때문이다. 

<br>
<br>

**<small>[참고 자료]: https://www.baeldung.com/java-wrapper-classes, https://www.geeksforgeeks.org/wrapper-classes-java/, https://www.scaler.com/topics/java/wrapper-classes-in-java/</small>**

```toc
```

