---
title: "[Spring/Project] Jenkins : 배포 자동화"
date: 2019-04-28 02:25:28 -0400
categories: Java/Spring Notification/project
---



# 개요

현재 **서버 환경구축**과, Jenkins를 활용하여 **테스트, 빌드 자동화**까지 설정하였다.

이제 배포 자동화를 구현하여 로컬에서 github로 push한 내용이 **자동으로 배포** 서버로 적용되는 CI환경과, nginx를 이용한 **무중단 배포**를 구현하도록 하자.

# Jenkins 자동배포 설정

## 로컬 어플리케이션 설정

**pom.xml**

```
<groupId>com.pushstone</groupId>
<artifactId>DailyNotificationServer</artifactId>
<version>0.0.2-SNAPSHOT</version>
<name>DailyNotificationServer</name>
<packaging>jar</packaging>
<description>Demo project for Spring Boot</description>

...

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>compile</scope>
</dependency>
```

dependency의 scope[^1]는 기본값이 compile이다. 즉 spring-boot-starter-web이 설정되어 있다면 굳이 tomcat의존성을 나눠서 추가할 필요가 없다.

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

이렇게 사용해도 똑같은 의존성이 설정된다.

> **scope는 compile(default), provided, runtime, test 4가지가 있다.**
>
> compile : compile이 설정된 의존성은 모든 빌드 작업에서 사용 가능하다.
>
> provided : 이 의존성은 runtime시에 JDK나 컨테이너에서 제공되어야 한다.
>
> runtime : 이 의존성은 런타임 시에는 필요하지만, compile시점에는 필요하지 않다.
>
> test : 이 의존성은 runtime시에는 필요하지 않고, 테스트 목적으로만 사용된다.

패키징 방식을 jar로 설정한다.

```
<packaging>jar</packaging> 
```

이 또한 설정하지 않을경우 default로 maven은 jar로 패키징합니다.

**application.properties**

```
server.port=80
server.compression.enabled=true
```

## Jenkins 설정

deploy 플러그인은 여러가지가 있다.

war 배포시에는 Deploy to container도 이용가능하지만, jar 파일 배포를 위해

