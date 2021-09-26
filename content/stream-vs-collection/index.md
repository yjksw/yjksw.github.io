---
emoji: ✊
title: Stream vs. Collection
date: '2021-03-20 23:00:00'
author: 코다
tags: 자바
categories: 자바
---

요약하자면 Stream과 Collection의 차이는 다음과 같다. 

```java
1. 스트림은 요소를 보관하지 않고 필요할 때 생성되거나 하위 Collection에 보관한다. 
2. 스트림은 원본을 변경하기보다 새로운 스트림을 생성하여 반환한다. 
3. 스트림 연산은 lazy operation이다. (따라서 무한 스트림도 가능한 것이다)
```

개념적으로 접근했을 때 Collection의 경우에는 어떠한 데이터를 담는 자료구조의 역할을 주로 하지만, Stream의 경우는 연산과 관련된 것이 주라고 볼 수 있다. 

- Quote

    Java *Collection*s offer efficient mechanisms to store and process the data by providing data structures like *[List](https://drafts.baeldung.com/java-linkedlist)*, *[Set](https://drafts.baeldung.com/java-hashset)*, and *[Map](https://drafts.baeldung.com/java-hashmap)*.

    However, the Stream API is useful for performing various operations on the data without the need for intermediate storage.

    출처: [https://www.baeldung.com/java-return-stream-collection](https://www.baeldung.com/java-return-stream-collection)

### Traversal

Collection은 여러번 데이터를 횡단할 수 있지만, Stream은 한번만 가능하며 source로부터 새로운 Stream을 추출해야 새롭게 traverse 할 수 있다. 

```java
Stream<Integer> numbers = ...;
numbers.forEach(...); //error 없음
numbers.forEach(...); //error 발생
```

### Lazy Operation

Collection은 요소를 보관하기 때문에 해당 Collection에 어떠한 요소가 추가되기 전에 operation을 우선 실행해야한다. <br>

하지만 Stream은 lazy하기 연산을 하기 때문에 우선 Stream에 담겨진 요소들에 대해서 선실행을 하지 않는다. 이후에 필요할 때 요소를 꺼내오고 연산을 하도록 한다. 또한 Stream은 불변이므로 요소를 추가하거나 삭제할 수 없다. 

### 외부반복 vs. 내부반복

외부에서 값을 꺼내서 반복해야하는 Collection과 다르게 Stream은 내부적으로 알아서 반복문을 돌면서 명령한 연산을 수행한다. <br>

내부 반복을 했을 경우 1) 반복자를 사용하여 명시적으로 표시할 필요가 없고 2) 병렬처리시 스레드간 공유자원에 대한 관리를 할 필요가 없다. 

### 유연성

Stream은 여러 operation의 조합으로 유연하게 데이터 연산이 가능한 장점이 있다. 어떤 특정 result set을 도출해서 consumer에게 넘겨줄 필요 없이 필요한 연산을 조합하여 바로바로 처리할 수 있다. 

### Functional Behavior

Stream은 functional 하기 때문에 기존 데이터를 변경시키지 않는다. 따라서 result set도 immutable (연산 중에) 하기 때문에 병렬 처리에 강하다. <br>

<br>

참고 링크:

- [https://bk-investing.tistory.com/42](https://bk-investing.tistory.com/42)
- [https://javaconceptoftheday.com/collections-and-streams-in-java/#:~:text=Difference Between Collections Vs Streams In Java %3A&text=Collections are mainly used to store and group the data,or remove elements from streams](https://javaconceptoftheday.com/collections-and-streams-in-java/#:~:text=Difference%20Between%20Collections%20Vs%20Streams%20In%20Java%20%3A&text=Collections%20are%20mainly%20used%20to%20store%20and%20group%20the%20data,or%20remove%20elements%20from%20streams).
- [https://www.baeldung.com/java-return-stream-collection](https://www.baeldung.com/java-return-stream-collection)

```toc
```