---
emoji: 🤹‍♀️
title: "[동적계획법] 이항계수"
date: '2020-09-24 23:00:00'
author: 코다
tags: 알고리즘
categories: 알고리즘
---

이런 말이 있다. 

> 동적 계획법이라는 말은 전문가들이 전문가들처럼 보여줄 수 있도록 해주는 말이고 일반인들에게는 그냥 '기억해서 풀기' 다.

이항계수에 관련한 성질은 기억해두면 이후 코딩이나 알고리즘 문제를 풀 때 유용하기 때문에 기록해 준다. 이항계수를 풀 때 중요한 성질은 다음과 같다. <br>

$$
{n \choose k} = {n \choose n-k}
$$

$$
{n \choose k} = {n-1 \choose k} + {n-1 \choose k-1}
$$

$$
\sum_{k=1}^n {n \choose k} = 2^n
$$

위의 공식은 이항계수의 정의식을 참고해서 유도하는 방법으로 이항 계수의 정의식을 알고 있어야 한다.

$$
{n \choose k} = _{n}\mathrm{C}_{k} = \frac{n!}{(n-k)!k!}
$$

## 동적 계획법을 활용한 이항계수 풀이

이항계수에 관련한 알고리즘 문제를 풀기 위해서 이항계수의 2번째 성질을 이용하기로 한다. 그 이유는 2번째 성질이 동적 계획법 활용에 알맞게 더 작은 부분으로 분할하여 정복 할 수 있는 성질을 잘 드러내고 있기 때문이다. 다음 방법을 사용해서 알고리즘을 풀어보자. <br>

여기서 일반 재귀나 분할 정복보다 동적 계획법에 알맞게 진행하기 위해서 memoization을 사용한다. <br>

```java
//DAC
import java.util.Scanner;

class Main{
	static long[][] value;
	public static void main(String[] args){
		Scanner sc = new Scanner(System.in);
		int n = sc.nextInt();
		int k = sc.nextInt();

		value = new long[n+1][k+1];

		coef(n, k);
		long result = value[n][k];
		System.out.println(result);
	}

	public static void coef(int a, int b){
		if (a==b){
			value[a][b] = 1;
			return;
		}
		if (b==0) {
			value[a][b] = 1;
			return;
		}
		else {
      
			if(value[a-1][b] == 0)
				coef(a-1, b);
			if(value[a-1][b-1] == 0) 
				coef(a-1, b-1);
			value[a][b] = value[a-1][b] + value[a-1][b-1];
			return;
		}
	}
}

```

<br>

단순히 이항계수의 정의를 이용한 유도식을 재귀를 통해서 구현한 것이다. 다음과 같이 구현하면 작은 숫자들에 대해서는 충분히 답을 낼 수 있지만 숫자가 커지게 되면 할당해야 하는 배열의 크기가 기하급수적으로 커지게되고, 그 결과 값 또한 long 타입으로도 담을 수 없기 때문에 매우 제한적이다. 따라서 [백준 11401](https://www.acmicpc.net/problem/11401)에서는 이항계수를 소수인 1,000,000,007로 나눈 프로그램을 작성하도록 되어 있다. 그 연산에 대해서는 2가지 접근 방법이 있다. <br>

우선, 왜 위에서 소개한 이항계수 정의식에 바로 \% 1,000,000,007을 하지 않는지에 대한 이유를 짚고 넘어가야 한다. 이항계수 정의식은 분수꼴이기 때문에 소수 p로 \% 연산을 했을 때 분자와 분모에 나뉘어서 적용되지 않는다. <br>

$$
\frac{N!}{K!(N-K)!}\%p\qquad  \Longrightarrow \qquad \text{나뉘어서 적용 불가}
$$


### 접근 방법  1. 확장 유클리드 알고리즘

확장 유클리드 알고리즘은 기본 원리로 유클리드 호제법으로 GCD를 구하는 것을 따라가며, 두 정수 A,B가 주어졌을 때, 베주 항등식인 $Ax+By=gcd(A,B)$ 에서 gcd(A,B)를 구하고 정수해 (x,y)를 구하는 알고리즘이다. <br>

여기서 사용하고 싶은 확장 유클리드 알고리즘의 항을 써보면 다음과 같다. <br>

$$
(AB^{-1}) \% p
$$

위에서 말했던 이항계수 정의식을 보면 $A$와 $B$에 각각 무엇이 대입되는지 알 수 있다. 여기서 확장 유클리드 알고리즘을 사용하는데, 확장 유클리드 알고리즘은 두 수의 최대공약수와 베주 항등식의 $x$와 $y$까지 구할 수 있는 알고리즘이다. <br>

$$
Bx + py =1 \qquad (i)
$$

$$
Bx \equiv 1 \pmod{p} \qquad (ii)
$$

위의 (i)에서 확장 유클리드 알고리즘으로 정수해 $(x, y)$를 구할 수 있는데 그럼 다음 식에 대입하면 원하는 식을 구할 수 있다. <br>

$$
(AB^{-1}) \% p \\= (AB^{-1} \cdot 1) \%p\\=(AB^{-1} \cdot Bx)\%p\\=Ax\%p
$$

결론은, **베주 항등식에서 구한 $x$와 정의식에서 정의한 $A$를 곱한 것을 $p$로 modular 하면 원하는 식을 구할 수 있다.** <br>

여기서 확장 유클리드 알고리즘을 이해하고 최종적으로 $x$를 구하는 것이 어렵게 느껴졌는데, 구현해보니 지나치게 길거나 복잡하지는 않았다. 재귀를 사용해서 구현하였다. <br>

<br>

```java
public static void euc(long p, long B) {
  if(p%B > 0){
    euc(B, p%B);
    temp = y;
    y = x - (p/B) * y;
    x = temp;
  } else {
    x = 0;
    y = 1;
  }
}
```


<br>
<br>

**<small>[참고 자료]: https://onsil-thegreenhouse.github.io/programming/problem/2018/04/02/problem_combination/</small>**

```toc
```