---
emoji: 🚀
title: JPA 프록시 알아보기
date: '2021-08-26 23:00:00'
author: 코다
tags: JPA
categories: JPA
---

## INTRO

- JPA는 DB의 데이터와 객체간의 모순을 해결하기 위해서 나온 것이다.
- 객체는 객체 그래프로 탐색이 가능하지만 데이터베이스에 저장된 데이터는 객체를 탐색하듯이 탐색하기가 어렵다.
- 따라서 데이터베이스에 저장된 데이터들을 가지고 객체 그래프 탐색이 가능하기 위해서 프록시 라는 기술이 나왔다.
- JPA에서는 연관된 객체를 로딩하는데 지연 로딩과 즉시 로딩이라는 두가지 기법을 사용한다.
- 여기서 지연 로딩시 프록시 기술이 사용된다.

<br>
<br>

## 기본 예시 Entity
1. Member
    ```java
    @Entity
    public class Member {

        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        private String name;

        @ManyToOne(cascade = CascadeType.PERSIST)
        @JoinColumn(name = "team_id")
        private Team team;

        protected Member() {
        }

        public void setTeam(Team team) {
            this.team = team;
        }

        //getter 및 생성자 생략
    }
    ```
<br>

2. Team 
    ```java
    @Entity
    public class Team {

        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        private String name;

        protected Team() {
        }

        //getter 및 생성자 생략 
    }
    ```

<br>
<br>

## 프록시
- 지연 로딩을 설정된 연관 객체를 가져올 때 프록시 객체를 가져온다.
- `em.find()`를 사용하면 실제 객체를 가져오고, `em.getReference()`를 사용하면 프록시 객체를 가져온다.
    - 프록시 객체를 가져온다는 것은 query는 실행되지 않는 것이다.
    - 특정 객체에 대해 `em.find()`를 하면 연관된 객체에 대해서는 설정되어 있는 `fetch` 종류에 따라 즉시로딩 혹은 지연로딩한다.

<br> 

### 실제 Entity 조회 - `find()`

<br>

```java
@DisplayName("em.find()는 실제 entity를 조회하여 가져온다.")
@Test
void find_actualEntity() {

    Member member = new Member("member1");
    Team team = new Team("Team A");
    member.setTeam(team);

    Member savedMember = memberRepository.save(member);

    testEntityManager.flush();
    testEntityManager.clear();

    EntityManager em = testEntityManager.getEntityManager();
    Member findMember = em.find(Member.class, savedMember.getId()); //select 퀴리가 호출된다.

    assertThat(findMember.getId()).isNotNull();
    assertThat(findMember.getName()).isEqualTo("member1");
    assertThat(findMember.getClass()).isEqualTo(Member.class); //실제 엔티티 클래스임을 확인할 수 있다.
}
```
- 위 테스트를 실행하면 다음과 같은 select 쿼리가 실행된다. 
```sql
select
    member0_.id as id1_0_0_,
    member0_.name as name2_0_0_,
    member0_.team_id as team_id3_0_0_,
    team1_.id as id1_1_1_,
    team1_.name as name2_1_1_ 
from
    member member0_ 
left outer join #ManyToOne, OneToOne 기본 fetch 전략은 EAGER이므로 left join을 실행해서 Team까지 즉시로딩한다. 
    team team1_ 
        on member0_.team_id=team1_.id 
where
    member0_.id=?
```

<br>

### 프록시 Entity 조회 - `getReference()`

<br>

```java
@DisplayName("em.getReference()는 해당 entity에 대한 proxy 객체를 가져온다.")
@Test
void getReference_proxyEntity() {

    Member member = new Member("member1");
    Team team = new Team("Team A");
    member.setTeam(team);

    Member savedMember = memberRepository.save(member);

    testEntityManager.flush();
    testEntityManager.clear();

    EntityManager em = testEntityManager.getEntityManager();
    Member findMember = em.getReference(Member.class, savedMember.getId()); //아무 쿼리가 호출되지 않는다.

    assertThat(findMember.getId()).isNotNull(); //getId()는 여전히 쿼리는 호출되지 않는다.

    assertThat(findMember.getClass()).isNotEqualTo(Member.class); //실제 Entity 클래스가 아니다.

    assertThat(findMember).isInstanceOf(Member.class); //프록시 객체는 실제 Entity 를 상속한다.

    System.out.println(findMember.getClass()); //출력 : class com.example.jpastudy.domain.Member$HibernateProxy$VGt5eoXR
}
```
- 위 테스트 코드는 select 쿼리를 호출하지 않는다. 
- `getId()`는 해당 entity에 대한 getter이기는 하나, 프록시는 해당 entity에 대한 식별자를 보관하므로 `getId()`를 호출하더라도 entity를 초기화하지 않는다. (여진히 쿼리가 실행되지 않는다.)
- 위에서 `findMember.getClass()`를 출력해보면 `class com.example.jpastudy.domain.Member$HibernateProxy$VGt5eoXR` 와 같이 프록시 객체 임을 확인할 수 있다. 

