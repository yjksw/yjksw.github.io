---
emoji: ⚡️
title: 내가 또 보기 위한 TCP 혼잡제어
date: '2021-10-22 23:10:00'
author: 코다
tags: 네트워크 
categories: 네트워크
---

## 🌩 왜 혼잡제어가 필요할까? 

- 라우터에 패킷이 몰리면 패킷이 유실되고 패킷을 재전송 하면서 네트워크는 더 혼잡해진다.
- 송신측에서 이러한 문제를 해결하기 위해 전송속도를 줄이는 혼잡 제어를 사용한다.

<br>

## 🌩 AIMD

- Additive Increase, Mutiplicative Decrese
- 패킷을 하나씩 보내고 문제없이 도착하면 window 크기를 1개씩 증가한다.
- 패킷 전송에 실패하면 속도를 절반으로 줄인다.
- 이 경우 나중에 네트워크에 진입한 쪽이 처음에는 불리하지만 점점 동일한 평형상태가 되기 때문에 공정하다.
- 네트워크 혼잡을 미리 감지하지는 못하고 혼잡하면 대역폭을 줄인다.
    
<p align="center"><img width="70%" src="https://user-images.githubusercontent.com/63405904/139091747-767ffc1e-b9a7-4894-9445-17930ef59391.png"></p>

<br>

## 🌩 Slow Start

- AIMD는 처음 전송속도를 올리는 것이 너무 느리다는 단점이 있다.
- slow start는 처음에는 문제가 없다면 윈도우 사이즈를 지수함수꼴로 증가한다.
- 혼잡 현상이 발생하면 window사이즈를 1로 떨어뜨린다. 하지만 이때는 네트워크의 혼잡율을 어느정도 예상할 수 있다.
- 따라서 혼잡 현상이 발생했던 window size의 크기의 반까지 지수함수 꼴로 증가시키고 이후부터 1씩 완만하게 증기시킨다. (임계점에 다다르면 지수함수는 너무 급격하므로 1씩 윈도우를 증가시킨다)
  
<p align="center"><img width="70%" src="https://user-images.githubusercontent.com/63405904/139091821-279101d0-e464-4644-a803-3ea6a7c2a678.png"></p>

<p align="center"><img width="70%" src="https://user-images.githubusercontent.com/63405904/139091877-c00e5fdd-d52f-4280-9f76-3ba9587b225a.png"></p>

<br>

## 🌩 Fast Retransmit 빠른 재전송

- 패킷을 받는 쪽에서 패킷이 도착하지 않고 다음 패킷이 도착하더라도 ACK를 보내는데 이때 잘 도착한 패킷의 다음 패킷을 ACK로 보낸다.
- 그러면 중복된 ACK 패킷이 계속 도착하므로 이때는 문제가 된 순번의 패킷을 재전송 해줄수 있다.
- 중복된 패킷을 3개 받으면 재전송을 하는데, 혼잡임을 감지하고 window size를 반으로 줄이다.

<p align="center"><img width="70%" src="https://user-images.githubusercontent.com/63405904/139092293-9e27653f-f3aa-44fd-867c-c2327be7b57f.png"></p>

<br>

## 🌩 Fast Recovery 빠른 회복

- 혼잡한 상태가 되면 window를 1이 아닌 반으로 줄이고 선형증가시킨다.
- 혼잡상황을 한번 겪으면 순수 AIMD 방식으로 동작한다.

<br>

## 🌩 TCP Reno

- 위 기법들을 사용한 TCP 혼잡제어이다.
- 먼저 slow start로 시작하고 임계점을 넘어가면 1씩 설정하여 윈도우 사이즈를 높인다.
- 위의 3 ACK Duplicated와 타임아웃을 구분하여 각기 다른 방식을 취한다.
  - 3 ACK Duplicated라면 윈도우 사이즈를 반으로 줄이고 선형적으로 증가시킨다.
  - 타임아웃이 발생하면 윈도우 크기를 1로 줄이고 slow start를 진행한다.
        
<p align="center"><img width="70%" src="https://user-images.githubusercontent.com/63405904/139091995-e0f1f659-47c8-486a-b7c4-d9ddc7ee2c9a.png"></p>


<br>
<br>

**[참고자료]**
- [https://evan-moon.github.io/2019/11/26/tcp-congestion-control/](https://evan-moon.github.io/2019/11/26/tcp-congestion-control/)

```toc
```