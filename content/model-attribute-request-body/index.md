---
emoji: ✊
title: "@ModelAttribute vs. @RequestBody"
date: '2021-03-25 23:00:00'
author: 코다
tags: 스프링부트
categories: 스프링부트
---

이번에 스프링 체스 자바 웹 어플리케이션을 사용하여 구현하면서 처음에는 모두 `@RequestParam` 으로 데이터를 가져왔었다. 하지만 인자가 너무 많아지는 경우 메서드에 파라미터가 많아지면서 가독성이 안 좋아졌다. 또한 DTO에 해당 데이터를 담아서 서비스 레이어에 전달해야하거나 할 때 일일이 데이터를 DTO에 담아서 가공해야 하는 작업을 해야하기도 했다. 코드를 구현할 때 손가락이 아프다면 수정할 부분을 찾으라고 했었는데 확실히 `@RequestParam`으로 받는 것은 손가락이 아팠다. <br>

아니나 다를까 리뷰어가 @ModelAttribute를 사용하는 걸 추천했다. 마침 레벨1 제이슨 톡방에서도 @ModelAttribute 에 대한 논의가 활발하길래 공부도 하고 코드에 적용을 하며 배운 것을 기록해본다. 

## @ModelAttribute vs. @RequestBody

간단하게 말하면 다음과 같다.

```java
1. @ModelAttribute은 form data로 오는 데이터를 저장한다. 
2. @RequestBody는 JSON/xml 타입으로 오는 body에 담긴 데이터를 저장한다.
```

`@ModelAttribute` 같은 경우는 parameter 값으로 DTO에 바인딩한다. 따라서 해당 DTO 객체에 `setter` 메소드가 반드시 있어야 한다. 따라서 타입에 대한 검증을 한 후에 setting을 한다. 

`@RequestBody`의 경우는 본문 body에 있는 Json/xml 타입을 바인딩하기 때문에 `HttpMessageReader`를 통해서 `ObjectMapper`를 한다. 

여기서 `HttpMessageReader`는 들어온 request body의 데이터값을 Java object로 역직렬화 해준다. 이때 역직렬화는 `ObjectMapper`의 `readValue()` 메서드로 변환하므로 setter가 필요가 없다. (단, 기본 생성자의 경우는 필요한 경우가 많다) 

## 사용예시

- form 데이터로 넘어오는 경우 @ModelAttribute를 사용해서 바인딩한다. 여기서 @RequestBody를 쓰면 에러가 발생한다.

    ```java
    @PostMapping("/white-game")
        public String whiteStart(Model model, HttpSession session, @ModelAttribute RoomInfoRequestDto roomInfoRequestDto) {
            String room = roomInfoRequestDto.getRoomName();
            String password = roomInfoRequestDto.getPassword();

            model.addAttribute("room", room);
            session.setAttribute("password", password);

            return "game";
        }
    ```

- 아래의 경우 js에서 데이터를 보내줄 content-type을 지정하지 않았을 때 오류가 났다. 이후에 @RequestBody는 json 타입을 받기 때문에 `Content-type`을  `application/json`으로 지정하니 잘 바인딩이 되었다.

    ```java
    @PutMapping("/game")
        public MoveResponseDto move(HttpSession session, @RequestBody MoveRequestDto moveRequestDto) {
            String password = (String) session.getAttribute("password");
            return chessService.move(moveRequestDto, password);
        }
    ```

- 번외로, 서버로 내보내는 content-type에서 json을 지정하고 싶을 경우 `Content-type : 'application/json'` 으로 지정하고 서버에서 받는 데이터의 경우 `Data-type : 'json'` 으로 지정해야 한다. (중요한 이유는 없고 그냥 설정이 그렇게 되어 있음)


```toc
```