---
emoji: 🐡
title: "이펙티브 자바 - 아이템 7 & 8"
date: '2021-12-17 23:00:00'
author: 코다
tags: 책 자바
categories: 책 자바
---

> 이 글은 몇몇 크루들과 이펙티브 자바 스터디를 하며 정리한 내용입니다. 🙌

## 🌩 [아이템 7] 다 쓴 객체 참조를 해제하라

JVM 언어를 사용한다면 GC가 알아서 사용되지 않는 객체를 해제할텐데 왜 이런 항목이 있는걸까? 

다음과 같은 경우에는 GC가 해당 객체가 다 쓴 객체인지 아닌지 판단할 수가 없다. 

Stack 자료구조를 구현한 예시이다. 

```java
public class Stack {
	
	private Object[] elements'
	private int size = 0;
	...// 이외 필드 및 메서드

	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}
		
		return elements[--size];
	}
}
```

다음과 같은 경우 다시 참조되지 않을 객체에 대한 해제가 이루어지지 않았기 때문에 '메모리 누수'가 발생한다. 이 프로그램이 오랜시간 실행이 되다보면 메모리 사용량이 늘어나 성능이 저하되거나 심하면 `OutOfMemoryError`를 일으킬수도 있다. 

위 코드와 같은 경우 `elements` 배열에 더이상 사용되지 않는 영역의 객체 참조를 배열이 여전히 가지고 있다. 예를 들면 `size` 바깥에 존재하는 객체들에 대해서 말이다. 

이렇게 GC에게 객체 처리를 맡기는 언어에서 이러한 메모리 누수를 찾기가 매우 어렵다. 또한 쓰이지 않지만 참조되고 있는 객체의 영향이 해당 객체에서 끝이 나는 것이 아니라 해당 객체들을 췸조하는 다른 여러 객체들도 회수할 수 없다. 

따라서 다음과 같이 null 처리를 통해 참조 해제를 해야 한다. 

```java
public Object pop() {
	if (size == 0) {
		throw new EmptyStackException();
	}
	
	Object result = elements[--size];
	element[size] = null;
	return result;
}
```

<br>

### 객체 참조를 직접 해제해야 하는 경우

그렇다고 항상 객체 참조를 하나하나 null 처리 해야한다는 것은 아니다. 이 로직은 가독성을 저하시키고 프로그램을 지저분하게 만든다. 객체 참조를 null 처리를 통해 직접 해제해야하는 경우는 **자기 메모리를 직접 관리하는 클래스**인 경우이다. 

예를 들어 `Stack` 클래스의 경우를 살펴보자. 이 클래스의 경우 객체가 아니라 객체 참조를 담는 element 배열을 사용한다. 그러니까 JVM에서 직접 객체 참조를 관리하는 것이 아니라 관리하는 영역을 따로 만든 것이다. 그렇기 때문에 어디부터 어디까지가 유효한 참조범위인지 GC는 알 수 없다. 

<br>

### 메모리 누수가 자주 발생하는 상황

또 다른 예로는 캐시가 있다. 캐시에 특정 객체를 넣어놓고 인지하지 못하면 메모리 누수가 발생한다. 이 경우 저자가 제안하는 몇가지 해결방법이 있다. 

첫 번째는 WeakHashMap을 사용하여 해당 캐시의 key 값이 더 이상 참조되지 않는다면 해당 엔트리를 삭제하는 것이다. 

```java
WeakHashMap<Integer, String> map = new WeakHashMap<>();

Integer key1 = 1000;
Integer key2 = 2000;

map.put(key1, "값1");
map.put(key2, "값2");

key1 = null;
```

이렇게 사용할 경우 key1에 대한 엔트리 값이 자동으로 GC에 의해 수거된다. 

<br>

> 다음 단락은 개인적인 의견 입니다 !!

