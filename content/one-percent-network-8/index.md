---
emoji: ⚡️
title: 성공과 실패를 결정하는 1%의 네트워크 원리_8
date: '2021-10-06 08:00:00'
author: 코다
tags: 네트워크 책 
categories: 네트워크 책 
---

> 다음은 *성공과 실패를 결정하는 1%의 네트워크 원리* 를 읽고 정리한 내용입니다. 본 글은 CH5. 서버측의 LAN에는 무엇이 있는가?_방화벽과 캐시 서버의 탐험입니다 🙌

<br>
<br>

## 🛺 [Story4] 캐시 서버를 이용한 서버의 부하 분산

### 1. 캐시 서버의 이용

- 프록시 구조를 사용하여 데이터를 캐시에 저장한다.
- 프록시는 웹 서버와 클라이언트 사이에 들어가서 웹 서버에 대한 액세스 동작을 중개한다.
    - 중개하는 과정에서 웹 서버에서 받은 데이터를 저장해두고 가능하면 해당 데이터를 대신하여 응답한다.
    - 웹 서버가 처리해야할 일을 실행하기 위해서 오랜 시간이 걸리는 반면 캐시 서버는 받은 데이터를 곧바로 송신만 하면 되기 때문에 매우 빠르다.
- 데이터가 자주 바뀌는 부분은 캐시 서버를 활용하기 어렵다. 하지만 캐시 서버에서 처리할 수 있는 얼마를 담당하면 웹 서버에 가는 부하도 줄어들어 처리속도도 향상된다.

### 2. 캐시 서버는 갱신일로 콘텐츠를 관리한다

- 캐시 서버가 동작할 때 캐시 서버를 웹 서버 대신 DNS에 등록한다. 따라서 요청이 오면 캐시 서버가 대신해서 데이터를 받는다.
    - 메세지를 받을 때 웹 서버의 수신 동작과 동일한 절차를 거쳐서 받는다.
    - 패킷을 만들고, 접속 동작을 실행하고 요청 메세지를 받는다.
- 이후 해당 요청에 대한 데이터가 저장되어 있는지 조사한다.
- 저장된 데이터가 없는 경우
    - `Via` 라는 필드 값을 헤더에 추가하여 캐시 서버를 경유했다는 것을 나타낸다.
        - 중요한 값은 아니며 캐시 서버의 설정에 따라서 추가되지 않는 경우도 있다.
    - 만일 여러대의 서버가 캐시 서버에 연결이 되어 있다면 요청의 URI에 따라서 웹 서버로 요청을 전송한다.
        - 이때 클라이언트는 캐시 서버로 변경된다.
    - 웹 서버에서 캐시 서버로 응답을 보내고 캐시 서버는 `Via` 헤더를 추가하여 클라이언트에게 응답을 한다.
    - 그리고 응답에 대한 메시지를 캐시 서버에 저장하고 저장한 일시를 기록한다.
- 데이터가 저장되어 있는 경우
    - 만일 캐시 데이터가 저장되어 있다면 해당 데이터가 변경 되지는 않았는지 확인하는 `If-Modified-Since` 헤더를 덧붙여서 웹 서버에 전송한다.
        - 만일 데이터가 없었다면 위 헤더는 추가되지 않는다.
    - 이때 웹 서버는 변경이 없다면 `304 Not Modified` 상태코드를 응답한다. 변경이 있다면 데이터가 없던 것과 마찬가지로 동작한다.

### 3. 프록시의 원점은 포워드 프록시이다

- 클라이언트측에 캐시 서버를 두는 경우이다. → 포워드 프록시
- 웹 서버에 대한 캐시 서버와 동일하게 동작하지만 추가로 방화벽을 실현하는 목적이 있었다.
- 방화벽은 인터넷에서의 부정침입을 막는 것이기 때문에 프록시에서 요청 메세지를 받아 인터넷으로 필요한 것을 통과시키는 역할을 한다.
- 프록시의 캐시를 이용하면 사내 LAN에서 더 빨리 데이터를 얻을 수 있다.
- 프록시를 사용하면 요청의 내용을 조사하기 때문에 더 자세한 조건을 설정해서 특정 사이나에 대한 액세스를 금지하는 등의 제한을 걸 수 있다.

#### 포워드 프록시 사용시 데이터 송신

- 요청 메세지의 URL과 관계없이 모든 요청을 포워드 프록시에 우선 송신한다.
- 요청 메세지의 내용도 변경된다.
    - 본래 웹 서버의 이름을 제외하고 URI에 데이터 경로를 적었는데, 포워드 프록시를 사용하는 경우 이름까지 그대로 요청 URL에 기록한다.
- URL에 적힌 그대로가 전송 대상이므로 서버측 캐시 서버와 같이 정해진 서버로 전송하는 것이 아니라 모든 서버에 전송할 수 있다.

### 4. 포워드 프록시를 개량한 리버스 프록시

- 포워드 프록시는 브라우저의 설정이 필요해 장애의 원인이 되기도 한다.
- 따라서 요청 메세지에 전체 URL이 아닌 URI 에 쓰여있는 디렉토리를 웹 서버에 대응시켜 전송할 수 있도록 했다 → 서버측에 설치하는 캐시 서버에서 채택한 방법으로 **리버스 프록시**라고 한다.

### 5. 트랜스패어런트 프록시

- 캐시 서버에서 전송 대상을 판단하는 방법이다.
    - IP 헤더의 수신처 IP 주소로 액세스 대상 웹 서버를 찾는 방법을 **트랜스패어런트 프록시**라고 한다.
