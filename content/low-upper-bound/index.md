---
emoji: 🤹‍♀️
title: "[번역] Lower and Upper Bound Theory"
date: '2020-09-04 23:00:00'
author: 코다
tags: 알고리즘
categories: 알고리즘
---

알고리즘에 대해서 배울 때 가장 먼저 다루는 부분이 바로 **Time Complexity** 이다. 기술이 발전하면서 메모리에 대한 부분은 상당 부분 해결이 되고 걱정하지 않아도 되는 부분이 되었다. 하지만 시간 복잡도 측면에서는 아무리 발전해도 부족한 부분이다. 왜냐하면 짧으면 짧을수록 더 좋기 때문이다. 따라서 알고리즘 강의를 들을 때에는 항상 Time Complexity에 대한 강의를 시작으로 배운다. 어떠한 문제에 대해서 여러가지 알고리즘을 사용하여 해결할 수 있을 때 무엇이 최적의 알고리즘인지를 판단하는 잣대는 해당 알고리즘으로 문제를 해결하는데 걸리는 시간이기 때문이다. 거기서 핵심적인 역할을 하는 두 theory에 대해서 [다음 글](https://www.geeksforgeeks.org/lower-and-upper-bound-theory/)의 내용을 번역 및 정리하면서 알아보자. <br>

Lower Bound와 upper Bound Theory는 어떠한 문제에 대한 가장 적은 복잡도를 가진 알고리즘을 선택하는데 핵심적인 역할을 한다. 구체적으로 이 이론들을 다루기 이전에 각각 Lower Bound와 Upper Bound가 무엇을 의미하는지 살펴보자. <br>

* **Lower Bound** - <br>

  L(n)이 알고리즘 A에 대한 수행 시간일 때, $L(n) >= C*g(n)$을 성립하는 $C$가 있는 경우 $g(n)$은 A의 lower bound이다. 어떠한 알고리즘의 lower bound는 Big Omega 로 나타낸다. <br>

* **Upper Bound** - <br>

  U(n)이 알고리즘 A에 대한 수행 시간일 때, $L(n) <= C*g(n)$을 성립하는 $C$가 있는 경우 $g(n)$은 A의 lower bound이다. 어떠한 알고리즘의 lower bound는 Big Oh(O) 로 나타낸다. <br>


## 1. Lower Bound Theory:

Lower Bound Theory에 의하면 어떠한 알고리즘의 lower bound에 대하여 다른 어떠한 알고리즘도 랜덤한 input에 대하여 L(n)보다 적은 시간 복잡도를 가질 수 없다. 또한 다르게 말하면 모든 알고리즘들이 **적어도 L(n)** 만큼의 시간을 가질 수 밖에 없다. <br>

*주의할 점: L(n)은 모든 알고리즘 중 최소의 복잡도를 나타낸다.*<br>

Lower Bound 는 그 어던 알고리즘에 있어서도 매우 중요하다. Lower Bound를 계산한 후에, 특정 알고리즘의 복잡도를 계산하여 L(n)과 같다면 해당 알고리즘이 최적의 알고리즘이라는 것을 알 수 있다. 이 글에서 우리는 어떠한 알고리즘의 lower bound를 찾는 방법들에 대해서 다루어 볼 것이다. <br>

명심해야 하는 것은 언제나 우리의 가장 중요한 목적이 **최적의 알고리즘**을 구하는 것이라는 것이다. 여기서 최적의 알고리즘이라고 함은 해당 알고리즘의 Upper Bound 가 해당 문제의 Lower Bound와 같은(U(n)=L(n)) 경우이다. *Merge sort*를 통해서 optimal algorithm을 살펴보자. <br>

### Trivial Lower Bound -

Lower bound를 찾는 가장 쉬운 방법이다. 이 방법은 Lower bound가 문제의 input의 개수와 output의 개수로 쉽게 알 수 있다고 하는 Trivial Lower Bound 방법이다. <br>

#### 예시: Multiplication of n*n matrix 

- Input: 2개의 행렬에 대해 $2n^2$개
- Output: 1 개의 n*n 행렬, $n^2$ 개

위의 input/output의 숫자를 보면 쉽게 lower bound가 $O(n^2)$라는 것을 알 수 있다. <br>

### Computational Model -

이 방법을 비교를 하는 모든 알고리즘에 대해서 사용할 수 있다. 예를 들어, sorting 문제의 경우 각 원소들을 비교하고 배열을 해야한다. 탐색하는 문제 또한 비슷하다. 예시를 통해서 해당 문제들의 lower bound를 구하는 법에 대해서 살펴보자. <br>

### Ordered Searching -

이미 정렬이 되어 있는 리스트에 대해서 탐색을 하는 경우이다. <br>

### Example 1: Linear Search

처음부터 시작하여 차례로 각 element를 순회하며 해당 원소가 찾던 원소인지 확인한다. 

### Example 2: Binary Search

중간에 있는 것과 비교하여 찾고자 하는 숫자가 클 경우, 반의 오른쪽 부분을, 작을 경우 반의 왼쪽 부분을 나누어서 탐색한다. 

### Calculation the lower bound: 

비교하는 최대 횟수는 n이고, 리스트의 tree에 총 k levels이 있다고 한다면..

1. Node의 갯수는 $2^k-1$
2. $2^k -1$에 대하여 worst case의 경우 upper bound 개수인 n 
3. 각 레밸에서 1번 비교를 하니, 비교하는 횟수는 $k>=|log2n|$

따라서 비교 문제에 있어서 n개의 원소를 가질 경우 복잡도는 log(n) 보다 작을 수 없다. 따라서 시간복잡도를 (log n)을 가진 binary serach가 최적화된 알고리즘이라고 판단할 수 있다. 

## 2. Upper Bound Theory

Upper Bound Theory는 해당 문제를 푸는데 최대 U(n)만큼이 시간이 뜬다는 것이다. 주로 worst case input인 경우 Upper Bound를 알 수 있다. <br>


```toc
```