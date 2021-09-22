---
emoji: ✊
title: "동작 파라미터부터 람다까지: 콜백함수를 곁들인"
date: '2021-09-22 23:00:00'
author: 코다
tags: 자바
categories: 자바
---

## [INTRO]
- 자바가 업데이트 되면서 메소드를 일급시민으로 취급할 수 있게 되었다.
- 이로 인해 동작 파라미터를 통해서 어떠한 동작을 인자로 넘길 수 있다. 
- 메소드를 일급 시민으로 취급하면서 함수형 인터페이스 등의 개념이 등장한다.
- 코드의 명확성을 증진시키기 위해 익명 클래스, 람다 함수, 메서드 참조 등등의 개념이 활용된다. 
- 동작 파라미터를 활용한 예시로 콜백 함수를 들여다보자. 

<br>

## [왜 동작 파라미터인가?]
- 코딩을 할 때 가장 중요한 요소 중 하나는 변화하는 요구사항에 대응하는 것이다. 
- **동작 파라미터화**는 나중에 실행할 코드 블록을 인수로 넘겨서 행동을 결정하는 것이다. 
  - 나중에 실행되도록 넘기는 콜백 함수와 같은 동일하게 작용한다. (내재된 개념이다)

- 이해를 돕기 위해 *모던 자바 인 액션* 에 나온 예제를 살펴보자. 
  - 상황1: 사과를 색으로 필터링 하는 요구사항을 구현한다. 

    ```java
    public static List<Apple> filterGreenApples(List<Apple> inventory) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple: inventory) {
            if (GREEN.equals(appleColor())) {
                result.add(apple);
            }
        }
        return result;
    }
    ```

  - 상황2: 사과를 무게로 필터링하는 요구사항을 구현한다. 
    ```java
    public static List<Apple> filterByWeight(List<Apple> inventory, int weight) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple: inventory) {
            if (apple.getWeight() > weight) {
                result.add(apple);
            }
        }
        return result;
    }
    ```

  - 위 두 코드가 상당히 유사하다. (실제로 작성할 때도 복붙하고 if문 안의 로직만 바꿨다.) 
  - 이것은 DRY(Don't Repeat Yourself)원칙을 반하며 좋지 않은 코드이다. 
  - 이 상황에서 중복을 줄이기 위해 자바에서 일급시민으로 승격된 메소드를 동작 파라미터로 넘겨서 개선한다. 

    1. 함수형 인터페이스 선언 
        ```java
        @FunctionalInterface
        public interface ApplePredicate {
            boolean test (Apple apple);
        }
        ```

    2. 함수형 인터페이스 구현체 생성
        ```java
        // 무거운 사과 필터링 predicate
        public class AppleHeaveyWeightPredicate implements ApplePredicate {
            
            @Override
            public boolean test (Apple apple) {
                return apple.getWeight() > 150;
            }
        }

        // 녹색 사과 필터링 predicate
        public class AppleGreenColorPredicate implements ApplePredicate {
            
            @Override
            public boolean test (Apple apple) {
                return GREEN.equals(apple.getWeight());
            }
        }
        ```

    3. 필터 메소드에 동작 파라미터 활용 
        ```java
        public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
            List<Apple> result = new ArrayList<>();
            for (Apple apple: inventory) {
                if (p.test(apple)) { //동작 파라미터화!!! 조건문을 캡슐화하여 인자로 전달 
                    result.add(apple);
                }
            }
            return result;
        }          
        ```

  - 전략 패턴이라고도 한다. 

<br>

## [First Class Citizen 일급 시민]
위에서 계속 일급 시민이라는 개념이 등장했다. 동적 파라미터와 함수형 인터페이스 등은 일급시민을 기반으로 탄성 배경과 개념을 설명할 수 있다. 

- 다음 조건을 만족하는 것을 일급 시민으로 취급한다. 
  - 변수에 값을 할당 가능
  - 함수의 인자로 넘길 수 있음
  - 함수의 반환값이 될 수 있음

스칼라나 그루비 등에서는 메소드를 일급값으로 취급하여 사용중에 있지만 자바에서는 메소드가 일급값이 아니었다. 하지만 이번에 자바8을 설계하면서 다른 언어에서 메소드를 일급값으로 취급하는 것의 장점을 살려서 자바에서도 가능하도록 설게를 했다. 

즉, 자바에서도 메소드를 일급값 취급하여 인자로 넘기거나 변수에 할당하고, 반환할 수 있게 되었다. 
- 함수형 인터페이스
- 메서드 참조
- 람다: 익명함수

<br>

## [익명 클래스, 메소드 참조, 람다]
동작 파라미터화를 통해서 변화된 요구사항을 하나의 인자로 처리하는 편리함을 경험했다. 하지만 언제나 코드의 명확성이 우선시되어야 한다 라는 조건을 충족시키기 위해서 동작 파라미터를 가독성 있게 작성하는 방법을 알아보자. 
<br>

1. 함수형 인터페이스 구현체
    - 위 예시에서 작성한 `ApplePredicate`이라는 인터페이스가 함수형 인터페이스이다. 
    - 함수형 인터페이스는 추상 메소드가 단 하나만 있는 인터페이스이다. (default 메소드는 다수 존재해도 된다.)
    - 이미 자바 표준으로 정의된 여러 표준 함수형 인터페이스가 있다. [참고링크](https://velog.io/@im_joonchul/%ED%91%9C%EC%A4%80-%ED%95%A8%EC%88%98%ED%98%95-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4)
    - 자바 8부터 `@FuntionalInterface`라는 어노테이션을 지원하는데, 해당 어노테이션을 통해 컴파일 시점에 해당 인터페이스가 함수형 인터페이스 조건을 충족하는지 확인 할 수 있다. (해당 어노테이션 없더라도 동작에는 차이가 없다.)
    - 위 예시처럼 함수형 인터페이스를 `implements` 한 구현체를 동작 파라미터로 넘겨서 활용할 수 있다. 

2. 익명 클래스 
    - 매번 함수형 인터페이스를 구현하는 클래스 파일을 생성하여 구현하고 명시적으로 인스턴스화 한 후 사용하는 것이 불편할 수 있다. 실질적인 로직과 관련이 없는 코드 라인이 많아지는 것도 문제다. 특히 한군데서만 쓰이는 구현체일 경우 더더욱 그렇다. 
    - 이때 익명 클래스를 사용하여 클래스 선언과 인스턴스를 동시에 하고, 필요한 즉시 구현하여 사용할 수 있다. 

    ```java
    public class AppleFactory {

        public statid void doService(List<Apple> apples) {
            // 익명 클래스 구현하기 
            ApplePredicate p = new ApplePredicate { // (*)
                public boolean test(Apple apple) { //(*)
                    return Green.eqauls(apple.getColor());
                }
            };

            filterApples(apples, p);
        }

        public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
            List<Apple> result = new ArrayList<>();
            for (Apple apple: inventory) {
                if (p.test(apple)) { 
                    result.add(apple);
                }
            }
            return result;
        }   
    }
    ```
  
3. 메소드 참조 
    - 하지만 여전히 불필요한 코드라인이 완전히 없어지지 않았다. 위 (*) 표시가 매번 익명 클래스를 작성할 때마다 반복되어 코드를 장황하게 만들 수 있다. 
    - 위 문제까지 없앨 수 있도록 메소드를 따로 선언해놓고 메소드 참조를 통해 인자로 넘길 수 있다. 

    ```java
    public static boolean isGreenApple(Apple apple) {
        return GREEN.equals(apple.getColor());
    }

    public static boolean isHeavyApple(Apple apple) {
        return apple.getWeight() > 150;
    }

    //메서드 참조 사용하기 
    filterApples(inventory, Apple::isGreenApple);
    filterApples(inventory, Apple::isHeavyApple);
    ```

4. 람다: 익명함수
   - 하지만 단 한번만 쓰이거나 한다면 따로 메소드를 선언하는 것조차 불필요한 작업일 수 있다.
   - 람다 익명함수를 사용하여 코드를 더 간결하게 만들 수 있다. 
  
  ```java
  List<Apple> apples = 
    filterApplies(inventory, (Apple apple) -> RED.equals(apple.getColor()));
  ``` 

  - 람다가 항상 좋은 것은 아니다. 람다 블록 안에 많은 코드가 구현되어야한다면 메소드를 따로 분리하고, 메소드 참조를 쓰는 것이 더 명확하게 읽히는 코드가 될 수 있다. 
  - 람다는 여러가지 형태로 더 간결해질 수 있다. 
  
  ```java
  ApplePredicate<Apple> p;

  p = (apple) -> RED.equals(apple.getColor()); //매개변수 형 생략 가능 
  List<Apple> apples = filterApplies(inventory, p);

  p = apple -> RED.equals(apple.getColor()); //매개변수 소괄호 생략 가능 
  List<Apple> apples = filterApplies(inventory, p);
  ```

- 추가로 Generic까지 적용하면 더 추상적이고 범용적으로 활용 가능하게 된다. 개인적으로 이번에 자바의 JDBCTemplate 라이브러리를 직접 구현해 보았을 때 아주 유용하게 사용했다. 라이브러리와 같은 코드들은 인자와 반환값이 매우 유연해야하기 때문에 동작 파라미터화 + Generic을 사용하면 구현하기 용이하다. 

<br>

## [동작 파라미터 활용하기: 콜백함수]
콜백함수란 나중에 실행될 코드블록을 의미하며 특정 로직을 수행 후 돌아와서 해당 로직을 수행하도록 하는 함수를 말한다. 
<br>
JDBCTemplate을 구현하면서 동작 파라미터를 활용한 콜백함수를 사용했다. 일반적으로 DB와 통신하는 부분은 `Connection 을 생성 -> Statement 생성 -> 쿼리 실행` 이라는 큰 틀 안에서 사소한 동작의 차이가 있다. 그렇기 때문에 각각을 (CUD vs R) 메소드로 분리하면 큰 틀에 해당하는 코드가 중복된다. 
[코드 전체보기](https://github.com/yjksw/jwp-dashboard-jdbc/tree/yjksw)

<br>

- CallBack 함수형 인터페이스 
  
  ```java
    @FunctionalInterface
    public interface CallBack<T> {
        T execute(PreparedStatement pstm) throws SQLException;
    }
  ```

- 큰 틀을 관장하는 `execute()` 메소드

  ```java
    private <T> T execute(String sql, Object[] args, CallBack<T> sqlExecution) { // 쿼리마다 다른 동작이 CallBack으로 넘겨진다.
        try (
            Connection conn = datasource.getConnection();
            PreparedStatement pstm = conn.prepareStatement(sql);
        ) {
            log.info("query : {}", sql);
            new ArgumentsSetter().setArguments(pstm, args);
            return sqlExecution.execute(pstm); // 각기 다른 동작을 수행하고 반환한다
        } catch (SQLException e) {
            log.error("SQLException thrown: {}", e.getMessage());
            throw new DataAccessException(e.getMessage());
        }
    }
  ```

- 콜백 함수 활용 부분 

  ```java
    public void update(String sql, Object... args) { // 매소드 참조 형식 
        execute(sql, args, PreparedStatement::executeUpdate);
    }

    public <T> T query(String sql, RowMapper<T> rowMapper, Object... args) {
        CallBack<T> execution = (pstm) -> { // 람다 형식
            ResultSet rs = pstm.executeQuery();
            return rowMapper.apply(rs);
        };

        return execute(sql, args, execution);
    }
  ```

<br>
<br>

**[참고링크]**
- [https://zzang9ha.tistory.com/303](https://zzang9ha.tistory.com/303)
- [https://velog.io/@im_joonchul/%ED%91%9C%EC%A4%80-%ED%95%A8%EC%88%98%ED%98%95-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4](https://velog.io/@im_joonchul/%ED%91%9C%EC%A4%80-%ED%95%A8%EC%88%98%ED%98%95-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4)
- [모던 자바 인 액션](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=200069290)


```toc
```
