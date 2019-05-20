---
title: "[Spring/번역] 컨트롤러 테스트 가이드 in Spring Boot"
date: 2019-03-16 18:09:28 -0400
categories: Java/Spring
tags : Java/Spring
---

# Spring Boot에서 RestController에 대한 단위 테스트, 통합 테스트를 하는 방법

> 이글은 [The Practical Developer](https://www.facebook.com/thepracticaldeveloper/)님의 게시글인  **"[Guide to Testing Controllers in Spring Boot](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#comment-401)"**을 허가받아 번역한 글임을 밝힙니다.
>
> 번역을 반겨주셨던 The Practical Developer님께 감사를 표합니다.



## 목차



1. [Unit and Integration Tests for RestControllers in Spring Boot](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Unit_and_Integration_Tests_for_RestControllers_in_Spring_Boot)

2. [Introduction](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Introduction)

   2.1. [The sample application](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#The_sample_application)

   2.2. [Server and Client Side Tests](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Server_and_Client_Side_Tests)

3. [Server-Side Tests](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Server-Side_Tests)

4. [Inside-Server Tests](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Inside-Server_Tests)

   4.1. [Strategy 1: MockMVC in Standalone Mode](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Strategy_1_MockMVC_in_Standalone_Mode)

   ​	4.1.1. [MockMVC standalone code example](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#MockMVC_standalone_code_example)
   ​	

   ​			4.1.1.1. [MockitoJUnitRunner and MockMVC](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#MockitoJUnitRunner_and_MockMVC)
   
   ​			4.1.1.2. [JacksonTester initialization](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#JacksonTester_initialization)

   ​			4.1.1.3. [Configure the Standalone Setup in MockMVC](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Configure_the_Standalone_Setup_in_MockMVC)

   ​			4.1.1.4. [Testing ControllerAdvices and Filters with MockMVC](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Testing_ControllerAdvices_and_Filters_with_MockMVC)

   ​			4.1.1.5. [Better Assertions with BDDMockito and AssertJ](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Better_Assertions_with_BDDMockito_and_AssertJ)

   4.2. [Strategy 2: MockMVC with WebApplicationContext](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Strategy_2_MockMVC_with_WebApplicationContext) 

   ​	4.2.1. [MockMVC and WebMvcTest code example](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#MockMVC_and_WebMvcTest_code_example)

   ​			4.2.1.1. [SpringRunner](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#SpringRunner)
   
   ​			4.2.1.2. [MockMVC Autoconfiguration](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#MockMVC_Autoconfiguration)    		

   ​			4.2.1.3. [Overriding beans for testing using MockBean](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Overriding_beans_for_testing_using_MockBean)

   ​			4.2.1.4. [No server calls](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#No_server_calls)

   ​	4.2.2. [Using MockMVC with a Web Application Context – Conclusions](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Using_MockMVC_with_a_Web_Application_Context_Conclusions)

5. [Outside-Server Tests](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Outside-Server_Tests)

   5.1. [Strategy 3: SpringBootTest with a MOCK WebEnvironment value](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Strategy_3_SpringBootTest_with_a_MOCK_WebEnvironment_value)
   	
   5.2. [Strategy 4: SpringBootTest with a Real Web Server](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Strategy_4_SpringBootTest_with_a_Real_Web_Server)
   	
   ​	5.2.1. [Spring Boot Test Code Example](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Spring_Boot_Test_Code_Example)

   ​			5.2.1.1. [Web Server Testing](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Web_Server_Testing)

   ​			5.2.1.2. [Mocking layers](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Mocking_layers)				
   
   ​			5.2.1.3. [TestRestTemplate](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#TestRestTemplate)
   
   ​	5.2.2. [SpringBootTest approach – Conclusions](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#SpringBootTest_approach_Conclusions)

   5.3. [Performance and Context Caching](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Performance_and_Context_Caching)
   	

6. [Conclusion](https://thepracticaldeveloper.com/2017/07/31/guide-spring-boot-controller-tests/#Conclusion)



스프링 부트에서 컨트롤러(웹 또는 API  계층)를 테스트하기위한 여러 방법들이 있습니다. 몇몇 방법은 순수한 단위테스트를 작성하는데 도움이 되고 또 몇몇은 통합테스트에 유용합니다. 이 포스트에서는 스프링 standalone모드에서 MockMVC를 사용한 세가지의 접근법을 다룰 것입니다.





### 도입

**스프링 부트**에서는 테스팅을 위한 몇가지 접근법들이 있다. 스프링 부트는 지속적으로 진화하는 framework이고, 새로운 버전에는 더 많은 옵션들이 추가되고 구버전은 하위 호환성을 위해 유지됩니다. 그 결과로 코드의 일부를 테스트 하는 방법들이 생겼고, 언제 어떤 테스트 방법을 적용해야하는지 불명확해졌다. 이 글에서는 독자들이 여러 테스트방법들의 다른 관점을 이해하도록 돕고 각 방법들이 왜 유용한지, 언제 쓰는게 좋은지 설명할 것입니다. 

이 글에서는 mocking 객체가 각기 다른 레벨들에서 쓰여질수 있는 등의 이유로 가장 모호하므로.  Controller 테스팅에 대해 중점적으로 다루겠습니다. 





### 예시 프로그램

우리는 각기 다른 개념들을 연습하기위해  이글에서 몇몇의 예시 코드를 사용할 것이다. 

> 이글의 모든 코드는 [GitHub: Spring Boot Testing Strategies](https://github.com/mechero/spring-boot-testing-strategies) 에서 이용가능하다. ( 이 코드가 유용하다고 생각하면 star을 눌러주세요! )

요약하자면, 위 코드는 REST API를 통해 공개된 entity들(superheroes)의 저장소이다. 다른 전략들을 사용할때 발생하는 일들에 대해 좀 더 잘 이해하려면 각 예시의 몇몇 특징들을 나열하는것도 중요하다.

- 만약에 superhero가 id로 찾아지지 않는다면, NonExistingHeroException이 발생한다. 이 안에는 예외를 발생시켜주고, 이것을 404코드(NOT_FOUND)로 변환해주는 스프링의 @RestControllerAdvice 어노테이션이 있다.

- HTTP통신에서 HTTP Response 헤더에 `X-SUPERHERO-APP` 을 추가하기 위해 SuperHeroFilter 클래스룰 사용할 것이다.





### 서버 사이드 테스트 vs 클라이언트 사이드 테스트  

먼저, 서버사이드와 클라이언트 사이드를 분리해보자.

서버사이드 테스트는 가장 확장된 방식의 테스트이다. request를 보내보고 서버가 어떻게 행동하는지 볼 수 있고, response의 구성과 내용 등을 체크할 수 있다.

클라이언트 사이드 테스트는 자주 쓰이지는 않지만, request가 어떻게 구성되어 있는지, 또 어떤 http method를 사용하는지 검증하고 싶을때 유용하다. 이 테스트에서는 서버의 행동을 mock(가짜로 구성)하고, 클라이언트에서 몇몇 코드를 호출하여 서버로 간접적으로 request를 보내볼 수있다. 이를 통해 정확하게 request요청이 있음을 알 수 있고, 그 내용을 검증 할 수 있다. response의 내용은 가짜로 구성 했었기에 신경쓰지 않아도 된다. 아쉽게도 이 내용에 대한 좋은 예시는 많이 없다.  [javadoc에서 제공한 예시](https://github.com/spring-projects/spring-framework/blob/master/spring-test/src/test/java/org/springframework/test/web/client/samples/SampleTests.java) 를 봐도 마찬가지이다. 어쨌든, 여기에서 중요한 점은 클라이언트 사이드 테스트는 클라이언트 어플리케이션을 만들 때 사용할 수 있고, 클라이언트에서 밖으로 나가는 request들을 검증하고 싶을때도 사용할 수 있다는 것이다.

**이 글에서는 서버사이드 테스트에 초점을 맞출것이다.** 이를 통해 서버 로직이 어떻게 동작하는지 확인할 수 있다. 클라이언트 테스트는 보통 request를 mock(가짜로 구성)해서 어떻게 서버가 어떻게 반응하는지 체크한다. 스프링에서 이러한 종류의 테스트들은 어플리케이션의 Controller Layer과 밀접한 연관이 있는데, 이는 Controller Layer가 Spring에서 HTTP handling을 담당하는 부분이기 때문이다.





### Server-Side Tests

서버사이드 테스트들을 자세히 보면, 방법을 두가지로 분류할 수 있다. 첫번째는 MockMVC 접근법을 사용하여 Controller 테스트를 작성하는 것이고, 두번째는 RestTemplate을 이용하는 것이다. 만약에 당신이 **진짜** 단위 테스트를 작성하고 싶다면, 첫번째 방식(MockMVC)를 택해야한다. 반면 통합테스트를 작성하고 싶다면 RestTemplate을 사용해야 한다. 왜냐하면 MockMVC를 이용하면 컨트롤러에 대한 테스트를 세분화 할 수 있기 때문이다. 반면 RestTemplate은 Spring의 WebApplicationContext를 (Standalone모드 여부에 따라 부분적으로 또는 완전히) 이용한다. 이 두가지에 대해 더 자세히 이해해보자.





### Inside-Server Tests

우리는 우리의 Controller의 로직을 웹서버를 구동하지 않고도 직접 테스트 할 수 있다. 이것을 바로 "inside-server testing"이라 부르는데, 이 테스팅은 단위 테스트의 개념에 더 가깝다. 이를 가능하게 하려면, 당신은 전체 웹서버의 행동을 mock해야한다. 그래서 테스트해야 하는 몇몇 부분들을 놓치기도 할 것이다. 하지만 여기서 놓친 부분들은 통합 테스트에서 완전히 커버되기 때문에 걱정하지 않아도 된다. 





## **방법 1 : MockMVC in Standalone Mode**

![Test MockMVC Standalone](https://thepracticaldeveloper.com/wp-content/uploads/2017/07/tests_mockmvc_wm.png)

 스프링에서는 standalone 모드에서 MockMVC를 사용한다면, inside-server test를 작성할 수 있다. 이렇게 함으로써 어떠한 context도 로딩하지 않고 테스트 할 수 있다. 아래 예시를 보자.





### MockMVC  코드 예시

```java
@RunWith(MockitoJUnitRunner.class)
public class SuperHeroControllerMockMvcStandaloneTest {
 
    private MockMvc mvc;
 
    @Mock
    private SuperHeroRepository superHeroRepository;
 
    @InjectMocks
    private SuperHeroController superHeroController;
 
    // This object will be magically initialized by the initFields method below.
    private JacksonTester<SuperHero> jsonSuperHero;
 
    @Before
    public void setup() {
        // We would need this line if we would not use MockitoJUnitRunner
        // MockitoAnnotations.initMocks(this);
        // Initializes the JacksonTester
        JacksonTester.initFields(this, new ObjectMapper());
        // MockMvc standalone approach
        mvc = MockMvcBuilders.standaloneSetup(superHeroController)
                .setControllerAdvice(new SuperHeroExceptionHandler())
                .addFilters(new SuperHeroFilter())
                .build();
    }
 
    @Test
    public void canRetrieveByIdWhenExists() throws Exception {
        // given
        given(superHeroRepository.getSuperHero(2))
                .willReturn(new SuperHero("Rob", "Mannon", "RobotMan"));
 
        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/2")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();
 
        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getContentAsString()).isEqualTo(
                jsonSuperHero.write(new SuperHero("Rob", "Mannon", "RobotMan")).getJson()
        );
    }
 
    @Test
    public void canRetrieveByIdWhenDoesNotExist() throws Exception {
        // given
        given(superHeroRepository.getSuperHero(2))
                .willThrow(new NonExistingHeroException());
 
        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/2")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();
 
        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.NOT_FOUND.value());
        assertThat(response.getContentAsString()).isEmpty();
    }
 
    @Test
    public void canRetrieveByNameWhenExists() throws Exception {
        // given
        given(superHeroRepository.getSuperHero("RobotMan"))
                .willReturn(Optional.of(new SuperHero("Rob", "Mannon", "RobotMan")));
 
        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/?name=RobotMan")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();
 
        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getContentAsString()).isEqualTo(
                jsonSuperHero.write(new SuperHero("Rob", "Mannon", "RobotMan")).getJson()
        );
    }
 
    @Test
    public void canRetrieveByNameWhenDoesNotExist() throws Exception {
        // given
        given(superHeroRepository.getSuperHero("RobotMan"))
                .willReturn(Optional.empty());
 
        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/?name=RobotMan")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();
 
        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getContentAsString()).isEqualTo("null");
    }
 
    @Test
    public void canCreateANewSuperHero() throws Exception {
        // when
        MockHttpServletResponse response = mvc.perform(
                post("/superheroes/").contentType(MediaType.APPLICATION_JSON).content(
                        jsonSuperHero.write(new SuperHero("Rob", "Mannon", "RobotMan")).getJson()
                )).andReturn().getResponse();
 
        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.CREATED.value());
    }
 
    @Test
    public void headerIsPresent() throws Exception {
        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/2")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();
 
        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getHeaders("X-SUPERHERO-APP")).containsOnly("super-header");
    }
}
```

코드에 대한 자세한 설명은 다음 섹션에서 하겠다.





### MockitoJUnitRunner 그리고 MockMVC

우리는 단위테스트를 실행하기 위해 **MockitoJUnitRunner**를 사용한다. **Mokito**에서 제공해주는 **MockitoJUnitRunner**는 내장 JUnit runner의 위에서 몇 가지 기능을 제공해준다. 첫번째 기능은 프레임워크가 사용중인지 탐지해주는 것이고, (예시 코드에는 사용되지 않는  [stub](https://ko.wikipedia.org/wiki/%EB%A9%94%EC%86%8C%EB%93%9C_%EC%8A%A4%ED%85%81)들이 없다.) 두번째 기능은 @Mock 어노테이션이 달린 field들을 초기화 해주는 것이다. 그래서 따로 Mockito.initMocks( ) 메서드를 호출할 필요가 없다. 

여기서 mock들을 어떻게 초기화 하는지 이해해보자. 우리의 **SuperHeroRepository**는 **@Mock**으로 가짜로 구성되어있다. 우리는 이 repository를 진짜 controller 클래스 안에 넣어야하고, 그래서 우리는 **SuperHeroRepository** 인스턴스에 **@InjectMocks** 어노테이션을 붙인다. 즉, 가짜 repository가 진짜 bean 인스턴스(진짜 repository) 대신에 컨트롤러에 주입된다.

각각의 테스트에서, 우리는 **MockMVC**를 진짜 대신 사용하여 모든 종류의 가짜 request(GET, POST, etc.)를 보낸다. 그러면 응답으로 **MockHttpServletResponse** 를 받게된다. 이 응답도 역시 실제 응답이 아님을 명심하자. 위 모든것은 진짜처럼 꾸며진 것들이다.



```java
// This object will be magically initialized by the initFields method below.
private JacksonTester<SuperHero> jsonSuperHero;

@Before
public void setup() {
    // We would need this line if we would not use MockitoJUnitRunner
    // MockitoAnnotations.initMocks(this);
    // Initializes the JacksonTester
    JacksonTester.initFields(this, new ObjectMapper());
    // MockMvc standalone approach
    mvc = MockMvcBuilders.standaloneSetup(superHeroController)
            .setControllerAdvice(new SuperHeroExceptionHandler())
            .addFilters(new SuperHeroFilter())
            .build();
}
```





### JacksonTester 초기화

**JacksonTest** 객체도 **JacksonTester.initFields()**메서드를 사용함으로써 자동주입 되었다, 이 유틸리티 클래스는 스프링과 함께 제공되고, 위에 보이듯이 [static 메서드를 사용함으로써 초기화](http://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/json/JacksonTester.html)된다. 그래서 약간 까다롭다.







### MockMVC에서 독립형(standalone) 설정을 구성하는 방법

다른 테스트들 보다 먼저 실행되는 Setup 메서드에서 우리는 MockMVC를 standalone 모드로 구성할 필요가 있다. 또한 **명시적으로 테스트 상황에서 사용할 Controller와 Controller Advice 그리고 HTTP Filter를 구성해야한다.** 이 advice와 filter파트들을 기본 클래스에 추가 할 수도 있다. 하지만 둘중 어떤 방법이든 이 접근법에는 주요한 단점이 있다. ControllerAdvice와 Filter 등을 자동으로 주입해줄 Spring context가 없기 때문에, ControllerAdvice와 Filter 등의 안에 설계된 당신의 로직중 일부를 여기에 구현되어야 한다는 점이다.





### ControllerAdvice들과 Filter들을 MockMVC에서 테스트하기



```java
    @Test
    public void canRetrieveByIdWhenDoesNotExist() throws Exception {
        // given
        given(superHeroRepository.getSuperHero(2))
                .willThrow(new NonExistingHeroException());
                
    // when
    MockHttpServletResponse response = mvc.perform(
            get("/superheroes/2")
                    .accept(MediaType.APPLICATION_JSON))
            .andReturn().getResponse();

    // then
    assertThat(response.getStatus()).isEqualTo(HttpStatus.NOT_FOUND.value()); // line 60
    assertThat(response.getContentAsString()).isEmpty();
}
```


```java
    @Test
    public void headerIsPresent() throws Exception {
        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/2")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();
 
        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getHeaders("X-SUPERHERO-APP")).containsOnly("super-header");
    }
}
```



어떻게 이 환경들을 검증할 수 있는지에 집중해보자, 코드의 60번째 줄, 존재하지 않는 id와 보낸 request가  NOT_FOUND코드로 끝나는지 검사한다. 이는 **ControllerAdvice** 가 잘 동작한다는 뜻이다.  또한 header가 존재함을 검증하기위한 코드가 있고, 이를 통해 **Filter** 또한 잘 동작함을 알 수 있다. 당신은 이 코드들로 간단한 연습을 해볼수 있다. advice와 filter를 설정하는 standalone setup부분을 제거하고 테스트를 실행해보라. 이 클래스들을 주입할 context가 없기때문에 테스트가 실패할 것이다.

좀더 이 글이 도움이 되기 위해 위의 테스트에 Filter와 ControllerAdvice테스트를 추가시켰다. 하지만 컨트롤러 테스트에서 하지 않기로 결정했다면, 헤더의 존재와 404상태코드를 검증하는 테스트들을 **통합테스트**에 남겨도 된다. ( 그것들을 이 코드와, standalone configuration에서 지우면 된다. ) 이렇게 하면 **더 깨끗한 버전의 단위 테스트를 작성할 수 있다. 즉, 다른 인터페이스는 빼고, 온전히 Controller 클래스의 로직만 테스트 할 수 있게 된다.**







### BDDMockito 와 AssertJ을 이용하여 더 좋은 Assertion(테스트)들 구현하기

소제목처럼 가독성 좋고, 자연스러운 스타일의 테스트를 쓰기위해 BDDMockito 와 AssertJ를 이용한다. 만약 이것들에 대해 더 자세히 알고싶다면 다른 부분을 먼저 읽고 [Write BDD Unit Tests with BDDMockito and AssertJ](https://thepracticaldeveloper.com/2018/05/10/write-bdd-unit-tests-with-bddmockito-and-assertj/) 를 참조하라.



## 방법 2 : MockMVC with WebApplicationContext



![Test MockMVC with Context](https://thepracticaldeveloper.com/wp-content/uploads/2017/07/tests_mockmvc_with_context_wm.png)



두번째 방법으로 컨트롤러에 대해 MockMVC가 또 포함된 단위테스트를 작성 할 수 있다. 이 방법에서는 Spring의 WebApplicationContext를 불러올 수 있다. 하지만 아직 inside-server 방식을 사용하고 있기때문에, 이 방법에서도 웹 서버는 구동하지 않는다.







### MockMVC 와 WebMvcTest 샘플 코드



```java
@RunWith(SpringRunner.class)
@WebMvcTest(SuperHeroController.class)
public class SuperHeroControllerMockMvcWithContextTest {
 
    @Autowired
    private MockMvc mvc;
 
    @MockBean
    private SuperHeroRepository superHeroRepository;
 
    // This object will be magically initialized by the initFields method below.
    private JacksonTester<SuperHero> jsonSuperHero;
 
    @Before
    public void setup() {
        // Initializes the JacksonTester
        JacksonTester.initFields(this, new ObjectMapper());
    }
 
    @Test
    public void canRetrieveByIdWhenExists() throws Exception {
        // given
        given(superHeroRepository.getSuperHero(2))
                .willReturn(new SuperHero("Rob", "Mannon", "RobotMan"));
 
        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/2")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();
 
        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getContentAsString()).isEqualTo(
                jsonSuperHero.write(new SuperHero("Rob", "Mannon", "RobotMan")).getJson()
        );
    }
 
    // ...
    // Rest of the class omitted, it's the same implementation as in Standalone mode
 
}
```

Standalone mode와 비교했을때 큰 차이점들이 있다.





### SpringRunner

위 테스트는 SpringRunner와 함께 실행된다. 그리고 이 사실이 context가 초기화되는 방법을 설명한다. 테스트를 실행할때 시작부분에서 어떻게 context가 빈들을 로딩하고 주입하는지 확인 할 수 있다.





### MockMVC Autoconfiguration

@WebMVCTest 어노테이션을 사용하면 MockMVC 인스턴스가 자동설정되고, context에서 사용이 가능해진다. ( 그래서 아래 코드에서 보이듯이, autowire을 사용할 수 있다. ) 게다가 우리는 테스트할 컨트롤러 클래스를 annotation에 설정 할 수 있다. spring은 해당 부분 컨텍스트( 컨트롤러및 그 주변 구성)만 로딩할 것이다.

annotaion의 수행은 Filter와 Controller Advice도 주입되어야 한다는걸 알정도로 똑똑해서 이 경우에는 setup() 메서드에 해당 설정들이 없습니다.



### Overriding beans for testing using MockBean

( MockBean을 사용해서 테스용 빈들로 오버라이딩하기)

현재 코드엔 @MockBean 어노테이션을 사용하여 repository가 spring context에 주입되어 있습니다. 컨트롤러가 주입되어 사용할 수 있기 때문에, We don’t need to make any reference to our controller class apart from the one in the annotation. 이 bean은 단지 진짜 repository의 역할을 대신할 것입니다.



### No server calls (서버 호출이 없습니다)

현재 우리가 테스트하고있는 response들도 가짜라는 사실을 명심해야합니다. 또한 이 테스트에는 어떠한 서버도 없습니다. 그래도 이 테스트는 controller 클래스 안의 로직과 체크하고 있기 때문에 타당한 테스트입니다. 동시에 주변 클래들도 테스트합니다.(**SuperHeroExceptionHandler**` and `**SuperHeroFilter**).



### Using MockMVC with a Web Application Context – Conclusions

( Web Application Context 와 함께 MockMVC 이용하기 : 결론 )

가장 중요한 차이점은 Spring의  context가 있기때문에 주변 배우들을 명시적으로 호출할 필요가 없다는 점입니다. 만약 새 필터나, 새 controllerAdvice나 또는 이 request-response 프로세스에 참여하는 다른 배우들을 생성한다면, 그것들은 자동으로 테스트에 주입될것입니다. 그러므로 우는 매뉴얼적인 구성에 대해 신경쓸 필요가 없습니다.  테스트에서 무얼 사용하는지에 대한 세밀한 제어는 없지만, 이는 실제로 일어나는 것과 더 가깝습니다. 어플리케이션을 실행할때 모든 요소들이 기본적으로 존재하니까요.

통합테스트에 조그만 변화가 있는것을 확인할수 있을겁니다. 이번엔 filter와 controller adice를 참조하지않고 즉시 테스트하고 있습니다. 만약 나중에 이 request-reponse flow에 개입하는 다른 클래스를 포함하면, 이 테스트에도 참여하게 됩니다.

만약 이 테스트가 한클래스의 동작 이상을 포함하게 된다면, 당신은 이 테스트릁 그 클래스 간의 통합테스트로 여겨도 됩니다. 이 경계는 모호한것 입니다. 당신은 테스트에 한 컨트롤러만 있다고 주장 할 수 있겠지만, 적절히 이 테스트를 수행하기 위해선 추가적인 구성이 필요합니다. 



## Outside-Server Tests

테스트를 위해 HTTP request를 application으로 보내고 있다면, 이른바 outside-server test를 수행하고 있는 것입니다. 이렇게 밖에 있을때에도 우리는 mock을 테스트들에 주입해서 단위테스트 비슷하게 코드를 짤수 있습니다. 예를들면, 간단한 3-계층의 어플리케이션이 있을때, service 층을 가짜로 구성해서 웹서버를 통해 컨트롤러만 테스트 할 수 있습니다. 하지만 실제로 이 접근법은 단위테스트보다 훨씬 무겁습니다. 특별히 configuration을 제외시키거나, 필요한것들만 불러오도록 설정하는 등 스프링을 설정하지 않으면 전체 어플리케이션 context를 로딩합니다.

스프링에선 **RestTemplate를 사용하여 request들을 보내** REST controller들에 대한 outside-server test를 작성할 수 있습니다. 또는 통합테스트를 위해 유용한 기능들 (권한 헤더나, 오류 용인 헤더등을 포함한)을 보유한 새로운 TestRestTemplate를 이용할 수도 있습니다.

스프링 부트에서는 @SpringBootTest 어노테이션을 이용할 수 있습니다. 그러면 context로 주입된 몇몇 bean들을 얻을 수 있고, 또한 **application.properties** 등 에서 로드된 속성들에 접근 할 수 있습니다. 이는 테스트를 위해 스프링부트의 전체를 제공하는 @ContextConfiguration을 대신할 수 있습니다.

스프링부트에서 테스트 전략들은 특징들의 수나 사용할 수 있는 옵션들을 감안할때 복잡하게 여겨질 것입니다. 이제부터 그 전략들을 살펴봅니다. 



## 방법 3: SpringBootTest with a MOCK WebEnvironment value

**@SpringBootTest**` 또는 **@SpringBootTest(webEnvironment = WebEnvironment.MOCK)**를 사용할때는 진짜 **HTTP server**을 로드할 필요가 없습니다. 어디서 들어본 말인가요?

**이는 2번째 전략 MockMVC with anplication context 와 비슷한 접근법입니다.** 즉 이론상으로 outside-server 테스트를 작성하기 위해서 @SpringBootTest를 사용해야 하지만, WebEnviroment를 mock으로 세팅했다면 우리는 inside-server 테스트로 전환하게 됩니다.

우리는 web server가 없기때문에 RestTemplate를 사용할 수 없지만, 그래서 @AutoconfigureMockMVC라는 외부 어노테이션을 통해 구성된 MockMVC를 사용합니다.  제 생각엔 이 방법은 모든방법중 가장 까다로운 방법입니다. 개인적으로 추천드리지 않습니다.

MockMVC와 특정 컨트롤러를 위해 띄워진 context를 이용한 두번째 전략이 더 낫습니다. ( 테스트 하고자 하는것을 더 잘 조작할 수 있기때문에 )





## 방법 4: SpringBootTest with a Real Web Server

![Spring Boot Test](https://thepracticaldeveloper.com/wp-content/uploads/2017/07/tests_springboot_wm-1.png)



@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT) 또는 @SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)를 사용할때는 실제 HTTP server로 테스트를 하는것이다. 이때 RestTemplate 또는 TestRestTemplate를 이용할 필요가 있다. 랜덤 포트를 사용하는것과 지정된 포트를 사용하는 것의 차이는 단지 기본값 8080포트( server.port 속성으로 지정했을때는 그 포트)는 사용되지않고, 임의로 지정된 포트번호를 사용한다는 점 뿐이다. 이는 병렬 테스트를 진행할때 포트 충돌을 피하는데 도움이 된다. 다음 코드를 보고 주요한 특징을 설명 하겠습니다.



## Spring Boot Test 샘플 코드

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class SuperHeroControllerSpringBootTest {
 
    @MockBean
    private SuperHeroRepository superHeroRepository;
 
    @Autowired
    private TestRestTemplate restTemplate;
 
    @Test
    public void canRetrieveByIdWhenExists() {
        // given
        given(superHeroRepository.getSuperHero(2))
                .willReturn(new SuperHero("Rob", "Mannon", "RobotMan"));
 
        // when
        ResponseEntity<SuperHero> superHeroResponse = restTemplate.getForEntity("/superheroes/2", SuperHero.class);
 
        // then
        assertThat(superHeroResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(superHeroResponse.getBody().equals(new SuperHero("Rob", "Mannon", "RobotMan")));
    }
 
    @Test
    public void canRetrieveByIdWhenDoesNotExist() {
        // given
        given(superHeroRepository.getSuperHero(2))
                .willThrow(new NonExistingHeroException());
 
        // when
        ResponseEntity<SuperHero> superHeroResponse = restTemplate.getForEntity("/superheroes/2", SuperHero.class);
 
        // then
        assertThat(superHeroResponse.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
        assertThat(superHeroResponse.getBody()).isNull();
    }
 
    @Test
    public void canRetrieveByNameWhenExists() {
        // given
        given(superHeroRepository.getSuperHero("RobotMan"))
                .willReturn(Optional.of(new SuperHero("Rob", "Mannon", "RobotMan")));
 
        // when
        ResponseEntity<SuperHero> superHeroResponse = restTemplate
                .getForEntity("/superheroes/?name=RobotMan", SuperHero.class);
 
        // then
        assertThat(superHeroResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(superHeroResponse.getBody().equals(new SuperHero("Rob", "Mannon", "RobotMan")));
    }
 
    @Test
    public void canRetrieveByNameWhenDoesNotExist() {
        // given
        given(superHeroRepository.getSuperHero("RobotMan"))
                .willReturn(Optional.empty());
 
        // when
        ResponseEntity<SuperHero> superHeroResponse = restTemplate
                .getForEntity("/superheroes/?name=RobotMan", SuperHero.class);
 
        // then
        assertThat(superHeroResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(superHeroResponse.getBody()).isNull();
    }
 
    @Test
    public void canCreateANewSuperHero() {
        // when
        ResponseEntity<SuperHero> superHeroResponse = restTemplate.postForEntity("/superheroes/",
                new SuperHero("Rob", "Mannon", "RobotMan"), SuperHero.class);
 
        // then
        assertThat(superHeroResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
    }
 
    @Test
    public void headerIsPresent() throws Exception {
        // when
        ResponseEntity<SuperHero> superHeroResponse = restTemplate.getForEntity("/superheroes/2", SuperHero.class);
 
        // then
        assertThat(superHeroResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(superHeroResponse.getHeaders().get("X-SUPERHERO-APP")).containsOnly("super-header");
    }
 
}
```

차이점을 중심으로 보겠습니다.



### 1. Web Server Testing

테스트 구동을 위해 SpringRunner를 이용하지만, RANDOM_PORT 모드를 명시하기 위해 @SpringBootTest 어노테이션을 붙입니다. 그렇게 함으로써 웹서버가 실행되고, 테스트를 진행합니다.

위 18번째줄 코드에서 외부 서버에 도착하려고 했을때와 같이, 템플릿을 이용해 request들을 발생시킵니다.

이젠 검증하고자 하는 response가 MockHttpServletResponse 대신 ResponseEntity이기 때문에 테스트에 조금의 변화가 있습니다

```JAVA
 @Test
    public void canRetrieveByIdWhenExists() {
        // given
        given(superHeroRepository.getSuperHero(2))
                .willReturn(new SuperHero("Rob", "Mannon", "RobotMan"));
 
        // when
        ResponseEntity<SuperHero> superHeroResponse = restTemplate.getForEntity("/superheroes/2", SuperHero.class); // Line 18
 
        // then
        assertThat(superHeroResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(superHeroResponse.getBody().equals(new SuperHero("Rob", "Mannon", "RobotMan")));
    }
```



### 2. Mocking layers

지금도 @MockBean 어노테이션으로 repository 계층을 가짜로 구성할 수 있습니다. ( 역 : real Server를 이용해 테스트하는 지금도 repository를 mocking할 수 있습니다.)



### 3. TestRestTemplate

@SpringBootTest 어노테이션을 사용함으로써 주입할 수 있는 TestRestTemplate bean을 얻을수 있습니다. 이 TestRestTemplat는 표준 RestTemplate와 똑같이 동작하지만, 이 전에 볼수 있듯이 추가적인 기능들을 제공합니다.



### 스프링부트 테스트의 접근법 - 결론 

### 

컨트롤러 계층을 테스트 한다는 목표는 같지만, 이 테스트(역 : @SpringBootTest 어노테이션을 이용하는 테스트)는 이전의 첫번째 전략(MockMVC in standalone mode) 과는 완전히 다른 관점으로 목표에 접근합니다. 전에는 주변배우(filter와 controller advice)도 없이 클래스만을 불러왔습니다. 하지만 이 테스트에서는 웹 서버를 포함해서 전체 Sprinng Boot context를 불러옵니다. SpringBootTest는 가장 무겁고, 단위테스트와도 가장 거리가 먼 테스트입니다.

이 테스트는 주로 통합테스트에 사용됩니다. 이 테스트의 핵심은 context에 있는 빈들을 가짜로 구성해서 대체해도 여전히 스프링 부트의 다른 클래스간의 상호 동작을 웹서버도 참여한채로 검증할 수 있다는 것입니다.

하지만 단위테스트에서는 이방법을 사용하지 않는것을 권장합니다. 테스트가 무거워질거고 내가 뭘 테스트 하고 있는지에 대한 컨트롤을 잃을수도 있습니다. 하지만 통합테스트에서는 이 방법을 추천드립니다. 이 테스트 방법은 어플리케이션에서 다른 요소들이 어떤 방식으로 함께 동작하는지 검증하는데 항상 유용합니다.



## 성능과 Context 캐싱

지금 당신은 첫번재 전략이 성능면에서 다른 전략들보다 훨씬 최적이다 라고 생각할지도 모릅니다. 또는 테스트 할때마다 전체 Spring Boot Context를 로딩해야하니 끔찍하게 동작한다고 생각할지도 모르죠. 하지만 그생각들은 100% 맞는말은 아닙니다. 스프링(Boot가 포함된)을 테스트를 위해서 사용할때, application context는 같은 테스트 단위동안에는 기본적으로 재사용 됩니다.

즉, 위 예시의 전략 2,3,4에서 Spring context는 첫번째 로딩된 이후호 재사용 됩니다. 하지만 주의하세요.  테스트가 context의 bean들을 수정하는 상황에서 context의 재사용은 부작용을 일으킬 수 있습니다. 만약 이런 상황이라면 @DirtiesContext 어노테이션으로 conntext를 다시 로드한다고 명시 함으로써 해결 할 수있습니다. ([관련 문서](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/integration-testing.html#__dirtiescontext)를 확인하세요)

# 결론

보셨다시피, Spring Boot에는 컨트롤러들을 한개씩 따로 테스트하는 많은 전략들이 있습니다. 우리는 이 글에서 가장 가벼운 방법부터 가장 무거운 방법까지 다루었습니다. 이제 언제 어떤 테스트를 사용해야 하는지에 대해 개인적인 의견을 적어보려고 합니다.

- 다른 것들의 동작은 배재한, 컨트롤러 로직만을 위한 단위테스트는 항상 작성하려고 해야합니다. 이때 첫번째 전략을 이용하세요 : **use MockMVC in Standalone mode**.
- 웹 계층과 관련된 주변 행동들을 테스트 해야한다면 ( 필터링, 인터셉터, 권한 등) 네번째 전략을 이용하세요 : **SpringBootTest with a web server** on **a random port**. 
  하지만, 이 테스트는 어플리케이션의 몇몇 부분들을 검증하는것 이기 때문에 분리된 통합테스트로 생각해야합니다. 필요하다면 순수 컨트롤러 층에대한 단위테스트를 해야 합니다. 즉, 테스트 단계들을 섞어버리는 것을 지양하고, 분리되도록 노력해야합니다.

이 가이드가 유용했길 바랍니다, 피드백이 있다면 댓글을 통해 남겨주세요!

이 가이드가 마음에 드셨나요? 제가 저술한 Learn Microservices with Spring Boot [on the Apress Store](http://www.kqzyfj.com/click-8535631-12831288?url=https%3A%2F%2Fwww.springer.com%2Fus%2Fbook%2F9781484231647%3Futm_medium%3Daffiliate%26utm_source%3Dcommission_junction_authors%26utm_campaign%3D3_nsn6445_product_PID%25zp%26utm_content%3Dus_10092017&cjsku=9781484231647) or [Amazon](http://amzn.to/2FSB2ME) 에서 더 많은 정보를 얻으실 수 있습니다.



