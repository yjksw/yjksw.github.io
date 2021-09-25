---
emoji: 🛠
title: 전략 패턴이란? 
date: '2021-02-30 23:00:00'
author: 코다
tags: 설계
categories: 설계
---

## 전략 패턴(Strategy Pattern)이란?

객체가 할 수 있는 **행위**에 대한 `전략 클래스` 생성하여 해당 행위들을 캡슐화(인터페이스화) 하여 사용하는 것이다. <br>

즉, 행위를 각각의 전략 클래스로 생성하고 수정이 필요한 경우 전략을 바꾸는 것으로 행위를 수정하도록 한다. <br>


## 왜 전략 패턴을 사용해야 할까?

예를 들어 움직이는 Bus, Train 이라는 객체가 있다고 하고 각각 move() 함수를 통해서 움직인다. <br>

그런데, Bus는 도로로 Train은 선로로 움직인다. 만약 이때 버스가 더 이상 길이 아니라 선로로 움직인다고 가정할 때, 버스의 move() 메소드를 선로로 움직이는 로직으로 수정해야 한다. <br>

이때 두 가지 문제가 발생한다. 

1. OCP (Open-Closed Principle)에 위배 : 수정에 닫혀있어야 하는데, `move()` 메소드를 직접 수정
2. 확장이 될 경우 메서드 중복 문제 : `move()` 메소드를 가진 여러 객체가 있을 때 일일이 수정을 해아함

**이 때 전략 패턴을 사용하면, 위 두 문제를 마주하지 않으면서 행위를 수정할 수 있다.** 

## 전략 패턴 구현

- 행위에 대한 메소드를 정의하는 `Strategy` 인터페이스를 생성하고 해당 인터페이스를 구현하는 구현체로 각기 다른 전략 로직을 구현한다. <br>

<p align="center"><img width="80%" src="https://user-images.githubusercontent.com/63405904/134760341-bf9b0e5b-8db9-489f-a529-63a46844a877.png"></p>

- 움직이는 객체인 `Bus`, `Train` 등등에 위 `MovableStrategy` 를 조합하여, `move()` 메소드에서 지정된 전략 패턴으로 실행되도록 한다. (Bus, Train 등을 Movable 이라는 추상 클래스나 부모 클래스로부터 확장을 해서 추가적으로 중복을 줄일 수도 있다.)

    ```java
    public class Moving {
        private MovableStrategy movableStrategy;

        public void move(){
            movableStrategy.move();
        }

        public void setMovableStrategy(MovableStrategy movableStrategy){
            this.movableStrategy = movableStrategy;
        }
    }
    ```

전략 패턴은 상태 패턴과 유사한 매커니즘으로 구조가 되어 있다. 다만 조금 다른 점은 전략 패턴이 대체하고자 하는 것은 `상속` 에 가깝다. 반면 상태 패턴이 대체하고자 하는 것은 수많은 `조건문` 이다. 두 패턴 모두 조합을 통해서 문제를 해결하려고 하기 때문에 그 생김새가 매우 유사하다. [참고]([https://github.com/KWSStudy/Refactoring/issues/2](https://github.com/KWSStudy/Refactoring/issues/2)) <br>


### 출처
- [https://victorydntmd.tistory.com/292](https://victorydntmd.tistory.com/292)

```toc
```


