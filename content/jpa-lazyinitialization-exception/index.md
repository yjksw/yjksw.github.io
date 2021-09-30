---
emoji: 🚀
title: LazyInitializationException 란?
date: '2021-08-04 23:00:00'
author: 코다
tags: JPA
categories: JPA
---

## LazyInitialization 에러 발생 이유

```java
    LazyInitializationException: could not initialize proxy - no session
```

- Lazy Loading을 하려고 하는데 세션이 사라져서 프록시 초기화가 불가능해 지연로딩을 못하는 경우에 발생하는 에러
- 가장 단순한 방법으로는 `FetchType` 을 `EAGER`로 설정해서 일괄적으로 부모 호출시 자식이 모두 즉시로딩으로 초기화되도록 하면 되지만, 비니지스적인 이유가 아닌 `LazyInitialization`을 피하기 위해 `EAGER`로 설정하는 것은 좋지 않다.
- 위 에러는 Lazy Loading을 하는 시점까지 session을 유지시켜주는 것이 핵심이며 주로 `@Transactional` 추가로 해당 문제는 해결된다.

<br>

## 서비스 외부에서 LazyLoading Collection 호출 시

서비스에서 LazyLoading을 하고 싶다면 `@Transactional` 을 붙여서 해결할 수 있지만 간혹 서비스 클래스 밖에서 지연로딩 지정되어 있는 컬렉션을 조회해야할 때가 있다. 그럴 때 `LazyInitialization` 에러를 피할 수 있는 방법은 두가지이다. 

### 1. `Hibernate.initialize()`를 사용해 서비스 내에서 조회

만일 서비스가 아닌 다른 클래스 내에서 JPA 조회를 하고 싶다면 세션이 유지되지 않는다. (`@Transactional` 어노테이션을 붙이는 것이나, `FetchType.EAGER` 을 붙이는 것이 모두 부적합하거나 비효율적이다) <br>

이때는 해당 컬렉션을 조회하는 메서드를 서비스 내에 만들어 호출하여 사용하고, 해당 메서드 내에서 `Hibernate.initialize(Entity)` 를 사용해서 프록시가 즉시 초기화 되도록 하면 된다. <br>

```java
@Transactional(readOnly = true)
	public List<Task> getTaskListEager(Long taskId) {
		List<Task> taskList = this.tasksRepository.findOne(taskId).getTaskList();
		Hibernate.initialize(taskList);
		return taskList;
	}
```

- **주의할 점:** `Hibernate.initialize()` 는 그 안에 중첩되어 있는 LazyCollection에 대해서는 가지고오지 못하므로 그것에 대한 `Hibernate.initialize()`는 따로 해줘야한다.

### 2. TransactionManager 사용

어노테이션이 아니라 Spring에서 제공해주는 TransactionManager를 직접 사용해서 session을 직접 지정하도록 한다. 

```java
@Autowired
private PlatformTransactioinManager transactionManager;

public List<Task> getTaskListEager(Long tasksId) {
	TransactionStatus transactionStatus = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
	List<Task> taskList = null;
	try {
		taskList = this.tasksRepository.findOne(tasksId).getTaskList();
		this.transactionManager.commit(transactionStatus);
	} catch (Exception e) {
		this.transactionManager.rollback(transactionStatus);
	}

	return taskList;
}
```

**[참고자료]**

- [https://thorben-janssen.com/lazyinitializationexception/](https://thorben-janssen.com/lazyinitializationexception/)
- [https://github.com/HomoEfficio/dev-tips/blob/master/Spring Data JPA - LazyInitialization 에러 - getOne().md](https://github.com/HomoEfficio/dev-tips/blob/master/Spring%20Data%20JPA%20-%20LazyInitialization%20%EC%97%90%EB%9F%AC%20-%20getOne().md)

```toc
```