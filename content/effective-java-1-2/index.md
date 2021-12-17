---
emoji: 🐡
title: "이펙티브 자바 - 아이템 1 & 2"
date: '2021-12-14 23:00:00'
author: 코다
tags: 책 자바
categories: 책 자바
---

> 이 글은 몇몇 크루들과 이펙티브 자바 스터디를 하며 정리한 내용입니다. 🙌

## 🌩 [아이템 1] 생성자 대신 정적 팩터리 메서드를 고려하라

### 정적 팩터리 메서드(static factory method)란?

해당 클래스의 인스턴스를 반환하는 정적 메서드이다. 

예를 들어 다음 박싱 클래스를 반환하는 정적 메서드를 보자.

```java
public static Boolean valueOf(boolean b) {
	return b? Boolean.TRUE : Boolean.FALSE;
}
```

이 챕터에서 추천하는 바는 public 생성자 대신 정적 팩터리 메서드를 고려해보라는 것이다. 정적 팩터리 메서드는 장단점이 있다. 따라서 항상 정적 팩터리 메서드가 더 우수하다는 것은 아니다. 다만 습관적으로 public 생성자를 사용하기 이전에 상황을 살피고 정적 팩터리 메서드가 더 적합하다면 해당 메서드를 사용하도록 추천한다. 

<br>

### 장점

1. 이름을 가질 수 있다. 
    
    여러 종류의 생성자가 필요로 할 경우 매개변수의 수를 드라게 해서 추가하곤 한다. 하지만 이 매개변수와 생성자 자체로는 반환될 객체의 특성이나 언제 이 생성자가 사용이 되는지 설명할 수 없다. 
    
    따라서 이름을 가진 정적 팩터리 메서드를 사용한다면 개발자가 각 생성 메서드가 무엇을 의짐하는지 알 수 있고 그 차이를 잘 드러낼 수 있다. 
    
