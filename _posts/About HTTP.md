## 웹과 HTTP

#### 웹

1. 월드 와이드 웹(World Wide Web)이란 인터넷에 연결된 사용자들이 서로의 정보를 공유할 수 있는 공간을 의미합니다.

2. 인터넷상의 하나의 서비스
3. 특징중 하나로 멀티미디어 정보를 하이퍼텍스트(문서 내부에 또 다른 문서로 연결되는 참조를 집어 넣음으로써 웹 상에 존재하는 여러 문서끼리 서로 참조할 수 있는 기술을 의미합니다.) 방식으로 제공한다는 점이 있다.

4. ~~온디맨드 서비스(사용자 요청에 즉시 반응하는 서비스)~~



#### HTTP

1. 웹의 어플리케이션 계층 프로토콜 HyperText Transfer Protocol 

2. HTTP는 두가지 프로그램으로 구현됨(클라이언트, 서버). 즉 서로 다른 종단 시스템의 각 프로그램들은 HTTP 메시지를 교환하여 통신함.
   HTTP는 메시지의 구조 및 클라이언트와 서버가 어떻게 메시지를 교환할 지(클라이언트가 서버에게 웹 페이지를 어떻게 요청할지 + 서버가 클라이언트에게 어떻게 웹 페이지를 전송할 지)를 정의함.

3. TCP를 전송 프로토콜로 사용함. 즉 클라이언트는 서버와 TCP연결을 진행하고, 연결이 되면 HTTP request 메시지를 소켓 인터페이스로 보냄. 
   즉, 이러한 계층구조로 인해 HTTP는 데이터 손실 또는 복구 과정등을 몰라도 된다.

4. 서버는 클라이언트에 대한 어떠한 상태정보도 저장하지 않는다. 따라서 HTTP를 비상태 프로토콜(stateless)라고 부르기도 한다.

#### 비지속 연결과 지속 연결

**비 지속 연결**

1. HTTP는 default값으로 지속 연결을 사용한다. 즉 여러 객체를 요청하기위한 url요청/응답 쌍들이 한 tcp연결상에서 전달된다. 
  but, HTTP 클라이언트와 서버는 비지속 연결을 사용하도록 설정할 수 있다.

2. 비지속 연결은 한 객체의 요청마다 다른 TCP연결 과정을 거친다. 비지속 연결에서는 동시성 정도(브라우저의 기본값은 보통 5~10개의 TCP연결을 동시에 설정)가 중요하다.

3. RTT(Round Trip time)은 패킷이 서버로 가서 클라이언트에 되돌아 오는데 걸리는 시간이다.



![ê´ë ¨ ì´ë¯¸ì§](https://phoenix.goucher.edu/~kelliher/s2011/cs325/feb04img8.png)

> 1. 클라이언트는 tcp 메세지를 서버로 보내고, 서버는 tcp메세지로 응답한다.
> 2. 클라이언트는 HTTP 요청 메세지를 TCP 연결로 보내면서, 핸드셰이크의 세번쨰 부분(응답)을 같이 보낸다.
>
>
> 즉 총 응답 시간은 2 RTT와 HTML파일 하나의 전송시간을 더한 값이다.



**지속 연결**

HTTP 1.1부터

서버가 응답을 보낸후에 TCP연결을 그대로 유지하는 방식

파이프라이닝 이용

최근 HTTP 2부터는 같은 연결상에서 다중 요청과 응답이 가능하고, 이 연결 내에서 HTTP메시지 요청과 응답의 우선순위 기법이 가능하도록 제안됨.



#### HTTP 메세지 포맷



**HTTP REQUEST 메시지**

![img](https://www.ntu.edu.sg/home/ehchua/programming/webprogramming/images/HTTP_RequestMessageExample.png)



![kurose_320719_c02f08](/Users/dadadamarine/Desktop/study/blog/dadadamarine.github.io/assets/images/kurose_320719_c02f08.gif)







**HTTP RESPONSE 메시지**

![img](https://www.ntu.edu.sg/home/ehchua/programming/webprogramming/images/HTTP_ResponseMessageExample.png)



![kurose_320719_c02f09](/Users/dadadamarine/Desktop/study/blog/dadadamarine.github.io/assets/images/kurose_320719_c02f09.gif)



**HTTP 상태코드**



#### 웹 캐싱

**개요**

웹 캐시, 또는 프록시 서버라고 함

원래의 웹 서버를 대신하여 HTTP요구를 충족시키는 네트워크 개체.

웹 캐시는 자체의 저장디스크를 가지며, 최근 호출된 객체의 사본을 저장 및 보존한다.

<https://joridari.tistory.com/category/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8/Web>

![ì¹ ìºì±ì ëí ì´ë¯¸ì§ ê²ìê²°ê³¼](http://pds13.egloos.com/pds/200901/10/34/a0100134_4967c7dba90d5.jpg)

![캐싱 서버 발전 과정](http://pds12.egloos.com/pds/200901/10/34/a0100134_4967c7dba26cb.jpg)

![웹 캐싱을 이용한 웹 서비스 인프라](http://pds11.egloos.com/pds/200901/10/34/a0100134_4967c7dc1ab77.jpg)



**기존 웹 캐싱** 

![캐싱 서버 발전 과정](http://pds12.egloos.com/pds/200901/10/34/a0100134_4967c7dba26cb.jpg)

기관 내부 캐싱자료 더 좋은거 찾기

1. 응답 시간 단축. (특히 클라이언트와 원 서버 사이의 병목 대역폭과, 클라이언트와 캐시 사이의 병목 대역폭의 차이에 따라)
2. 한 기관에서 인터넷으로 접속하는 링크상의 웹 트래픽을 대폭으로 줄일 수 있다. 즉 많은 트래픽을 지역화 할 수 있다.
3. CDN(Content Distribution Network)이 예이다. 



**조건부 GET**

GET요청과 If-modified-since를 이용함 - 캐시의 복사본시 새것이 아닐수도 있는 문제 해결



0. 웹 캐싱은 처음 브라우저에게 객체를 전달할때, 자신에게도 객체를 같이 저장한다. 이떄 객체와 더불어 HTTP Response의 Last-Modified에 해당하는 마지막 수정날짜를 함께 저장한다.

1. 웹 캐싱은 서버에 request를 보낼 때 If-modified-since헤더를 같이 보낸다. 이때 이 헤더의 값은 객체와 함께 저장되었던 Last-Modified값이다.
   즉 이 조건부 GET을 통해 서버에게 그 객체가 명시된 날짜 이후에 수정된 경우만 객체를 보내도록 요청한다.
2. 서버는 이 조건부 GET에 대한 응답을 보낼때, 명시된 날짜 이후로 수정되지 않았다면 상태코드로 304 Not Modified와, 비어있는 body를 보낸다.

