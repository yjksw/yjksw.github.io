---
emoji: ✊
title: "자바빈 규약 (번외: Serialization)"
date: '2021-06-08 23:00:00'
author: 코다
tags: 자바
categories: 자바
---

간단한 JavaBean 규약에 대해서 알고 넘어가기

## JavaBean

자바빈 규약을 따르는 Java Class를 말한다. 

## JavaBean 규약

1. defulat 패키지가 아닌 패키지 하위에 있는 클래스
2. 기본 생성자가 존재 (no-arg constructor)
3. Property는 모두 private으로 선언
4. Getter/setter를 통해서 properties를 조작
5. `Serializable`을 implement 하여 직렬화 가능

## 번외 : Serialization & Deserialization

- Serialization : converting state of an object into a byte stream
- Deserialization: reverse process of serialization

해당 객체에 영속성을 부여하기 위해서 사용되는 매커니즘이다. <br>

Java 객체를 serialize 하게 하기 위해서는 `java.io.Serializable` 인터페이스를 구현하도록 한다. 해당 인터페이스는 멤버변수나 메소드가 존재하지 않는 marker interface(Cloneable이나 Remote와 같은) 이다. <br>

Serializable하면 1) 해당 객체 그대로에 영속성을 부여할 수 있으며 2) 네트워크 상에서 byte stream으로 전송이 가능하다. <br>

**Serialization 특징**

- 부모 클래스가 Serializable interface를 구현하면 자식 클래스는 자동으로 Serializable 하다.
- non-static 멤버 변수만 Serialization 으로 저장될 수 있다. (static 과 transient 데이터는 불가)
- 비밀번호 등의 보안으로 인해 어떠한 멤버 변수가 serialize 되어 저장되지 않기를 원한다면 해당 데이터를 trasient 데이터로 지정하도록 한다.

    ```java
    class Member implements Serializable {
    	transient String password;
    	String name;
    	...
    }
    ```

- 해당 객체가 deserialized 될 때 해당 객체의 생성자는 호출되지 않는다.
- Serializable 한 객체와 연관되어 있는 객체 또한 Serializable 인터페이스를 반드시 구현해야 한다.

    ```java
    class ObjectA implements Serializable {	
    	//ObjectB는 반드시 Serializable을 구현해야 함	
    	ObjectB oj = new ObjectB();
    }
    ```

**SerialVersionUID**

Serialization을 진행하면서 각 Serializable class는 `SerialVersionUID` 라는 id를 할당받는다. 해당 id를 통해서 직렬화된 객체의 sender와 receiver를 판별하는데, sender와 reciever는 동일해야한다. 만일 동일하지 않다면 `InvalidClassException`이 deserialize 할 때 발생한다. 

<br>
<br>

**[참고자료]**
- [https://dololak.tistory.com/133](https://dololak.tistory.com/133)
- [https://www.geeksforgeeks.org/javabean-class-java/#:~:text=JavaBeans are classes that encapsulate,public getters and setter methods](https://www.geeksforgeeks.org/javabean-class-java/#:~:text=JavaBeans%20are%20classes%20that%20encapsulate,public%20getters%20and%20setter%20methods).
- [https://www.geeksforgeeks.org/serialization-in-java/](https://www.geeksforgeeks.org/serialization-in-java/)
- [https://www.javatpoint.com/serialization-in-java#:~:text=Serialization in Java is a,is converted into an object](https://www.javatpoint.com/serialization-in-java#:~:text=Serialization%20in%20Java%20is%20a,is%20converted%20into%20an%20object).


**[MORE]**
- non-static 만 serialization 가능한 이유
- SerialVersionUID

```toc
```