2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다. 
    
    미리 만들어놓은 인스턴스나 새로 생성한 인스턴스 캐싱하여 재사용하는 추가 로직을 가지고 있을 수 있다.
    
    - 예를 들어 `Boolean.valueOf()` 메서드는 객체를 아예 생성하지 않고 미리 생성된 객체만을 반환한다.
    
    이렇게 미리 만들어놓은 인스턴스를 활용하는 것은 해당 코드의 성능도 높여주지만 개발자가 반환되는 인스턴스를 철저하게 통제할 수 있다는 이점도 제공한다. 이러한 클래스를 인스턴스 통제(instance-controlled) 클래스라고 한다. 
    
    - 인스턴스를 통제한다면 클래스를 싱글톤(Singleton)이나 인스턴스 불가(noninstantiable)로 만들 수 있다.
    - 불변 값에서 동치를 보장하는 인스턴스를 제공할 수도 있다.
    - Flyweight pattern의 근간이 된다. Flyweight pattern 이란 불필요한 new 연산자를 줄이고 필요한 인스턴스만 만들어 최대한 공유할 수 있도록 한 디자인 패턴이다.
    - 열거 타입이 인스턴스를 하나만 만들어지는 것을 보장하도록 해준다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다. 
    
    반환 할 때 반드시 해당 클래스의 객체만 반환하는 것이 아니라 필요시 하위 타입의 객체를 반환하도록 하는 것은 엄청난 유연성을 가질 수 있게 해준다. 
    
    이 유연성을 활용하면 반환할 때 그 구현체를 공개하지 않고 반환하여 활용할 수 있도록 하므로 코드가 간결하고 작게 유지될 수 있다. 
    
    - 자바 컬렉션 프레임워크의 경우 여러 추가 기능을 덧붙인 45개의 유틸리티 구현체를 제공한다. 이 구현체들은 아래 사진에서 볼 수 있는 것과 같이 `java.util.Collections`에서 정적 팩토리 메서드를 통해서 얻어질 수 있다.

        <p align="center"><img width="75%" src="https://user-images.githubusercontent.com/63405904/146213695-ebca67cd-c845-4eba-8c35-3a97bf683f3f.png"></p>
        
    - 위 클래스들은 공개되어 있지 않기 때문에 자바 컬렉션 프레임워크 자체를 외부에서 바라보았을 때 훨씬 작고, 응용하면서 익혀야하는 개념과 난이도가 대폭 줄어든다.
    - 또한 그 인터페이스가 명시한 대로 동작할 것을 알기 때문에 해당 구현 클래스가 무엇인지 자세히 살펴볼 필요도 없다. 즉, 정적 팩토리 메서드를 사용한 클라이언트는 해당 객체의 구현체가 아닌 인터페이스만으로 다룰 수 있는 이점이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다. 
    
    반환 타입의 하위 타입이기만 한다면 해당 클래스의 객체를 반환할 수 있다. 이것은 매번 다음 릴리즈 때 필요에 의해 다른 클래스의 객체를 반환할 수 있도록 해준다. 
    
    - 예를 들면 `EnumSet` 같은 경우 정적 팩토리 메서드로만 객체를 생성할 수 있는데, 매개변수에 적인 원소의 수에 따라서 `RegularEnumSet` 인스턴스를 반환하거나 `JumboEnumSet` 인스턴스를 반환한다.
    - 다음 릴리즈 때 둘 중 하나의 구현체의 이점이 없어진다면 큰 번거로움 없이 둘 중 하나를 삭제하면 된다.
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. 
    
    **위 유연함을 제공하기 때문에 service provider framework가 가능하다.**
    
    흔히 알고 있는 대표적인 service provider framework는 `JDBC`이다. 여기서 provider는 구현체이다. 이러한 service provider framework에서는 클라이언트에게 구현체를 제공하는 역할을 프레임워크가 담당하기 때문에 클리아언트는 인터페이스로 기능만 제공받고 구현체로부터는 분리될 수 있다. 
    
    클라이언트가 서비스의 인스턴스를 얻을 때 서비스 접근 API를 사용하여 조건에 따라 필요한 구현체를 얻는데 여기서 정적 팩터리 메서드의 유연성이 이것을 가능하게 한다. 
    
<br>

### 단점

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다. 
    
    하지만 이것은 **상속보다 컴포지션을 사용**하고 **불변 타입을 보장**하도록 한다면 이 제약이 그렇게 큰 단점으로 다가오지는 않는다. 
    
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다. 
    
    어떤 메서드를 사용하여 해당 객체를 생성할 수 있는지 규정되어 있지 않으므로 찾기가 어렵다. 다만 흔히 사용하는 몇가지 명명 방식이 있기는 하다. (예를 들면 `from`, `of`, `valueOf`, `instance` 등등.. 필요하면 찾아보길)
    
<br>
<br>

## 🌩 [아이템 2] 생성자에 매개변수가 많다면 빌더를 고려하라

앞서 정적 팩터리 메서드의 장점에 대해서 이야기했다. 하지만 정적 팩터리 메서드도 여전히 한계점이 있는데 그것은 매개변수가 너무 많거나, 그 중 선택적인 요소들이 많을 때 **가독성과 안정성**을 유지시키는 것이 어렵다. 

- 여기서 **가독성이**라고 하는 것은 여러 매개변수가 있을 때 몇번째가 어느 필드에 관한 값인지 알기 어려운 것이다.
- 여기서 **안정성**이라고 하는 것은 setter를 사용했을 때 중간에 일관성이 깨지게 되는 것을 말한다.

<br>

### 생성자(정적 팩터리 메서드)와 setter의 한계점

