---
emoji: 🖥
title: 쿠키와 세션 알아보기
date: '2021-09-04 23:00:00'
author: 코다
tags: 웹
categories: 웹
---

## INTRO
- HTTP는 Stateless 무상태성을 가지고 있다. 따라서 데이터를 상태로 저장하지 않는다.
- HTTP가 무성태성이기 때문에 클라이언트에 대한 데이터를 유지하고 싶을 때는 쿠키 또는 세션을 이용한다. (이전에 요청을 보낸 동일한 사용자임을 확인하고 싶은 경우 등등)
- 쿠키는 클라이언트가 정보를 가지고 브라우저에서 저장 및 관리한다. 
- 세션은 서버가 데이터를 가지고 저장 및 관리한다. 

<br>
<br>

## Cookie
- 쿠키는 클라이언트가 정보를 가지고 브라우저에서 해당 정보를 저장한다. 
- 따라서 요청을 보낼 때마다 브라우저에서 저장된 쿠키 데이터를 HTTP 헤더에 추가하여 서버에 보낼 수 있다.
    - HTTP 메세지 자체는 무상태성이기 때문에 매번 쿠키값을 보내주어야 한다. 
- 쿠키에 저장되는 값의 형태는 text 이다. 

### 쿠키의 단점 
- 클라이언트가 관리하는 것이기 때문에 데이터가 쉽게 훼손 될 수 있다. 
    - 실제로 크롬 브라우저에서 개발자 도구 -> Application 탭에 가면 쿠키 데이터를 저장하는 저장소를 볼 수 있다. 
    - 여기서 값을 조회, 수정, 삭제를 할 수 있다. 
    <img width="636" alt="스크린샷 2021-09-04 오후 3 10 59" src="https://user-images.githubusercontent.com/63405904/132084874-b28a2e15-6ade-4fec-87a5-87434675f0b4.png">
- 다른 사람이 쉽게 열람할 수 있다. 따라서 민감한 정보를 저장할 수 없다. 

<br>

### 쿠키 설정하기 
- 쿠키를 설정하고 싶을때는 아래와 같이 서버에서 `Set-Cookie` 헤더로 값을 보내면 된다. 
    ```
    HttpResponse : 

    HTTP/1.1 302 Found 
    Set-Cookie: cookieValue=thisIsCookie!!!
    Location: http://localhost:8080/index.html 
    ```

- 그 이후 모든 클라이언트의 요청에는 위에 설정한 쿠키값이 Cookie 헤더로 서버에 전송된다. 
    ```
    HttpRequest : 

    GET /login HTTP/1.1
    Host: localhost:8080
    Connection: keep-alive
    Pragma: no-cache
    Cache-Control: no-cache
    Cookie: cookieValue=thisIsCookie!!!
    ```

<br>
<br>

## Session
- 세션은 서버의 메모리에서 데이터를 관리한다. 
- 외부에 공개하기 위험한 정보를 서버의 메모리에 안전하게 저장할 수 있다. 
- 세션을 식별하기 위한 고유한 id인 session id가 부여되고 클라이언트는 해당 세션 Id만을 저장하고 관리하면 된다. 
- 세션에 저장되는 값의 형태는 object 이다. 

### 세션의 단점 
- 서버의 메모리에서 지나치게 많은 세션 데이터를 관리하기 힘들 수 있다. 

<br>

### 세션 설정하기 
- 서버에서 세션을 설정할 때 `HttpServletRequest`의 `getSession()`을 호출하면 해당 request의 세션을 생성해서 반환한다. 아래에 세션 생성 및 반환 시점에 대해서 추가로 설명한다. 
- 반환된 세션에 `setAttribute()`를 통해 속성값을 지정한다. 해당 값이 세션에 저장되는 데이터 정보이다. 
- 세션의 `getId()`를 호출해 해당 세션의 id를 `Set-Cookie` 헤더에 `JSESSIONID` 라는 key로 추가하여 클라이언트에게 응답힌다. 
- 이후에 쿠키로 등록된 세션 id를 통해서 세션을 유지하고 속성값에 접근할 수 있다. 
- **코드로 보기**
    - 스프링부트 웹 프로젝트라고 가정한다. 
    ```java
    @GetMapping("/login")
    public ResponseEntity<> login(String account, String password, HttpServletRequest request) {
        User loginUSer = new User(account, password);
        
        HttpSession session = HttpServletRequest.getSession();
        session.setAttribute("user", user);
        String sessionId = session.getId();
        
        HttpHeaders responseHeaders = new HttpHeaders();
        responseHeaders.set("Set-Cookie", "JSESSIONID=" + sessionId);
        responseHeaders.set("Location", "index.html");

        return ResponseEntity.found(responseHeaders);
    }
    ```
- 그럼 다음과 같이 응답이 나간다. 
    ```
    HTTPResponse : 

    HTTP/1.1 302 Found 
    Set-Cookie: JSESSIONID=2cfd4acb-0b67-4ce9-90c4-97ee3242e51b 
    Location: http://localhost:8080/index.html 
    ```
