---
emoji: 🖥
title: Springboot 언제 어떤 테스트를 사용할까
date: '2021-08-17 23:00:00'
author: 코다
tags: 스프링부트 테스트
categories: 스트링부트 테스트
---

## Intro
---
- 스프링부트 프로젝트를 진행하다보면 웹 mvc에 대한 테스트를 진행해야할 때가 있다. 
- 때로는 각 layer에 대한 슬라이스 테스트를 작성하거나, 일부분에 대한 통합 테스트만을 진행할 때 Mock 테스트를 해야할 때도 있다. 
- 테스트 관련 annotation에 대해서 정리하고 각 annotation의 차이 및 언제 무엇을 사용하면 좋을지 정리해본다. 

<br>
<br>

## `@SpringBootTest`
---
<br>

- `@SpringBootTest` 어노테이션은 `@SpringBootApplication` 을 찾아 해당 configuration에 맞추어 실제 Spring web context를 실행햔다. 
- Spring context의 설정으로 그대로 적용해서 테스트를 진행해야 할 경우에 해당 어노테이션을 붙여서 테스트를 하는 것이 좋다. 
- 하지만 전체 컨텍스트를 로드하는 만큼 굉장히 오랜 시간이 걸린다. 
- 실제로 `@SpringBootTest` 어노테이션이 붙은 테스트를 돌려본다면 다음과 같은 스프링 컨텍스트르 로딩하는 긴 로그가 찍히는 것을 확인할 수 있다. 

```
//... 일부 생략
20:37:26.255 [main] DEBUG org.springframework.test.context.support.TestPropertySourceUtils - Adding inlined properties to environment: {spring.jmx.enabled=false, org.springframework.boot.test.context.SpringBootTestContextBootstrapper=true, server.port=0}

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.2)

//... 일부 생략 
 > Running with Spring Boot v2.5.2, Spring v5.3.8
2021-08-16 20:37:26.559 DEBUG 4487 --- [           main] c.w.p.a.user.UserAcceptanceTest          : Running with Spring Boot v2.5.2, Spring v5.3.8
2021-08-16 20:37:26.560 INFO  main o.s.boot.SpringApplication.logStartupProfileInfo L:663 
 > The following profiles are active: security,test
2021-08-16 20:37:26.560  INFO 4487 --- [           main] c.w.p.a.user.UserAcceptanceTest          : The following profiles are active: security,test
2021-08-16 20:37:27.252  INFO 4487 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2021-08-16 20:37:27.328  INFO 4487 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 70 ms. Found 4 JPA repository interfaces.
2021-08-16 20:37:28.062  INFO 4487 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 0 (http)
2021-08-16 20:37:28.068  INFO 4487 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-08-16 20:37:28.068  INFO 4487 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.48]
2021-08-16 20:37:28.115  INFO 4487 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-08-16 20:37:28.116  INFO 4487 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1379 ms
```

<br>
<br>
<br>

## `@AutoConfigureMockMvc`
---
<br>

- 위 어노테이션은 다음과 같이 `MockMvc`를 주입받아서 톰캣 서버를 띄우지 않은 상태로 API 요청 부분을 Mocking 해서 사용할 수 있다. 하지만 스프링 컨텍스트의 빈을 모두 로드하는 것은 `@SpringBootTest` 와 동일하다. 
- `@SpringBootTest` 와 `@AutoConfigureMockMvc`를 사용해서 테스트 코드를 실행해보면 위 `@SpringBootTest`를 사용했을때와 다르게 Tomcat을 시작하는 로그가 찍히지 않는 것을 확인할 수 있다. 

> 테스트 코드 

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @DisplayName("사용자는 내 프로필을 조회할 수 있다.")
    @Test
    void getAuthenticatedUserProfile_LoginUser_Success() throws Exception {
        // given
        UserProfileResponseDto responseDto = UserFactory.mockLoginUserProfileResponseDto();

        // when
        ResultActions perform = mockMvc.perform(get("/api/profiles/me")
            .header(HttpHeaders.AUTHORIZATION, "Bearer testToken")
            .contentType(MediaType.APPLICATION_JSON_VALUE)
            .accept(MediaType.ALL));

        //...뒤 코드 생략

    }
}
```

> 실행 시 로그 

```
//...일부 생략 
21:41:00.201 [main] DEBUG org.springframework.test.context.support.TestPropertySourceUtils - Adding inlined properties to environment: {spring.jmx.enabled=false, org.springframework.boot.test.context.SpringBootTestContextBootstrapper=true}

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.2)

