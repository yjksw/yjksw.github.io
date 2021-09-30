---
emoji: 🖥
title: "@Transactional"
date: '2021-05-10 23:00:00'
author: 코다
tags: 스프링부트 웹 데이터베이스
categories: 스프링부트 웹 데이터베이스
---

## 트랜잭션을 사용하는 이유

트랜잭션을 사용하면 각각의 단위로 나누어져있는 작업의 단위를 하나로 합칠 수 있다. 즉, 일련의 연산들을 하나의 독립된 작업 단위로 보고 **하나**로 취급하기 위해서 사용하는 것이다. <br>

언제 일련의 연산들을 하나로 봐야 할 때가 생길까? <br>

예를 들어서 돈을 송금하는 시스템이 있다고 가정해보자. 계좌A에서 계좌B로 돈을 송금해야 할 때, 계좌A에 충분한 잔액이 있는 것을 확인하고 돈을 송금하기 위해서 돈을 차감했다. 그리고 계좌B에 입금을 하려고 하는 순간 예외가 발생하면서 입금을 하지 못했다. 그런데 계좌A에서는 여전히 돈이 차감된 상태이다. 중간에 송금하려고 했던 돈이 사라지게 된 것이다. <br>

이때, 위의 과정을 `@Transactional`로 관리를 하게 된다면 위의 여러 작업들을 하나의 단위로 보고 중간에 예외가 발생한다면 위에서 실행중이던 작업을 한꺼번에 롤백해준다. 

## 트랜잭션 기본 방법

2개 이상의 쿼리를 하나의 커넥션으로 묶어 DB에 전송하고, 에러가 발생할 경우 자동으로 모든 과정을 rollback 한다. 따라서 트랜잭션은 하나 이상의 쿼리를 처리할 때 동일한 connection 객체를 공유하도록 한다. <br>

트랜잭션은 코드기반의 트랜잭션(Programmactic Transaction)과 선언적 트랜잭션(Declarative Transaction)을 지원한다. Spring에서는 어노테이션을 활용한 선언적 트랜잭션을 주로 사용한다. 

## 트랜잭션의 성질

워낙 많은 곳에서 트랜잭션의 성질에 대해서 말하지만 기록을 위해서 그대로 한번 옮겨보았다. 

- 원자성(Atomicity) : 한 트랜잭션 내에서 실행한 작업들을 하나로 간주
- 일관성(Consistency) : 일관성 있는 데이터베이스 상태를 유지
- 격리성(isolation) : 동시에 실행되는 트랜잭션들이 서로 영향을 미치지 않도록 격리. 따라서 트랜잭션의 동시 접근 데이터에 대한 제어가 필요함
- 지속성(Durability) : 트랜잭션 성공시 결과가 항상 저장

## 다수의 트랜잭션 실행 시 발생 가능한 문제상황

**Dirty Read**

- A가 변경 후 커밋하지 않은 상태에서 B가 조회를 한다.
- A가 중간에 문제가 생겨서 롤백되었으면 B는 잘못된 값을 조회한 상황이 된다.

**Non-Repeatable Read** 

- A가 반복적으로 쿼리를 사용하는 사이에 B가 값을 변경하고 커밋을 해서, A의 쿼리 중간에 결과가 달라지는 상황이 된다.
- 한 트랜잭션 안에서 같은 쿼리를 두번 실행했을 때, 데이터 불일치 문제 발생.

**Phantom Read** 

- A가 특정 범위를 조회하는 쿼리를 두번 이상 실행할 때, B가 중간에 값을 추가해서 이후 실행된 A의 쿼리에 이전에 없던 유령 데이터가 생기는 문제가 발상한다.

## 문제상황을 해결하기 위한 격리수준

**사용방법**

`@Transactional(isolation = Isolation.DEFAULT)`

**DEFAULT**

- 기본 격리 수준이며 DB의 격리수준을 따른다.
- MySql → Repeatable-read, Oracle → Read committed

**READ_UNCOMMITED(level 0)** 

- 커밋되지 않은 데이터에 대한 읽기를 허용
- Dirty Read 발생 가능
- 데이터 잠금을 위한 간접 비용이 적고 교착 상태에 빠질 위험이 없어 성능이 빠름

