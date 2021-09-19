---
emoji: 🚊
title: CDN 알아보기
date: '2021-09-02 23:00:00'
author: 코다
tags: network cdn infra 
categories: 인프라
---

## Intro
---
- 이번에 프로젝트를 진행하면서 보안상의 이유로 직접 S3 서버에 접근할 수 없었기 때문에 AWS에서 제공하는 CDN 서비스인 CloudFront를 통해서 이미지 등의 리소스에 접근해야 했다. (CDN서버의 본래 목적과는 다소 다른 이유로 사용했다.)
- CDN은 어떤 기술이며, 장점이 무엇이고, 어떻게 동작하는지에 대해서 알아본다. 
- CDN을 퉁해 누릴 수 있는 보안적인 이점은 무엇이며, 프로젝트를 진행하면서 CloudFront를 어떻게 활용했는지에 대해서 작성한다. 

<br>
<br>

## CDN이란 무엇인가?
---
CDN은 Content Delivery Network의 약자이다. 직역하자면 컨텐츠를 전달해주는 네트워크이다. CDN 컨텐츠를 전송하는 물리적인 서버가 지리적으로 여러곳에 상주하며 유저와 가까이 위치한 서버에서 요청한 컨텐츠를 고속으로 제공해준다. 
<br>

CDN이 제공하는 컨텐츠는 `HTML`, `javascript` 파일, `css`, 이미지, 동영상 등의 대부분의 인터넷 콘텐츠이다. CDN 서비스에 대해서 설명하는 예시에 항상 등장하는 어플리케이션은 넷플릭스이다. 전세계 곳곳에서 유저들이 넷플릭스 컨텐츠를 요청하면 가장 가까이 상주하고 있는 CDN 서버에서 넷플릭스 컨텐츠를 빠르게 유저에게 제공한다. 이외에 Facebook, Amazon 등도 사용 중이다. 
<br>

CDN을 자체로 웹을 호스팅 할 수는 없다. 다만 웹의 컨텐츠를 `캐싱`하여 호스팅하는 웹을 대신하여 전송해서 웹의 성능을 개선한다.
<br>
<br>

## CDN 장점
---
1. 캐싱으로 컨텐츠 고속 전송
    - 사용자와 물리적으로 가까운 CDN 서버에서 컨텐츠를 전송하므로 사용자의 입장에서 컨텐츠 로드 시간이 매우 단축된다. 또한 CDN 서버는 랜덤으로 배치되어 있는 것이 아니라, 전세계 트래픽이 많은 영역에 전략적으로 위치해 있다. 

    - 캐싱 과정 
        1. 사용자가 컨텐츠를 요청하면 가장 가까운 CDN 엣지 서버로 요청이 간다. 
        2. (최초 요청일 경우) CDN 엣지 서버에서 원본 서버로 요청을 보낸다. 
        3. 원본 서버가 해당 컨텐츠를 엣지 서버에 응답한다. 
        4. 엣지 서버가 사용자에게 컨텐츠를 응답한다. 
        <br>
        <p align="center"><img width="450" src="https://user-images.githubusercontent.com/63405904/130417611-508fcc23-79f2-4ae7-923b-5a0989a2cf54.png"><br>이미지 출처: https://www.cloudflare.com/ko-kr/learning/cdn/performance/</p>

    - 그 이후부터는 동일한 컨텐츠에 대한 요청이 있을 때 해당 컨텐츠가 동일한 CDN 엣지 서버에 요청을 보내고 원본 서버에 요청을 보낼 필요 없이 CDN 엣지 서버가 컨텐츠를 반환한다. 이때 속도가 굉장히 향상된다. 
        <br>
        <p align="center"><img width="450" src="https://user-images.githubusercontent.com/63405904/130417751-e2d97680-8446-401e-bdec-90b95560f87f.png"><br>이미지 출처: https://www.cloudflare.com/ko-kr/learning/cdn/performance/</p>

<br>

2. CDN의 failover
    - failover란? 서버가 갑자기 중단되어 서버에 요청을 보내던 트래픽에 대해 정상적인 응답을 하지 못하는 것을 방지하는 것
    - CDN은 요청을 보내던 origin server가 죽으면 정상적으로 응답을 할 수 있는 서버로 reroute 하여 사용자가 안정적으로 그 응답을 받을 수 있도록 한다. 
    <br>
    <p align="center"><img width="433" src="https://user-images.githubusercontent.com/63405904/130594395-77cf44f3-049c-4a4c-9be2-f59678f77b7c.png"><br>이미지 출처: https://www.cloudflare.com/ko-kr/learning/cdn/cdn-load-balance-reliability/</p>

<br>

3. 로드밸런싱 및 DDos 공격 완화
    - 로드 밸런서는 네트워크 트래픽을 여러 서버에 분산해서 성능을 개선하는 것이다. 
    - CDN은 GSLB(Global Server Load Balancing)으로 로드 밸런싱 기술을 제공한다. (DNS와 GSLB의 차이점에 대해서 학습해도 좋다.) [GSLB 참고](https://www.cloudflare.com/ko-kr/learning/cdn/glossary/global-server-load-balancing-gslb/)
    - CDN은 데이터센터의 로드 밸런싱으로 사용자의 요청을 가능한 서버에 분산해서 요청한다. (GSLB를 사용하기 때문에 요청 서버에 대한 헬스체크도 수행하여 안정적이다.)
    - 속도도 개선시키고, 트래픽도 감소시키므로 DDos 공격도 방지할 수 있다. 
        <p align="center"><img width="630" src="https://user-images.githubusercontent.com/63405904/130593424-74c410aa-b465-4fee-8dd1-e4213090064a.png"><br>이미지 출처: https://www.cloudflare.com/ko-kr/learning/cdn/cdn-load-balance-reliability/</p>

<br>
<br>

## 프로젝트에서 CDN 사용 목적
---
- 다음은 프로젝트의 인프라 구조이다. 
    <p align="center"><img width="605" src="https://user-images.githubusercontent.com/63405904/130720611-034be1a1-c358-4ce5-8fac-8383a87d19e0.png"></p>
- 일반적인지는 잘 모르겠지만, 현재 S3 버킷에 프론트 서버가 올라가있다. (그렇지 않더라도 이미지 및 동영상 리소스가 S3 버킷에 저장되어 있다.) 
- 프로젝트를 하는데 보안상의 이슈로 S3 버킷에 대한 접근을 전체공개할 수 없었고, Cloud Front를 통해서 우회하여 접근하도록 설계했다. 
- CDN의 본래 목적은 리소스를 캐싱하여 빠르게 로딩하는 것이지만 이번 프로젝트에서는 S3 버킷 사용 목적으로 설계했다. 
- 프론트 서버를 분산하거나, 진행중인 프로젝트(개발자 친화적 SNS)의 특성상 이미지 및 동영상 리소스가 굉장히 많아져서 S3 버킷이 추가되면 로드 밸런싱, CDN failover 등의 이점을 누릴 수 있을 것이라고 생각한다. 

<br>
<br>

**[참고자료]**
- https://www.cloudflare.com/ko-kr/learning/cdn/performance/
- https://www.cloudflare.com/ko-kr/learning/cdn/cdn-load-balance-reliability/

```toc
```