- 포워드 프록시에서 처럼 브라우저에 설정할 필요도 없고 리버스 프록시처럼 전송 대상을 리버스 프록시로 설정하고 DNS에 등록할 필요도 없다.
    - 만일 트랜스패어런트 프록시에 DNS에 등록된다면 수신처 IP가 해당 프록시가 되므로 수신처 IP를 조사해서 패킷을 중개하는 구조를 사용할 수 있다.
- 그래서 트랜스 패어런트 프록시를 브라우저에서 웹 서버로 요청 메세지가 흘러가는 길목에 설치하거나 한 길로 수렴하는 네트워크의 길목에 설치하여 사용한다. (길마다 프록시를 설치해야할 수도 있다)

<br>

## 🛺 [Story5] 콘텐츠 배포 서비스

### 1. CDN을 이용한 부하 분산

- 서버 측 캐시와 클라이언트 측 캐시의 부하 경감 효과가 각각 다르다.
    - 서버 측 캐시는 웹 서버에 들어오는 요청에 대한 부하를 분산시킨다.
    - 클라이언트 측 캐시는 인터넷에 들어오는 패킷 수를 줄여 인터넷 트래픽을 억제한다.
- 인터넷 트래픽을 억제하기 위해서는 (특히나 대용량 이미지나 영상 데이터에 대해) 클라이언트 측에 캐시 서버를 두는 것이 더 좋다. 하지만 그것은 웹 서버 개발자가 제어할 수 있는 부분이 아니다. (브라우저를 통한 설정이 필요하기 때문이다.)
- 따라서 해결책으로 프로바이더와 계약하여 웹 서버 개발자가 제어할 수 있는 클라이언트 가까이 있는 캐시 서버를 이용하는 것이다. → CDN

<p align="center"><img width="70%" src="https://user-images.githubusercontent.com/63405904/139086526-535450f9-f459-4d2d-bc6c-748046ff5174.png"><br>출처: 상위 1% 네트워크</p>

- 모든 프로바이더에 캐시 서버를 설치하는 것은 어려우니 우선 중요한 프로바이더마다 캐시 서버를 설치한다.
- 서버 운영자가 직접 설치하고 프로바이더와 계약하는 것이 어려우므로 그것을 대신 하고 캐시 서버를 대출하는 CDN 서비스가 등장했다.
    - CDN 서버는 여러 웹 서버의 데이터를 캐싱할 수 있으므로 여러 웹 서버 개발자들이 공동으로 이용하여 이용 비용을 절감할 수 있다.

### 2. 가장 가까운 캐시 서버의 관점

- CDN을 사용하기 위해서는 클라이언트가 가장 가까운 캐시 서버를 찾을 수 있어야한다.

#### 최초 방법

- DNS 서버가 IP주소를 응답할 때 가장 가까운 캐시 서버의 IP 주소를 응답하도록 설정한다.
    - DNS 서버에서 복수의 IP가 등록된 경우 RR로 응답하는 것을 변경한다.
    - 응답할 때 RR 방식이 아니라 클라이언트와 캐시 서버의 거리를 판단하여 가장 가까운 캐시 서버 IP를 반환하도록 한다.
- 가장 가까운 거리를 측정하는 방법
    - 캐시 서버의 설치 장소에 있는 라우터에서 경로 정보를 모은다. (캐시 서버 갯수만큼의 경로표가 모인다)
    - 웹 서버 측 DNS 서버에서 해당 경로표를 입수하여 클라이언트의 DNS 요청 패킷의 송신처 IP주소와의 경로 및 거리를 측정한다.
        - 이때 클라이언트 측 DNS 서버와의 거리를 측정하기 때문에 대략적인 거리이다.
        - 인터넷 경로 정보에 지나는 프로바이더와 대략적인 거리 정보가 있다.
    - 클라이언트 DNS 서버와 가장 가까운 캐시의 라우터를 알 수 있다.

### 3. 리피터용 서버로 액세스 대상을 분배한다

#### 두번째 방법

- 리다이렉트를 나타내는 `Location` 필드를 사용하여 액세스 대상을 가장 가까운 캐시 서버로 돌리는 방법이다.
- 먼저 DNS 서버에서 웹 서버의 IP주소를 회답하고 클라이언트가 해당 IP 주소(리다이렉트용 서버)로 요청을 보낸다. 리다이렉트용 서버는 경로표를 가지고 있어 가장 가까운 캐시 서버로 리다이렉트 하도록 `Location` 필드를 설정하여 응답한다.
- HTTP 요청이 많아지므로 어느정도 오버헤드가 있다. 하지만 클라이언트 DNS 서버와의 거리가 아닌 클라이언트 간의 거리를 조사하므로 더 정확하다.
- 더 정확하게 하기 위해 최적의 캐시 서버에 액세스하는 스크립트 프로그램을 내장한 페이지를 반송할 수도 있다.

### 4. 캐시 내용의 갱신 방법에서 성능의 차이가 난다

- 캐시서버를 이용할 때 갱신 내용 유무를 확인하느라 네트워크가 혼잡해질 수도 있다.
- 이 점을 개선하기 위해 확인을 하지 않고 데이터가 업데이트 된다면 즉시 갱신할 수 있다.
    - CDN 캐시 서버에는 이러한 기능이 내장되어 있다.
- 캐시에는 변하지 않는 부분만 캐싱하는 것이 효율적이다.

```toc
```