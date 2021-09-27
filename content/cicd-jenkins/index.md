---
emoji: 🖥
title: "젠킨스를 활용한 CI/CD 적용기"
date: '2021-05-19 23:00:00'
author: 코다
tags: 웹 운영
categories: 웹 운영
---

## 자바 + 스프링 MVC 프로젝트 배포과정 (별도 인스턴스 활용)
- 이번에 몇몇 크루들과 미션을 진행하면서 웹을 처음으로 호스팅 해보았다. 웹을 배포 할 때 더욱 편리하다는 DevOps의 꽃 ci/cd를 학습해보기 위해서 6명이 모여서 한번 적용해보았다. 적용하면서 밟은 단계들을 기록해둔다. 
- 아래와 같이 그대로 적용하다가 본 프로젝트에 맞게 어느정도 커스텀하여 다르게 설정한 것도 있다. 특히 버전같은 것들은 좀 outdated 된 정보일 수 있다. 
- 추후에 진행할 팀 프로젝트에 큰 도움이 될 것 같다. 

### docker 설치

```java
sudo apt-get update && \
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
sudo apt-key fingerprint 0EBFCD88 && \
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
sudo apt-get update && \
sudo apt-get install -y docker-ce && \
sudo usermod -aG docker ubuntu && \
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
sudo chmod +x /usr/local/bin/docker-compose && \
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### EC2에서 Jenkins key 받기 및 적용

```java
$ sudo wget -q -O - [https://pkg.jenkins.io/debian/jenkins.io.key](https://pkg.jenkins.io/debian/jenkins.io.key) | sudo apt-key add -

$ sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'

$ sudo apt update

$ sudo apt install jenkins

//실행중 확인
$ systemctl status jenkins
```

### Jenkins 포트 번호 변경

젠킨스는 내부적으로 톰캣 서버를 이용하므로 기본포트 8080을 이용한다. 대부분의 스프링 프로젝트도 8080 톰캣 포트를 이용하기 때문에 젠킨스의 포트번호를 변경해야한다. 

- Jenkins 홈 디렉토리

    ```java
    $ cd /var/lib/jenkins
    ```

- Jenkins 기본 설정파일 & 로그 파일

    ```java
    //기본 설정 파일 
    //여기서 포트번호 변경
    $ cd /etc/default/jenkins

    //로그파일
    $ cd /var/log/jenkins/jenkins.log
    ```

- 포트 변경 후 재시작

    ```java
    $ sudo systemctl restart jenkins
    ```

### Jenkins 접속 및 설정

- Jenkins 접속

    ```java
    서버IP:변경포트번호
    ```

- 암호 가져오기

    ```java
    $ sudo docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
    ```

    - 해당 암호를 접속한 Jenkins에 입력
- Install suggested plugins 로 추천 플러그인 설치
- 어드민 계정 생성
- 플러그인 설치
    - `Deploy to Container` - 배포 후 빌드 파일 container에 배치
    - `Public Over SSH` - ssh로 빌드파일 전송

### Jenkins Key 생성

```java
//유저 변경 & 홈 디렉토리로 이동 
$ sudo su jenkins && cd 

//키 생성
$ ssh-keygen -t rsa 
```

→ 배포용 어플리케이션(톰캣?)의 키도 생성해야함 

### SSH KEY 교환

- Jenkins 인스턴스에서 유저 변경후 public ssh key 확인

    ```bash
    $ sudo su jenkins
    $ cd
    $ .ssh/id_rsa.pub
    ```

- Jenkins의 public ssh key 를 tomcat (배포서버) 에 저장

    ```bash
    //배포 서버에 키 복사
    $ vi .ssh/authorized_keys
    ```

### Jenkins 시스템 설정

- Publish over SSH → Path to key : jekins private ssh key 경로 기입
- SSH Servers → Application Server 주소, user name, remote directory 경로 기입

### Global Tool 설정

- JDK 경로 기입

### Jenkins 프로젝트 생성 → Github 연동

- Jenkins 에 Github 프로젝트 추가하고 Credentials 추가
- Jenkins에서 배포 버튼을 누르면 빌드, 배포가 이루어짐

## Docker + Jenkins 배포 과정

### Docker 에서 Jenkins 이미지 받기 및 컨테이너 실행

```bash
//이미지 받기
$ sudo docker pull jenkins/jenkins:버전

//이미지 확인
$ sudo docker images

//컨테이너 실행
$ sudo docker run -d -p 8181:8080 -v /jenkins:/var/jenkins_home --name jm_jenkins -u root jenkins/jenkins:버전
```

- -d : 도커 백그라운드 모드
- -p : 호스트와 컨테이너의 포트 연결 (포워딩)
- -v : 호스트와 컨테이너 디렉토리 연결 (마운트)
- —name : 컨테이너 이름 설정
- -u : 사용자 지정

### IP와 포트로 Jenkins 브라우저 접속 및 admin 생성

- 비밀번호 확인

    ```bash
    $ sudo docker logs jenkins
    ```

### 플러그인 설치

- 위와 동일

### Github 연결

- Jenkins 브라우저에서 새로운 Item → Freestyle project
- 레포지토리 URL 입력
- Credentials 추가
    - Kind → username with password (테스트용)
        - 계정 이름 및 비밀번호 입력
    - Kind → 실재로는 SSH 연동을 많이 사용한다. 해당 내용은 아래에 기입.
- Branch Specifier 추가
    - ex. `*/master`
- Builder Trigger → Github hook trigger
- Build → execute shell
    - push 시 실행하도록 등록
    - shell 스크립트 작성 필요 *

### Jenkins Github SSH 연동

- 현재 사용자 확인

    ```bash
    $ ps aux | grep jenkins
    ```

- 사용자 전환

    ```bash
    //jnkins가 서버에 등록된 사용자가 아닌 경우가 많으므로 su -u 로 전환이 안될 수 있는 것 참고 
    $ sudo -u jenkins /bin/bash
    ```

- .ssh 디렉토리 생성

    ```bash
    $ mkdir /var/lib/jenkins/.ssh
    $ cd /var/lib/jenkins/.ssh
    ```

- ssh 키 생성

    ```bash
    $ ssh-keygen -t rsa -f /var/lib/jenkins/.ssh/github_ansible-in-action

    //비밀키, 공개키 생성 확인 
    $ ls -al 
    ```

- Github 설정 - 공개키 등록
    - Settings → Deploy keys → add deploy key → 공개키 복붙

        ```bash
        $ cat /var/lib/jenkins/.ssh/github_ansible-in-action.pub
        ```

- Jenkins 브라우저에 비밀키 등록
    - Credentials → System → Global credentials → Add credentials → 비밀키 복붙

        ```bash
        $ cat /var/lib/jenkins/.ssh/github_ansible-in-action
        ```

    - Kind → SSH Username with private key

        Username → github_ansible-in-action(Job에서 보여줄 인증키 이름이므로 원하는 것 설정)

### Github 설정

- Setting → Integration & Services → Add Service → Jenkins hook url → Jenkins 주소 입력
    - 주소 뒤에 `/github-webhook/` 추가

⇒ 슬랙 연동도 가능

<br>
<br>

**[참고링크]**

- [https://jojoldu.tistory.com/139](https://jojoldu.tistory.com/139)
- [https://jojoldu.tistory.com/442](https://jojoldu.tistory.com/442)


```toc
```