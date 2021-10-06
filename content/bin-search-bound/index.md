---
emoji: 🤹‍♀️
title: "[알고리즘] 이진탐색 응용: UpperBound와 LowerBound"
date: '2020-09-04 23:00:00'
author: 코다
tags: 알고리즘
categories: 알고리즘
---

이진탐색의 응용 버전으로 상/하한선을 찾는 알고리즘이다. 이를 응용한 [문제](https://www.acmicpc.net/problem/10816)를 풀면서 배운 개념을 정리해둔다. <br>

이진탐색을 사용하면 모든 요소들을 다 방문하면서 탐색하는 것보다 훨씬 더 효율적으로 원하는 요소를 탐색할 수 있다. 하지만 이진탐색의 경우, <u>중복되지 않은 값이 주어진 경우</u>이거나, <u>해당 요소의 존재 여부</u>만을 가리기 위해서 일 경우에만 사용이 가능하다. 위의 문제의 경우, 중복되는 값들이 존재하며 그 값들이 총 몇개가 있는지도 확인할 수 있어야 하기 때문에 일반적인 이진탐색을 통해서는 답을 도출할 수 없다. 그래서 찾은 알고리즘 <mark>Upper Bound</mark> 와 <mark>Lower Bound</mark> 알고리즘 이다. **이진탐색에서 살짝 변형되어서 중복된 자료가 있을 때 유용하게 탐색할 수 있는 알고리즘**이다. <br>

아래 그림을 보면 lower bound와 upper bound에 대해서 더 잘 이해할 수 있다. <br>

![image](https://user-images.githubusercontent.com/63405904/111030306-c4d70300-8444-11eb-8b82-7ad2c3cc0ec1.png)

## Upper Bound Algorithm

```
> Upper Bound Theory:
	K 값보다 큰 값(>)이 처음 나오는 위치를 리턴해주는 알고리즘. 
```

구현은 이진 탐색과 매우 유사하지만 약간의 변형이 있다. <br>

1. **탐색의 범위를 크기가 n일 때:** index만큼인 n-1까지 탐색하는 것이 아니라, n까지 탐색해야 함.
2. **탐색하는 숫자 k를 찾았거나, mid 값보다 k가 작을 경우:**  해당 값이나 위치를 return하는 것이 아니라, 중간 값 이후부터 분할한 오른쪽 부분을 재 탐색함. 
3. **탐색하는 숫자 k보다 mid에 있는 값이 클 경우:** 본래 mid를 제외한 start~mid-1까지를 탐색했으나, 여기서는 start~mid 탐색해 mid값을 재포함 함. 


여기서 1번을 시행하는 이유는 만약 배열에 있는 모든 요소, 혹은 마지막 요소보다 큰 값에 대해서 return 해야할 경우, 배열의 크기 만큼을 리턴해야 하므로 end 요소에 이진 탐색처럼 `array.length-1`이 아닌 `array.length`가 들어가야 한다. <br>

2번의 경우에는 해당 값 k를 찾았을 때도, *<u>k보다 큰 값이 처음 등장하는 위치</u>* 를 찾아야 하기 때문에, 해당 다음 위치부터 end index에 대해서 마지막 하나 남을 때까지 재탐색 해야 한다. <br>

3번의 경우 mid 인덱스를 재포함하는 이유는 해당 mid에 있는 값이 upper bound 일 수 있으나, 탐색 숫자 k보다 <u>큰 첫번째 숫자임을 보장할 수 없기 때문에</u> 탐색을 이어서 진행하는 것이다. <br>

**코드:**

```java
static int searchUpper(int start, int end, int num){
  if(start>=end)
    return start;
  
  int mid = (start+end)/2;
  
  if(num_list[mid]<=num){
    return searchUpper(mid+1, end, num);
  } else {
    return searchUpper(start, mid, num);
  }
}
```

<br>

## Lower Bound Algorithm

Lower bound 또한 Upper bound와 매우 유사하게 진행되므로 위의 설명을 참고하면 된다. 다만 upper bound와 다르게 작용하는 부분은, 딱 한 부분이다. <br>

* 탐색하고 있는 k가 등장했을 때, upper bound 알고리즘에서는 mid+1 값부터 오른쪽 반을 탐색하였는데, lower bound 알고리즘에서는 mid를 포함시켜 왼쪽 반을 탐색하도록 한다. <br>

이 부분은 lower bound이 경우 숫자 k 가 처음 등장한 위치를 찾아야 하기 때문에 k 가 등장하더라도 그 이전에 k가 존재하지는 않는지 확인해야 하기 때문이다. <br>

<br>

**코드:**

```java
static int searchLower(int start, int end, int num){
  if(start>=end)
    return start;
  
  int mid = (start+end)/2;
  
  if(num_list[mid]>=num){
    return searchLower(start, mid, num);
  } else {
    return searchLower(mid+1, end, num);
  }
}
```

<br>
<br>

**<small>[참고 자료]: http://bajamircea.github.io/coding/cpp/2018/08/09/lower-bound.html, [https://jackpot53.tistory.com/33#:~:text=lower%20bound%EB%8A%94%20%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%82%B4,%EB%A5%BC%20%EB%A6%AC%ED%84%B4%ED%95%B4%EC%A3%BC%EB%8A%94%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98%EC%9D%B4%EB%8B%A4.](https://jackpot53.tistory.com/33#:~:text=lower bound는 데이터내,를 리턴해주는 알고리즘이다.) </small>**


```toc
```