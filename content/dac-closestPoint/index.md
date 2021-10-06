---
emoji: 🤹‍♀️
title: "[알고리즘]분할정복 - 백준 2261 가장 가까운 두 점"
date: '2020-09-17 23:00:00'
author: 코다
tags: 알고리즘
categories: 알고리즘
---

분할정복 알고리즘을 배울 때 나오는 유명한 문제 중 하나이다. 하지만 난이도가 굉장히 높기 때문에 쉽게 접근하기 어려웠는데, 분할 정복에 남은 마지막 문제를 그냥 안풀고 넘어가기엔 마음에 걸려서 마음먹고 공부해보기로 했다. <br>

백준 사이트에서도 검색을 추천하여 알고리즘을 공부하기를 권하기 때문에 검색을 통해 [좋은 글](https://octorbirth.tistory.com/274)을 발견했다. 그리고 해당 문제의 솔루션을 이해하는데만 집중했다. <br>

분할정복 문제를 반복해서 풀어보니 분할정복은 DP 만큼이나 여러가지 형태의 문제가 있으니 최대한 많은 문제들을 풀어보는 것이 중요하다는 것을 알 수 있었다. 그리고 여러 문제를 풀어 본 결과 다음을 깨달을 수 있었다. <br>

- 분할정복에서 분할을 하는 이유 중 하나는 **굳이 필요 없는 연산/비교 등을 하지 않기 위해서**이다. 

다르게 이야기하면 **쓸데없는 것을 쳐내기 위해서** 특정 기준에 따라서 계속 분할을 하는 것이다. [백준 2261](https://www.acmicpc.net/problem/2261) 문제를 보면 어떤 의미인지 알 수 있다. 이 사실이 나로 하여금 더 구현을 잘하게 해주지는 못하지만 *개발자 마인드*를 갖추는데 어느 정도 일조했다고 생각한다. 알고리즘 문제들을 풀면 풀수록 쌓이는 *개발자 마인드* 룰을 통해서 새로운 문제를 바라보더라도 개발자스럽게 생각해야지 적합한 알고리즘을 찾을 수 있다.<br>

이렇게 지나지게 많은 비교 연산을 해야할 때 필요 없는 것이 무엇인지부터 접근해야 한다. <br>

<br>

## 문제 해결

가장 가까운 두 점 문제의 솔루션을 요약하면 다음과 같다. <br>

1. x값 기준으로 정렬.
2. 중간을 기준으로 왼쪽과 오른쪽을 나눔.
3. 왼쪽 가장 가까운 거리 d1, 오른쪽 가장 가까운 거리 d2 찾아냄. 
4. d1과 d2 중 더 최소값을 d 에다가 저장함. 
5. 중간으로 가로지르는 점들 중 중앙과 d 이상 차이나는 점들을 제외함.
6. 해당 점들을 y 기준으로 정렬해서 위의 점과 높이가 d 이상 차이나는 점들을 제외하여 비교하여 d3을 구함. 

<br>

![image](https://user-images.githubusercontent.com/63405904/111621456-5c649900-882b-11eb-9fc0-3f1dbdaca71e.png)

<br>

위의 그림을 보면 보다 직관적으로 이해할 수 있다. 위 과정을 반복하면서 최소값을 지속적으로 업데이트하면 최종적으로 최소값을 찾을 수 있다. <br>

이렇게 x 값을 기준으로 정렬해서 제외한 후에, 아래 사진과 같이 y 값을 기준으로 또 정렬하여 제외시키면 된다. <br>

![image](https://user-images.githubusercontent.com/63405904/111621504-69818800-882b-11eb-9fb4-7be3a658c7b8.png)

## 솔루션 구현

다음 솔루션을 구현하기 위해서 이해하거나 응용하면 좋을 개념들은 다음과 같다. 

1. JAVA comparator
2. 재귀
3. 객체 생성 

"완전히 모르는 건 아닌데?" 라고 생각하기 쉽지만 제대로 생각해보면 나는 잘 활용하지 않았던 개념들이 있었다. 예를 들어, comparator를 생성하여 정렬하기 보다 조금 돌아가지만 이전에 하던 방식으로 일일이 정렬하는 방법을 주로 사용하고, 객체를 생성하여 코드가 직관적이게 되기 보다 배열에 나만 아는 규칙으로 끼워 넣는 경우가 많았던 것 같다. 한번 이 모든 개념들을 제대로 응용해서 좋은 코드를 짜보자. <br>

아래의 대부분의 코드는 [사이트](https://octorbirth.tistory.com/274)에서 참고하고 내가 조금의 업그레이드를 시킨 정도이다. <br>

### JAVA Comparator 

JAVA Comparator는 배열이나 객체 등을 sorting 하기 위해서 매우 유용한 인터페이스이다. 숫자가 아닌 무언가, 또는 조금 다른 기준을 통해서 정렬을 하기 원할 때, 이 comparator를 정의하여 사용하면 매우 유용하다. <br>

기본적인 사용방법은 다음과 같다.<br>

1. Comparator를 implement 한 class 정의 
2. `Arrays.sort`나, `Collections.sort`에서 내부 정렬 기준을 구현 하면됨. 

비슷한 기능을 하는 인터페이스로는 Comparable이 있다. 이것은 어떠한 클래스에서 implement 하여 내부에 있는 compareTo 함수를 통해 클래스 기본 정렬 기준을 설정하는 것이다. <br>

| Comparable | 클래스의 기본 정렬 기준을 설정하는 인터페이스              |
| ---------- | ---------------------------------------------------------- |
| Comparator | 기본 정렬 기준과는 다르게 정렬하고 싶을 때 이용하는 클래스 |

이번에 **가장 가까운 두 점** 문제에서는 Comparator를 사용하여 sort 메소드를 통해서 사용할 예정이다. 우선 점들을 x좌표 기준으로 정렬하고, 이후에 y 기준으로 정렬하는 2가지 기준으로 정렬하는 클래스를 생성한다. <br>

```java
class xComparator implements Comparator<Point> {
  public int compare(Point p1, Point p2){
    return p1.x - p2.x;
  }
}

class yComparator implements Comparator<Point> {
  public int compare(Point p1, Point p2) {
    return p1.y - p2.y;
  }
}
```

### 재귀 Recursion

거의 모든 알고리즘 문제의 일부분이 되는 재귀이지만 이번 문제에서 재귀를 활용하면서 재귀에 대해서 한층 더 이해할 수 있었다. 재귀에도 tail-recursion과 head-recursion이 나누어져 있고, 코드가 복잡해 질수록 더더욱 어느 타이밍에 재귀를 호출하는지가 매우 중요하다. <br>

이번 풀이에서는 앞서 소개한 문제해결 방식을 재귀적으로 반복하여 매번 왼쪽, 오른쪽, 중간 가로지르는 부분으로 분할해 최소 거리를 찾도록 하였다. 이렇게 head-recursion으로 호출한 후에 return 된 최소 거리를 저장하고 이후에 처리해야 할 코드를 수행한다. <br>

### 객체 생성

매번 이런 두 점과 같은 문제가 나올 때, 2차원 배열을 생성해서 수행했었다. 물론 그래도 아무 문제가 없고 이런 경우가 메모리나 속도 측면에서 더욱 효율적인 경우가 많다. 하지만 객체를 생성해서 데이터를 저장해야만 할 때가 있는데, 객체 생성을 해서 저장하는게 익숙하지 못해서 하지 못하는 경우가 많기 때문에 이번 문제에서 점의 좌표를 클래스 객체에 한번 담에 보았다. 생각보다 매우 간단하지만 첫 걸음이 어려워서 자주 사용하지 못했다고 생각한다. 

```java
class Point {
  int x;
  int y;
  
  public Point(int x, int y){
    this.x = x;
    this.y = y;
  }
}
```

## 전체 코드

다음과 같은 구성을 가지고 있다. 

1. 점들 사이의 최소값을 분할정복으로 찾는 minDistance()
2. 특정 기준 이하의 점들 사이의 최소값을 brute-force로 찾는 searchMin()
3. 점들 사이의 거리를 계산하는 distance()
4. xComparator와 yComparator
5. Point 클래스 객체


```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.StringTokenizer;
import java.util.Arrays;
import java.util.Collections;

class Main {
  static Point[] value;
  public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    StringTokenizer st = null;
    
    int n = Integer.parseInt(br.readLine());
    value = new Point[n];
    
    for(int i=0;i<n;i++){
      st = new StringTokenizer(br.readLine());
      int x = Integer.parseInt(st.nextToken());
      int y = Integer.parseInt(st.nextToken());
      value[i] = new Point(x, y);
    }
    
    Arrays.sort(value, new xComparator());
    
    int answer = minDistance(0, n-1);
    System.out.println(answer);
    br.close();
    
    return;
  }
  
  public static int minDistance(int begin, int end){
    int size = end-begin+1;
    if(size<=3) {
      return searchMin(begin, end);
    }
    
    int mid = (begin+end)/2;
    int d1 = minDistance(begin, mid);
    int d2 = minDistance(mid+1, end);
    
    int result = Math.min(d1, d2);
    
    ArrayList<Point> mid_list = new ArrayList<>();
    
    for(int i=mid-1;i>=begin;i--) {
      int xDist = value[mid].x - value[i].x;
      if((xDist*xDist) < result) {
        mid_list.add(value[i]);
      } else
        break;
    }
    
    for(int i=mid+1;i<=end;i++) {
      int xDist = value[mid].x - value[i].x;
      if((xDist*xDist) < result) {
        mid_list.add(value[i]);
      } else
        break;
    }
    
    Collections.sort(mid_list, new yComparator());
    int mlist_size = mid_list.size();
    
    for(int i=0;i<mlist_size-1;i++){
      for(int j=0;j<mlist_size;j++){
        int yDist = mid_list.get(i).y - mid_list.get(j).y;
        if(yDist*yDist < result) {
          int dist = distance(mid_list.get(i), mid_list.get(j));
          if(dist < result)
            result = dist;
        } else
          break;
      }
    }
    return result;
  }
  
  public static int distance(Point a, Point b){
    return(a.x-b.x)*(a.x-b.x) + (a.y-b.y)*(a.y-b.y);
  }
}
  
class xComparator implements Comparator<Point> {
  public int compare(Point p1, Point p2){
    return p1.x - p2.x;
  }
}

class yComparator implements Comparator<Point> {
  public int compare(Point p1, Point p2) {
    return p1.y - p2.y;
  }
}

class Point {
  int x;
  int y;
  
  public Point(int x, int y){
    this.x = x;
    this.y = y;
  }
}
```

```toc
```