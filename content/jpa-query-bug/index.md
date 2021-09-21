---
emoji: 🚀
title: JPA N+1 문제 및 해결방법 알아보기
date: '2021-08-26 23:00:00'
author: 코다
tags: JPA 스프링
categories: JPA 스프링
---

# INTRO 

- JPA에서는 데이터와 객체지향으로 설계 사이의 모순을 해소하기 위해서 나온 기술이다. 
- 많은 객체들은 내부에 Collection 형태로 다른 객체에 대한 참조가 가능하게 설계된다. 
    - 예) `Team` 객체 내부에 `Team`에 속해있는 `List<Member>` 와 팀에 할당된 `List<Locker>`가 존재한다. 
- 이때 상위 객체를 select 하면서 하위 객체를 가져오는 경우 다음 두가지 fetch 타입에 각각 다음과 같은 문제가 있다. 
    - `fetch = FetchType.EAGER`일 경우 : `MultipleBagFetchException`이 발생
    - `fetch = FetchType.LAZY`일 경우 : `N+1` 문제 발생
- 상위 엔티티에서 다수의 collection 형태의 연관엔티티를 가지고 있을 때 여러 상황 및 문제와 해결 방법에 대해서 공부해본다. 

<br>

# Entity 상황

- `Team`과 `Member`가 1:N 연관관계 - `FetchType.LAZY`, `cascadeType.PERSIST`
- `Team`과 `Locker`가 1:N 연관관계 - `FetchType.LAZY`, `cascadeType.PERSIST`
- 참고 : 우선 모든 연관관계는 `LAZY`로 적용하고 테스트 상황에 따라 `EAGER`로 변경

1. `Team` 엔티티
    ```java
    @Entity
    public class Team {

        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        private String name;

        @OneToMany(
            mappedBy = "team",
            cascade = CascadeType.PERSIST,
            fetch = FetchType.EAGER,
            orphanRemoval = true
        )
        private List<Member> members = new ArrayList<>();

        @OneToMany(
            mappedBy = "team",
            cascade = CascadeType.PERSIST,
            fetch = FetchType.EAGER,
            orphanRemoval = true
        )
        private List<Locker> lockers;

        protected Team() {}

        //...생성자 생략 

        public void addLocker(Locker locker) {
            locker.setTeam(this);
            lockers.add(locker);
        }

        public void setMembers(List<Member> members) {
            members.stream()
                .forEach(member -> member.setTeam(this));

            this.members = members;
        }

        //...getter 생략
    }
    ```

2. `Member` 엔티티
    ```java
    @Entity
    public class Member {

        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        private String name;

        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "team_id")
        private Team team;

        protected Member() {}

        //생성자 및 setter, getter 생략
    }
    ```

3. `Locker` 엔티티
    ```java
    @Entity
    public class Locker {

        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        private String name;

        @ManyToOne
        @JoinColumn(name = "team_id")
        private Team team;

        protected Locker() {}

        //생성자 및 setter, getter 생략
    }
    ```

<br>

# JPA에서 collection fetch join

- Team에 대한 모든 정보가 필요한 경우 Team을 가져오면서 Members와 Lockers 정보도 모두 함께 가지고 와야하는 경우가 있을 수 있다. 
- JPA에서는 `OneToMany`관계에서 fetch join을 할 경우 중복 데이터가 발생한다. 따라서 해결을 하기 위해서 쿼리에 `distinct`를 추가해주어 중복을 없애야한다. 

    1. `distinct`가 없는 collection fetch join
        - TeamRepository.java
        ```java
        @Query("select t from Team t join fetch t.members")
        List<Team> findAllJoinFetchMembers();
        ```

    2. `distinct` 있는 collection fetch join
        - TeamRepository.java
        ```java
        @Query("select distinct t from Team t join fetch t.members")
        List<Team> findAllJoinFetchMembers();
        ```

    3. 실제 실행된 query 
        ```sql
        select
            team0_.id as id1_2_0_,
            members1_.id as id1_1_1_,
            team0_.name as name2_2_0_,
            members1_.name as name2_1_1_,
            members1_.team_id as team_id3_1_1_,
            members1_.team_id as team_id3_1_0__,
            members1_.id as id1_1_0__ 
        from
            team team0_ 
        inner join
            member members1_ 
                on team0_.id=members1_.team_id
        ```
    <br>

    - Test Method 
        ```java
        @BeforeEach
        void setUp() {
            Team teamA = new Team("teamA");
            teamA.addLocker(new Locker("L1"));
            teamA.addLocker(new Locker("L2"));
            Team teamB = new Team("teamB");
            teamB.addLocker(new Locker("L3"));
            teamB.addLocker(new Locker("L4"));

            Member memberA1 = new Member("memberA1");
            Member memberA2 = new Member("memberA2");
            Member memberA3 = new Member("memberA3");

            Member memberB1 = new Member("memberB1");
            Member memberB2 = new Member("memberB2");
            Member memberB3 = new Member("memberB3");

            List<Member> teamAMembers = List.of(memberA1, memberA2, memberA3);
            List<Member> teamBMembers = List.of(memberB1, memberB2, memberB3);

            teamA.setMembers(teamAMembers);
            teamB.setMembers(teamBMembers);

            teamRepository.save(teamA);
            teamRepository.save(teamB);

            testEntityManager.flush();
            testEntityManager.clear();
        }

        @DisplayName("collection fetch join 데이터에 대해 데이터 뻥튀기가 발생한다.")
        @Test
        void collectionFetchJoin_DuplicatedDataSelected() {

            List<Team> teams = teamRepository.findAll(); //members와 lockers가 지연로딩이므로 select 쿼리 1개
            assertThat(teams.size()).isEqualTo(2);

            testEntityManager.flush();
            testEntityManager.clear();

            System.out.println("======================================");

            List<Team> teamsWithMembers = teamRepository.findAllJoinFetchMembers();
            assertThat(teamsWithMembers.size()).isEqualTo(6); // members에 대한 fetch join을 할 경우(distinct 없음) 데이터 뻥튀기가 발생한다.

            List<Team> teamsWithMembers = teamRepository.findAllJoinFetchMembers(); 
            assertThat(teamsWithMembers.size()).isEqualTo(2); // members에 대한 fetch join을 할 경우(distinct 있음) 데이터 중복이 일어나지 않는다.
        }
        ```

