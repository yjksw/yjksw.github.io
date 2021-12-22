---
emoji: 🐡
title: "이펙티브 자바 - 아이템 9 & 10"
date: '2021-12-18 23:00:00'
author: 코다
tags: 책 자바
categories: 책 자바
---

> 이 글은 몇몇 크루들과 이펙티브 자바 스터디를 하며 정리한 내용입니다. 🙌


## 🌩 [아이템 9] try-finally 보다는 try-with-resources를 사용하라

자바에서 close 메서드를 직접 호출해서 닫아주어야하는 자원들 

- `InputStream`, `OutputStream`, `java.sql.Connection` 등등

<br>

### try-finally 사용시 단점

- try-finally를 사용한다면 닫아야 하는 자원이 많아질수록 매우 복잡해진다.
- try 블록과 finally 블록에서 모두 예외가 발생할 수 있는데, 만일 둘다 예외가 발생했을 경우 이후에 일어난 예외가 첫번째 예외를 삼켜서 디버깅을 어렵게 한다.

<br>

### try-with-resources로 해결

- 짧고 간결하여 읽기가 수월하다.
- try 내부와 `close()`에서 모두 예외가 발생 하더라도 첫번째 예외만 보여지고 두번째 예외는 `suppressed` 되어 출력된다.
- catch 블록을 함께 사용하여 여러 예외를 처리할 수 있다.

<br>

## 🌩 [아이템 10] equals는 일반 규약을 지켜 재정의하라

대부분은 `equals`를 재정의 하지 않는 것이 가장 좋다. 책에서는 특히나 다음과 같은 경우면 재정의 하지 않는 것을 추천한다. 

1. 각 인스턴스가 본질적으로 고유하다. 
2. 인스턴스의 '논리적 동치성(logical equality)'를 검사할 일이 없다. 
3. 상위 클래스의 equals가 하위 클래스에도 적합하게 구현되어 있다. 
4. 클래스가 private이거나 package-private이고 equals를 호출할 일이 없다. 
5. 인스턴스가 하나만 만들어지도록 통제되는 싱글톤이나 enum 인 경우이다.

**다음과 같은 경우는 equals를 재정의 하는 것이 좋다.** 

- 객체의 식별성(identity)가 아닌 논리적 동치성을 확인해야 할 때 (VO 같은 경우)

<br>

### Object 명세에 적힌 일반 규약

> equals는 동치관계(equivalence relation)을 구현하며 다음을 만족한다. 

`Object` 명세에서 말하는 **동치관계**란 무엇일까? 

- 집합을 서로 같은 원소의 부분집합으로 나누고 같은 집합 속에 있는 원소는 서로 언제나 교환될 수 있어야 하는 것이다.

<br>

첫 번째 조건, **반사성**

- 객체는 자기 자신과 같아야 한다.
- `x.equals(x) == true`

<br>

두 번째 조건, **대칭성** 

- 두 객체의 동치 여부는 같아야 한다.
- `x.equals(y) == true`면 `y.equals(x) == true`
- 다음 예에서는 대칭성이 만족되지 않은 경우이다.
    
    ```java
    public final class CasInsensitiveString {
    	private final String s;
    
    	public CasInsensitiveString(String s) {
    		this.s = Objects.requireNonNull(s);
    	}
    
    	@Override
    	public boolean equals(Object o) {
    		if (o instanceof CasInsensitiveString) {
    			return s.equalsIgnoreCase((CasInsensitiveString) o).s;
    
    		if (o instanceof String) { // 문제가 되는 지점 !!! 
    			return s.equalsIgnoreCase((String) o);
    		}
    
    		return false;
    }
    ```
    
    - 위 코드에서는 `CasInsensitiveString` 클래스 인스턴스 뿐 아니라 String에 대해서도 equals 메서드를 지원한다. 여기서 문제는 `CasInsensitiveString**.**equals(String)` 의 경우는 잘 성립하는데 `String.equals(CasInsensitiveString)`는 성립하지 않는다.
    - 이렇게 대칭성이 성립되지 않으면 해당 객체를 사용하는 다른 객체들의 결과를 예측하기가 어렵다.

<br>