**READ_COMMITTED(level 1)** 

- 커밋 확정된 데이터에 대해서만 읽기를 허용
- Dirty Read 방지

**REPEATABLE_READ(level 2)** 

- 트랜잭션이 완료될 때까지 `SELECT` 하는 데이터에 대해서 shared-lock이 걸리므로 해당 데이터는 수정이 불가하다.
- 트랜잭션이 종료되기 전가지 한번 조회한 값은 계속 같은 값으로 조회되도록 한다.
- `Non-Repeatable Read` 문제를 방지한다. (같은 값을 반복적으로 조회할 때 중간에 다른 값이 조회되는 문제)

**SERIALIZABLE(level 3)**

- 트랜잭션이 완료될 때까지 `SELECT` 하는 데이터에 대해서 shared-lock이 걸리므로 해당 데이터는 수정과 입력이 불가하다.

하지만 격리수준이 올라갈 수록 성능 저하의 우려가 있다는 것을 명심하자. 

## Transaction 안에 Transaction : 전파속성

하나의 트랜잭션 안에서 또 다른 트랜잭션이 발생하면 어떻게 처리가 될지 전파속성에 따라서 다르게 관리할 수 있다. 여러가지 전파속성이 있지만,  그중 몇가지만 다루어보자. 

**사용방법**

`@Transactional(propagation=Propagation.REQUIRED)`

**PROPAGATION_MANDATORY**

부모의 트랜잭션 내에서 실행되고 없으면 예외가 발생한다. 

**PROPAGATION_NESTED** 

기존에 트랜잭션이 있는 경우, 포함하여서 실행한다. 

**PROPAGATION_NEVER** 

트랜잭션이 있는 상황에서 다른 트랜잭션이 실행된다면 예외를 발생한다. 

**PROPAGATION_REQUIRED (기본설정)** 

트랜잭션이 있으면 그 상황에서 실행하고, 없으면 새로운 트랜잭션을 실행한다. 즉, 부모의 트랜잭션에서 실행하거나, 없으면 새로운 트랜잭션을 생성하는 것이다. 

**PROPAGATION_REQUIRED_NEW**

자신만의 고유한 트랜잭션을 실행한다. 