<br>

# `FetchType.EAGER` - `MultipleBagFetchException` 발생 


- 현재는 위에 연관관계 fetchType이 `LAZY`로 되어 있지만 두 가지를 `EAGER`로 설정하여 즉시로딩 할 경우 `MultipleBagFetchExcpetion`이 발생한다. 
- JPA에서는 다수의 `OneToMany`로 연관관계가 맺어져 있는 연관 엔티티에 대한 즉시로딩을 막고 있는데, 위에서 언급한 컬렉션 즉시로딩 시 발생하는 데이터 뻥튀기에 대한 제어가 어렵기 때문이다. 
- 발생 예외 - <br>
    `Caused by: org.hibernate.loader.MultipleBagFetchException: cannot simultaneously fetch multiple bags:` 

<br>

# `FetchType.LAZY` - `N+1 문제` 발생

- `Team`의 `Member`와 `Locker`가 각각 `OneToMany` 연관관계가 맺어져 있다. 
- 다음과 같이 메소드를 실행 할 때 N + 1 쿼리가 발생한다. 
- 실행 서비스 로직
    ```java
    @DisplayName("Team의 Members, Lockers 관련 서비스 로직이 있을 때 N+1 문제가 발생한다.")
    @Test
    void findAll_TooManyQuery() {

        List<Team> teams = teamRepository.findAll(); //모든 teams를 조회하는 select 쿼리 1개가 발생한다.

        List<String> memberNames = teams.stream() //각 팀 A개에 대한 각각의 Member select 쿼리 N개가 발생한다.
            .map(Team::getMembers)
            .flatMap(Collection::stream)
            .map(Member::getName)
            .collect(toList());

        List<String> lockerNums = teams.stream() //각 팀 A개에 대한 각각의 Locker select 쿼리 N개가 발생한다.
            .map(Team::getLockers)
            .flatMap(Collection::stream)
            .map(Locker::getName)
            .collect(toList());

        assertThat(teams.size()).isEqualTo(2);
        assertThat(memberNames.size()).isEqualTo(6);
        assertThat(lockerNums.size()).isEqualTo(4);
    }
    ```
- 실제 발생 쿼리
    ```sql
    Hibernate: #Team 전체 조회
    select
        team0_.id as id1_2_,
        team0_.name as name2_2_ 
    from
        team team0_
    Hibernate: #첫번째 Team의 Member 조회
        select 
            members0_.team_id as team_id3_1_0_,
            members0_.id as id1_1_0_,
            members0_.id as id1_1_1_,
            members0_.name as name2_1_1_,
            members0_.team_id as team_id3_1_1_ 
        from
            member members0_ 
        where
            members0_.team_id=?

    Hibernate: #두번째 Team의 Member 조회
        select
            members0_.team_id as team_id3_1_0_,
            members0_.id as id1_1_0_,
            members0_.id as id1_1_1_,
            members0_.name as name2_1_1_,
            members0_.team_id as team_id3_1_1_ 
        from
            member members0_ 
        where
            members0_.team_id=?
    
    Hibernate: #첫번째 Team의 Locker 조회
        select
            lockers0_.team_id as team_id3_0_0_,
            lockers0_.id as id1_0_0_,
            lockers0_.id as id1_0_1_,
            lockers0_.name as name2_0_1_,
            lockers0_.team_id as team_id3_0_1_ 
        from
            locker lockers0_ 
        where
            lockers0_.team_id=?
    
    Hibernate: #두번째 Team의 Locker 조회
        select
            lockers0_.team_id as team_id3_0_0_,
            lockers0_.id as id1_0_0_,
            lockers0_.id as id1_0_1_,
            lockers0_.name as name2_0_1_,
            lockers0_.team_id as team_id3_0_1_ 
        from
            locker lockers0_ 
        where
            lockers0_.team_id=?
    ```
