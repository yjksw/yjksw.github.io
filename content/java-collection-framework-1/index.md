---
emoji: ✊
title: "JCF 파헤치기 1 - 기본 & ArrayList"
date: '2021-12-03 23:00:00'
author: 코다
tags: 자바
categories: 자바
---

## 💡 Intro

- **데이터의 그룹을 저장하는 클래스들을 표준화한 프로그래밍 방식**
- 컬렉션 프레임워크는 다수의 데이터를 다루는 여러 클래스를 제공하여 개발자의 부담을 덜어준다.
- 인터페이스와 다형성을 이용해서 객체지향적으로 설계가 되어 있기 때문에 추상적이고 재사용성이 높은 좋은 프레임워크이다.

<br>

## 🌩 핵심 인터페이스

컬렉션에 담기는 데이터를 크게 3가지로 나누어 각각을 인터페이스로 정의해두었다. 그리고 3가지 중 List, Set의 공통점을 뽑아 따로 인터페이스로 추상화 되어 있다. 

<p align="center"><img width="80%" src="https://user-images.githubusercontent.com/63405904/144552104-6d0d4ba5-92c9-464a-8e14-3580a0972f6c.png"></p>

각 인터페이스와 특징은 다음과 같다. 

1. List: 순서가 있으며 중복이 허용된 데이터의 집합 
    - ArrayList, LinkedList, Stack, Vector, etc.
2. Set: 순서가 없으며 중복을 허용하지 않는 데이터의 집합 
    - HashSet, TreeSet, etc.
3. Map: 키와 값의 쌍으로 이루어진 데이터의 집합이며 순서를 유지하지 않으며 키는 중복을 허용하지 않음
    - HashMap, TreeMap, Hashtable, etc.

모든 컬렉션 클래스는 위 3개의 인터페이스 중 하나를 구현하고 있으며 해당 인터페이스의 이름이 클래스명에 포함되어 있다. (예외로 Vector, Hashtable, Stack 같이 이전에 이미 존재하던 것들은 이름에 인터페이스명을 포함하고 있지 않기도 하다. 기존 호환을 위해 남겨져 있기는 하나 되도록 새로 정의된 컬렉션 클래스를 쓰는 것을 추천한다.)

### Collection 인터페이스

컬렉션 클래스에 저장된 데이터를 읽고, 추가하고 삭제하는 등 컬렉션을 다루는 기본적인 메서드들을 정의

예시) 

- add, addAll, clear, contains, equals, isEmpty, remove, etc.

### List 인터페이스

- 상속계층도

<p align="center"><img width="70%" src="https://user-images.githubusercontent.com/63405904/144552206-76fe42b0-368c-4c8c-a4b3-2907d8c1ac14.png"></p>

### Set 인터페이스

- 상속계층도

<p align="center"><img width="70%" src="https://user-images.githubusercontent.com/63405904/144552215-20cedfd3-3a44-4d56-ab5a-0f69b24fa62d.png"></p>

### Map 인터페이스

- 상속계층도

<p align="center"><img width="70%" src="https://user-images.githubusercontent.com/63405904/144552224-6c84a3c4-28cf-4608-8d19-aad933cfa988.png"></p>

- 값을 반환하는 `values()` 의 반환 타입은 Collection이고, `keySet()`의 반환 타입은 Set이다. 전자는 중복을 허용하고 후자는 중복을 허용하지 않는다.

### Map.Entry 인터페이스

- Map 인터페이스 내부에는 Map.Entry 라는 인터페이스가 하나 더 있다.
- Map에 저장되는 **key-value 쌍을 다루기 위해 내부적으로 정의한 인터페이스**이다.
- 객체지향적으로 설계하도록 유도한 인터페이스이며, Map 인터페이스를 구현하면 Map.Entry 인터페이스도 함께 구현해야한다.

```java
public interface Map<K, V> {
	
	int size();

	boolean isEmpty();

	interface Entry<K, V> {
		
		K getKey();

		V getValue();

		V setValue(V value);

		...

	}
}
```

<br>

## 🌩 ArrayList

- Object 배열을 이용하여 순차적으로 데이터를 저장한다.
    
    ```java
    public class ArrayList extends AbstractList implements List, RandomAccess, Cloneable, java.io.Serializable {
    	...
    	transient Object[] elementData; 
    	...
    }
    ```
    

### 왜 `elementData` 는 `trasient` 설정이 되어 있을까? 🤔

우선 `transient`는 해당 클래스를 직렬화 할 때 직렬화 대상이 되지 않도록 하는 키워드이다. 그럼 핵심 데이터를 담는 `elementData`가 `transient` 설정이 되어 있다면 직렬화 대상에서 제외가 될 텐데 왜 이런 설정이 되어 있을까? 

우선 짚고 넘어가고 싶은 것은 직렬화는 좋은 기술이지만 고려해야할 부분들이 매우 많은 까다로운 기술이라는 것이다. 자세한 것은 다음 [링크](https://www.youtube.com/watch?v=3iypR-1Glm0)를 참고해보자. 

따라서 ArrayList 클래스를 보면 `serialize`, `deserialize` 하는 메소드인 `writeObject()`, `readObject()`를 직접 구현하고 있다. 

### 배열의 초기 크기는 어떻게 산정이 될까? 🤔

**초기 크기를 지정한 경우의 내부 구현을 살펴보자.**

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}
```

- 초기 값이 0 이라면 `EMTPY_ELEMENTDATA` 라는 것을 지정하고 아니라면 입력 값 크기 만큼의 배열을 선언한다.

**초기 크기를 지정하지 않은 경우의 내부 구현을 살펴보자.**

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() {
	this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

- 빈 Object 배열을 선언하여 할당한다.
- 하지만 주로 ArrayList를 활용할 때 초기 크기 뿐 아니라 이후에도 크기를 정해주지 않았으며 그냥 element를 추가만 하는 경우가 많았다.
- 그러면 언제 배열의 크기를 재할당 하는 것일까? 다음 코드를 살펴보자.

```java
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}

public boolean add(E e) { // element를 add 하는 로직
    modCount++;
    add(e, elementData, size);
    return true;
}

private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData, newCapacity(minCapacity));
}

private Object[] grow() { //추가 공간이 필요한 경우 capacity를 늘리는 로직 
    return grow(size + 1);
}
```

- ArrayList는 내부적으로 size라는 인스턴스 변수를 두고 몇개의 요소들이 있는지 트랙킹한다. 새로운 요소를 추가할 때 이 추가 공간이 있는지 확인하고 없다면 `grow()` 메소드를 호출한다.
- `grow()`는 `Arrays.copyOf()`를 사용하여 새로운 크기 만큼의 배열을 생성하고 기존 데이터를 옮긴다.
- 시스템 적으로 오버헤드가 많고 처리시간이 많이 소요되는 작업이므로 ArrayList를 사용할때는 저장할 개수보다 조금 더 여유있기 기본 사이즈를 할당하는 것이 좋다.

<br>

## 🛋 느낀 점

- 내용 추가 예정 ‼️ 안 끝남 주의 ‼️

<br>
<br>

**[참고자료]**
- 자바의 정석
- Java 내부구현 코드

```toc
```