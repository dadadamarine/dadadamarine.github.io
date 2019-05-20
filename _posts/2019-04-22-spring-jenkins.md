---
title: "[Spring/Project] Jenkins 사용 무작정 따라하기"
date: 2019-04-22 02:25:28 -0400
categories: Java/Spring Notification/project
---



> 이글은 Jojoldu님의 글 [docker를 이용한 CI 구축 연습하기 (젠킨스, 슬랙)](https://jojoldu.tistory.com/139) 를 바탕으로 제작된 실습 후기입니다.



# 젠킨스 환경 설정

![image-20190422130113773](/assets/images/image-20190422130113773.png)

Docker conatiner에 Jenkins를 설치하고, 패스워드를 알아낸다.



그후 32769 포트에 접속해서 

![image-20190422130208869](/assets/images/image-20190422130208869.png)

패스워드를 입력하고 설치를 한다.

그럼 에러가 난다.



[관련 공지](<https://jenkins.io/blog/2018/12/10/the-official-Docker-image/>)를 확인하였다. 

```
jenkins/jenkins를 사용해야한다.

jenkins 와 jenkinsci/jenkins는 이미지 업데이트가 중단되었다.
```



docker kitematic보다는

```
docker pull jenkins/jenkins

docker images # 다운받은 이미지 확인

docker run -p 32769:8080 --name jenkins jenkins/jenkins
# 다운받은 이미지로 컨테이너를 생성한다. 컨테이너 이름은 jenkins, 로컬의 32769포트를 컨테이너의 8080포트와 연결
```

명령어를 사용하도록 하자.

> 도커 명령어는 [도커 무작정 따라하기: 도커가 처음인 사람도 60분이면 웹 서버를 올릴 수 있습니다!](<https://www.slideshare.net/pyrasis/docker-fordummies-44424016>) 를 참조하시면 됩니다.



![image-20190422135828825](/assets/images/image-20190422135828825.png)

이런 화면이 나오신다면 컨테이너가 잘 실행 된것입니다.



```
# 종료되어있는 컨테이너를 켜기
docker start jenkins

# 현재 실행중인 컨테이너 확인
docker ps
```

실행 결과

![image-20190422140737771](/assets/images/image-20190422140737771.png)





이제 아래와 같은 비밀번호 입력화면이 나오면

![image-20190422141237762](/assets/images/image-20190422141237762.png)



터미널을 위로 올려서 찾은 패스워드를 입력합니다.

![image-20190422141101540](/assets/images/image-20190422141101540.png)

![image-20190422142700698](/assets/images/image-20190422142700698.png)

이렇게 드디어 정상실행!





# 웹 훅 설정

## 중략

**젠킨스 기본설정**

**ngrok 설정**

![image-20190422145900529](/assets/images/image-20190422145900529.png)



## github 웹 훅 설정

[참조](<https://taetaetae.github.io/2018/02/08/github-web-hook-jenkins-job-excute/>)

### 1. CSRF 해제

![image-20190422145958787](/assets/images/image-20190422145958787.png)

맨 아래 보이는 Configure Global Security로 들어간다.

![image-20190422150704071](../../../../../Library/Application Support/typora-user-images/image-20190422150704071.png)



### 2. jenkins Job을 설정 

[참조](<https://taetaetae.github.io/2018/02/08/github-web-hook-jenkins-job-excute/>)



### 3. Github Webhook 설정

![image-20190422152537017](/assets/images/image-20190422152537017.png)





### 4. 결과

설정 이후 로컬에서 github 로 push를 날리면 자동으로 빌드를 진행한다.

![image-20190422152403926](/assets/images/image-20190422152403926.png)



빌드도 무사히 완료하였다.

![image-20190422152738868](/assets/images/image-20190422152738868.png)



# 슬랙 알림 설정

### 1. 슬랙에서 젠킨스 플러그인 설치

### 2. 젠킨스에서 Slack Notification 설치

### 3. 슬랙에서 나온 BaseUrl, Integration Token, Channel을 입력

![image-20190422154752680](/assets/images/image-20190422154752680.png)

> Integration Test 입력방법. 
>
> 1. Add 클릭
> 2. Kind를 Secret Text로 바꾼후 Secret칸에 슬랙에서 복사한 토큰을 입력



![image-20190422155210497](/assets/images/image-20190422155210497.png)

Test Connection을 눌러서 Success가 나온다면 통과!



### 4. 빌드 프로젝트에 빌드 후 조치 등록

![image-20190422155554937](/assets/images/image-20190422155554937.png)





### 5. 결과

![image-20190422160341982](/assets/images/image-20190422160341982.png)

성공!