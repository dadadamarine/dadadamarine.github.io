---
title: "[Spring/Project] nginx로 무중단 배포 세팅하기"
date: 2019-04-28 02:25:28 -0400
categories: Java/Spring Notification/project
---



# 개요

개발환경 : ubuntu , spring boot, maven 



지금 **CI(Continuous Integration)** 환경 구축까지 완료하였다. 

이제 nginx를 사용하여한 **무중단 배포**를 구현하도록 하자.



무중단 배포에 관한 내용은 jojoldu님의 [Nginx를 활용한 무중단 배포 구축하기](https://jojoldu.tistory.com/267)글을 참조해가며 구축하겠습니다.



# 무중단 배포란?

기존 CI까지 구축한 내용은 배포하는 시간동안 새로운 Jar실행 전까지 기존 Jar을 종료시켜 놓기때문에 서비스가 되지 않습니다.

또한 이렇게 서버를 종료하고 배포하는 방식은 롤백하기도 어렵습니다.



무중단 배포를 구현하는 방법에는 여러가지가 있습니다. 

종류마다 구현방식도 약간씩 다릅니다.

이 글에서는 nginx를 이용하여 무중단 배포를 구현하겠습니다. 

nginx의 내용과 작동방식은 위의 김동욱님 블로그를 참고하시면 쉽게 이해하실 수 있습니다.



# 무중단 배포 구축하기

먼저 서버에 nginx를 설치합니다.

 

## Nginx 설치

직접 소스를 받아 컴파일 하는 방식도 있지만, 편의성을 위해 package를 통한 설치를 하겠습니다.

```bash
$ sudo apt-get update
$ sudo apt-get install nginx
```

 

제대로 설치되었는지 버전을 확인해 봅니다.

```bash
$ nginx -v
nginx version: nginx/1.14.0 (Ubuntu)
```



이제 Nginx를 실행합니다.

```bash
sudo service nginx start
```



잘 실행되었는지 확인합니다.

```bash
$ ps -ef | grep nginx
root      90247      1  0 05:50 ?        00:00:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data  90248  90247  0 05:50 ?        00:00:00 nginx: worker process
pushsto+  90417  89366  0 06:03 pts/0    00:00:00 grep --color=auto nginx
```



서버에 접속해보면 다음과 같은 화면이 보여집니다.

![image-20190428150505674](/Users/dadadamarine/Desktop/study/blog/dadadamarine.github.io/_posts/assets/images/image-20190428150505674.png)



## Nginx 리버스 프록시 설정

현재 Nginx가 배포한 스프링부트 프로젝트를 바라 볼 수 있도록 설정합니다.



**리버스 프록시**


리버스 프록시란?

가짜 서버에 request가 들어왔을때, 프록시 서버가 배후 서버(reverse server)로 데이터를 요청해서 가져오는 것.

기본적으로 nginx는 비동기 처리방식을 채택하고 있다. 



![ì¤í¬ë¦°ì·, 2017-07-03 20-49-56](http://i.imgur.com/yReDKjj.png)

<center>출처 : https://whatisthenext.tistory.com/123</center>



장점

Reverse 서버에 request에 대한 응답대기 프로세스를 생성하지 않아도 되는 장점이 생긴다. 

보안상의 이점 : 또한 외부 사용자로부터 실제 내부망에 있는 서버의 존재와 정보를 숨길 수 있다. 

로드밸런싱 : proxy서버가 내부 서버의 정보를 알고 관리하므로, 부하 여부에 따라 요청을 분배 할 수 있다.



**Nginx 환경설정 수정**



참조 : [설정파일의 요소에 대한 자세한 설명](<https://whatisthenext.tistory.com/123>) , [Nginx 가상 호스트에 대한 설명](<https://opentutorials.org/module/384/4529>) , [순방향 vs 역방향 프록시](<https://medium.com/sjk5766/nginx-reverse-proxy-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-e11e18fcf843>) , [Nginx 주요 설정](<https://sarc.io/index.php/nginx/61-nginx-nginx-conf>)




```bash
sudo vi /etc/nginx/nginx.conf
```

nginx 설정파일을 열어서 Virtual Host에 대한 설정을 해준다.

nginx는 nginx.conf 파일에서 `location` 지시어를 사용하여 요청을 배분한다.



conf 파일에 가상 호스트 설정(server{})을 추가합니다.

```bash
html -{
	...

    server {
            listen           80; 					# 기본 포트는 80
            server_name      localhost;
            root /home/pushstone/NotificaionServer  # document root 설정

            location / { # proxy_pass 설정. 특정 확장자 요청을 넘기는 설정
                    proxy_pass http://localhost:8080;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header Host $http_host;
            }
    }
    
   	...
}

#mail {
	...
}
```



Proxy_pass : 요청이 올 경우 http://localhost:8080로 요청을 전달합니다.

proxy_set_header XXX : 실제 요청 데이터를 header의 각 항목에 할당하는 역할

















------

참조

------

주석

