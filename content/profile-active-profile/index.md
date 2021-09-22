---
emoji: 🖥
title: 웹 개발 시 Profile 전략 - @Profile & @ActiveProfile
date: '2021-08-19 23:00:00'
author: 코다
tags: 스프링부트
categories: 스프링부트
---

## Intro

- 프로젝트를 진행하다 보면 상황에 따라 각기 다른 운영환경을 설정해야할때가 있다. 그때마다 properties 설정 파일에 가서 설정되어있는 운영 환경을 바꾸고 돌리기는 어렵다. 
- 이때 각기 다른 `Profile`를 적용해서 상황에 따라 적합한 `Profile` 설정을 따르도록 할 수 있다. 

<br>

## yml 파일로 설정 나누기 - 간단하게 

- `yml` 또는 `properties`를 통해서 profile 설정을 나눌 수 있다. 
- 각각 원하는 환경에 대한 설정정보를 `yml`, `properties`에 기재한 후 `application-{profile-name}.yml` 또는 `application-{profile-name}.properties`로 지정한다. 
    <p align="center"><img width="317" alt="스크린샷 2021-08-17 오후 8 42 39" src="https://user-images.githubusercontent.com/63405904/129719783-aeb9d93e-4c22-473e-9221-0b553393287d.png"><br>resources 하위에 나누어진 profiles 설정들</p>

- 여러 profile 환경으로 나눠져 있을 경우 어떤 profile을 기본적으로 실행할 것인지 `application.properties`에 지정해 주어야 한다. 
    ```properties
        spring.
            profiles.
                active = local
    ```
- 나누어진 profile을 적용하기 위해서는 `$ java -jar -Dspring.profiles.active={profile-name} [jar파일명]`로 적용하고자하는 프로필을 지정하여 실행하거나 `@ActiveProfile` 어노테이션을 활용할 수 있다. 

<br>

### 사용 예시

- 본인은 프로젝트 진행 시 다음과 같이 local, prod, test로 환경을 나누었다. 

1. application-local.yml

    ```yml
    spring:
    datasource:
        url: jdbc:mariadb://localhost:3306/pickgit?serverTimezone=UTC&characterEncoding=UTF-8
        username: ***
        password: ***
    jpa:
        show-sql: true
        properties:
        hibernate:
            dialects: org.hibernate.dialect.MySQL57Dialect
            format_sql: true
            default_batch_fetch_size: 1000
        generate-ddl: true
        hibernate:
        ddl-auto: none
    ```

2. application-prod.yml

    ```yml
    spring:
        datasource:
            url: jdbc:mariadb://localhost:13306/pickgit?serverTimezone=UTC&characterEncoding=UTF-8
            username: ***
            password: ***
            driver-class-name: org.mariadb.jdbc.Driver
    jpa:
        show-sql: true
        properties:
        hibernate:
            dialects: org.hibernate.dialect.MySQL57InnoDBDialect
            format_sql: true
            default_batch_fetch_size: 1000
        generate-ddl: true
        hibernate:
        ddl-auto: validate
    ```

3. application-test.yml

    ```yml
    spring:
    datasource:
        url: jdbc:h2:mem:~/test;MODE=MySQL;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
        username: **
        password: **
    jpa:
        show-sql: true
        properties:
        hibernate:
            dialects: org.hibernate.dialect.MySQL57Dialect
            format_sql: true
            default_batch_fetch_size: 1000
        generate-ddl: true
        hibernate:
        ddl-auto: create-drop
    ```

4. application.yml
   -  `spring.profiles.active:` 로 기본 profile 환경을 세팅할 수 있다. <br>
   -  `spring.profiles.include:` 로 포함할 다른 profile 설정을 지정할 수 있다. (주로 공통으로 적용될만한 보안 관련 profile, aws 관련 설정 등을 include로 포함한다.)

<br>

## @Profile 어노테이션으로 환경별 설정 다르게 하기 

