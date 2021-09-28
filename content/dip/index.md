---
emoji: 🍀
title: DIP 의존관계 역전의 원칙
date: '2021-03-13 23:00:00'
author: 코다
tags: 스프링 웹
categories: 스프링 웹
---

스프링 강의 중 DAO vs. Repository의 차이점에 대해서 논의하다가 다음과 같은 표현이 나왔다. 

- Repository의 추상 인터페이스는 Domain Layer에 속하며 Domain 객체들을 관리하고 생애주기를 같이한다. 그 구현체인 SimpleJpaRepository는 Infrastructure에 속한다. 추상화된 repository 인터페이스를 사용하면서 추상에 의존하고 구체에 의존하지 않도록 구성(DIP) 하여 유연성 있는 시스템을 구성한다.

여기서 나오는 DIP는 무엇이고 위와 같은 구성이 어떻게 우연성을 제공하는 걸까? 

### DIP 요약

- **Dependency Inversion Principle**의 약자이다.

본래 객체는 상위 계층이 하위 계층에 의존한다. DIP는 그 관계를 역전시켜서 상위 계층이 하위 계층의 구현에서 독립하도록 한다. 그러기 위한 원칙 두가지는 다음과 같다. 

1. 상위 모듈과 하위 모듈이 서로 의존하는 것이 아니라 모두 추상화에 의존한다. 
2. 추상화가 구현에 의존하는 것이 아니라 구현이 추상화에 의존해야 한다. 

한마디로 요약하면 다음이다. <br>

**"The DIP is about inverting the classic dependency between high-level and low-level components by abstracting away the interacting between them."**

출처 : [https://www.baeldung.com/java-dependency-inversion-principle](https://www.baeldung.com/java-dependency-inversion-principle) <br>

즉 상위 모듈과 하위 모듈 사이에 추상화를 껴서 서로를 의존하는 것이 아니라 인터페이스를 통해서 협력하도록 하는 것이다. 

### DIP 구현

다음과 같은 구현이 있을 때 어떤 것이 DIP 일까?

```sql
public class StringProcessor {
    
    private final StringReader stringReader;
    private final StringWriter stringWriter;
    
    public StringProcessor(StringReader stringReader, StringWriter stringWriter) {
        this.stringReader = stringReader;
        this.stringWriter = stringWriter;
    }

    public void printString() {
        stringWriter.write(stringReader.getValue());
    }
}
```

1. `StringReader`, `StringWriter` 가 인터페이스이고 `StringProcessor`와 같은 패키지에 존재한다. (구현체는 함께 있지 않다)
2. `StringReader`, `StringWriter`가 인터페이스이면서 `StringProcessor`와 다른 패키지에 존재한다. 

즉 구현체와 따로 분리되어 있어서 StringProcessor는 인터페이스에 의존하고 인터페이스는 언제나 변경이 가능하여 구현체에 존재하지 않도록 한다.

```toc
```