위 방법은 추천하지 않는다. 우선 Map 자료구조의 `key`나 `value`의 값이 불변과 `final`이 아닐 경우 찾기 매우 어려운 버그가 발생하기 너무 쉽다고 생각한다. 특히나 `key`와 같은 경우는 불변이나 `final`이 아닐 경우 개발자가 예상하지 못하는 시점에 변경이 일어나고 의도하지 않은 `key`와 `value`가 매핑되어 있을 수 있다. 

두 번째 방법은 엔트리의 가치를 시간에 따라 점점 떨어뜨리고 오래된 엔트리를 주기적으로 청소해주는 것이다. `ScheduledThreadPoolExecutor`를 사용하여 백그라운드 스레드로 유효기간이 지난 엔트리를 삭제하는 로직을 실행할 수 있다. 또는 `LinkedHashMap` 자료그조의 `removeEldestEntry()` 를 사용하여 오래된 엔트리를 삭제할 수 있다. (추가로 복잡한 캐시 처리에는 `java.lang.ref` 패키지를 사용하라고 제안한다. 여러 종류의 객체 참조에 대한 패키지인데 자세한 것을 이 [링크]([https://d2.naver.com/helloworld/329631](https://d2.naver.com/helloworld/329631))를 참고해보자.) 

메모리 누수가 자주 발생할 수 있는 마지막 경우는 리스너(Listener) 혹은 콜백(callback)이다. 콜백을 등록하고 해제하지 않으면 콜백이 쌓이며 메모리 누수가 발생한다. 

<br>

## 🌩 [아이템 8] finalizer와 cleaner 사용을 피하라

### 수행 시점을 보장할 수 없다

finalizer와 cleaner의 큰 단점은 어느 시점에 수행되는지 보장할 수 없다는 것이다. 따라서 반드시 닫아야 하는 파일 닫기와 같은 로직을 finalizer 혹은 cleaner로 수행한다면 파일이 언제 닫힐지 몰라 열어두게 되고 한계를 넘으면 프로그램이 실패한다. 

finalizer의 스레드는 다른 스레드에 비해 우선순위가 낮아서 오랜시간 처리가 되지 않을 수도 있다. 반면 cleaner는 해당 스레드를 제어할 수 있지만 여전히 GC의 통제하에 수행하기 때문에 실제로 수행하는 시점을 알기 어렵다. 

<br>

### 수행 여부를 보장할 수 없다

수행 시간도 큰 단점이지만 수행 여부에 대한 보장도 되지 않는다. 해당 작업이 종료되었는지와 상관없이 프로그램이 종료될 수 있기 때문에 프로그램 내부의 일시적인 것이 아닌 영구적은 수정 작업에서 finalizer나 cleaner를 사용하게 되면 시스템 자체가 서서히 죽어갈 수도 있다. 

- 예를 들면 DB와 관련된 공유 자원의 lock 해제 등의 로직 등이 제대로 이루어졌는지 확인되지 않고 프로그램이 종료될 수 있는 것이다. 그러면 엄청난 성능 저하가 일어나고 시간이 지나면 왜인지 모른체 시스템이 다운 될 수 있다.

<br>

### 예외 발생시 알 수 없다

또 다른 큰 단점은 finalizer 수행 중 예외가 발생하면 **1) 예외는 무시되지만 2) 작업은 순간 종료된다**(남은 작업을 처리하지 않는다). 예외가 발생해도 경고도 출력되지 않고 무시되며 프로그램은 여전히 실행되기 때문에 어떤 객체가 중단되었고 훼손 되었는지 알 수 없다. cleaner는 자신의 스레드를 통제하기 때문에 관련 문제가 적게 발생한다. 

<br>

### 성능 문제가 발생한다

try-with-resources를 사용한 경우 객체 생성 후 GC가 수거하기까지 12ns가 걸리지만 finalizer를 사용하면 550ns 가 걸린다. 즉, 50배의 성능저하가 일어나는 것이다. 안전망 형태 (추후 설명)로 사용하면 빨라지지만 여전히 5배가 느려진다. GC의 성능 저하를 초래하는 것이다. 

<br>

### finalizer 공격에 노출된다

예를 들어 다음과 같은 경우이다. 

- 생성자나 직렬화 과정에서 예외가 발생하여 불완전한 객체가 생긴다.
- 이때 악의적인 하위 클래스의 finalizer에서 정적 필드에 자기 자신을 참조하여 GC가 불완전한 해당 클래스를 수집하지 못하게 된다.
- 이렇게 일그러진 객체가 생성되어 일반적으로 허용되지 못하는 작업을 수행하게 될 수도 있다.

위 상황을 예방하기 위해서는 다음 방법들을 사용해보자. 

- 객체를 final로 설정하여 하위 클래스 생성을 막자. (하지만 이것은 하위 클래스로의 확장 가능성 자체를 막아버리는 단점이 존재한다)
- final이 아닌 클래스라면 아무일을 하지 않는 finalize 메서드를 만들고 final로 선언한다.

<br>

### 가장 좋은 해결 방법

- `AutoCloseable`을 구현하고 close 메서드를 호출해라.
- 혹은 `try-with-resource`를 사용해라.
- 번외로 각 객체는 자신이 닫혔는지를 항상 확인 하는 것이 좋다.  `close()` 메서드 내부에서 닫힐 때 필드에 저장하고 해당 객체를 호출할 때 필드를 검사해 만일 닫힌 객체였다면 `IllegalStateException`을 던져야 한다.

<br>

### 그럼 언제 finalizer/cleaner를 사용하면 좋을까

- close를 호출하지 않은 것에 대한 안전망 역할
    - 자바의 경우 `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor` 는 안전망 finalizer를 제공한다.
- 네이티브 피어(native peer)와 연결된 객체 관련 메모리 해제
    - 네이티브 피어는 네이티브 메서드로 기능을 위임한 네이티브 객체이다.
    - 자바 객체가 아니므로 GC는 그 존재를 알지 못한다.
    - 하지만 당장 자원이 회수되거나 성능 저하를 감수하지 않으려면 close 메서드를 사용해야 한다.
- cleaner 같은 경우는 Autocloseable의 안전망으로 많이 사용된다.
    - 예를 들어 아래 같은 경우에는 cleaner가 반드시 회수해야할 자원인 `numJunkPiles`를 청소한 후 객체를 닫는 것이다.
    
    ```java
    public class Romm implements AutoCloseable {
    	private static final Cleaner cleaner = Cleaner.create();
    
    	private static class State implements Runnable { //static class로 선언 !! 
    		int numJunkPiles;
    
    		State(numJunkPiles) {
    			this.numJunkPiles = numJunkPiles; // 본래는 이 부분이 native peer 참조를 담는 final long 변수
    		}
    
    		@Override
    		public void run() {
    			System.out.println("방 청소");
    			numJunkPiles = 0;
    		}
    	}
    
    	private final State state;
    
    	private final Cleaner.Cleanable cleanable;
    
    	public Room(int numJunkPiles) {
    		state = new State(numJunkPiles);
    		cleanable = cleaner.register(this, state);
    	}
    
    	@Override
    	public void close() {
    		cleanable.clean();
    	}
    }
    	
    ```
    
    - 위 코드에서 run 메서드가 호출되는 경우는 Room의 close 메서드를 호출하거나 cleaner가 State의 run 메서드를 호출하는 경우이다. (안정망)
    - **여기서 주의할 점은 State 인스턴스가 Room 인스턴스를 참조하지 않는 것이다.** 만일 참조하게 된다면 순환참조가 생겨서 GC가 Room 인스턴스를 절대로 회수 할 수 없다. 따라서 위 코드에서는 State가 정적 중첩 클래스로 구현이 되어 있다. 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖는다. (람다도 마찬가지이므로 조심해야 한다.)
    - Cleaner는 단순한 안전망이기 때문에 반드시 `try-with-resource`를 사용하여 즉각적으로 `close()` 메서드가 호출되도록 구현하는 것이 가장 좋다.

```toc
```