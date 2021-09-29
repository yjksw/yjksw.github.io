---
emoji: 🖥
title: "Hash와 Salt"
date: '2021-06-21 23:00:00'
author: 코다
tags: 웹 자바 
categories: 웹 자바 
---

## 들어가기 전에

암호화(Encryption)과 해싱은 다른 개념

- 암호화 - 양방향이므로 복호화가 가능
- 해싱 - 단방향이므로 복호화가 불가능

<br>

## 단방향 해시 함수 (One-Way Hash Function)

기본적으로 패스워드 등의 보안의 문제가 걸린 정보를 DB에 저장할 때 평문으로 저장하지 않고 해싱한 값을 저장한다. (평문으로 저장할 경우 DB가 해킹되었을 때 심각한 문제가 발생한다) <br>

단방향 해시 함수를 사용해서 원본 내용을 완전히 새로운 내용으로 **다이제스트**(**digest**)로 매핑한다. 이때 매핑하는 것을 **해시**라고 한다. 이것은 단방향이므로 복호화할 수 없다. <br>

<p align="center"><img width="90%" src="https://user-images.githubusercontent.com/63405904/135313771-acb38bc5-e482-41a0-a0b8-2a7415bd2903.png"></p>

- 해시 함수 종류
    1. SHA
    2. MD
    3. HAS
    4. WHIRLPOOL

### 한계점

- Rainbow Table
    
    동일한 데이터를 동일한 해시 함수로 연산한 **다이제스트**는 동일한 값을 가진다. 따라서 여러 값들에 대한 다이제스트를 모아놓은 **Rainbow Table**이라는 것이 존재하고 이것을 통해서 원본 데이터를 유추할 수 있다. 
    
- Brute-force
    
    해싱 자체가 빠른 검색을 하기 때문에 반대로 다이제스트를 얻는 과정도 빠르게 실행된다. 무작위 데이터를 대입하여 다이제스트를 비교(해싱이므로 더 빠르게 수행됨)하여 원본 데이터를 유추할 수 있다. 
    
<br>

## 단방향 해시 함수 보완

### Key Stretching

n번의 해시를 통해서 다이제스트의 다이제스트를 얻어내어 해커 입장에서 원문 데이터를 얻는데 시간을 더 오래 소요하게 한다. (Brute-force 무력화) <br>

<p align="center"><img width="90%" src="https://user-images.githubusercontent.com/63405904/135314064-923f6f7a-d9bf-4987-9996-05c371b77aa9.png"></p>

### Salt

원본 데이터에 임의의 문자열을 덧붙여서 해싱을 해서 다이제스트를 얻어내는 방법이다. 따라서 다이제스트의 원문을 알아낸다고 하더라도 사용자가 입력한 원본 password를 알아내는 것은 어려워진다. <br>

<p align="center"><img width="90%" src="https://user-images.githubusercontent.com/63405904/135314140-e59783ad-2587-4ba2-98d0-32763bb66f49.png"></p>

위 두가지 방법을 모두 사용하여 다이제스트에 대한 보안성을 더 높인다. <br>

<p align="center"><img width="90%" src="https://user-images.githubusercontent.com/63405904/135314236-2f313ce8-33aa-4c3a-9c39-2ec9fb18ea45.png"></p>

<br>

## 간단하게 구현해보기

1. SALT 크기를 지정
    
    너무 짧으면 안전하지 않다. <br>
    
    랜덤 함수를 하용해 SALT를 생성하는 것이 좋으나, `java.util.Random` 은 암호학적으로 안전하지 않아서`java.security.SecureRandom`을 사용한다.
    
2. 해시 함수를 쓰기 위해서 `java.security.MessageDigest` 외부 라이브러리를 Import 한다. (이외의 다른 라이브러리도 존재한다)
3. 랜덤 함수를 통해서 SALT를 얻어 사용자 입력 password에 덧붙인다.
4. `MessageDigest` 라이브러리의 `update()`를 통해서 문자열을 해싱하여 해당 라이브러리에 저장한다. 
5. `MessageDigest` 라이브러리의 `digest()` 를 통해서 다이제스트를 얻는다. 

<br>
<br>

**[참고링크]**

- [https://st-lab.tistory.com/100](https://st-lab.tistory.com/100)

```toc
```