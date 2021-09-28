---
emoji: 🖥
title: "JWT (JSON Web Token) 알아보기"
date: '2021-06-13 23:00:00'
author: 코다
tags: 웹 보안
categories: 웹
---

## JWT(JSON Web Token) 배경

이전에 인증 절차를 거치려면 사용자의 해싱값을 DB에 저장하고 매번 요청이 있을 때마다 해당 해싱값을 검증해야한다. 검증시 DB에 접근하는 쿼리가 실행되어야하는데 성능면에서 좋지 않다. 따라서 JWT가 등장하게 되고 위와 같은 절차를 거치지만 DB 접근 쿼리가 필요하지 않게 된다. 

- *JWT 정의: A string that is sent in the HTTP request (from client or server) to validate the authenticity of the client. It is saved on the client-side only.*

    [출처]([https://medium.com/jspoint/so-what-the-heck-is-jwt-or-json-web-token-dca8bcb719a6](https://medium.com/jspoint/so-what-the-heck-is-jwt-or-json-web-token-dca8bcb719a6)) 

<br>

## JWT 특징

- compact
- self-contained
- digitally signed : it is signed using a secret key(**HMAC** algorithm) or public/private key pair using (**RSA** or **ECDSA**)
    - sgined 토큰 이라면 claim의 무결성(integrity)를 검증할 수 있다.
    - 만일 토큰이 public/private key로 signed 되었다면, 시그니처는 private key를 들고있는 쪽이 signed 한 것이라는 것을 보장한다.

<br>

## JWT 언제 사용하면 좋을까

- **Authorization**: 유저가 로그인하여 어떠한 요청들을 보낼 때, JWT를 통해서 인증을 한다. JWT가 사용되는 가장 대표적인 시나리오다. `Single Sign On` 이라고 하는데, 이 때 JWT를 사용하면 부하도 적고 여러 다른 도메인에서 쉽게 사용될 수 있다. (cross-origin일 때 용이)
- **Information Exchange**: 정보들을 안전하게 전달하려고 할 때 JWT를 사용하기도 한다. JWT를 signed 할 수 있으니 송신자가 누군지 보장할 수 있다. 또한 시그니처를 통해서 header와 payload를 계산하기 때문에 해당 컨텐츠가 무결함을 보장할 수 있다.

<br>

## JWT 생성

secret key를 사용해서 JWT를 생성하며, 해당 secret key는 private하여 외부에 공개되거나 JWT 토큰에 주입될 경우가 없다. 또한 JWT를 클라이언트에게 받았을 때 서버에 저장되어 있는 secret key를 통해서 검증한다. <br>

JWT 생성하기 위해서 [여기]([https://jwt.io/](https://jwt.io/)) 에서 원하는 라이브러리를 다운 받아서 사용한다. 

<br>

## JWT 구조

`xxxx.yyyy.zzzz` 으로 header, payload, signature가 (`.`)로 연결되어 있다. 

- Header
    - type of token (ex. JWT)
    - signing algorithm
    - JSON 타입, `Base64Url`로 인코딩
- Payload
    - claims - entity(유저) 정보나 이외의 추가 정보 등등
    - registered - 등록되어 있는 claims로 항상 추가하기를 권장하는 유용한 claims
        - iss(issuer), exp(expiration time), sub(subject), aud(audience) 등등
        - JWT은 compact 하기 때문에 3글자로 제한하여 정의함
    - public -  JWT 사용하는 입장에서 정의할 수 있는 claims
        - 충돌을 피하기 위해서 JWT Registry에 정의된 것들만 사용하거나, 충돌방지할 수 있는 URI 형태
    - private - 커스텀 claims로 양쪽에서 합의하여 공유하기로 한 정보에 대해서 서술한 claims
    - JSON 타입, `Base64Url`로 인코딩, 때문에 어느 누구나 디코딩하여 접근할 수 있는 정보이기 때문에 민감한 정보는 담지 않도록 한다.
- Signature
    - 인코딩 된 header, payload, secrete을 조합하여 header에 있는 알고리즘으로 시그니처를 생성
    - 이 시그니처를 통해서 담긴 정보의 무결성을 보장
    - private key로 signed 되었을 경우 JWT sender 또한 보장 가능

### JWT Signature

위 header와 payload는 쉽게 디코딩해 내용을 확인할 수 있다. 따라서 해당 token을 보안에 사용할 수 있도록 하는 요소는 signature이다. <br>

제 3자가 토큰의 header나 payload를 변경하지 않았음을 보장하기 위해서 secret key와 해당 키로 생성된 signaturer를 사용한다. <br>

Token 검증을 할 때, 전송된 JWT에 있는 header, payload를 가지고 서버에 저장되어 있는 secret을 통해서 test signature를 생성한다. test signature와 token에 있는 처음 token 생성시 추가된 original signature와 동일한지 비교하여 데이터 변경 여부를 체크한다. 이 방법을 통해서 token은 데이터 무결성을 보장한다. <br>

<br>

## 왜 JWT?

Simple Web Tokens(**SWT**)와 Security Assertion Markup Language Tokens(**SAML**) 비교했을 때 장점들이다. <br>

JSON은 우선 XML보다 간단(덜 장황함)하여 인코딩 했을 때, 그 사이즈가 훨씬 작고 **SAML**보다 compact 하다. <br>

<p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/135121193-926e8de5-411d-49dc-873c-09cfb0a552df.png"></p>

<p align="center"><img width="85%" src="https://user-images.githubusercontent.com/63405904/135121388-68caa14c-464b-449e-8901-70491a30b765.png"></p>

보안적 측면에서는 **SWT**는 대칭키를 활용하는 HMAC 알고리즘만 사용이 가능한데, **JWT**, **SAML**은 public/private key 페어를 사용할 수 있다. 더해서 XML signing보다 JSON signing이 훨씬 쉽다. <br>

또한 잘 구축되어 있는 여러 JSON 파서들을 활용할 수 있다. <br>

마지막으로 활용 측면에서 **JWT**는 인터넷 환경에서 많이 쓰인다. 그렇기 때문에 클라이언트의 입장에서 **JWT**를 여러 환경(특히, 모바일 환경에서도)에서 활용하기 용이하다는 장점이 있다.  <br>

<br>
<br>

**[참고자료]**

- [https://jwt.io/introduction](https://jwt.io/introduction)
- [https://stackoverflow.com/questions/31309759/what-is-secret-key-for-jwt-based-authentication-and-how-to-generate-it](https://stackoverflow.com/questions/31309759/what-is-secret-key-for-jwt-based-authentication-and-how-to-generate-it)


```toc
```