- [Publish Over SSH Plugin](https://wiki.jenkins.io/display/JENKINS/Publish+Over+SSH+Plugin)
- [SSH2Easy Plugin](https://wiki.jenkins.io/display/JENKINS/SSH2Easy+Plugin)

두가지 [젠킨스(Jenkins)에서 원격(Remote)으로 배포하기](https://yookeun.github.io/tools/2018/04/14/jenkins-remote/)글의 추천대로 `Publish Over SSH Plugin` 를 이용하겠다.

1. Jenkins 플로그인 - 설치가능 에서 `publish over ssh`를 검색하여 다운로드 (재시작 후 설치 클릭)
2. 시스템 설정에서 `Publish over SSH`칸 작성

2번까지는 [참고](https://yookeun.github.io/tools/2018/04/14/jenkins-remote/) 블로그를 참조하시면 쉽게 따라하실 수 있습니다.

1. 해당 item의 구성으로 가서 `빌드환경`작성

[![image-20190428001204636](https://github.com/dadadamarine/dadadamarine.github.io/raw/master/assets/images/image-20190428001204636.png)](https://github.com/dadadamarine/dadadamarine.github.io/blob/master/assets/images/image-20190428001204636.png)

> Source files : jar파일의 위치. maven의 경우 이 파일은는 target 폴더 하위에 위치합니다.
>
> Remove prefix : 소스 파일 앞부분 경로. 여기서는 target이다.
>
> Remote directory : 업로드할 경로. 시스템 설정에서 지정한 Remote Directory를 포함한다. 즉 /usr이라는 경로를 지정했었으면 /usr 하위의 폴더로 경로가 이어진다.
>
> Exec command : 실행할 명령어를 적는다.

name 부분의 고급설정에 가서 해당 부분을 체크하고 서버의 계정/패스워드를 입력한다.

[![image-20190428001527001](https://github.com/dadadamarine/dadadamarine.github.io/raw/master/assets/images/image-20190428001527001.png)](https://github.com/dadadamarine/dadadamarine.github.io/blob/master/assets/images/image-20190428001527001.png)

## 에러수정

MongoDB로 환경설정을 한 이후여서 테스트 에러가 납니다.

```
Caused by: com.mongodb.MongoTimeoutException: Timed out after 30000 ms while waiting to connect. Client view of cluster state is {type=UNKNOWN, servers=[{address=localhost:27017, type=UNKNOWN, state=CONNECTING, exception={com.mongodb.MongoSocketOpenException: Exception opening socket}, caused by {java.net.ConnectException: Connection refused (Connection refused)}}]
```

현재 Jenkins서버에는 MongoDB가 설치되어 있지않기때문에 연결되지 않습니다.

따라서 테스트시에 h2 DB를 사용하도록하여 젠킨스 서버에서 테스트 코드가 돌아갈 수 있게 합니다.

**외부설정**

어플리케이션 에서 사용하는 설정값들을 어플리케이션 안팎에 정의하는 것. 또한 이 properties간에는 우선순위가 존재한다.

테스트에서 이 값을 바꾸는(오버라이딩) 방법

[![image-20190428020046250](https://github.com/dadadamarine/dadadamarine.github.io/raw/master/assets/images/image-20190428020046250.png)](https://github.com/dadadamarine/dadadamarine.github.io/blob/master/assets/images/image-20190428020046250.png)

1. test/resources에 application.properties를 만듭니다.

2. Project Structure(command + ;)의 module에서 source -> test -> resources 폴더를 누르고, 중간의 test resources버튼을 눌러 등록합니다.

   빌드시에 src밑에 파일들을 classpath에 먼저 넣고 넣고. 테스트코드를 컴파일하고 classpath에 넣음. 그때 application.properties파일이 덮어씌워진다.

   하지만 이 방법은 test의 properties파일이 덮어쓴 값때문에 원래 소스의 properties 값을 참조하는 곳에서 값이 없다는 에러가 발생할 수 있다.

3. 다른방법 : 파일명을 상이하게 저장하여(필자는 application-test.properties로 설정) test의 resource에 넣어두고

```
@TestPropertySource(locations = "classpath:/application-test.properties")
```

어노테이션을 Test클래스 위에 설정하여 통해 그 properties를 읽어 추가로 사용한다. ( 이때는 같은 property key가 있을때만 오버라이딩함 & 이렇게 설정한 값들은 프로퍼티 우선순위가 2번째로 높음)

> 사실상 embedded mongo는 properties를 따로 설정하지 않아도 테스트 시에 embedded mongo로 테스트를 진행한다.

**embeded mongo**

```
<dependency>
    <groupId>de.flapdoodle.embed</groupId>
    <artifactId>de.flapdoodle.embed.mongo</artifactId>
    <scope>test</scope>
</dependency>
```

scope를 test로 설정하면, 테스트시에 자동으로 내장 DB로 실행한다.

## 실행결과

[![image-20190428063143147](https://github.com/dadadamarine/dadadamarine.github.io/raw/master/assets/images/image-20190428063143147.png)](https://github.com/dadadamarine/dadadamarine.github.io/blob/master/assets/images/image-20190428063143147.png)

Jenkins상에서 build가 성공한 모습

[![image-20190428064130723](https://github.com/dadadamarine/dadadamarine.github.io/raw/master/assets/images/image-20190428064130723.png)](https://github.com/dadadamarine/dadadamarine.github.io/blob/master/assets/images/image-20190428064130723.png)

왼쪽은 Jenkins 자동배포가 성공한 모습 / 오른쪽은 실제 서버에 빌드된 파일이 배포된 모습 



**aws서버에 배포**

다른 프로젝트를 aws서버에 젠킨스를 통해 배포할시에는 key 등록, 권한문제등 추가적인 설정이 필요했으나 전체적으로 비슷하였다.



------

참조

- [[Jenkins\] 자동배포 설정](https://osc131.tistory.com/67)
- [spring boot jar 파일로 배포하기(deploy)](https://www.leafcats.com/178)
- [메이븐(Maven)은 알고 스프링(Spring)을 쓰는가?](https://jeong-pro.tistory.com/168)
- [Maven Dependency Scopes](https://www.baeldung.com/maven-dependency-scopes)

------

주석

[^1]: 이 의존성이 적용되는 지점을 설정하는 것. [참조](https://www.baeldung.com/maven-dependency-scopes)

