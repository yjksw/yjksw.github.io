---
emoji: 🖥
title: OAuth 알아보기
date: '2021-09-03 23:00:00'
author: 코다
tags: 웹
categories: 웹
---

## 1. INTRO
<br>

- 많은 어플리케이션에서 소셜 로그인을 지원하는데, 이때 사용되는 것이 OAuth 2.0 이다. 
- 간단하게 이야기하면 OAuth 2.0 이란 사용자의 정보에 대한 권한을 부여하는 `산업 표준 프로토콜`의 일종이다. 
> (정의) 제 3의 앱이 자원의 소유자인 서비스 이용자를 대신하여 서비스를 요청할 수 있도록 자원 접근 권한을 위임하는 방법 <br> *출처: 금융보안원 "OAuth 2.0 개요 및 보안 고려사항" 보안연구부-2015-030*

- 즉, 정보 소유자 (서비스 이용자)를 대신하여 앱이 다른 서비스에 등록되어 있는 자원에 대한 접근을 요청하는 권한을 위임한다. 
- 아래 글은 [링크](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2) 원문을 번역하고 일부 요약한 것이다. 

<br>
<br>
<br>

## 2. OAuth 주요 개념 

<br>
1. **리소스 소유자** (Resource Owner) - 어플리케이션이 인가 요청을 하는 정보의 소유자이다. 즉, 그 정보를 소유하고 있는 '사용자'를 말한다. 
2. **클라이언트** (Client) - 리소스 소유자의 정보를 요청하는 어플리케이션이다. 
3. **리소스 서버** (Resource Server) - 리소스 소유자의 정보를 보유하고 있는 서버이다. 
4. **인가 서버** (Authorization Server) - 클라이언트가 사용자에 대한 정보 권한을 요청할 때, 그 사용자에 대해 검증하고 클라이언트 어플리케이션에 access token을 발급하는 인가 서버이다. 편의상 리소스 서버와 인가 서버를 구별하지 않고 이해해도 좋다. 

<br> 
<br>

## 3. 간단히 보는 OAuth 프로토콜 흐름

<br>
<p align="center"><img width="540" alt="스크린샷 2021-08-17 오후 2 41 53" src="https://user-images.githubusercontent.com/63405904/129669922-d91d3405-3315-4b29-9e47-f8015dec6b13.png">출처 : Digital Ocean의 An Introduction to OAuth 2.0</p>
<br>

1. 어플리케이션 유저에게 리소스에 대한 인가를 요청한다. 
2. 유저는 해당 인가를 승인한다. 
3. 어플리케이션은 리소스를 보유하고 있는 인가 서버에 access token을 요청한다. 
4. 본 어플리케이션이 인증이 되고, 인가 승인을 인증하면 인가 서버는 해당 어플리케이션에 access token을 발급하고 인가 절차가 마무리된다. 
5. 인가를 받은 어플리케이션은 리소스 서버에 access token과 함께 리소스를 요청한다.
6. 리소스 서버에서 access token을 확인하면 해당 리소스를 어플리케이션에 제공한다. 

<br>
<br>

## 4. 어플리케이션 등록

<br>

- OAuth 기능을 추가하기 이전 본 어플리케이션(클라이언트)를 리소스를 요청하고자 하는 제 3 서비스(Google, Kakao, Github 등등)에 등록해야한다. 
- 등록 정보 
    ```
    - 어플리케이션 이름
    - 어플리케이션 웹사이트
    - Redirect URI or Callback URL
    ```
- 위에서 리다이렉트 URI는 인가 후, 인가 서버(리소스 서버)에서 code를 전송해줄 주소를 말한다. 
<br>

- 어플리케이션 등록 시 다음 정보를 발급 받는다. 다음은 `Client Credentials`이라고 도 한다. 
    1. Client Id - 외부에 노출되는 어플리케이션 identity 이다.
    2. Client Secret - 외부에 공개되어서는 안되는 key로 어플리케이션을 인증하고 유저의 계정에 접근을 요청할 때 사용한다. (Access Token 발급 시)
<br>
<br>

## 5. Authorization 부여하기 

<br>

- OAuth 2.0은 3가지 종류로 인가를 부여한다. 
    1. Authorization Code
    2. Client Credentials
    3. Device Code

