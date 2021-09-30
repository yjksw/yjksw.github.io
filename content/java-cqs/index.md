---
emoji: ✊
title: "CQS(Command Query Separation) 간단히 알아보기"
date: '2021-07-17 23:00:00'
author: 코다
tags: 자바
categories: 자바
---

## Intro

함수는 **시스템의 상태를 바꾸는지**에 따라서 크게 두 가지로 나뉜다. 1) Command 2) Query. 이 두 가지 분류법에 대해서 하나의 함수가 두 가지 경우를 모두 담당하는 것은 좋지 않다. 

<br>

## Command vs. Query

- **Query**
    
    주어진 쿼리에 대한 결과값을 반환하고 시스템의 상태를 변화시키지는 않는다.
    
    다른 값을 바꾸지 않고 오직 질문에만 대답한다. 
    
    부작용에서 자유롭다. (read-only)
    
    ```java
    list = [1, 2, 3, 4, 5];
    
    print(max(1));
    
    5
    ```
    
- **Command**
    
    값을 반환하지 않아도 시스템의 상태를 변화시킨다. (영구적)
    
    부작용이 생길 여기자 있다. (mutator, modifier)
    
    ```java
    list = [1, 2, 3, 4, 5];
    list.reverse();
    
    print(list)
    
    [5, 4, 3, 2, 1]
    ```
    

이렇게 함수를 두 가지로 나누는 것은 유용하다. 현재 사용하는 함수가 상태를 바꾸지 않는 query 라면 신뢰를 가지고 사용할 수 있다. 하지만 상태를 바꾸는 command라면 함수간 순서에 주의를 기울이고 부작용이 생길 여지를 인지하고 있어야 한다. 

<br>

## CQS - Command Query Separation

***Betrand Meyer***라는 프랑스 학자는 하나의 함수가 command 이거나 query 이어야 한다고 주장한다. 이러한 프로그램 원칙을 CQS라고 하며 한 함수에서는 둘은 분명히 구분해야 한다고 말했다. <br>

하지만 스택의 pop() 연산 같은 경우는 위 두가지를 모두 담당하기도 한다. 이러한 예외 상황은 융통성 있게 허용하는 것이 전반적으로 더 효율적일 수 있다. <br>

위 개념은 현재 진행중인 프로젝트에서 다음과 같은 코드에 대한 크루의 코드리뷰로 접하게 되었다. <br>

```java
Post newPost = post.update(updatedContent);
```

위와 같은 경우 CQS를 준수하지 않은 것이므로 둘 중 하나를 담당하도록 구현로직을 바꾸고 필요하면 재조회하는 방향으로 리팩토링 했다. <br>

<br>
<br>

**[참고자료]**

- [https://shoark7.github.io/programming/knowledge/command-and-query-method](https://shoark7.github.io/programming/knowledge/command-and-query-method)

```toc
```