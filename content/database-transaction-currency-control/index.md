---
emoji: 💾
title: "Transaction의 동시성 제어(Currency Control)"
date: '2021-07-01 23:00:00'
author: 코다
tags: 데이터베이스
categories: 데이터베이스
---

## 동시성 문제 발생 가능 상황

두개의 트랜잭션이 모두 읽는 연산을 하는 경우 문제가 되지 않는다.

- 하나의 트랜잭션은 read, 하나는 write인 경우 (**Isolation**으로 해결)
    - **Dirty Read**
        
        **상황**: 트랜잭션1이 write 할 때 트랜잭션2가 update된 데이터를 읽었지만 트랜잭션1이 rollback 되었을 때 발생 <br>
        
        **문제**: 트랜잭션2가 무효된 데이터를 읽었음 <br>
        
    - **Non-repeatable Read**
        
        **상황**: 트랜잭션1이 데이터를 read하고, 트랜잭션2가 데이터를 write 한 후, 트랜잭션1이 다시 동일한 데이터를 read 할 경우에 발생 <br>
        
        **문제**: 트랜잭션이 1이 동일한 read를 했음에도 불구하고 바뀐 데이터를 읽음 <br>
        
    - **Phantom Read**
        
        **상황**: 트랜잭션1이 데이터(범위)를 read하고, 트랜잭션 2가 데이터를 추가(insert) 했는데, 트랜잭션1이 다시 데이터를 read한 경우 <br>
        
        **문제**: 동일한 read를 실행하였는데, 이전에 없었던 값이 추가됨 <br>
        
- 두개의 트랜잭션이 모두 wrtie인 경우 (**Currency Control**로 해결)
    - **Lost Update**
        
        **상황**: 두개의 트랜잭션이 한 개의 데이터를 동시에 update 할 때 발생 <br>
        
        **문제**: 하나의 트랜잭션의 내용을 다른 트랜잭션이 덮어씀으로 이전에 update 한 내용이 손실됨 <br>
        
    - **Inconsistency(or Unrecoverability)**
        
        **상황**: 한 트랜잭션(T1)이 두개의 일관된 값을 읽어야하는데 다른 트랜잭션(T2)이 그 사이에 실행될 때 발생 <br>
        
        **문제**: `T1`이 두개의 값 `X`, `Y`를 읽어 수정할 때, `T1`이 `X`만 읽어서 수정한 상태에서 `T2`가 `X`와 `Y`를 수정하고 이후에 `T1`이 `Y`를 읽어 수정함. 이때 `T1`의 입장에서 하나는 갱신 이전의 값, 또 하나는 갱신 이후의 값을 읽어서 데이터가 불일치한 모순이 일어남. <br>
        
    - **Cascading Rollback**
        
        **상황**: 한 트랜잭션(T1)이 두개의 값을 읽고 수정하는 사이에 다른 트랜잭션(T2)이 완료되었지만 T1이 rollback 되어야 할 때 발생<br>
        
        **문제**: `T1`이 두개의 값 `X`, `Y`를 읽어서 수정할 때, `T1`이 `X`를 수정한 상태에서 `T2`가 `X`를 수정하고 해당 트랜잭션을 완료함. 이후 `T1`에서 오류가 발생해 rollback이 되어야하는 상황에서 `T2`는 이미 완료되었으므로 `X` 값에 대한 rollback이 되지 못하여 문제가 발생함. <br>
        
<br>

## 트랜잭션 스케줄

위 문제를 해결하기 위해서 트랜잭션 스케줄을 통해 연산들의 실행 순서를 제어하여 데이터에 오류가 없도록 해야한다. 

1. Serial Schedule 직렬 스케줄 
    
    모든 트랜잭션이 순차적으로 실행
    
2. Nonsercial Schedule 비직렬 스케줄
    
    트랜잭션이 상관없이 병행하여 실행
    
3. Serializable Schedule 직렬 가능 스케줄
    
    서로 영향을 주지 않는 스케줄의 경우 병행 실행하고, 나머지는 실행 순서를 제어하여 실행
    

직렬 스케줄의 경우 오류가 생기지 않도록 보장할 수는 있지만, 효율이 떨어진다는 단점이 있다. 따라서 트랜잭션을 최대한 직렬 가능한 스케줄로 만드는 것이 중요하고 **Locking** 기법을 통해서 트랜잭션 실행 순서를 제어해 직렬 가능한 스케줄로 만들어 데이터에 생길 수 있는 오류를 제어한다. <br>

예를 들어, 두개의 트랜잭션이 모두 READ 연산을 한다면 연산 순서가 중요하지 않으므로 병행 실행한다. <br>

또 두개의 트랜잭션이 다른 데이터에 접근한다면 연산 순서가 중요하지 않으므로 병행 실행한다. <br>

만일 한 데이터에 대해서 읽고 쓰는 작업이 한꺼번에 발생한다면 실행순서가 중요하므로 Lock을 통해서 그 순서를 제어한다. <br>

<br>

## Locking 기법

여러 트랜잭션이 동일한 데이터에 대해서 병행 접근을 하지 못하도록 제어해서 데이터간 문제가 발생하지 않도록 하는 기법 