<br>
<br>

## 프록시 객체 특징 및 초기화  

- 초기화는 영속성 컨텍스트에 실제 객체가 없을 경우 엔티티 생성을 요청하여 영속성 컨텍스트에 실제 entity를 로드하고 참조값을 저장하는 것이다. 
- 프록시 객체는 실제 객체를 상속한다. 따라서 사용할 때는 실제 Entity처럼 사용할 수 있다. 
<p align="center" ><img width="500" alt="스크린샷 2021-08-15 오후 3 58 10" src="https://user-images.githubusercontent.com/63405904/129469950-dd48c26f-5554-4fdd-9e1f-5d38399ae46b.png"><br>이미지 출처 : 자바 ORM 표준 JPA 프로그래밍 (저자: 김영한)</p>
<br>

- 프록시 객체는 실제 Entity에 대한 식별자와 해당 Entity의 참조값을 저장하고 있다. 따라서 `getId()`로 식별자 조회 시 초기화 하지 않는다. 
- 더불어 프록시 객체가 초기화 될 때 실제 entity 객체로 대체되는 것이 아니라, 프록시에 저장되어 있는 entity 참조값으로 실제 객체 메소드에 위임하여 메소드가 실행된다. 
<p align="center" ><img width="500" alt="스크린샷 2021-08-15 오후 4 03 26" src="https://user-images.githubusercontent.com/63405904/129470031-d0c8bcd1-ea2a-479c-a45f-bc3afa4f20de.png"><br>이미지 출처 : 자바 ORM 표준 JPA 프로그래밍 (저자: 김영한)</p>

<br>

### Getter 호출시 (`getId()` 제외) 프록시 객체 초기화 - `getReference()` -> `getName()`

<br>

```java
@DisplayName("proxy 객체는 getName() 등을 호출 할 경우 초기화 된다.")
@Test
void getReference_proxyEntity_getName() {

    Member member = new Member("member1");
    Team team = new Team("Team A");
    member.setTeam(team);

    Member savedMember = memberRepository.save(member);

    testEntityManager.flush();
    testEntityManager.clear();

    EntityManager em = testEntityManager.getEntityManager();
    Member findMember = em.getReference(Member.class, savedMember.getId()); //아무 쿼리가 호출되지 않는다.

    assertThat(findMember.getId()).isNotNull(); //getId()는 여전히 쿼리는 호출되지 않는다.
    assertThat(findMember.getName()).isEqualTo("member1"); //select 쿼리가 실행된다.

    assertThat(findMember.getClass()).isNotEqualTo(Member.class); //초기화 되어도 여전히 실제 Entity 클래스가 아니다.
    assertThat(findMember).isInstanceOf(Member.class); //프록시 객체는 실제 Entity 를 상속한다.
}
```
<br>

```sql
select
    member0_.id as id1_0_0_,
    member0_.name as name2_0_0_,
    member0_.team_id as team_id3_0_0_,
    team1_.id as id1_1_1_,
    team1_.name as name2_1_1_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.id 
where
    member0_.id=?
```
- getName() 실행 시 위 select 쿼리가 실행된다. 

<br>

### 프록시 특징
- 처음 단 한번만 초기화 된다. 
- 프록시 객체가 대체되는 것이 아니라 프록시 객체를 통해 실제 entity 접근하는 것이다. 
- 타입 체크시 주의해야 한다. (아래 더 자세히 다룸)
- 영속성 컨텍스트에 이미 해당 entity가 있다면 `getReference()`를 해도 실제 entity가 조회된다. 
- 영속성 컨텍스트를 통해 초기화 하므로 영속성 컨텍스트와 연결이 없다면 `LazyInitializationException` 예외가 발생한다. 

<br>
<br>

## 프록시 객체 타입 체크 

- 프록시 객체의 타입은 `getClass()` 로 비교할 경우 프록시 객체 클래스가 조회되기 때문에 비교하기 위해서는 `instanceOf` 를 사용해야 한다. 
- 많은 경우 프록시 객체를 `getClass()`로 실제 entity와 비교하여 버그가 발생한다. 

```java
@DisplayName("proxy 객체의 타입은 실제 entity 객체 타입과 다르다. getClass()로 비교할 경우 false가 반환된다.")
@Test
void proxyType_getClass() {

    Member member = new Member("member1");
    Team teamA = new Team("Team A");
    member.setTeam(teamA);

    Member savedMember = memberRepository.save(member);

    testEntityManager.flush();
    testEntityManager.clear();

    //프록시 객체를 가져오기 위해 Member의 Team 프로퍼티를 FetchType.LAZY로 설정
    EntityManager em = testEntityManager.getEntityManager();
    Member findMember = em.find(Member.class, savedMember.getId()); //Team은 지연로딩으로 프록시객체이다.
    Team findTeam = findMember.getTeam();

    assertThat(teamA.getName()).isEqualTo(findTeam.getName()); //기존 TeamA와 프록시 팀인 findTeam의 필드값은 동일하다.
    assertThat(teamA.getClass()).isNotEqualTo(findTeam.getClass()); //프록시 팀읜 findTeam의 class는 Team이 아니다.

    assertThat(findTeam).isInstanceOf(Team.class); //isInstanceOf로 비교하면 상속 받은 Team.class 이다.
}
```

