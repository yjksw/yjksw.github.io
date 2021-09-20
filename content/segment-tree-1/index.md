---
emoji: 🤹‍♀️
title: [알고리즘] 세그먼트 트리를 활용한 히스토그램 문제 풀이_1
date: '2020-09-09 23:00:00'
author: 코다
tags: Algorithm
categories: Algorithm
---

히스토그램에서 가장 큰 직사각형의 크기를 찾는 알고리즘을 풀다가, 관련 문제의 풀이법을 간단히 찾아서 금방 해결할 줄 알았으니 구현에서 의도치 않은 오랜 시간이 걸렸다. 

먼저 [문제](https://www.acmicpc.net/problem/6549)의 해결 방법을 요약하면 다음과 같다. 

> 1. 히스토그램 중, 높이가 가장 낮은 min 값과 해당 너비값을 곱하여 넓이를 구함. 
> 2. 해당 최소값을 기준으로 히스토램을 나누어서 1번을 반복함. 
> 3. 더 이상 나눌 수 없을 때까지 반복하며 매번 넓이의 max 값을 업데이트 함. 

다음은 [백준 블로그](https://www.acmicpc.net/blog/view/12)에 있는 문제 해설에서 가져온 그림이다. 위의 해결 방법을 이해하는데 도움이 된다. 


![histogram](https://user-images.githubusercontent.com/63405904/109445062-10bb9c80-7a82-11eb-9887-9047f1485785.png){: width="80%"}


처음에 단순히 이 풀이방법을 배열과 재귀를 사용해서 구현하는 방법으로 시도를 했었다. 사이트에 나와있는 테스트 케이스가 통과하길래 바로 채점을 했더니 결과는 *시간초과* 였다.. 개인적으로 알고리즘을 할 때 가장 어려운 부분이 답을 출력이 되지만 시간초과가 나올 때 인 것 같다. 문제설명 밑에 해당 문제를 세그먼트 트리를 사용한 분할정복으로 풀 수 있다고 하길래 세그먼트 트리에 대해서 공부하면서 정리한 내용과 처음에 접근했던 방식에 대해서 쓰려고 한다. 

1. 배열/재귀를 사용해서 풀었던 방법: <mark> 시간초과 </mark>
2. 세그먼트 트리/분할정복을 사용해서 푼 방법: <mark>통과</mark>



### 배열과 재귀를 사용한 첫번째 접근 방법

배열과 재귀를 사용한 접근 방법은 간단하지만 번거롭다. 매번 나뉘어진 구간 사이에서의 **최솟값**을 찾는 과정을 반복해야 하기 때문이다.  

#### 접근 방법 1: ArrayList

ArrayList를 사용해서 탐색 API를 사용해서 최솟값 구하기

```
1. ArrayList의 일부 구간을 subList를 사용하여 List에 저장한다.
2. Collections.min() 메소드를 사용하여 최솟값을 추출하여 넓이를 구한다. 
3. indexOf() 메소드를 사용하여 최솟값의 index를 구한다. 
4. 다음과 같은 재귀로 반복한다.
	* 맨 첫번째 요소가 최소값일 경우: 두번째 요소부터 마지막 요소까지 재귀
	* 맨 마지막 요소가 최소값일 경우: 첫본째 요소부터 마지막 이전 요소까지 재귀
	* 중간의 어느 값이 최소값일 경우: (시작 요소, 최소값 위치 -1)과 (최소값 위치 +1, 마지막 요소)로 나누어서 재귀
```

위의 방식은 조금만 큰 값이 들어가도 바로 *시간 초과*가 결렸다. 이유는 ArrayList의 경우 일반 배열과 달리 초반에 메모리 할당이 되지 않기 때문에 추가/삭제 시 메모리 할당을 매번 해줘야 한다. 따라서 일반적으로 일반 배열이 더욱 빠르다. 그래서 두번째 접근 방식으로 일반 배열을 사용하는 것을 택했다. 일반 배열을 사용하면 최소값을 찾는 등의 메소드를 사용하기는 어렵지만 최소값을 찾는 구현은 어렵지 않고, 시간 복잡도도 비슷하기 때문에 시도해 보았다. 

**코드:**

```java
static void solve(int startIndex, int lastIndex){
  long area = 0;
  if(startIndex == lastIndex){
    area = value.get(startIndex);
    if(area>max)
    	max = area;
    return;
  }
  
  List<Integer> list = value.subList(startIndex, lastIndex-1);
  long min = Collections.min(list);
  area = min*(lastIndex-startIndex+1);
  int index = startIndex + list.indexOf(min);
  if(area > max)
    max = area;
  
  if(index == startIndex)
    solve(index+1, lastIndex);
  else if(index == lastIndex)
    solve(startIndex, index-1);
  else{
    solve(startIndex, index-1);
    solve(index+1, lastIndex);
  }
}
```





#### 접근 방법 2: Arrays

Array를 사용해서 최소값을 구하기

```
1. for-loop를 사용해서 최소값 구하기
2. 최소값 사용하여 넓이 구하기 
3. 접근 방법 1에서와 같이 재귀하기
```

ArrayList를 사용했을 때보다는 빨랐기 때문에 더 많은 test case를 통과할 수 있었다. 하지만 여전히 시간초과에 걸렸다. 

문제에서 나온 직사각형의 갯수 제한은 100,000이고 재귀 초기함수가 `(startIndex==lastIndex)` 일 때이기 때문에 각각하나씩 모두 접근한다. 이때마다 해당 구간의 최소값을 찾기위해 O(n)만큼 탐색을 하니 시간 초과가 걸릴만 하다. 때문에 문제의 태그에서 나온 세그먼트 트리에 대해서 공부하고 활용해보기로 했다. 

**코드:**

```java
static void solve(int startIndex, int lastIndex){
  long area = 0;
  long min = -1;
  long index = -1;
  if(startIndex==lastIndex){
    area = value[startIndex];
    if(area>max)
      max=area;
    return;
  }
  
  for(int temp=startIndex;temp<=lastIndex;temp++){
    if(min<0 || value[temp]<min){
      min = value[temp];
      index = temp;
    }
  }
  
  area = min*(lastIndex-startIndex+1);
  
  if(area>max)
    max = area;
  
  if(index == startIndex)
    solve(index+1, lastIndex);
  else if(index == lastIndex)
    solve(startIndex, index-1);
  else{
    solve(startIndex, index-1);
    solve(index+1, lastIndex);
  }
}
```





#### 접근 방법 3: Segment Tree

세그먼트 트리는 <mark>주어진 쿼리에 빠르게 응답하기 위해 만들어진 자료구조</mark>이고, 그 사용법은 쿼리마다 상이하다. 가장 대표적으로 세그먼트 트리를 사용할 때 내는 예시는 구간 합을 구하는 문제이다. 하지만 이 글에서는 *히스토그램에서 가장 큰 직사각형* 푸는 문제에 적용된 세그먼트를 설명할 것이다. 

풀이에 세그먼트 트리를 활용할 수 있는 상황은 다음 두가지와 같다. **1. 쿼리 형식으로 문제가 주어진 경우 2. 시간 복잡도를 log로 만들고 싶을 경우**. <small>개인적으로 구간에 관련한 문제가 나올 경우, 시간 복잡도를 줄이기 위해 세그먼트 트리 사용을 하는 것이 좋은 것 같다.</small>

세그먼트 트리는 주로 이진트리를 이용하며, 주로 완전 이진 트리 Full Binary Tree에 가깝다. 그렇기 대문에 세그먼트 트리를 사용하면 다음과 같은 성능을 지닌다. 

1. 쿼리의 결과값 구하기: O(lgN)
2. 값 업데이트 하기: O(lgN)

히스토그램 문제에서는 쿼리의 결과값 구하는 과정의 시간 복잡도가 O(lgN)이 되면서 성능이 매우 좋아지게 된다. 



###### Segment Tree 란?

<img src="https://user-images.githubusercontent.com/63405904/109445201-6e4fe900-7a82-11eb-8e6c-09edb7e236a4.png" alt="histogram" style="zoom:70%;" />

히스토그램 문제에서는 위에서 말했듯 다음 두가지 풀이를 반복한다. 

1.  최소값 기준으로 구간 나누기

2. 나뉘어진 구간에서 재귀로 1) 반복하기

히스토그램에서 중요 요소는 최소값이기 때문에 각 구간의 최소값의 위치를 저장하도록 한다. 따라서 이후에 특정 구간의 최소값을 찾을 때 <mark>O(lgN)</mark>만큼의 시간복잡도로 최소값을 찾을 수 있다. 위의 이진 세그먼트 트리는 10개의 원소가 있다고 가정했을 때 각 구간이 나뉜 것을 보여준다. **세그먼트 트리에서 모든 leaf node는 원래 배열의 자기자신 element**이다. 

요약하자면 segment tree의 구성요소는 다음과 같다. 

* Leaf node :  원래 배열의 그 수의 위치. <small>(이 값은 응용 문제에 따라서 달라진다.)</small>
* 다른 node:  왼쪽 자식과 오른쪽 자식 중 더 최소값의 위치.

히스토그램 문제에서 Segment tree 구현을 요약하면 다음과 같다. 

```
1. 초기화 함수: 구간이 자기 자신일 경우, leaf node이므로 해당 위치를 기록함. 
2. 재귀 함수: 구간을 반으로 나누어서 재귀함.
3. 일반 함수: 자신의 왼쪽 자식 노드와, 오른쪽 자식 노드 위치의 값을 비교하여 더 작은 값의 위치를 트리의 해당 노트에 입력함. 
```

전체 구간 `0 ~ n-1`까지부터 시작해 재귀를 하면 각 구간마다의 <mark>최솟값의 위치</mark>를 기록한 *lgN* 높이 만큼의 segment tree가 생성된다. 이 세그먼트 트리를 사용해서 더 빠른 방법으로 최소값을 탐색하고 제일 앞에서 설명한 방법을 통해서 정답을 도출하면 된다. 

---

다음 글에서 구체적으로 세그먼트 트리를 구현하는 방법과 히스토그램에서 응용된 방법에 대해서 다루도록 하겠다. 



**<small>[참고 자료]: https://www.acmicpc.net/blog/view/12, https://www.crocus.co.kr/648, https://www.acmicpc.net/blog/view/9 </small>**

```toc
```