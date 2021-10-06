---
emoji: ✊
title: "[JAVA] 구글에서 제공하는 Java Coding Convention Guide"
date: '2020-11-28 23:00:00'
author: 코다
tags: 자바
categories: 자바
---

프리코스를 진행하면서 구글에서 제공하는 javaGuide를 읽고 해당 convention을 따라서 코딩 하도록 하기 위해서 해당 문서를 정독했다. 원래 알고 있던 부분들도 있고 아닌 부분들도 있는데, 이렇게 잘 문서화 되어 있다는 것을 처음 알았다. 다음은 해당 문서를 읽으면서 두고두고 참고할 내용들을 정리한 것들이다. <br>

다음 사이트 참고: [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)

## 1. Source file structure

Java 소스 파일은 다음과 같은 구조를 가지고 있다. 순서에 유의하여 구조화 되어 있다. 

1. 만약 존재한다면, license or copyright information 
2. Package 명시
3. Import statements
4. 단 하나의 top-level class 

→ 위의 4 section을 1줄 간격(exactly one blank)으로 나눈다. 

<br>

### 1-1. copyright information

소스파일 맨 위에 시작 주석으로 파일 클래스 이름, 버전 정보, 날짜, 저작권 주의를 보여주는 주석으로 시작한다. 

### 1-2. Import Statements