여러 매개변수가 있고 몇몇 선택적인 요소들이 있을 경우 흔히 사용하는 방법은 **점층적 생성자 패턴**(telescoping constructor pattern)이다. 이 패턴을 사용할 경우 보통 생성자의 수가 너무 많아 가독성이 떨어지거나 설정하고 싶지 않은 매개변수까지 포함하기 쉽다. (따라서 0 또는 null로 해당 값을 입력하게 된다) 

두번째 가능한 방법은 자바빈즈 패턴JavaBeans Pattern이다. 이 경우 생성자는 기본으로 두고 setter를 통해서 원하는 매개변수의 값을 설정하는 것이다. 가독성은 좋아지나 **불변 객체를 포기**해야하며 여러 메서드를 호출해서 각각 초기값을 입력해주어야 하기 때문에 해당 작업이 완료되기 이전에는 **일관성이 보장되지 않은 상태**이다. 따라서 버그 가능성이 매우 높다. (스레드 안정성이 낮다.)

<br>

### 빌더 패턴 Builder Pattern을 통한 대안

빌더 패턴을 사용할 경우 필수 매개변수(초기값이 반드시 있어야 하는 필드)로 빌더 객체를 생성한다. (**해당 객체를 직접 생성하는 것이 아니다.**) 이후 빌더를 통해서 선택 매개변수들을 하나씩 지정하게 되고 `build()` 메서드 호출을 통해 필요 매개변수들이 대입된 객체를 얻게 된다. 

```java
public class User {
	private final String name; // 필수 매개변수
	private final int age; // 필수 매개변수
	private final String school;
	private final String address;

	private User(Builder builder) {
		name = builder.name;
		age = builder.age;
		school = builder.school;
		address = builder.address;
	}

	public static class Builder {
		private final String name;
		private fianl int age;

		private final String school = ""; // 기본값
		private final String address ""; // 기본값

		public Builder(String name, int age) {
			this.name = name;
			this.age = age;
		}

		public Builder school(String school) {
			this.school = school;
			return this;
		}

		public Builder address(String address) {
			this.address = address;
			return this;
		}

		public User build() {
			return new User(this);
		}
	}
}
	
```

이처럼 Builder를 사용한다면 해당 클래스를 불변으로 유지하면서 기본값 매개변수는 반드시 지정하도록, 그리고선택 매개변수는 원하는 것을 선택해서 사용할 수 있도록 한다. 

Builder를 사용한다면 메서드 체이닝 방식으로(fluent API, method chaining) 사용되기 때문에 가독성이 좋고 일관성을 유지시킬 수 있다. 

불변식(invariant)을 검증하고 싶다면 `build()`에서 사용하는 생성자에 검증로직을 넣어서 보장하도록 한다. 

**번외) 불변(immutable) vs. 불변식(invariant)**