- 이후 클라이언트가 요청을 보낼 때마다 쿠키에 세션 ID 값이 포함되어서 보내어 진다. 이 아이디를 통해서 세션을 매핑하고 세션이 유지되는 것을 확인할 수 있다. 
    ```
    HTTPRequest : 

    GET /login HTTP/1.1
    Host: localhost:8080
    Connection: keep-alive
    Pragma: no-cache
    Cache-Control: no-cache
    Cookie: JSESSIONID=2cfd4acb-0b67-4ce9-90c4-97ee3242e51b
    ```

<br>

### 세션은 언제 생성될까 ?
- 이번에 직접 HTTP 서버를 구현하는 프로젝트를 하면서 세션을 구현하면서 의문이 들었다. 
- `request.getSession()`을 하면 항상 세션을 반환하는데 그럼 모든 HttpRequest에 대한 세션이 생성되는 것인지 궁금했다. 
- `request.getSession()`을 하면 유지되던 세션, 혹은 새로 생성된 세션이 반환되므로 모든 요청에 대한 세션을 생성한다고 생각할 수 있지만 그렇지 않다. 
- 세션을 만드는 것도 비용이 필요하기 때문에 세션이 필요할 때만 lazy 하게 생성한다. 
<br>

- `HttpSession`의 경우 세션은 ServletContainer에서 생성이 된다. 
- 따라서 `HttpServletRequest`에서 세션을 요청할 때 서블릿 컨테이너에서 내부적으로 생성해서 세션을 반환한다. 
- 실제로 세션이 생성되는 시기는 `@Autowired`로 세션이 주입되는지, 메서드 인자로 주입되는지, request에서 `getSession()`을 호출하는 지에 따라서 다르므로 참고하자. 
- 나는 주로 controller 메서드의 인자로 세션을 호출하는데 그때는 컨트롤러 메서드를 호출하는 즉시 세션을 생성/요청해서 주입해준다. 
    ```java
    @PostMapping("login")
    public ResponseEntity<ResponseMessage> login(@RequestBody User user, HttpSession session) {
        //..로직 생략 
    }
    ```
<br>

- 세션을 유지 할 때는 헤더에 관련 키가 있다면(`JSESSIONID`와 같은 쿠키값) 해당 세션을 요청해서 반환하고, 없다면 새로운 세션을 생성해서 반환한다. 

### 주의할 점
- 세션이 쿠키보다 보안이 좋은 것은 사실이나, 쿠키에 저장된 `JSSESIONID`를 탈취하여 다른 브라우저의 쿠키값으로 설정한다면 서버는 위 세션 아이디와 동일한 브라우저로 판단해 요청을 처리한다. 
- 예를 들어서 A 사용자가 로그인해서 부여받은 세션 ID를 다른 브라우저의 쿠키값으로 집어넣기만 해도 A 사용자가 로그인한 상태로 요청을 보내는 것으로 처리되므로 여전히 위험할 수 있다. 

<br>
<br>

## 언제 무엇을 사용하는 것이 좋을까?
- 언제 세션을 사용하고 언제 쿠키를 사용하는지에 대해서는 물론 답이 없다. 
- 웹을 구현하면서 워낙 많은 경우가 있기 때문에 각각 다른 이유들을 비교해서 더 적합한 것을 선택하면 된다. 

- 나의 경우에는 민감한 정보가 있을 때는 세션을, 그렇지 않은 경우는 쿠키로 관리한다.

<br>
<br>

## 번외) static `HttpSessions` 클래스 테스트하기 `MockStatic<>` 
- 직접 Http 서버를 구현하면서 세션에 고유한 id를 부여하기 위해 `UUID.randomUUID()`를 통해서 구현했다. 
- `HttpSession` 을 관리하는 `HttpSessions`는 `static`으로 관리하고 있고, 해당 클래스에서 `UUID.randomUUID()`를 사용해 새로운 HttpSession을 생성하고 아이디를 부여해 리턴 및 저장한다. 
- 매번 랜덤한 값을 id로 부여하기 때문에 테스트 코드를 짜는데 어려움이 있었고, Mocking을 하려고 했으나 HttpSessions가 static이므로 기존의 모킹 방식으로는 테스트하기가 어려웠다. 
- 따라서 static 을 모킹할 수 있는 방법으로 테스트를 진행했다. 
    ```java
    private static MockedStatic<HttpSessions> mockHttpSessions;

    String sessionId = "id";
    when(HttpSessions.getSession()).thenReturn(new HttpSession(sessionId));
    ```

<br>
<br>
<br>

**[참고자료]**
- [https://hahahoho5915.tistory.com/32](https://hahahoho5915.tistory.com/32)
- [https://soon-devblog.tistory.com/2](https://soon-devblog.tistory.com/2)
- [https://www.youtube.com/watch?v=OpoVuwxGRDI](https://www.youtube.com/watch?v=OpoVuwxGRDI)

```toc
```