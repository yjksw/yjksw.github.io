---
emoji: 🖥
title: 대칭키와 비대칭키 비교하기
date: '2021-06-21 23:00:00'
author: 코다
tags: 웹 보안
categories: 웹
---

## Intro

암호화(encryption)에는 3가지 기술이 있다. 

1. Symmetric encryption - 대칭키
2. Asymmestric encryption - 비대칭키
3. Hash functions(keyless) - 해싱

여기서는 대칭키, 비대칭키에 대해서만 다룰 것인데 둘다 각각의 장단점이 있다. <br>

대칭키와 비대칭키의 간단한 차이점 

- 우선 모두 key를 사용해서 데이터를 encrypt/decrypt 한다.
- 대칭키의 경우 동일한 key를 가지고 암호화/복호화를 하기 때문에 사용하기가 쉽다.
- 비대칭키의 경우 public key를 사용해서 데이터를 암호화하고, private key를 사용해서 복호화한다.

<br>

## Symmetric encryption 대칭키?

대칭키를 사용하면 데이터의 암호화/복호화 모두 하나의 key를 사용한다. 그리고 해당 키를 수령인과 공유한다. (수령인이 암호화된 데이터를 받았을 때 복호화를 위해서 필요) <br>

<p align="center"><img width="90%" src="https://user-images.githubusercontent.com/63405904/135317204-d0bcfee1-fc3a-4fc8-ab1a-dc3e644d772c.png"></p>

### 대칭키 장단점

**장점**

- 세팅이 쉽고 간단하다. (jiffy 순간적으로 처리된다)
- 개념이 간단하기 때문에 여러 환경에서 거의 모두 적용이 가능하다.

**단점**

- 비밀키가 수령인과 공유되어야 한다.
- 따라서 보안적 측면에서 다소 위험한 부분이 있다.

<br>

## Asymmetric encryption 비대칭키?

비대칭키 방식은 데이터를 암호화/복호화 하는데 두개의 key가 필요하다. public key를 사용해서 데이터를 암호화하고, private key를 통해서 데이터를 복호화한다. <br>

암호화된 데이터를 수령하는 수령인은 반드시 private key를 가지고 있어야 하는데, 그것을 private하게 유지하기 위해서 local 하게 저장하는 것이 가장 좋다. 

### 비대칭키 장단점

**장점**

- 비밀키를 공유하지 않아도 된다.
- digital sigining을 지원하기 때문에 수령인의 identity를 보장할 수 있고, 메세지의 오염여부를 알 수 있다.

**단점**

- 시간과 이외의 다른 많은 노력들이 소요된다.
- 이메일이나 데이터를 보낼 때, 상대방이 key pair를 생성했는지 항상 확인해야한다.
- 만일 key을 잃어버리면 복구할 수 없다.

<br>

## 언제 무엇을 사용할까?

- 만일 빠르게 암호화된 메세지를 전송하고 싶을 경우 Symmetric encryption 대칭키를 사용하기를 추천한다.

<br>
<br>

**[참고링크]**

- [https://blog.mailfence.com/symmetric-vs-asymmetric-encryption/](https://blog.mailfence.com/symmetric-vs-asymmetric-encryption/)

```toc
```