불변은 해당 객체에 대한 그 어떠한 변경도 허용하지 않는 것이다. 불변식은 프로그램이 실행되는 동안 반드시 만족해야하는 조건이다. 값에 대한 변경과 상관없이 해당 객체가 가지고 있는 필수 요구사항이라고 할수 있다. (나이는 항상 양수여야 한다 등등 

<br>

### 빌더패턴의 장점 - 계층적 클래스에서의 응용

각 계층에서 각각의 빌더를 두어 사용한도록 한다. 상위 클래스에서는 추상 빌더를, 하위 클래스에서는 구현체 빌더를 사용하도록 한다. Generic과 추상 메서드를 활용한 빌더 패턴이다. 

코드를 통해서 이해하는 것이 훨씬 빠르므로 코드로 확인해보자. 

- 최상위 추상 클래스 `Pizza`

```java
public abstract class Pizza {
	public enum Topping {HAM, MUSHROOM, ONION}
	final Set<Topping> toppings; //상위 클래스 필드

	Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone();
	}

	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class); // element 타입을 지정한 빈 EnumSet 자료구조이다.
		
		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self(); // 하위 클래스에서 그 값이 달라지므로 추상메서드 self()를 반환
		}

		abstract Pizza build(); // 하위 클래스의 생성자 호출

		protected abstract T self(); // 하위 클래스의 Builder 반환 메서드 
	}
}	
```

- 첫 번째 하위 클래스 `NyPizza` - 사이즈 지정이 필수

```java
public class NyPizza extends Pizza {
	public enum Size { SMALL, MEDIUM, LARGE }
	private final Size size;

	private NyPizza(Builder builder) {
		super(builder); // 상위 클래스의 topping을 지정
		size = builder.size; // 하위 클래스의 필드를 지정
	}

	public static class Builder extends Pizza.Builder<Builder> {
		private final Size size;

		public Builder(Size size) {
			this.size = Objects.requireNonNull(size);
		}

		@Override
		public NyPizza build() {
			return new NyPizza(this);
		}

		@Override
		protected Builder self() {
			return this;
		}
	}
}
```

- 두 번째 하위 클래스 `Calzone` - Sauce 추가 여부 지정

```java
public class Calzone extends Pizza {
	private final boolean sauceInside;

	private Calzone(Builder builder) {
		super(builder);
		sauceInside = builder.sauceInside;
	}
	
	public static class Builder extends Pizza.Builder<Builder> {
		private boolean sauceInside = false; // 기본값 지정
		
		public Builder sauceInside() {
			sauceInside = true;
			return this;
		}	
		
		@Override
		public Calzone build() {
			return new Calzone(this);
		}

		@Override
		protected Builder self() {
			return this;
		}
	}
}
```

상위 코드를 잘 이해해보면 계층적 클래스에서 빌더를 통해 구현체의 필수 매개변수를 지정하고 공통된 상위 클래스의 매개변수 지정까지 함께 처리할 수 있는 것을 확인할 수 있다. 

또한 하위 클래스에서 `build()`를 정의하기 때문에 구현체의 객체를 반환할 수 있다. 즉, `NyPizza.Builder`는 `NyPizza`를 반환할 수 있다. (상위 클래스에서는 `Pizza`를 반환하도록 되어 있다.)

이렇게 하위 클래스의 메서드에서 상위 클래스에서 정의한 반환값이 아닌 하위 클래스의 타입을 반환하는 것을 **공변 반환 타이핑(covariant return typing)** 이라고 한다. 이것을 통해 클라이언트가 형변환을 굳이 하지 않고 Builder를 사용할 수 있게 된다. 

위에서 구현한 Builder는 다음과 같이 사용할 수 있다. 

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
		.addTopping(SAUSAGE)
		.addTopping(ONION)
		.build();

Calzone calzone = new Calzone.Builder()
		.addTopping(HAM)
		.sauceInside()
		.build();
```

- Builder의 추가적인 이점
    
    위의 `addToppings()` 처럼 가변인수(varargs) 매개변수를 여러개 지정할 수 있다. 
    
    즉, 개수가 한정되어 있지 않은 가변 인자를 여러개 연달아 지정할 수 있는 이점을 누릴 수 있다. 
    
<br>

### 빌더패턴의 단점

객체를 만들기 전 빌더 관련 로직을 구현하고 생성해야 하므로 성능에 민감한 상황에서 영향을 끼칠 수 있다. 

또한 추가 코드가 매우 많아지기 때문에 매개변수가 적을 때는 오히려 생산성이 훨씬 떨어진다. 

하지만 저자는 API는 일반적으로 시간이 지날수록 매개변수가 많아지기 때문에 나중에 빌더 패턴으로 전환하기보다는 애초에 빌더로 시작하는 것을 추천한다. 

<br>

### 요약 - 언제 빌더를 사용하는게 좋을까?

- 매개변수의 수가 많을 때
- 매개변수의 수가 많고 매개변수 중 다수가 선택 매개변수 일 때 (또는 같은 타입이라서 점층적 생성자 패턴을 사용하지 못할 때)
- 가독성이 필요하고 안정성이 중요할 때

```toc
```