- `@Profile`을 사용하면 해당 어노테이션에서 지정한 환경으로 어플리케이션 실행 시 configuration을 설정할 수 있다. 
- `@Profile` 어노테이션은 다음과 같이 3가지 방법으로 (더 있을 수도 있다) 사용할 수 있다. 

    1. @Configuration 클래스 안에 method 위에 
        
        ```java
        @Configuration
        public class InfrastructureConfiguration {
        
            @Bean
            @Profile("test")
            OAuthClient mockGithubOAuthClient() {
                return new MockGithubOAuthClient();
            }

            @Bean
            @Profile("prod")
            OAuthClient githubOAuthClient() {
                return new GithubOAuthClient();
            }
        }
        ```

        - OAuthClient 주입할 때 "test" profile 환경이면 `MockGithubOAuthClient`가 주입되고 "prod" profile 환경이면 `GithubOAuthClient`가 주입된다. 
  
    2. @Configuration 클래스 안에 static class 위에
        
        ```java
        @Configuration
        public class InfrastructureConfiguration {
        
            @Configuration
            @Profile("test")
            static class MockOAuthClient() {
                
                @Bean
                OAuthClient mockGithubOAuthClient() {
                    return new MockGithubOAuthClient();
                }
            }

            @Configuration
            @Profile("prod")
            static class MockOAuthClient() {
                
                @Bean
                OAuthClient githubOAuthClient() {
                    return new GithubOAuthClient();
                }
            }
        }
        ```
        
        - 중첩 클래스로도 설정할 수 있다. 보기에 가독성이 좋은 것으로 선택하면 된다. 
    
    3. 인터페이스를 구현하는 구현체 class 위에 설정 
        - Bean으로 등록되어 있는 클래스에 적용할 수 있다. (`@Configuration`, `@Component`)
        - OAuthClient 라는 인터페이스가 있을 때, 해당 인터페이스를 구현하는 구현체를 Profile에 따라 나누어서 적용할 수 있다. 
        
        ```java
        @Component
        @Profile("!test") //profile이 테스트가 아닐 경우 적용된다. 
        public class GithubOAuthClient implements OAuthClient {

            //... 프로퍼티 생략

            @Override
            public String getLoginUrl() {
                return String.format("https://github.com/login/oauth/authorize"
                        + "?client_id=%s"
                        + "&redirect_uri=%s"
                        + "&scope=%s",
                    clientId, redirectUrl, scope);
            }

            @Override
            public String getAccessToken(String code) {
                //... 세부 로직 생략 
            }

            //... 생략 
        }
        ```
        
        - 위 로직은 Github OAuth관련 로직이므로 사용자 로그인 행위가 없는 테스트에서 실행하기 어렵기 때문에 아래와 같이 Profile이 "test"로 설정 되었을 경우 MockOAuthClient가 주입되도록 설정한다. 
        
        ```java
        @Component
        @Profile("test") //profile이 테스트가 아닐 경우 적용된다. 
        public class MockGithubOAuthClient implements OAuthClient {

            //... 프로퍼티 생략

            @Override
            public String getLoginUrl() {
                return String.format("https://github.com/login/oauth/authorize"
                        + "?client_id=%s"
                        + "&redirect_uri=%s"
                        + "&scope=%s",
                    clientId, redirectUrl, scope);
            }

            @Override
            public String getAccessToken(String code) {
                return "token";
            }

            //... 생략 
        }
        ```

<br>

## @ActiveProfile 어노테이션으로 profile 설정 적용하기 

- 이전에 말한대로 `yml` 또는 `properties` 파일에 profile 설정을 나누고 `application.properties` 등의 파일에 profile 을 지정하거나 jar를 실행하면서 환경을 지정할 수 있다. 
- 하지만 어떤 profile 환경에서도 항상 특정 profile 환경으로 실행되어야 할 때가 있다. 예를 들어서 test 코드 같은 경우는 항상 test profile 환경으로 돌아가야한다. 
- 이때 해당 클래스 상단에 @ActiveProfile("test") 등으로 설정하면 해당 profile이 적용되어 실행된다. 
- 
    ```java
    @DataJpaTest
    @ActiveProfiles("test")
    class OAuthServiceIntegrationTest {
        //...테스트 코드 생략 
    }
    ```

<br>
<br>

**[출처]**
- http://wonwoo.ml/index.php/post/1933
- https://www.baeldung.com/spring-profiles
- https://umanking.github.io/2019/04/13/spring-profile/

```toc
```