- 전파속성 관련 참고링크: [https://happyer16.tistory.com/entry/트랜잭션-전파-속성-propagation-롤백-예외](https://happyer16.tistory.com/entry/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EC%A0%84%ED%8C%8C-%EC%86%8D%EC%84%B1-propagation-%EB%A1%A4%EB%B0%B1-%EC%98%88%EC%99%B8)

## Transaction 추가 옵션

`@Transactional(readOnly = true)`

- 트랜잭션 작업 내에서 쓰기 작업이 일어나면 예외를 발생시킨다.

`@Transactional(rollbackFor = Exception.class)`, `@Transactional(rollbackForClassName={"NullPointerException"})`

- 기본적으로는 런타임 예외에 대해서 롤백을 하는데, 그 대상을 바꿀 수 있다.

`@Transactional(timeout = 10)`

- 지정한 시간 내에 작업을 완료하도록 설정할 수 있다.

`@EnableTransactionManagement`

- 빈 인스턴스에 트랜잭션을 적용하도록 한다.

<br>

## 트랜잭션 동작 원리

일반적으로 JPA를 사용할 때 스프링과 같은 IoC 컨테이너를 사용하지 않을 경우 transaction management 기능을 직접 구현해줘야 한다. 트랜잭션의 흐름은 다음과 같다. 

```java
UserTransaction utx = entityManager.getTransaction();

try {
	utx.begin();
	businessLogin();
	utx.commit();
} catch (Exception e) {
	utx.rollback();
	throw e;
}
```

## 스프링에서 `@Transactional`

스프링에서 트랜잭션을 알아서 관리해주지만 troubleshooting을 위해서 내부 동작 원리를 알아야 할 필요가 있다. <br>

이해하기 위해서 필요한 개념 2가지

- the persistence context
- the database transaction

두번째 개념인 database transaction은 우리가 사용하는 메서드 레벨의 `@Transactional` 로 그 생명주기와 범위가 설명된다. 이 database  transaction은 persistence context의 범위 안에서 일어난다. <br>

JPA에서 persistence context는 `EntityManager`를 말한다. Persistence context는 한정된 자바 객체들의 상태를 확인하고, 변경 사항들이 DB에 반영이 되도록 관리한다. 따라서 database transaction과 비슷하다고 생각할 수 있지만, 둘은 다른 개념이다. 주로 하나의 Entity Manager가 여러 database transactions 에 대해 사용된다. <br>

우선 트랜잭션 동작원리에 대해서 알기 이전에 `@PersistenceContext` 의 동작 방식에 대해서 아는 것이 중요하다.  해당 어노테이션은 컨테이너의 시작 시점에서 단 한번 entity manager를 주입하는 것처럼 보인다. 실은 `EntityManager` 는 인터페이스이고 스프링 빈으로 주입되는 것은 실제 entity manager가 아닌, *context aware proxy* 이며, runtime 중 실제 entity manager에서 책임을 위임한다.  

**트랜잭션 관리를 위해 필요한 3가지 components**

- EntityManager Proxy
    - 비지니스 로직에서 EntityManager 관련 메소드를 호출했을 때 entity manager를 직접적으로 호출하지 않는다. 비지니스 로직은 proxy에 의존하고 해당 proxy가 스레드에서 현재 entity manager를 추출한다.
- Transactional Aspect
    - `TransactionInterceptor` 로 구현이 되며, 비지니스 메소드 이전과 이후에 (before & after) 호출된다.
    - Before moment: 실행될 비지니스 메서드가 기존에 실행되고 있는 database transaction의 범위 내에서 실행되어야 하는 것인지, 새로운 transaction을 시작해야 하는지 판단
    - After moment: 해당 transaction이 커밋, 롤백, 실행중지 되어야 하는지 등등을 판단
    - 여기서 'before moment' 에 해당하는 책임은 Transactional Aspect 에서 실제로 담당하지 않고, 판단하는 책임을 Transaction Manager에 위임한다.
- Transaction Manager
    - 다음 두가지에 대해서 판단 및 처리한다.
        1. 새로운 Entity Manager가 생성되어야 하는지
        2. 새로운 database transaction이 시작되어야 하는지 

        이 두가지는 Transactional Aspect의 'before moment' 로직이 호출되었을 때 판단되어야 한다. 

    - 판단은 다음에 기반한다.
        1. 다른 transaction의 실행 중 여부
        2. 해당 transaction 메서드의 propagation 속성 (예를 들어 `REQUIRES_NEW` 일 경우 항상 새로운 transaction을 시작한다)
    - 만일 새로운 transaction을 생성하도록 했다면 다음이 실행된다
        1. 새로운 entity manager 생성
        2. 생성된 entity manager를 현재 쓰레드에 바인딩
        3. DB connection pool에서 커넥션 가져오기
        4. 해당 커넥션을 현재 쓰레드에 바인딩 
    - Entity manager와 connection 모두 현재 스레드(즉, 모두 스레드 단위로 실행 및 관리)에 바인딩 되어 있고, transaction이 끝났을 때 Transaction Manager가 제거한다. **따라서 현재 entity manager나 connection이 필요한 경우 현재 스레드에서 추출하여 사용하고, 이 부분을 EntityManager proxy가 담당한다.**

*Spring configuration에서 위 @Transactional 매커니즘이 동작하도록 설정해준다.* 

 <br>
 <br>

**[참고링크]**
- [https://dzone.com/articles/how-does-spring-transactional](https://dzone.com/articles/how-does-spring-transactional)
- [https://hleee.medium.com/격리-수준-3287d4bcc64d](https://hleee.medium.com/%EA%B2%A9%EB%A6%AC-%EC%88%98%EC%A4%80-3287d4bcc64d)
- [https://mangkyu.tistory.com/30](https://mangkyu.tistory.com/30)


**MORE**
- 하나의 Entity Manger(persistence context) 에 여러 database transactions가 연관되어 있는 경우는 무엇일까?