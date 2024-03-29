---
emoji: 🐡
title: "이펙티브 자바 - 아이템 11 & 12"
date: '2021-12-19 23:00:00'
author: 코다
tags: 책 자바
categories: 책 자바
---

> 이 글은 몇몇 크루들과 이펙티브 자바 스터디를 하며 정리한 내용입니다. 🙌

## 🌩 [아이템 11] equals를 재정의하려거든 hashCode도 재정의하라

`equals`와 함께 `hashCode`도 재정의하지 않으면 `HashMap`이나 `HashSet`의 원소로 사용할 때 일관성이 무너진다. 

Object 명세에 따르면 다음 규약이 있다. 

> eqauls(Object)가 두 객체를 같다고 판단하면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다. 

다시 말해서 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다는 것이다. 해시코드가 같지 않으면 다음 코드에서 일관성이 깨진다. 

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "사용자");

String result = m.get(new PhoneNumber(707, 867, 5309));
// result.equals("사용자") != false 이다.

```

<br>

### 좋은 hashCode를 작성하기

1. hashCode의 로컬 int 변수 result를 첫번째 핵심 필드의 해시코드로 초기화 한다. (여기서 해시코드는 다음 2.a 단계대로 계산한다.)
2. 다음 핵심 필드 들에 대해서 다음과 같이 해시코드를 계산하고 result 필드를 갱신한다. 
    1. 기본 타입 필드라면 Type.hashCode(f)를 수행한다. Type은 해당 기본 타입의 박싱 클래스다. 
        
        만일 참조 클래스라면 hashCode를 재귀적으로 호출할 수 있으니 이 필드의 표준형을 만들어서 해당 표준형(canonical representation)의 hashCode를 호출하고 null 이라면 0을 사용한다. 
        
        필드가 배열이면 각각의 원소를 별도의 필드로 다루어 해시코드를 계산한다. Arrays.hashCode를 사용할 수 있다. 
        
    2. a에서 계산된 해시코드로 result를 갱신한다. 
        
        `result = 31 * result + c;` 이다.
        
3. result를 반환한다. 
    
    
- 참고로 파생 필드는 제외해도 된다.
- equals에서 사용되지 않는 필드는 반드시 제외한다.
- 단계 b에서 31 * result를 하는 순서에 따라서 result 값이 달라지므로 해시 효과를 높여준다. 그렇지 않으면 anagram인 경우 해시코드가 같아지면서 성능이 저하된다. (O(n)이 된다) 31은 홀수이면서 소수이기 때문에 적합하다. (그 이유는 책에 더 자세히 나와있다)

이렇게 완성된 hashCode의 예시를 보면 다음과 같다. 

```java
@Override public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```

**Objects 클래스의 정적 메서드 활용하기**

- 임의 개수의 객체를 받아서 해시코드를 계산해주는 정적 메서드 hash를 제공한다.
- 하지만 속도는 다수 느리다. 입력 인수를 위한 배열이 만들어지고 박싱/언박싱이 일어나기 때문에 성능이 민감한 요소라면 사용하지 않는 것이 좋다.

```java
@Override 
public int hashCode() {
	return Objects.hash(lineNum, prefix, areaCode);
}
```

- 클래스가 불변이고 해시코드 계산 비용이 크다면 캐싱 방식을 고려하는 것이 좋다.
- hashCode를 지연초기화 하는 것도 하나의 방법인데 이때는 스레드 안전성을 고려하여 구현해야 한다.
- 성능을 높이기 위해 핵심 필드를 생략해서 해시코드를 계산하면 안된다.
- hashCode의 생성 규칙을 API 사용자가 자세히 알 필요가 없다. 클라이언트가 이 값에 의지하지 않게 되고 이후에 계산 방식을 바꿀 수도 있다.

<br>

## 🌩 [아이템 12] toString을 항상 재정의하라

Object에서 기본적으로 제공하는 `toString`은 `클래스명@16진수_해시코드` 를 주로 반환하여 사용자가 필요한 형태로 반환하는 경우가 거의 없다. `toString` 은 **간결하며 사용자가 읽기 쉬운 형태의 유익한 정보**를 반환해야 한다. 따라서 이 메서드를 항상 재정의 하는 것이 필요하다. 

`toString`을 잘 구현한다면 `println`, `printf`, `+ 연산자`, `assert` 등에서 유용하게 사용되며 시스템 디버깅이 훨씬 용의해진다. 또한 직접 호출하지 않아도 오류 메세지 로깅 시 매우 읽기 좋은 유용한 정보를 표시한다. 

`toString`은 해당 객체가 가진 주요 정보를 모두 반환하는 것이 좋다. 하지만 객체가 너무 크다면 표현하기 무리가 있다. 이럴 경우에는 요약 정보를 담는 것이 좋다. **어쨋든 자기 스스로를 굉장히 잘 표현한 문자열이어야 한다.** 

toString을 구현할 때 반환값의 형식을 문서화 할지 정하는데, 값 클래스라면 문서화 하는 것이 좋다. 그대로 입출력에서 사용하고 CSV 파일처럼 사람이 읽을 수 있는 데이터 객체로 저장될 수 있다. 이 경우 문자열과 객체를 상호 전환할 수 있는 생성자나 정적 팩토리 메서드를 제공해주는 것이 좋다. 

포맷을 명시하는 것의 단점으로는 그 포맷이 평생 쓰이게 된다. 따라서 다른 포맷을 적용하고 싶다면 해당 포맷에 맞추어서 파싱하고 새로운 객체를 만들어서 데이터를 저장하는 코드를 작성해야 한다. 그 다음에 포맷이 바뀌게 된다면 굉장히 번거로워진다. 

또한 toString에서 반환하는 정보에 대한 접근자를 각각 제공하는 것이 좋다. 그렇지 않다면 사용자는 toString을 파싱해서 사용할 수밖에 없어진다. 

정적 유틸리티 클래스는 toString을 제공할 필요가 없으며 열거 타입도 마찬가지이다. 

추상 클래스같은 경우 하위 클래스에서 공통적으로 사용해야할 문자열 표현이 있다면 추상 클래스에서 toString을 재정의해야한다.

```toc
```