- `Team`을 전체 조회할 때 지연로딩으로 인해 각 `Team` 마다 `Member` 조회 쿼리, `Locker` 조회 쿼리가 팀의 개수 N 만큼 추가로 나가므로 `N+1 문제`가 발생한 것이다.
- 팀의 개수가 많으면 많을수록 추가적인 쿼리가 많이 나가 성능이 더욱 떨어진다. 
- 해결하기 위해서는 연관관계 엔티티를 즉시로딩하거나 비슷한 방법을 사용하여 적은 쿼리로 select 할 수 있도록 해야한다.  

<br>

## 하나의 collection `fetch join`을 할 경우 

- 우선 하나의 Collection이라도 `fetch join` 한다면 어떻게 동작이 될까? 
- Team을 조회할 때 다음과 같은 쿼리를 사용해서 Lockers 컬렉션만 join fetch 하도록 한다. 
    ```java
    @Query("select distinct t from Team t join fetch t.lockers")
    List<Team> findAllJoinFetchLockers();
    ```
- 실행 서비스 로직은 바로 위와 같고, teams를 조회할 때문 위의 쿼리를 사용했다. 
- 실제 발생 쿼리
    ```sql
    Hibernate: #전체 팀 조회 시 lockers를 join fetch 한다. 
    select
        distinct team0_.id as id1_2_0_,
        lockers1_.id as id1_0_1_,
        team0_.name as name2_2_0_,
        lockers1_.name as name2_0_1_,
        lockers1_.team_id as team_id3_0_1_,
        lockers1_.team_id as team_id3_0_0__,
        lockers1_.id as id1_0_0__ 
    from
        team team0_ 
    inner join
        locker lockers1_ 
            on team0_.id=lockers1_.team_id

    Hibernate: #첫번째 Team의 Member 조회
        select
            members0_.team_id as team_id3_1_0_,
            members0_.id as id1_1_0_,
            members0_.id as id1_1_1_,
            members0_.name as name2_1_1_,
            members0_.team_id as team_id3_1_1_ 
        from
            member members0_ 
        where
            members0_.team_id=?

        select #두번째 Team의 Member 조회
            members0_.team_id as team_id3_1_0_,
            members0_.id as id1_1_0_,
            members0_.id as id1_1_1_,
            members0_.name as name2_1_1_,
            members0_.team_id as team_id3_1_1_ 
        from
            member members0_ 
        where
            members0_.team_id=?
    ```
    - Lockers 컬렉션을 join fetch 해서 즉시로딩 했으니, lockers 컬렉션 관련 서비스 로직 실행 시 추가 쿼리는 발생하지 않았다. 
    - 여전히 Members에 대해서는 team N개 만큼의 추가 쿼리 N개가 발생했다. 
    - 어느정도 쿼리가 줄긴 했으나, 여젼히 `N+1 문제`가 발생하고 있다. 

<br>
<br>

# 다수의 collection `fetch join`을 할 경우 - `MutipleBagFetchException` 발생

- 그렇다면 두 개의 컬렉션을 모두 fetch join 한다면 쿼리를 최적화 할 수 있지 않을까 생각할 수 있다. 
- 하지만 두개의 OneToMany 연관관계에 있는 collection fetch join을 할 경우 `MutipleBagFetchException` 예외가 발생한다. 
- 두개의 collection을 fetch join 하는 JPQL
    ```java
    @Query("select distinct t from Team t join fetch t.members join fetch t.lockers")
    List<Team> findAllJoinFetchAll();
    ```
- 발생 예외 - <br>
    `Caused by: org.hibernate.loader.MultipleBagFetchException: cannot simultaneously fetch multiple bags:` 

- collection을 `fetch join` 하면서 데이터 뻥튀기가 일어나기 때문인데 두 개 이상의 컬렉션을 할 경우 그 복잡도가 높아지기 때문에 JPA에서는 예외로 처리한다. 
- 위 쿼리를 실행하기도 전에 `JPA Entity Manager`에서 오류를 발견하여 어플리케이션 실행 시작 당시 `Bean Creation` 단계에서 예외를 던져 어플리케이션이 실행되지 못하게 한다. 

<br>
<br>

# 또 다른 문제, JPA Pagination
---
<br>

