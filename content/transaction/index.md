---
emoji: 🚊
title: Transaction의 동작제어
date: '2021-03-09 23:00:00'
author: 코다
tags: 데이터베이스
categories: 데이터베이스
---

## Transaction 이란?

- 개인이 설정할 수 있는 작업의 최소 단위이다.
- Transaction을 기준으로 `commit`을 할 수도, `rollback`을 할 수도 있다.
- Transaction을 사용할 때 DBMS에서 ACID를 제공받을 수 있다.
    - **Atomic(원자성)** : 한꺼번에 모두 처리가 되거나, 한꺼번에 모두 처리가 되지 않도록 원자성을 부여한다. 데이터 관련 일부만 처리되었을 때 생길 복잡한 상황과 부작용을 막을 수 있다.
    - **Consistency(일치성)** : 하나의 데이터가 처리되었을 때 관련된 다른 테이블 혹은 상황에서 일관된 논리가 수행 되도록 하는 것을 보장한다 (ex. A 에서 1000원이 차감되면 B에서 1000원이 증감되어야 하는 상황 등등.)
    - **Isolation(독립성)** : 데이터가 처리되는 도중 다른 일이 중간에 일어나지 않도록 해당 데이터를 보호하도록 보장. 중간에 다른 일이 끼어들어 부작용이 생기는 것을 방지한다.
    - **Durability(영구보존성)** : 데이터를 DB에 저장하여 보존하도록 한다.

## JDBC에서 Transaction 설정 및 응용하기

- JDBC에서 `Connection`을 생성할 때 해당 `Connection`은 default로 `setAutoCommit(true)` 로 설정이 되어 있다.
    - 이 말은 각 SQL statements가 모두 기본 transaction으로 래핑되어 있다는 뜻이다.
- 개인적으로 작업 단위를 설정해서 ACID를 보장받으며 처리하고 싶을 경우 다음과 같이 설정해야 한다.

    ```sql
    Connection con = DriverManager.getConnection(...);
    con.setAutoCommit(false); 

    //sql 실행

    if (결과 정상 처리) {
     con.commit();
    } else {
    	con.rollback();
    }
    ```

## JDBC Savepoint 설정하기

- Transaction을 통해서 commit 과 rollback 작업단위를 설정할 수 있지만, 하나의 작업 단위 내에서도 rollback 하고 싶은 지점을 직접 설정할 수도 있다.
    - 이때는 `setAutoCommit(false)`로 설정해야지 아니면 자동으로 각 sql 문이 하나의 transaction으로 묶인다.
- Savepoint를 설정해서 원하는 작업 포인트로 rollback 할 수 있다.
    - setSavepoint(String savepointName) : 새로운 savePoint를 설정하고 해당 객체를 리턴
    - releaseSavepoint(Savepoint savepoint) : 해당 savepoint 지점을 해제한다.
- 사용 예시:

    ```sql
    try {
    	con.setAutoCommit(false); 
    	Statement stmt = con.createStatement();

    	SavePoint savePoint1 = con.setSavepoint("savepoint1");
    	//sql 쿼리 실행
    	con.commit();
    } catch (SQLException) {
    	con.rollback(savepoint1);
    }
    ```

**[참고자료]** : [http://yimoyimo.tk/transaction_DI/](http://yimoyimo.tk/transaction_DI/), [https://hamait.tistory.com/345](https://hamait.tistory.com/345)

```toc
```