<br>
<br>

## 영속성 컨텍스트에 있는 entity getReference

- 영속성 컨텍스트에 조회하고자 하는 entity가 이미 존재한다면 `getReference()`를 사용하더라도 프록시 객체가 아닌 실제 entity 가 반환된다. 
<br>

```java
@DisplayName("이미 영속성 컨텍스트에 있는 entity 조회시 getReference로 조회해도 실제 entity가 반환된다.")
@Test
void getReference_AlreadyInPersistenceContext() {
    Member member = new Member("member1");
    Team teamA = new Team("Team A");
    member.setTeam(teamA);

    Member savedMember = memberRepository.save(member); //영속성 컨텍스트에 존재햔다.

    EntityManager em = testEntityManager.getEntityManager();
    Member findMember = em.getReference(Member.class, savedMember.getId()); // 프록시 객체를 가져오듯 메서드를 호출한다.

    assertThat(findMember.getClass()).isEqualTo(Member.class); //프록시가 아닌 실제 entity 클래스를 가져온다. 추가적인 쿼리는 실행되지 않는다.
}
```

<br>
<br>

## 준영속 entity 초기화 

- 준영속이 된 프록시 객체에서 초기화를 시도할 경우 하이버네이트의 `LazyInitializationException`이 발생한다. 
- 프록시의 초기화는 영속성 컨텍스트와 매우 밀접하게 관련이 있으므로 영속성 컨텍스트와의 관계가 끊어지면 에외가 발생한다. 

```java
@DisplayName("준영속 프록시 객체에 대한 초기화 시 예외가 발생한다.")
@Test
void getReference_InitializationExcpetion() {

    Member member = new Member("member1");
    Team teamA = new Team("Team A");
    member.setTeam(teamA);

    Member savedMember = memberRepository.save(member);

    testEntityManager.flush();
    testEntityManager.clear();

    EntityManager em = testEntityManager.getEntityManager();
    Member findMember = em.find(Member.class, savedMember.getId()); //Team 을 lazy 로딩 설정하여 프록시 객체로 가져온다.

    em.clear(); //먼저 영속성 컨텍스트를 clear 해주어야한다. 
    em.close(); //프록시 객체를 가져온 영속성 컨텍스트를 종료한다.

    assertThatThrownBy(() -> findMember.getTeam().getName()) //영속성 컨텍스트가 종료된 이후에 초기화를 시도하면
        .isInstanceOf(LazyInitializationException.class); //하이버네이트 예외가 발생한다.
}
```
- `Member`의 `Team`을 지연로딩으로 `fetch` 타입을 설정하여 `Member` 조회시 `Team`은 프록시 객체이도록 한다. 
- 영속성 컨텍스트를 닫은 후에 초기화를 시도하면 예외가 발생한다. 
    - 단순히 `Member.getTeam()` 이 초기화를 실행하지 않고 실제 `Team`의 필드가 사용될 때 (`getName()` 등) 초기화가 된다. 

<br>
<br>

## 프록시 객체 확인

- 프록시 객체 확인을 위해서는 `PersistenceUnitUtil.isLoaded(Object entity)`를 사용해서 확인할 수 있다. 
- 때때로 jpa repostiory에 대한 단위 테스트를 작성하려고 할 때, 의도한대로 지연로딩 또는 즉시로딩이 되었는지 확인할 때 사용할 수 있다. 
- 만일 초기화가 안된 프록시 객체라면 위 결과값이 `false`가 반환된다. 

```java
@DisplayName("프록시 객체를 확인할 수 있다. ")
@Test
void PersistenceUnitUtil_Proxy() {

    Member member = new Member("member1");
    Team teamA = new Team("Team A");
    member.setTeam(teamA);

    Member savedMember = memberRepository.save(member);

    testEntityManager.flush();
    testEntityManager.clear();

    EntityManager em = testEntityManager.getEntityManager();
    Member findMember = em.find(Member.class, savedMember.getId()); //Team 을 lazy 로딩 설정하여 프록시 객체로 가져온다.

    boolean memberIsLoaded = em.getEntityManagerFactory()
        .getPersistenceUnitUtil().isLoaded(findMember);

    boolean teamIsLoaded = em.getEntityManagerFactory()
        .getPersistenceUnitUtil().isLoaded(findMember.getTeam()); //EntityManagerFactory에서 PersistenceUnitUtil을 가져온다.

    assertThat(memberIsLoaded).isTrue(); //findMember 는 실제 entity 이므로 결과값이 true 이다.
    assertThat(teamIsLoaded).isFalse(); //team 은 프록시 객체이므로 결과값이 false 이다.
}
```

```toc
```