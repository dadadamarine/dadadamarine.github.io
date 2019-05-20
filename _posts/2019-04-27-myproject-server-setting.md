---
title: "[Spring/Project] 서버환경 세팅"
date: 2019-04-27 02:25:28 -0400
categories: Java/Spring Notification/project
---



# 개요

서버환경 : Ubuntu (bionic 18.04) 



# Docker 설치

**Docker CE  vs Docker EE**

전자는 개인이나 소규모 팀이 컨테이너 기반의 앱들을 실험할 수 있는 버젼으로 무료입니다.

후자는 엔터프라이즈급의 개발을 하는 용도로 설계된 버젼으로 유료입니다.

이 글에선 CE(Community Edition)을 사용합니다.



**도커를 설치하는 3가지 방법**

도커 CE를 설치하는 방법은 여러가지가 있고, 각 블로그마다 정리된 방법이 상이합니다. 



[Docker 문서](<https://docs.docker.com/install/linux/docker-ce/ubuntu/>)에 따르면 세가지 방식이 있습니다.

>- **도커 저장소를 사용하는 방법. 설치와 업그레이드가 용이한 방법으로 추천하는 방법이다.**
>- DEB (Ubuntu 파일 확장자, debian package)를 다운받아 설치와 업그레이드를 수동으로 관리하는 방법. 이 방법은 인터넷접근이 되지않는 air-gapped system[^1]에 도커를 설치할때 유용하다.
>- 일부 유저는 테스팅과 개발환경에서 자동화된 [스크립트](<https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-convenience-script>)를 사용하여 도커를 설치합니다.

 

 **저장소를 사용해 설치하기**

Docker CE를 설치하기전에 도커 저장소를 설정해야 합니다. 그 후에 설정한 저장소로부터 도커를 업데이트 할 수 있습니다.



**저장소 설정하기**

0. apt 패키지를 update합니다.

```bash
$ sudo apt-get update
```

   

1. apt가 HTTPS를 통해 저장소를 사용할 수 있도록 패키지를 설치합니다.

```bash
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

 

2. 도커의 공식 GPG key를 추가합니다. 

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

fingerprint로 key를 확인합니다.

```bash
$ sudo apt-key fingerprint 0EBFCD88

pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

`9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`가 나와야 합니다. 



3. 안정적인 저장소 사용을 위해 다음 명령을 실행합니다.

각 컴퓨터의 architecture에 맞춰 명령어를 실행하시면 됩니다.

우분투의 architecture는 `uname -m`명령어로 확인 가능합니다.

```bash
 sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```



**Docker CE 설치**

1. 최신버전의 Docker CE와 Containerd[^2]

```bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io 
```

 


> 만약 도커 저장소가 여러개가 있을 때,  `apt-get install` 또는 `apt-get update`명령어에 버젼을 명시하지 않으면 가장 최신의 version을 설치합니다. 
> 특정 버전의 Docker CE를 설치하는 방법은 [Docker document]() 에 자세히 나와있습니다. 여기서는 생략하도록 하겠습니다.



# 설치된 도커 사용하기

**Docker CE 버전확인**

```bash
$ docker version
Client:
 Version:           18.09.5
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        e8ff056
 Built:             Thu Apr 11 04:43:57 2019
 OS/Arch:           linux/amd64
 Experimental:      false
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/version: dial unix /var/run/docker.sock: connect: permission denied
```



**사용자 도커 그룹에 추가하기**

도커는 설치시에 docker 그룹을 자동으로 추가합니다.

```bash
$ cat /etc/group
...
docker:x:999:
```

현재는 그룹에 등록된 사용자가 없습니다.



gpasswd 또는 usermod 명령어로 그룹에 **현재 사용자**를 추가합니다.

```bash
sudo gpasswd -a $USER docker
```



```bash
$ cat /etc/group
...
docker:x:999:pushstone
```

사용자가 추가된 것을 확인 할 수 있습니다.



이제 도커를 재시작 하면 sudo명령어 없이 도커를 사용할 수 있습니다.

```bash
`$ sudo service docker restart`
```



**sudo 권한 에러**

```bash
$ docker run hello-world
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/create: dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.
```



계정을 아예 로그아웃 하고 다시 로그인하면 정상작동합니다.

저의 경우 ssh connection을 종료시키고 다시 연결했습니다.





# Docker mongo 설치



[로컬환경 설정 및 배포환경 설정](<https://dadadamarine.github.io/java/spring/project/spring-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/>) 글에 따라 mongo이미지를 설치하고 컨테이너를 실행시킵니다.

이미지를 다운받고, 백그라운드에서 컨테이너 실행후, 접속합니다.

```bash
docker pull mongo
docker run --name mongo -p 27017:27017 -d mongo
docker exec -it mongo /bin/bash
```



# Spring Boot 환경설정

## 보안 그룹 설정

현재 docker는 docker라는 그룹으로 관리되고 있습니다.



어플리케이션 사용을 위한 `springNotification`그룹과 git 사용을 위한 `git`그룹을 생성하겠습니다.

```bash
sudo groupadd springNotification
sudo groupadd git
```



## openjdk 8 설치

현재 서버엔 openJDK 8 버전이 설치되어있습니다.

```bash
$ java -version
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-8u191-b12-2ubuntu0.18.04.1-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
```

여러개의 자바 버전이 설치되어 있다면 

```bash
sudo /usr/sbin/alternatives --config java
```

명령어를 통해 버전을 변경 할 수 있습니다.



## git 설치

**PPA[^3] (Personal Package Archive) 사용**

Ubuntu 설치 후 아래의 명령어를 통해 PPA를 사용가능하게 만든다.

```bash
sudo apt-get install python-software-properties
sudo apt-get install software-properties-common
```



**Git repository 등록&업데이트 후 설치**

```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git-core
```



**설치 확인**

```bash
$ git version
git version 2.21.0
```



**SSH 공개키 만들기**

해당 내용은 http프로토콜을 사용하신다면 넘기셔도 무관합니다.

참조 : [Git 프로토콜](<https://git-scm.com/book/ko/v1/Git-%EC%84%9C%EB%B2%84-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C>)



1. 사용자의 SSH 키들은 기본적으로 사용자의 ~/.ssh 디렉토리에 저장합니다. 

```bash
# git 사용자로 로그인한 상태

cd ~
mkdir -p .ssh 
cd .ssh 
```



2. ssh-keygen 프로그램으로 키 생성

명령어 입력시 경로를 입력하고, 패스워드를 두번 입력합니다.
(암호를 비울시 키 사용시에 암호를 묻지 않습니다.)

```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/pushstone/.ssh/id_rsa): id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
```



3. 확인

 .pub 파일이 공개키 입니다. 이제 .pub파일의 내용을 복사해서 Git 서버관리자에게 보내면 됩니다.

```bash
$ ls
id_rsa  id_rsa.pub  known_hosts
```



4. ssh-agent에 추가

ssh-agent를 백그라운드로 시작한 후에 방금 생성한 id_rsa를 추가합니다.

```bash
eval "$(ssh-agent -s)"
$ ssh-add ~/.ssh/id_rsa
```



5. 공개키 등록

git hub 내 계정 profile 아래 Settings에 들어가 ssh를 추가합니다.

```bash
vi ./.ssh/id_rsa.pub
```

해당 파일로 들어가 내용을 복사해서 붙여넣으시면됩니다. 앞뒤의 내용 (ex. ssh-rsa )까지 포함해서 복사해줍니다.



6. 테스트

먼저 config설정을 합니다.

```bash
git config --global user.email "useremail@email.com"
```

```
ssh -T git@github.com

Hi dadadamarine! You've successfully authenticated, but GitHub does not provide shell access.
```

위와 같은 메세지가 뜨신다면 성공입니다.



# Tomcat

Spring boot는 내장 서블릿 컨테이너인 톰캣이 자동적으로 설정된다. 이는 ServletWebServerFactoryAutoConfiguration클래스가 담당한다.

또한 기존에 web.xml에서 설정했던 DispatcherServlet 관련 설정은 DispatcherServletAutoConfiguration클래스를 통해 처리한다.

[참조](<https://engkimbs.tistory.com/755>)



# 끝

다음 내용은 jenkins 서버에서 notification 서버로 자동 배포를 설정하겠습니다.

 



---

**참조**

- [[Docker] 도커 소개와 우분투에 Docker CE 설치하기](<https://www.bsidesoft.com/?p=7820>) - 도커 사용법 뿐 아니라 간단한 소개도 있는 글 

- [Ubuntu(우분투) 16.04 git 설치하기](https://firework-ham.tistory.com/1)

---

**주석**

[^1]:[HACKER LEXICON: WHAT IS AN AIR GAP?](<https://www.wired.com/2014/12/hacker-lexicon-air-gap/>)
[^2]: 도커의 컨테이너 런타임 포맷. [참조](<https://m.blog.naver.com/PostView.nhn?blogId=alice_k106&logNo=220972654815&proxyReferer=https%3A%2F%2Fwww.google.com%2F>)
[^3]:최신 버전의 패키지 다운로드를 위한 저장소등록 [참조](<https://webdir.tistory.com/197>)