세 번째 조건, **추이성**

- a, b가 동치고 b, c가 동치면 a, c도 동치이다.
- 주로 구체 클래스의 하위 클래스에서 추가가 된 필드에 대해서 equals를 추가하려고 할 때 발생한다.
    
    ```java
    // 추이성이 깨지는 경우
    ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
    Point p2 = new Point(1, 2); 
    ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
    
    p1.equals(p2) == true; // 위치만 비교
    p2.equals(p3) == true; // 위치만 비교
    
    p1.equals(p3) == false; // 색상까지 비교 
    ```
    
- `ColorPoint`의 `equals()` 내부 구현을 보면 다음과 같이 되어 있다.
    
    ```java
    @Override
    public boolean equals(Object o) {
    	if (!(o instanceof Point)) {
    		return false;
    	}
    
    	// 만일 o가 Point면 좌표만 비교한다. 
    	if (!(o instanceof ColorPoint)) 
    		return o.equals(this);
    	}
    
    	return super.equals(o) && ((ColorPoint o).color = color;
    ```
    
    - 이런 식으로 구현이 되었을 때 주의할 점은 무한 재귀에 빠질 수도 있다는 것이다. 위와 같이 구현된 `SmellPoint` 를 만들고 `myColorPoint.equals(mySmellPoint)`를 호출하면 무한 재귀로 인해 `StackOverflowError`를 일으킨다.
    - `instanceof` 대신 `getClass()` 를 쓸수 있지만 이것은 LSP를 위반하여 하위 클래스가 상위 클래스 대신 대체될 수 없다.
- 가능한 해결 방법은 추상 클래스의 하위 클래스를 활용하던지 상속보다 컴포지션 룰을 적용하던지이다.
    
<br>

네 번째 조건, **일관성**

두 객체가 같다면 수정이 되기 전까지 앞으로도 같아야 한다는 뜻이다. 특히 불변 클래스라면 한번 같다면 영원히 같아야 한다. 

**일관성** 조건을 만족시키기 위해서는 equals에 신뢰할 수 있는 자원만 들어와야 한다. 예를 들어서 특정 URL의 IP주소가 그 자원 중 하나라면 이것은 네트워크에 따라서 매번 달라질 수 있기 때문에 항상 결과가 같지 않다. 

<br>

다섯 번째 조건, **null-아님**

모든 객체가 null과 같지 않아야 한다는 뜻이다. 동치성을 검사하기 위해서 받은 객체를 적절히 형변환 하여 필드 값을 알아내기 때문에 여기서 `instanceof` 로 검사할 때 `ClassCastException`으로 예외를 발생시킨다. 

<br>

### 좋은 equals 메서드 구현하기

1. `==` 를 통해 자기 자신이면 `true` 반환한다. 
2. `instanceof` 로 입력의 타입을 검증한다. 
3. 입력을 올바른 타입으로 형변환 한다. 
4. 입력 객체와 자신의 **핵심 필드**는 모두 일치하는지 검사한다. 

- 자바에서 float와 double 같은 경우는 정적 메서드인 `Float.compare`와 `Double.compare`로 비교한다. 특수한 부동소수 값 때문이다.
- null을 정상값으로 취급하는 참조 타입 필드가 있다면 `Objects.equals(Object, Object)`를 사용하여 NPE를 방지하자.
- `CaseInsensitiveString` 처럼 다루기 복잡한 클래스라면 해당 필드의 표준형(canonical form)을 저장하여 표준형 끼리 비교하자. 가변이라면 이 표준형을 매번 갱신해야해서 어렵지만 불변이라면 더욱 적합하다.

<br>

### equals의 성능 고려하기

- 다를 가능성이 크거나 비교하는 비용이 싼 필드를 먼저 비교하는 것이 좋다.
- 객체의 논리적 상태와 관련이 없는 필드(락 필드 등등)는 비교하지 않도록 한다.
- equals를 재정의할 때 hashCode도 함께 재정의 해야 한다.
- 필드의 동치성만 검사해도 equals 규약이 대부분 만족되니 너무 복잡하게 구현하지 않는 것이 좋다.
- equals의 매개변수는 반드시 Object 이도록 한다.

```toc
```