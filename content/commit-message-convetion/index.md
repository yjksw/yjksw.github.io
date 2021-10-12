---
emoji: 👀
title: "[GitHub] Commit Message Convetion"
date: '2020-11-29 23:00:00'
author: 코다
tags: 기타
categories: 기타
---

Github에 익숙하지 않기 때문에 커밋은 나에게 push를 해서 업로드를 하기 위한 중간과정 중 하나였다. 하지만 다른 곳에서 깃헙이나 프로젝트 진행을 하면서 커밋을 하는 단위의 중요성과 깃헙의 최대 장점인 프로젝트를 되돌리기 위한 커밋 메세지의 중요성에 대해서 여러번 들었었다. 이번에 프리코스를 시작하면서 커밋 메세지에 대한 가이드를 읽고 정리해보기로 했다. <br>

[참고 사이트]([https://gist.github.com/stephenparish/9941e89d80e2bc58a153#recognizing-unimportant-commits](https://gist.github.com/stephenparish/9941e89d80e2bc58a153#recognizing-unimportant-commits))

<br>

## CHANGELOG.md 생성하기

- changelog에는 3개의 section이 있다: new features, bug fixes, breaking changes.
- 이러한 정보들은 배포가 될 때 script로 생성이 되어야 하며 해당하는 commit과 함께 제공되어야 한다.
- 해당 로그들을 보는 방법들은 다음과 같다.
    1. 지난 release 이후에 발생한 모든 subject(커밋 메세지의 첫번째 라인) 조회:

        ```java
        git log <lasg tag> HEAD --pretty=format:$s
        ```

    2. 이번 release의 새로운 feature:

        ```java
        git log <last release> HEAD --grep feature
        ```

### Recognizing unimportant commits

- 사소한 버그 수정 등과 같이 중요하지 않은 커밋들을 걸러낼 수 있다. 코드의 logic이 수정된 부분들이 아닌 경우에는 다음과 같은 명령어로 무시할 수 있다.

    ```java
    git bisect skip $(git rev-list --grep irrelevant <good place> HEAD)
    ```

### History 브라우징 시 정보 제공을 위한 커밋

- 커밋 메세지를 작성할 때 가능한 많은 정보들을 제공하는 것이 좋다.
- 그렇기에 메세지로 무슨 변경이나 추가가 있었는지 확인 할 수 있는데, 해당 메세지가 일정한 convention을 지닐 필요가 있다.

<br>

## Commit Message 형식

```java
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

- 커밋 메세지는 100자를 넘지 않는다. 이래야지 깃헙이나 깃 툴을 사용할 때 메세지 읽기가 쉽다.

### Subject line

어떤 변경이 일어났는지에 대한 간단명료한 설명을 담고 있다. (커밋 메세지의 첫 줄)

1. Allowed `<type>` 
    - feat (feature)
    - fix (bug fix)
    - docs (documentation)
    - style (formatting, missing semi colons, ...)
    - refactor
    - test (when adding missing tests)
    - chore (maintain)
2. Allowed `<scope>` 
    - 적용 범위를 나타내는 것으로 커밋에 대한 부가적인 정보이다. (선택 사항)
    - 적용 범위에 대한 예시는 이러하다: $location, $browser, $compile, $rootScope, ngHref, ngClick, ngView, 등등
3. `<subject>`
    - 현재형으로 작성한다: "change" → x "changed" or "changes"
    - 첫 문자를 대문자로 작성하지 않는다.
    - (.)을 작성하지 않는다.
4. 해당 커밋에 major 한 변화가 있다면 큰 변화가 있기 때문에 호환이 안되는 부분들이 있을 수 있다. 해당 부분들을 footer에 반드시 작성하게 되는데, 해당 메세지를 확인하지 못할 수도 있기 때문에 다음과 같이 `BREAKING CHANGE: 설명` 있음을 표시한다. 
    - `예: fead(pipeling)!: Add pipeline function`

### Message body

- 커밋 메세지와 같이 현재형 동사로 작성한다.
- 수정의 동기화, 수정 이전과의 비교를 명시한다.
- 본문에 여러개가 있을 경우에는 (-)로 구분한다.

### Message footer

- 커밋이 어떤 이슈에서 왔는지 촘조 정보들을 추가하는 용도로 사용.
- 특정 이슈와의 연관을 표현하기 위해 `close #123 #245` 같이 커밋 메세지를 추가한다.
- Breaking changes에 대해서 footer에 작성한다

    어떤 것이 수정되었는지, 수정이 된 정의, migration note 등이 추가되도록한다. 

    ```java
    BREAKING CHANGE: isolate scope bindings definition has changed and
        the inject option for the directive controller injection was removed.
        
        To migrate the code follow the example below:
        
        Before:
        
        scope: {
          myAttr: 'attribute',
          myBind: 'bind',
          myExpression: 'expression',
          myEval: 'evaluate',
          myAccessor: 'accessor'
        }
        
        After:
        
        scope: {
          myAttr: '@',
          myBind: '@',
          myExpression: '&',
          // myEval - usually not useful, but in cases where the expression is assignable, you can use '='
          myAccessor: '=' // in directive's template change myAccessor() to myAccessor
        }
        
        The removed `inject` wasn't generaly useful for directives so there should be no code using it.
    ```

<br>

## 예시

```java
feat($browser): onUrlChange event (popstate/hashchange/polling)

Added new event to $browser:
- forward popstate event if available
- forward hashchange event if popstate not available
- do polling when neither popstate nor hashchange available

Breaks $browser.onHashChange, which was removed (use onUrlChange instead)
```

```java
fix($compile): couple of unit tests for IE9

Older IEs serialize html uppercased, but IE9 does not...
Would be better to expect case insensitive, unfortunately jasmine does
not allow to user regexps for throw expectations.

Closes #392
Breaks foo.bar api, foo.baz should be used instead
```

```java
docs(guide): updated fixed docs from Google Docs

Couple of typos fixed:
- indentation
- batchLogbatchLog -> batchLog
- start periodic checking
- missing brace
```

```toc
```