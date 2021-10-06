---
emoji: 🤹‍♀️
title: 분할정복 기본 알아보기 
date: '2020-07-21 23:00:00'
author: 코다
tags: 알고리즘
categories: 알고리즘
---

다음은 분할정복 Divide and Conquer Algorithm에 대한 지식 블로그 **번역 및 요약+추가정리** 내용이다. <br>

<small>원문: https://www.geeksforgeeks.org/divide-and-conquer-algorithm-introduction/</small> <br>

이 글에서는 **분할정복**이 유용한 경우와 분할정복으로 해결할 수 있는 문제의 접근방식에 대해서 다룰 것이다. 다음이 이 글에서 다룰 내용들이다. <br>

1. DAC(분할정복) 소개 및 정리
2. DAC을 사용하는 알고리즘
3. DAC 알고리즘을 사용한 재귀
4. ~~DAC를 사용한 문제 예시~~

<br>

## Divide and Conquer:

1. **Divide**: 문제를 분할이 가능한 경우까지 분할한다.
2. **Conquer**: 분할된 sub problem을 재귀로 호출해 정복한다.
3. **Combine**: 해결된 sub problem들을 합쳐 문제의 해결책을 찾는다. 

<br>

## DAC가 응용된 알고리즘 기법들:

1. Binary Search(이진탐색): 검색하고자 하는 value를 주어진 배열의 중간 인덱스((start+end)/2)와 비교한다. <small>이때 배열은 오름차순으로 정렬된 상태이다.</small> value가 중간 인덱스의 값과 일치하면 해당 index를 반환한다. 작다면 중간 인덱스를 기준으로 왼쪽 배열에 재귀호출하여 앞의 탐색을 반복한다. 만일 크다면, 중간 인덱스를 기준으로 오른쪽 배열을 재귀적으로 탐색한다. 

   * <mark>Divide</mark>: 중간 인덱스를 기준으로 재귀호출을 통해 분할함. <br>
   * <mark>Conquere</mark>: 분할된 배열 내에서 value를 탐색하여 정복함. <br>

![image](https://user-images.githubusercontent.com/63405904/111792910-14677400-8908-11eb-8cb1-9bdbffc79340.png)

1. Quick Sort(퀵 정렬): 정렬의 한 기법으로 pivot element를 기준으로 pivot보다 더 작은 요소들은 pivot 기준 왼쪽에, 큰 요소들은 pivot 기준 오른쪽으로 분할한다. 각각 분할된 부분배열에서도 각각 pivot을 정해 동일하게 정렬하고 더 이상 분할할 수 없을 때(부분배열의 크기가 0또는 1이 됨)까지 반복하여 정렬한다. 

   * <mark>Divide</mark>:  배열을 pivot element의 기준으로 비균등한 크기로 나눈다. 재귀호출을 통해 더 나누어질 수 없을 때까지 나눈다. <br>
   * <mark>Conquer</mark>: 부분 배열을 pivot element 기준으로 크기 정렬을 하여 정복한다. 이때 순환 호출을 통해서 분할 정복을 반복 실행한다. <br>

![image](https://user-images.githubusercontent.com/63405904/111793059-395be700-8908-11eb-9a9a-8b690b3012f4.png)

1. Merge Sort(합병정렬): 정렬의 한 기법으로 순훈호출로 배열을 반으로 분할하여 정렬하고 합병하여 정렬하는 기법이다. 

   * <mark>Divide</mark>: 배열을 반으로 분할한다. <br>
   * <mark>Conquer</mark>: 분할된 부분배열을 정렬하고 부분 배열이 충분히 작지 않으면 순환 호출로 다시 분할 정복을 적용한다. <br>

![image](https://user-images.githubusercontent.com/63405904/111793003-2a753480-8908-11eb-8183-c3fe1f88e15c.png)

1. Closest Pair of Points: 여러 포인트들의 집합 속에서 가장 서로 가까운 포인트 쌍을 찾는 문제이다. 해당 문제는 모든 포인트 쌍의 거리를 비교하여 min 값을 찾을 때에는 O(n^2) 만큼의 시간 복잡도를 가지지만 DAC 알고리즘을 사용하면 O(nLogn) 의 시간 복잡도로 해결이 가능하다. 


<br>
<br>

<small>이미지 출처: https://m.blog.naver.com/PostView.nhn?blogId=horajjan&logNo=220310806806&proxyReferer=https:%2F%2Fwww.google.com%2F, https://gmlwjd9405.github.io/2018/05/10/algorithm-quick-sort.html, https://www.geeksforgeeks.org/divide-and-conquer-algorithm-introduction/ </small> 

```toc
```