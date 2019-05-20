---
title: "[Spring/Project] Spring WEb MVC 구조"
date: 2019-05-02 02:25:28 -0400
categories: Java/Spring
---



# Spring Framework 의 구조

Spring Framework 의 전체구조는 아래와 같다. [출처](<http://www.increpas.pe.kr/worker_course1.html?num=2&c_type=3>)

![img](http://increpas.com/ems/upload/images/springframework.png)



## Spring Web MVC

### Spring MVC 구조

**내가 자주 보아왔던 아래의 사진은 사실 Spring Web MVC의 구조에 해당한다. 스프링 프레임워크 전체로 보았을때 정말 빙산의 일각에 불과하다.** 

![img](https://t1.daumcdn.net/cfile/tistory/9934E4335A0E8EA01F)



Spring RESTful의 구조는 다음과 같다. [출처](<https://jeong-pro.tistory.com/96>)

![img](https://t1.daumcdn.net/cfile/tistory/99F609335A0E8EA010)



### Spring MVC의 처리 순서

위의 두가지 구조에서 가장 중점적인 역할을 하는것은 DispatcherServlet이다.  DispatcherServlet이 어떤 역할을 하는지 자세히 알아보자. 

처리 순서에 대해 잘 설명되어 있는 [참조](<https://jeong-pro.tistory.com/96>)사이트를 참조하였다.



1. Servlet Container에서 들어오는 모든 Request를 가로챈다.  

2. **HandlerMapping**에게 어떤 컨트롤러에게 요청을 위임할지 물어본다.

3. 매핑된 컨트롤러가 있다면, @RequestMapping을 타고가 요청을 처리할 메서드로 보낸다.

4. 컨트롤러는 해당 요청을 처리할 서비스를 주입(DI)받아 비즈니스로직 처리를 Service에 위임한다.
5. Service는 데이터베이스 관련 처리를 DAO에게 위임한다.
   ...
6. 이렇게 다시 돌아와 결과를 받은 컨트롤러는 Model객체와 함께 어떤 View를 보여줄지에 대한 정보(ModelAndView)를 담아 Dispatcher Servlet로 전달한다.
7. Dispatcher Servlet은 ViewResolver에게 전달받은 뷰의 정보를 넘긴다.
8. ViewResolver는 해당 View(템플릿 파일)을 찾아 DispatcherServlet에게 알려준다. 
   (servlet-context.xml에서 suffix, prefix를 통해 /WEB-INF/views/index.jsp 이렇게 만들어주는 것도 ViewResolver)
9. Dispatcher Servlet은 응답할 View에게 Render를 시키고, View는 응답 로직을 처리해서 돌려준다.
10. DispatcherServlet이 클라이언트에게 렌더링된 Response객체를 보낸다.

 

> 각각에 대한 자세한 설명은 다음 [참조글](<https://www.baeldung.com/spring-dispatcherservlet>)에 잘 나와있다. 



### DispatcherServlet

** DispatcherServlet의 구성**

Spring MVC framework의 DispatcherServlet은 단순히 request를 전달하는 것 이상의 기능을 한다. Spring IoC 컨테이너와 완전이 합쳐져서 스프링의 특징등을 이용할 수 있게 해준다. [출처1](<https://minwan1.github.io/2018/05/28/2018-05-28-spring-mvc/>), [출처2](<https://minwan1.github.io/2017/10/08/2017-10-08-Spring-Container,Servlet-Container/>)



![img](https://i.imgur.com/IUf4orm.png)

![img](https://i.imgur.com/PlDF42i.png)

ContextLoaderListener는 root-context.xml에 등록된 SpringContainer를 구동한다. 이때 비즈니스 로직과, DAO등의 객체들(Web환경에 독립적인 객체들)이 생성되어 Root WebApplicationContext에 빈으로 등록된다. DispatcherServlet은 WebApplicationContext를 생성하여 자신이 직접 사용하는 컨트롤러를 포함한 웹 관련 빈을   등록한다. 



**DispatcherServlet의 생성시점**

  웹 컨테이너(supports servlet. tomcat같은)는 사용자의 요청에 대해 request와 response 객체를 생성한다. 그리고 앞에서 살펴본것처럼 배포서술자를 통해 어떤 DispatcherServlet가 처리할지를 알아낸다. 만일 해당 클래스가 한번도 실행된 적이 없다면, 새로 인스턴스를 생성하고, 초기화를 한다. 이때 관련 빈들도 같이 초기화해준다. [참조   Spring MVC - DispatcherServlet 동작 원리](<https://jess-m.tistory.com/15>) . 동작 원리에 대한 더 자세한 설명 또한 이 글에 나와있다.



**Root and child contexts**

한개의 Web Application은 여러 DispatcherServlet를 사용 할 수 있다. 이때 각 DispatcherServlet은 각각 자신의 Application context를 가진다. Root WebApplicationContext는 전역적으로 선언되어 각 child WebApplicationContext는 이 Root context의 값을 참조 할 수 있다. 반대는 불가능 하다. [참조](<https://howtodoinjava.com/spring-mvc/contextloaderlistener-vs-dispatcherservlet/>)

 

### Spring MVC의 외부

좀더 크게 거시적으로 바라본 Spring MVC의 모습은 어떨까? [출처](https://jojoldu.tistory.com/28)

![img](https://t1.daumcdn.net/cfile/tistory/2236F14757BBD25119)



Spring MVC는 Servlet들의 생명주기를 관리하는 Servlet Contrainer에 포함된 Servlet중 하나이다. [참조](<https://jojoldu.tistory.com/28>), [서블릿 컨테이너의 역할과 서블릿의 생명주기](<https://minwan1.github.io/2017/10/08/2017-10-08-Spring-Container,Servlet-Container/>)



**WebApplicationContext vs ApplicationContext**



![img](https://t1.daumcdn.net/cfile/tistory/276A9339579B5CDB2C)

WebApplicaionContext는  ApplicationContext를 상속받은 인터페이스이다. 웹 어플리케이션을 위한 ApplicationContext로 사용되고, ApplicationContext에 추가적으로 Bean 영역(bean scope)를 정의하고 있다. [출처](<https://jojoldu.tistory.com/28>)