1. Wildcard imports는 지양한다. 
    - (*) 추가해서 전체를 한꺼번에 import 하는 것.
    - [관련 참고 사이트](https://medium.com/@tharakamd.12/is-it-bad-to-use-wildcard-imports-in-java-1b46a863b2be#:~:text=Wildcard%20imports%20tell%20java%20compiler,performance%20may%20lower%20a%20bit)
2. 한 줄이 너무 길어도 line wrapping 하지 않는다.
3. static imports를 하나의 block에 non-static imports를 하나의 block에 넣고 두 block 사이만 한 줄 간격이 있다. 
4. 각 block 내에서는 ASCII sort order에 따라서 정렬한다. 
5. class는 static import 가 아닌 normal import 한다. 

### 1-3 Class Declaration

1. Top-level 클래스는 각 소스파일 당 단 1개만 존재한다. 

<br>

## 2. Formatting

### 2-1. 괄호

1. optional인 경우에도 괄호를 쓴다. 
   
    - `if, else, for, do, while` 이 비어 있거나 한 줄만 있더라도 괄호를 추가한다.
2. 비어 있지 않은 블록의 경우 다음과 같이 한다. 
    - '{' 앞에 줄 간격 두지 않는다.
    - '{' 이후에  줄 간격 둔다.
    - '}' 이전에 줄 간격 둔다.
    - '}' 이후에 다음과 같은 경우에만 줄 간격을 둔다.
        - statement가 끝났을 때, 메소드, constructor, class가 끝났을 때
        - , 나 else 가 그 다음에 나오는 경우에는 줄 간격을 두지 않는다.

    ```java
    return () -> {
      while (condition()) {
        method();
      }
    };

    return new MyClass() {
      @Override public void method() {
        if (condition()) {
          try {
            something();
          } catch (ProblemException e) {
            recover();
          }
        } else if (otherCondition()) {
          somethingElse();
        } else {
          lastThing();
        }
      }
    };
    ```

3. 빈 블록의 경우: 

    다음 두 경우 모드 가능하나, multi-block 인 경우에는 consice 하게 할 수 없다. 

    ```java
    // This is acceptable
      void doNothing() {}

      // This is equally acceptable
      void doNothingElse() {
      }
    ```

    ```java
    // This is not acceptable: No concise empty blocks in a multi-block statement
      try {
        doSomething();
      } catch (Exception e) {}
    ```

### 2-2. 블록 indentation: +2 spaces

- 새로운 블록일 경우 2 만큼 들여쓰기 한다.
- 하지만 우테코에서는 +2 가 아니라 +4 만큼 들여쓰기 하도록 한다.

### 2-3 한 줄에 한 statement만 작성한다.

- 각 statement는 줄 간격을 둔다.

### 2-4 Column limit: 100

다음과 같은 경우가 아니라 한 줄에 100자가 넘지 않도록 line-wrapping을 한다. 

1. line-wrapping이 불가능한 경우
2. package / import 일 경우
3. shell에 복사 붙여넣기 해야 하는 comment일 경우

### 2-5 Line-wrapping

- 다음과 같은 상황에서 line break를 하여 line-wrapping 한다.
    1. non-assignment operator일 경우 줄 간격은 해당 Operator 앞에서 break 한다. 

        다음과 같은 것들에도 적용된다: 

        - dot separator (.)
        - two colons of method reference (::)
        - an ampersand in a type bound (<T extends Foo & Bar>)
        - pipe in a catch block ( catch (FooException | BarException e) )
    2. assignment-operator일 경우 해당 operator 다음에 line break 한다. 
    3. ( 앞에 있는 메소드나 constructor 이름은 붙어 있도록 한다. 
    4. , 같은 경우 그 앞의 토큰과 붙어 있는다. 
    5. lambda 의 → 다음에는 line break 하지 않는데, lambda body가 single expression 인 경우를 제외하고는 반드시 ( 다음에 line break 해야 한다. 

        ```java
        MyLambda<String, Long, Object> lambda =
            (String label, Long value, Object obj) -> {
                ...
            };

        Predicate<String> predicate = str ->
            longExpressionInvolving(str);
        ```

- line-wrapping 이후에는 +4 만큼의 들여쓰기를 한다.

### 2-6 공백 Whitespace

1. Vertical Whitespace
    - 다음과 같은 경우 빈 줄이 들어간다.
        1. consecutive members 나 initializers of a class 사이에 빈 줄 
            - 필드, constructors, methods, nested classes, static initializer, instance initializer
            - 두 필드 사이에 공백은 선택이다.
    - 빈 줄은 가독성을 위해서 필요한 곳에 어디든 추가될 수 있다.
2. Horizontal whitespace 
    1. if, for, catch 뒤에 있는 '(' 사이에 공백
    2. else, catch 앞에 있는 '}' 사이에 공백 
    3. '{' 앞에 공백.
        - 예외1: annotation 안에 있는 '{' 앞에는 공백 없음
        - 배열 안에 원소로 인한 '{' 앞에는 공백 없음
    4. binary 나 ternary operator 앞 뒤로 공백 넣는다. 
        - <T extends Foo & Bar>
        - `catch (FooException | BarException e)`
        - `(String str) -> str.length()`
        - (::) 이나 (.) 앞 뒤에는 공백 없음
    5. ,:; 나 ')' 뒤에 공백 있음.
    6. 변수이름과 type 사이에 공백: List<String> list

<br>

## 3. Naming

### 3-1. Package names

1. 패지키 이름은 Camel Case도 아니고 전부 소문자로 띄어쓰기 없이 이루어진다. 

### 3-2. Class names

1. 클래스 이름은 UpperCamelCase로 이루어 진다. 
2. 주로 noun이나 noun phase이다. 
3. Test 클래스의 경우 뒤에 Test가 붙는다. 

### 3-3 Method names

1. 메소드 이름은 lowerCamelCase로 나타난다.
2. 주로 verb 이다. 

### 3-4 Constant names

1. 상수의 경우 CONSTANT_CASE 와 같이 전부 대문자, _ 로 구분되어 있다. 
2. 여기서 상수라고 하는 것은 static final field 이며, 잘 변하지 않고, 메소드에 이거에 의한 부작용이 없는 숫자를 말한다. 
3. 예시: 

    ```java
    // Constants
    static final int NUMBER = 5;
    static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
    static final ImmutableMap<String, Integer> AGES = ImmutableMap.of("Ed", 35, "Ann", 32);
    static final Joiner COMMA_JOINER = Joiner.on(','); // because Joiner is immutable
    static final SomeMutableType[] EMPTY_ARRAY = {};
    enum SomeEnum { ENUM_CONSTANT }

    // Not constants
    static String nonFinal = "non-final";
    final String nonStatic = "non-static";
    static final Set<String> mutableCollection = new HashSet<String>();
    static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
    static final ImmutableMap<String, SomeMutableType> mutableValues =
        ImmutableMap.of("Ed", mutableInstance, "Ann", mutableInstance2);
    static final Logger logger = Logger.getLogger(MyClass.getName());
    static final String[] nonEmptyArray = {"these", "can", "change"};
    ```

### 3-5 Non-cnastant field names

1. Non-constant field(static 이거나 아니거나)의 경우 lowerCamelCase로 되어 있다. 주로 noun 이다. 

### 3-6 이외의 다른 CamelCase

1. Parameter, local variable, type variable 모두 lowCamelCase로 쓴다. 


```toc
```