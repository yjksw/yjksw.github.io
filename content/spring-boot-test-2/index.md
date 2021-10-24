---
emoji: 🖥
title: Springboot 테스트 다시 한번 알아보기_중요한 건 여러 번 😊 
date: '2021-10-23 22:00:00'
author: 코다
tags: 스프링부트 테스트
categories: 스프링부트 테스트
---

> 다음 [링크]([https://www.baeldung.com/spring-boot-testing](https://www.baeldung.com/spring-boot-testing))를 읽고 정리한 내용입니다 🙌 <br> 이전에 작성했던 [글](https://yjksw.github.io/spring-boot-test/)이 있습니다. 

스프링부트에서 지원하는 여러 테스팅 기법들을 통해서 단위 테스트나 스프링 컨텍스트를 띄우는 통합 테스트를 진행할 수 있다. 
사전 준비로는 스프링부트 프로젝트에 `org.springframwork.boot.spring-boot-start-test` 의존성을 추가해야한다.

<br>

## 🌩 @SpringBootTest 통합테스트

- 통합테스트는 어플리케이션의 여러 레이어의 통합 로직을 테스트 하는 것이다. 따라서 mocking을 하지 않는다.
- 원칙적으로는 통합테스트는 단위테스트와 분리되어 있어야하며 실행 또한 분리해서 실행해야 한다. 다른 profile 환경으로 나누고 통합테스트만을 분리하여 실행해야한다.
    - 이렇게 해야하는 이유 중 하나는 통합 테스트는 어플리케이션 컨텍스트를 띄우는 작업을 필요로 하기 때문에 상대적으로 긴 시간이 소요된다.
    - 또한 실제 데이터베이스의 실행을 필요로 하기도 한다.
- `@SpringBootTest` 은 컨테이너 전체를 띄우는데 유용하다. 이 어노테이션은 테스트에 사용될 ApplicationContext를 생성하여 테스트를 진행한다.
    - `@SpringBootTest`의 `SpringBootTest.webEnvironment.MOCK` 을 통해 mock 서블릿 환경에서 컨테이너를 실행할 수 있다.
- `@TestPropertySource` 어노테이션을 통해 properties 파일을 지정할 수 있다.

<br>

## 🌩 @TestConfiguration 을 활용한 테스트 설정

- `@SpringBootTest`는 어플리케이션 컨텍스트 전체를 띄우는 것이기 때문에 `@Autowired` 를 통해 자동주입하는 bean 은 모두 컴포넌트 스캔을 통한다는 것이다.
- 하지만 테스트를진행하면서실제 어플리케이션 컨텍스트와 다른 테스트용 설정 빈들을 주입하고 싶을 수 있다. 이때 `@TestConfiguration` 어노테이션을 활용한다.
- 사용하는 방법은 2가지 이다.
    1. static inner class
        
        ```java
        @RunWith(SpringRunner.class)
        public class EmployeeServiceImplIntegrationTest {
        
            @TestConfiguration
            static class EmployeeServiceImplTestContextConfiguration {
                @Bean
                public EmployeeService employeeService() {
                    return new EmployeeService() {
                        // implement methods
                    };
                }
            }
        
            @Autowired
            private EmployeeService employeeService;
        }
        ```
        
    2. separated test configuration class
        
        ```java
        @TestConfiguration
        public class EmployeeServiceImplTestContextConfiguration {
            
            @Bean
            public EmployeeService employeeService() {
                return new EmployeeService() { 
                    // implement methods 
                };
            }
        }
        ```
        
        - `@TestConfiguration`이 명시된 클래스는 component scanning에서 제외되어 있다. 따라서 해당 설정을 사용하고 싶은 테스트에 `@Import` 어노테이션을 통해 명시해주어야 한다.
        
        ```java
        @RunWith(SpringRunner.class)
        @Import(EmployeeServiceImplTestContextConfiguration.class)
        public class EmployeeServiceImplIntegrationTest {
        
            @Autowired
            private EmployeeService employeeService;
        
            // remaining class code
        }
        ```
        
<br>

## 🌩 @MockBean 을 활용한 모킹

- 특정 layer의 로직만 테스트하고 싶은 경우 해당 layer나 클래스가 의존하고 있는 다른 빈의 동작에 대해서는 크게 상관하고 싶지 않을때가 있다.
- 따라서 `@MockBean`을 활용하여 실제 의존 빈을 사용하는 것이 아니라 해당 빈이 지정된 값을 반환하도록 할 수 있다.
    
    ```java
    @RunWith(SpringRunner.class)
    public class EmployeeServiceImplIntegrationTest {
    
        @TestConfiguration
        static class EmployeeServiceImplTestContextConfiguration {
     
            @Bean
            public EmployeeService employeeService() {
                return new EmployeeServiceImpl();
            }
        }
    
        @Autowired
        private EmployeeService employeeService;
    
        @MockBean
        private EmployeeRepository employeeRepository;
    
        // write test cases here
    }
    ```
    
- 여기서 다음과 같이 `EmployeeRepository` 의 행동을 지정하고  테스트를 돌리면 EmployeeService 에서 repository 부분은 지정한 것과 같이 행동한다.
    
    ```java
    @Before
    public void setUp() {
        Employee alex = new Employee("alex");
    
        Mockito.when(employeeRepository.findByName(alex.getName()))
          .thenReturn(alex);
    }
    
    @Test
    public void whenValidName_thenEmployeeShouldBeFound() {
        String name = "alex";
        Employee found = employeeService.getEmployeeByName(name);
     
         assertThat(found.getName())
          .isEqualTo(name);
     }
    ```
    
<br>

## 🌩 @DataJpaTest 를 활용한 통합테스트

- Persistence layer를 테스트하고 JPA를 사용하고 있다면 `@DataJpaTest` 어노테이션이 해당 레이어를 테스트하는 여러 세팅을 해준다.
    - H2 설정
    - Hibernate, Spring Data, Datasource 설정
    - `@EntityScan` 실행
    - SQL 로깅 실행
- 테스트 이전에 데이터베이스에 테스트 데이터를 미리 넣을 수 있도록 `TestEntityManager`를 활용한다.

<br>

## 🌩 @WebMvcTest 를 활용한 단위 테스트

- 주로 Controller는 Serivce에 의존한다.
- Controller에 대한 단위테스트를 진행하기 위해서는 service layer 코드를 모킹해야 한다.
- 이때 `@WebMvcTest` 어노테이션을 활용할 수 있다. 이 어노테이션은 Spring MVC 인프라를 자동으로 설정해준다.
- 주로 `@WebMvcTest` 하나의 컨트롤러에 국한되며 `@MockBean` 어노테이션을 활용해 의존 객체를 모킹할 수 있다.
- `@WebMvcTest` 는 또한 `MockMvc` 에 대한 자동설정을 해 전체 HTTP 서버를 구동하지 않고 MVC 컨트롤러를 테스트할 수 있도록 한다.
    
    ```java
    @Test
    public void givenEmployees_whenGetEmployees_thenReturnJsonArray()
      throws Exception {
        
        Employee alex = new Employee("alex");
    
        List<Employee> allEmployees = Arrays.asList(alex);
    
        given(service.getAllEmployees()).willReturn(allEmployees);
    
        mvc.perform(get("/api/employees")
          .contentType(MediaType.APPLICATION_JSON))
          .andExpect(status().isOk())
          .andExpect(jsonPath("$", hasSize(1)))
          .andExpect(jsonPath("$[0].name", is(alex.getName())));
    }
    ```
    
<br>

## 🌩 각종 자동화 테스트

스프링부트에서는 전체 어플리케이션의 일부분을 로딩하고 특정 레이어만 테스트할 수 있는 자동화 어노테이션을 많이 제공한다. 설명한 몇가지를 소개해보려보 한다. 

- @JdbcTest : JPA 어플리케이션을 테스트하는데 사용될 수 있다. 하지만 이 어노테이션이 사용되는 테스트는 DataSource를 필요로하는 테스트 이다.
- @DataRedisTest : Redis 어플리케이션을 테스트할 수 있는 어노테이션이다. @RedisHash 클래스를 스캔하고 Spring Data Redis 레포지토리를 default 설정한다.

<br>
<br>

## 🛋 느낀점
- 스프링부트 테스트는 [이전 글](https://yjksw.github.io/spring-boot-test/)에서도 한번 다루었던 이야기 이다. 다시 한번 쓰게 된 이유는 [레거시 리팩토링 미션](https://github.com/yjksw/jwp-refactoring)을 진행하면서 코드를 보호하는 테스트의 정도에 대한 고민이 되었기 때문이다. 
- 테스트를 작성할수록 어느정도의 테스트까지 작성하는 것이 좋은지는 정답이 없는 것 같다. 다만 테스트 코드 내에서 모순이 있어 항상 성공하는 테스트를 만들지 않기 위해서 주의해야한다. 또한 실패하는 케이스에 대한 작성도 꼼꼼히 해야 한다. 
- 테스트코드는 다다익선일까 ?


```toc
```