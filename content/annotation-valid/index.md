---
emoji: 🖥
title: "@Valid 어노테이션 초간단 입문"
date: '2021-05-24 23:00:00'
author: 코다
tags: 스프링부트
categories: 스프링부트
---

**@Valid** - 스프링 부트에서 어노테이션으로 validation을 할 수 있도록 기능을 추가해주는 것. 즉, controller에서 인자를 받을 때 유효성 검사를 할 수 있도록 해주는 것이다. 

DTO의 필드나, 도메인 객체의 필드 위에 유효성 검사를 하고 싶은 어노테이션을 추가하고, controller의 인자 앞에 **@Valid** 를 추가해서 붙여준다. 

## java.validation 어노테이션

```java
@NotNull

@Null

@NotEmpty

@NotBlank 

@Size(min=, max=)

@Pattern(regex=) : 정규식 만족

@Max(숫자) 

@Min(숫자) 

@Positive

@PositiveOrZero

@Negative

@NegativeOrZero

@Email

@Digits(integer=, fraction=) : 대상 수가 지정된 정수와 소수 자리수보다 작은가? 

@DecimalMax(value=)

@DecimalMin(value=)

@AssertFalse

@AssertTrue
```

### 사용 예시

```java
public class Member {
	
	@NotNull(message= "id는 필수 값입니다.")
	@Size(min = 5, max = 10)
	private String id;
	
	@Max(value = 25, message = "25세 이하만 가능합니다") 
	@Min(value = 18, message = "18세 이상만 가능합니다")
	private int age;

	@Pattern(regxp = "...")
	private String email;

```