- 위와 같은 서비스 로직을 실행할 때 뿐 아니라 JPA Pagination을 적용하려고 할 때 collection fetch join의 문제가 발생한다. 
- collection fetch 를 할 경우 데이터 뻥튀기가 발생하기 때문에 Paging에 애매하게 되기 때문에 `select` 할 때 `WARN`이 발생한다. 
<br>

```log
2021-08-22 22:04:27.898  WARN 4825 --- [    Test worker] o.h.h.internal.ast.QueryTranslatorImpl   : HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
Hibernate: 
    select
        distinct team0_.id as id1_2_0_,
        members1_.id as id1_1_1_,
        team0_.name as name2_2_0_,
        members1_.name as name2_1_1_,
        members1_.team_id as team_id3_1_1_,
        members1_.team_id as team_id3_1_0__,
        members1_.id as id1_1_0__ 
    from
        team team0_ 
    inner join
        member members1_ 
            on team0_.id=members1_.team_id
```
- 그렇기 때문에 Pageable을 적용하면서 N+1 문제에 대해 `fetch join`만을 사용해서 해결하기는 어렵다.

<br>
<br>

# 해결방법: BatchSize 적용하기 

- 위 문제를 해결할 수 있는 방법 중 하나는 BatchSize를 적용하는 것이다. 
- `spring.jpa.properties.hibernate.default_batch_fetch_size={batchSize}` 를 `application.properties`에 지정하면 설정된 Batch Size만큼 `IN` 쿼리로 날라간다. 
- 위와 동일한 테스트 코드 실행 (Pagination을 추가한 것 외에는 동일히다.)
    ```java
    @DisplayName("Batch size를 적용해 Pagination을 하면 N+1문제나 경고가 뜨지 않는다.")
    @Test
    void findWithPagination() {
        Pageable pageable = PageRequest.of(0, 2);
        List<Team> teams = teamRepository
            .findAll(pageable)
            .toList();

        List<String> memberNames = teams.stream() //최대 batch size만큼 IN 쿼리로 한번에 처리된다
            .map(Team::getMembers)
            .flatMap(Collection::stream)
            .map(Member::getName)
            .collect(toList());

        List<String> lockerNums = teams.stream() //최대 batch size만큼 IN 쿼리로 한번에 처리된다
            .map(Team::getLockers)
            .flatMap(Collection::stream)
            .map(Locker::getName)
            .collect(toList());

        assertThat(teams.size()).isEqualTo(2);
        assertThat(memberNames.size()).isEqualTo(6);
        assertThat(lockerNums.size()).isEqualTo(4);
    }
    ```

- 실제 실행된 쿼리
    ```sql
    Hibernate: # 전체 팀 조회 쿼리
    select
        team0_.id as id1_2_,
        team0_.name as name2_2_ 
    from
        team team0_ limit ?
    Hibernate: 
        select
            count(team0_.id) as col_0_0_ 
        from
            team team0_
    Hibernate: # Team 2개의 Id가 IN 절로 한번에 조회된다. 
        select
            members0_.team_id as team_id3_1_1_,
            members0_.id as id1_1_1_,
            members0_.id as id1_1_0_,
            members0_.name as name2_1_0_,
            members0_.team_id as team_id3_1_0_ 
        from
            member members0_ 
        where
            members0_.team_id in (
                ?, ?
            )
    2021-08-23 02:41:39.149 TRACE 5511 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
    2021-08-23 02:41:39.149 TRACE 5511 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [BIGINT] - [2]
    Hibernate: # Team 2개의 Id가 IN 절로 한번에 조회된다. 
        select
            lockers0_.team_id as team_id3_0_1_,
            lockers0_.id as id1_0_1_,
            lockers0_.id as id1_0_0_,
            lockers0_.name as name2_0_0_,
            lockers0_.team_id as team_id3_0_0_ 
        from
            locker lockers0_ 
        where
            lockers0_.team_id in (
                ?, ?
            )
    ```

- 이전에는 전체 팀 조회 쿼리 1개 + Team 2개에 대한 Members 조회 쿼리 2개 + Team 2개에 대한 Lockers 조회 쿼리 2개해서 총 5개의 쿼리가 나갔다. 
- Batch size를 적용한 이후에는 전체 팀 조회 쿼리 1개 + Team의 Members 조회 쿼리 1개 + Team의 Lockers 조회 쿼리 1개 총 2개의 쿼리가 나갔다. 
- Team의 사이즈가 클 수록 더 쿼리 차이가 많이 난다. 
- Batch size는 1000을 넘어가지 않도록 설정하는 것이 좋다.

<br>
<br>
<br>

**[참고자료]**
- https://jojoldu.tistory.com/457
- https://www.inflearn.com/questions/14663
- https://jojoldu.tistory.com/165
- https://woowacourse.github.io/tecoble/post/2021-07-26-jpa-pageable/ 

```toc
```