//...일부 생략 
2021-08-16 21:41:03.332  INFO 4963 --- [           main] o.s.b.t.m.w.SpringBootMockServletContext : Initializing Spring TestDispatcherServlet ''
2021-08-16 21:41:03.333  INFO 4963 --- [           main] o.s.t.web.servlet.TestDispatcherServlet  : Initializing Servlet ''
2021-08-16 21:41:03.334  INFO 4963 --- [           main] o.s.t.web.servlet.TestDispatcherServlet  : Completed initialization in 1 ms
2021-08-16 21:41:03.349  INFO 4963 --- [           main] c.w.p.u.u.p.UserControllerTest           : Started UserControllerTest in 3.144 seconds (JVM running for 3.765)
2021-08-16 21:41:03.524  INFO 4963 --- [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2021-08-16 21:41:03.525  INFO 4963 --- [ionShutdownHook] .SchemaDropperImpl$DelayedDropActionImpl : HHH000477: Starting delayed evictData of schema as part of SessionFactory shut-down'
2021-08-16 21:41:03.532  INFO 4963 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2021-08-16 21:41:03.537  INFO 4963 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```


## `@WebMvcTest`
---
- `@WebMvcTest`의 경우 `@AutoConfigureMockMvc`와 비슷하게 서버가 모킹이 되지만 MVC와 관련된 빈들만 로드한다. 그렇기 때문에 MVC 관련 빈들만 사용한다면 위 어노테이션을 사용하는 것이 더 부화가 적다. 
- 해당 어노테이션 안에 특정 클래스를 지정하면 해당 클래스와 관련한 mvc 빈들이 올라가므로 더 효율적으로 사용할 수 있다. 
```java
@AutoConfigureRestDocs
@WebMvcTest(UserController.class)
@ActiveProfiles("test")
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @DisplayName("사용자는 내 프로필을 조회할 수 있다.")
    @Test
    void getAuthenticatedUserProfile_LoginUser_Success() throws Exception {
        // given
        UserProfileResponseDto responseDto = UserFactory.mockLoginUserProfileResponseDto();

        // when
        ResultActions perform = mockMvc.perform(get("/api/profiles/me")
            .header(HttpHeaders.AUTHORIZATION, "Bearer testToken")
            .contentType(MediaType.APPLICATION_JSON_VALUE)
            .accept(MediaType.ALL));

        //...뒤 코드 생략

    }
}
```
- 위와 동일하게 Tomcat 관련 로그는 찍히지 않는다. 대신 `SpringBootMockServletContext`이 모킹된 서블릿을 초기화 하는 로그를 확인할 수 있다. 
```
2021-08-16 23:46:28.495  INFO 5504 --- [           main] o.s.b.t.m.w.SpringBootMockServletContext : Initializing Spring TestDispatcherServlet ''
2021-08-16 23:46:28.495  INFO 5504 --- [           main] o.s.t.web.servlet.TestDispatcherServlet  : Initializing Servlet ''
2021-08-16 23:46:28.496  INFO 5504 --- [           main] o.s.t.web.servlet.TestDispatcherServlet  : Completed initialization in 1 ms
```
<br>

- 컨트롤러 슬라이스 테스트 8개에 대한 각 MockMvc 테스트를 비교했을 때, 미세하지만 `@WebMvcTest`가 더 빠른 것을 확인할 수 있다. 더 많은 숫자의 테스트 일 때 그 차이는 더 커진다. 
<br>

<p align="center"><img width="500" alt="스크린샷 2021-08-16 오후 11 48 46" src="https://user-images.githubusercontent.com/63405904/129583308-1e306f3c-0b51-4c97-8606-ee810a739289.png">@WebMvcTest 사용</p>


<p align="center"><img width="598" alt="스크린샷 2021-08-16 오후 11 50 11" src="https://user-images.githubusercontent.com/63405904/129583673-8bd736da-f88d-407f-a894-40f1e29a94fa.png">@AutoConfigureMockMvc 사용</p>

<br>
<br>
<br>

## `@ExtendWith(MockitoExtension.class)`
---
<br> 

- `@ExtendWith`는 JUnit에서 제공하는 기능이다. (위 테스트 어노테이션은 SpringBoot에서 제공한다.)
- 흔히 코드에서 `@ExtendWith(MockitoExtension.class)`으로 많이 사용하고 위의 모킹과 어떠한 부분이 다른지에 대해서 많이 헷갈려한다. 
- 위 모킹은 서블릿 컨테이너에 대한 모킹을 스프링 부트 에서 제공하는 것이다. 
- `@ExtendWith(MockitoExtension.class)` 은 Mock이라는 가짜 객체를 지원하는 Mockito 테스트 프레임워크를 JUnit5와 연동하여 사용하도록 하는 것이다. (두 개의 테스트 프레임워크의 결합)
- 주로 하나의 Layer의 단위테스트를 할 때 나머지 객체들을 모킹하도록 지원한다. 
- 즉, 서블릿 컨테이너와 상관없이 어느 한 객체에 대한 단위 테스트를 진행하면서 의존하고 있는 다른 객체의 행동을 stub하여 제어할 때 사용된다. 

<br>
<br>
<br>

**[출처]**
- https://spring.io/guides/gs/testing-web/
- https://elevatingcodingclub.tistory.com/61
- https://meetup.toast.com/posts/124
- https://www.baeldung.com/spring-boot-testing
- https://junit.org/junit5/docs/current/user-guide/#extensions
- https://pinokio0702.tistory.com/143


```toc
```