- 위 3가지 중 이번 프로젝트에서는 `Authorization Code`를 사용했다. `Authorization Code`가 가장 빈번하게 사용이 되는데 server-side applications에 가장 최적화 되어 있기 때문이다. 
- 아래가 `Authorization Code`를 발급받는 기본 흐름인데, 보면 어플리케이션이 user-agent(유저의 웹 브라우저)와 소통할 수 있어야 하며 user-agent를 통해 라우팅 된 API authorization 코드를 받을 수 있어야 한다. 

<p align="center"><img width="688" alt="스크린샷 2021-08-17 오후 4 08 28" src="https://user-images.githubusercontent.com/63405904/129680013-50c21f7f-e1a2-4bb7-89fc-361450ef6c3e.png">출처 : Digital Ocean의 An Introduction to OAuth 2.0</p>

1. 리소스를 요청하고자 하는 서버에 기재되어 있는 API authorization endpoint URI 에 1) client_id 2) redirect_uri 3) response_type 4) scope 등을 지정해서 요청을 보낸다. 
    - 본인은 프로젝트에서 Github 소셜 로그인을 구현했었는데 `https://github.com/login/oauth/authorize?client_id=%s&redirect_uri=%s&scope=%s` 로 유저 인가 request를 보내도록 되어 있었다. 
2. 그럼 해당 리소스 서버(Github 등)에서 인가를 허가할지 말지 여부를 물어보는 페이지로 이동하여 유저가 허가 또는 거절을 누른다. 
3. 유저가 허가한다면 리소스 서버(인가 서버)는 user-agent(브라우저)가  어플리케이션 등록 시 기재한 redirect URI로 authorization code와 함께 리다이렉트 하도록 한다. 
4. 어플리케이션은 리소스 서버(인가 서버)에 1) client_id 2) client_secrete 3) grant_type 4) code 5) redirect_uri 와 함께 Access Token 발급 request를 보낸다. 
5. 리소스 서버(인가 서버)에서 요청을 검사하고 유효하다면 AccessToken을 반환한다. 
    `{"access_token":"ACCESS_TOKEN","token_type":"bearer","expires_in":2592000,"refresh_token":"REFRESH_TOKEN","scope":"read","uid":100101,"info":{"name":"developer_coda","email":"coda@coda.com"}}
`
<br>
<br> 

## 6. Access Token 활용하기

<br>

- 이후 어플리케이션은 리소스 서버에 원하는 정보 또는 행위를 요청할 때 헤더에 access token을 담아서 요청하도록 한다. 
- Access token을 어플리케이션에 어떻게 관리할지는 내부 논의 후 정해서 저장하면 된다. (Redis, JWT 등등 이 있다.)
- 본인은 프로젝트에서 JWT 토큰의 value로 저장하고 유저에게 JWT 토큰을 반환했다. 

<br>
<br>

## 7. 전체적인 흐름도 

<br>

<p align="center"><img width="700" alt="스크린샷 2021-08-17 오후 4 44 00" src="https://user-images.githubusercontent.com/63405904/129685067-5d0bbbdc-0224-42e8-bed4-fe984cb036ec.png">출처 : NHN Cloud - OAuth 2.0 대표 취약점과 보안 고려 사항 알아보기</p>

<br>
<br>

## 8. 번외) OAuth의 대표 취약점 

<br>

### 8.1. CSRF(Cross Site Request Forgery) 공격 
- OAuth 인증 진행 시, 발급받은 `Authorization Code`와 이전에 발급받은 `client_secret`을 함께 보내어 리소스 서버에 Access Token을 요청하는 단계가 있다. 
- `client_secret`은 CSRF token과 같은 역할을 해서 중간에 CSRF 공격을 예방하는 역할을 하는데, 만일 이것에 대한 검증이 누락되거나 취약하면 CSRF 공격에 의해 Authorization Code가 탈취되어 사용자의 계정이 노출될 수 있다.
- `client_secret`에 대한 검증이 필요하다. 

<br>

### 8.2. Convert Redirect 
- 유저가 로그인한 후 인가를 승인하고 Authorization Code에 대해서 발급할 때 리다이렉트 되는 redirect_uri에 대한 검증이 안될 경우 공격자가 해당 공격 서버의 uri로 대체하여 Authorization Token을 탈취할 수 있다. 
- Redirect URI에 대한 Full path 검증을 진행해야한다. 

<br>
<br>
<br>

**[출처]** 
- https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2
- https://meetup.toast.com/posts/105

```toc
```