- Shared-lock 공유락
    
    읽기를 할 때 거는 lock <br>
    
    read 트랜잭션은 서로 영향을 주지 않으므로 shared-lock이 걸린 상태에서 추가 shared-lock을 걸 수 있음 <br>
    
    read 트랜잭션 중 쓰기가 발생할 경우 데이터가 손상되어 문제가 발생하므로 추가 exclusive-lock은 걸 수 없음 <br>
    
- Exclusive-lock 베타락
    
    읽고 쓰기를 할 때 거는 lock <br>
    
    write 트랜잭션은 읽거나 쓰는 트랜잭션 모두에게 영향을 주기 때문에 exclusive-lock이 걸린 상태에서는 다른 lock을 걸 수 없음 <br>
    

### 2단계 로킹 규약 2PLP

lock을 걸고 해제할 때 제한을 두어서 두 트랜잭션이 동시 실행되면서 생기는 데이터 일관성이 깨지는 현상을 방지하는 것이다. 로킹 단계를 2가지로 구분 지어서, 각 단계에 다른 제한을 두어 트랜잭션 동시 실행으로 발생할만한 데이터 오류를 방지한다. 

- Growing Phase 확장 단계
    
    해당 트랜잭션이 새로운 Lock 연산은 가능하나 unlock 연산을 가능하지 않은 상태
    
- Shrinking Phase 축소 단계
    
    해당 트랜잭션이 unlock 연산만 가능하고 lock 연산은 가능하지 않은 상태 
    

하지만 2PLP을 준수하므로 **deadlock**이 발생할 여지가 있다. <br>

→ 2단계 로킹 규약을 적용하지 않아서 데이터의 일관성이 깨진 경우 <br>

 <p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/135496848-c9c42362-63c5-493f-a2e9-fcd05f5cf6c6.png"></p>

→ 2단계 로킹 기법을 사용하여 일관성 문제 해결한 경우 <br>

 <p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/135496984-e425dd70-3f11-4718-a42e-28513c3452bb.png"></p>

### Deadlock 데드락 (교착상태)

두개의 트랜잭션이 각자의 데이터에 lock을 걸어놓고 상대의 데이터에 요청을 하므로 무한 대기 상태에 빠지게 되는 문제 

<br>

## 트랜잭션 Isolation Level

- **READ UNCOMMITTED (LEVEL 0)**
    
    데이터를 읽을 때 shared-lock을 걸지 않고, write 작업 시 exclusive-lock만 거는 것(갱신손실문제 방지) <br>
    
    다른 트랜잭션에 shared-lock/exclusive-lock이 걸렸더라도 데이터를 대기하지 않고 읽는 것<br>
    
    따라서 Dirty read, non-repeatable read, phantom read 모두 방지할 수 없다.<br>
    
- **READ COMMITED (LEVEL 1)**
    
    데이터를 읽을 때 shared-lock을 걸지만, 트랜잭션이 끝나기 전에도 unlock 가능<br>
    
    다른 트랜잭션에 shared-lock이 걸렸다면 데이터를 읽지만, exclusive-lock이 걸린 경우에는 읽지 못함<br>
    
    Dirty read 방지 가능, non-repeatable read, phantom read 방지 불가능<br>
    
- **REPEATABLE READ (LEVEL 2)**
    
    본 데이터에 설정된 shared-lock과 exclusive-lock을 트랜잭션 종료시까지 유지<br>
    
    트랜잭션이 완료될 때까지 SELECT문이 사용하는 모든 데이터에 대해서 shared-lock이 걸린다. 따라서 update가 불가능하지만, insert에 대해서는 다른 제약이 없다.<br>
    
    다른 트랜잭션이 공유한 공유락은 읽고, 배타락은 읽지 않음<br>
    
    다른 isolation level에 비해서 동시성이 낮아 잘 사용하지 않는 것을 추천<br>
    
    Dirty read, non-repeatable read 방지, phantom read 방지 불가능<br>
    
- **SERIALIZABLE (LEVEL 3)**
    
    데이터의 집합 범위에 잠금을 설정함<br>
    
    다른 사용자가 데이터를 update 혹은 insert 할 때 모두 트랜잭션을 격리함<br>
    
    트랜잭션이 완료될 때까지 SELECT문이 사용하는 모든 데이터에 대해서 shared-lock이 걸린다.<br>
    
    **인덱스에 공유락을 설정**한다. 따라서 다른 트랜잭션의 insert가 불가능<br>
    
    Dirty read, non-repeatable read, phantom read 방지 <br>
    

<br>
<br>

**[참고링크]**

- [https://mangkyu.tistory.com/30](https://mangkyu.tistory.com/30)
- [https://medium.com/pocs/동시성-제어-기법-잠금-locking-기법-319bd0e6a68a](https://medium.com/pocs/%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%A0%9C%EC%96%B4-%EA%B8%B0%EB%B2%95-%EC%9E%A0%EA%B8%88-locking-%EA%B8%B0%EB%B2%95-319bd0e6a68a)
- [https://goodmilktea.tistory.com/62](https://goodmilktea.tistory.com/62)## 동시성 문제 발생 가능 상황

```toc
```