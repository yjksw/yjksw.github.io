---
emoji: 🛠
title: Boolean 대신 timestamp
date: '2021-03-17 23:00:00'
author: 코다
tags: 설계
categories: 설계
---

- 이것은 정답이 아니라 한 블로그에 기술된 하나의 의견이다. 읽어보고 신선한 접근이라고 생각해서 정리해둔다. [링크]([https://changelog.com/posts/you-might-as-well-timestamp-it](https://changelog.com/posts/you-might-as-well-timestamp-it))

데이터베이스에서 boolean 값을 지정해서 저장해야하는 경우들이 있다. `is_published`, `is_signed`, `is_finished` 등등을 기록해야하는 경우들이다. 이 경우에 boolean으로 저장하지 말고 timestamp로 저장하도록 해보자! 글쓴이의 말을 인용하자면 "단 한번도 후회한적이 없다". <br>

Boolean 값으로 저장할만한 데이터는 언제 해당 데이터가 set 되었는지에 대한 timestamp를 제공함으로 잃는 것이 없다. 아무리 해당 시간 데이터가 필요하지 않더라도 말이다. 이렇게 구현을 하게 된다면 `null` 은 `false`로 `non-null`은 `true`로 간주되어 처리하면 될 것이다. <br>

```java
//boolean 사용
if (is_finished) {
	return true;
} else {
	return false;
}

//timestamp 사용
if (finished_at) {
	return true;
} else {
	return false;
}
```

따라서 자연스럽게 `deleted_at`, `hidden_at`, `signed_in_at` 등등으로 변환될 것이다. 

### 의견

글쓴이의 말이 일리가 있다. 큰 구현의 차이나 처리의 차이 없이 동일한 연산을 수행할 수 있고, 더 많은 정보를 제공하는 이점이 있다. <br>

하지만 해당 데이터가 `null`로 지정이 되어 있는 시점이 있다는 것이 해당 코드를 취약하게 만들 수도 있을 것 같다. <br>

 이 부분에 대한 다른 크루들의 생각을 첨부! <br>

<p align="center"><img width="90%" alt="_2021-04-25__7 01 27" src="https://user-images.githubusercontent.com/63405904/134771368-791c2734-8e0e-49e4-9b58-b9608036c69f.png"></p>