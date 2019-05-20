---
title: "[Spring/Project] Controller 단위테스트 코드와 설명 1"
date: 2019-04-24 02:25:28 -0400
categories: Java/Spring
---



# 개요

진행중인 스터디 프로젝트에서 컨트롤러를 수정해야할 필요가 생겼다. 마침 이 프로젝트에서 컨트롤러에 대한 테스트를 작성하지 않고 AceeptanceTest만 가지고 진행했었기 때문에, 컨트롤러 테스트를 먼저 작성하여 TDD로 컨트롤러를 작성하고자 한다.



> 컨트롤러 테스트의 원론에 대한 설명은 블로그 글 [[Spring/번역] 컨트롤러 테스트 가이드 in Spring Boot](<https://dadadamarine.github.io/java/spring/spring-boot-validation/>) 을 추천드립니다.



> **요약 : 컨트롤러에 대한 단위테스트는 크게 세 방법으로 나눌 수 있습니다.**
>
> 1. Standalone Mode의 MockMVC테스트
>    - 해당 Controller외에 어떠한 빈도 띄우지 않습니다. 
>
> 
>
>
> 2. WebApplicationContext[^1]와 함께하는 MockMVC테스트
>    - ControllerAdvice, Filter와 환경 빈들과 함께 테스트합니다. 
>
> 
>
> 3. Real Web Server로 진행하는 SpringBootTest (사실상 Integration Test)
>    - RestTemplate을 이용한 Outside-Server Test를 진행합니다.
>





# 테스트 작성 및 코드설명

현재 순수 Controller만을 위한 단위테스트를 작성하기 위해 1번 방법,

파라미터 어노테이션이 있는 메서드를 테스트하기위한 2번 방법 두가지를 사용합니다.



## 1. Standalone Mode의 MockMVC테스트 코드 

```java
@RunWith(MockitoJUnitRunner.class)
/* 
MocktoJUnitRunner를 이용해 단위테스트를 실행. MocktoJUnitRunner는 기본JUnit Runner에 비해 @Mock 어노테이션이 붙은 field들을 초기화 해주는등 더 많은 기능을 제공한다.
*/
public class ApiMenuCategoryControllerTest {
    private static Logger log = LoggerFactory.getLogger(ApiMenuCategoryControllerTest.class); // 로그를 위한 로거 선언
	
    public static final String URI_MENU_CATEGORY = "/api/menuCategory"; // URI로 쓰이는 상수

    private MockMvc mockMvc; 

    @Mock
    private MenuCategoryService menuCategoryService;

    @InjectMocks
    private ApiMenuCategoryController apiMenuCategoryController;

    private JacksonTester<MenuCategory> jsonMenuCategory;

    private List<MenuCategory> categories = new ArrayList<>();

    @Before
    public void setup() {
        JacksonTester.initFields(this, new ObjectMapper());

        mockMvc = MockMvcBuilders.standaloneSetup(apiMenuCategoryController)
                .setControllerAdvice(new ExceptionHandlerExceptionResolver())
                .build();

        MenuCategory fstCategory = new MenuCategory(null, "카테고리1");
        fstCategory.setId(1l);
        fstCategory.addChild(new MenuCategory(1l, "카테고리1의 하위 카테고리"));
        categories.add(fstCategory);
        categories.add(new MenuCategory(null, "카테고리2"));
    }

    @Test
    public void getCategoriesTest() throws Exception {
        //given
         when(menuCategoryService.findCategories())
                .thenReturn(categories);

        //when
        MockHttpServletResponse response = mockMvc.perform(get(URI_MENU_CATEGORY).accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();

        //then
        log.debug(response.getContentAsString());
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
    }

    @Test
    public void createTest() {
    }

    @Test
    public void deleteTest() {
    }
}
```



**MockMvc**

```java
	MockMvc mockMvc; // Spring MVC의 동작을 재현하게 도와주는 클래스. 
```

참고 : [MockMVC에 상세설명](<https://itmore.tistory.com/entry/MockMvc-%EC%83%81%EC%84%B8%EC%84%A4%EB%AA%85>)



**@Mock**

```java
  @Mock
    private MenuCategoryService menuCategoryService;
```

mock() 메서드를 대체하는 어노테이션. 

```java
@Test
public void example(){
    Person p = mock(Person.class);
    assertTrue( p != null );
}
```

mock() 메서드는 위와같이 넘겨준 클래스타입의 인스턴스를 만들어서 반환해준다.

참고 : @Mock, @Spy, @InjectMocks, when()등에 대한 설명 - [Mockio 사용법](<https://jdm.kr/blog/222>) 



**@InjectMocks**

```java
   @InjectMocks
    private ApiMenuCategoryController apiMenuCategoryController;
```

클래스 내부에서 다른 클래스를 포함해야 하는경우에 사용. 

@Mock , @Spy 어노테이션이 붙은 Mock객체를 자신의 멤버 클래스와 일치하면 주입시킨다.

참고 : @Mock, @Spy, @InjectMocks, when()등에 대한 설명 - [Mockio 사용법](<https://jdm.kr/blog/222>) 

 

**mockMvc 초기화**

```java
mockMvc = MockMvcBuilders.standaloneSetup(apiMenuCategoryController)
                .setControllerAdvice(new ExceptionHandlerExceptionResolver())
                .build();
```

mockMvc를 apiMenuCategoryController에 대한 standalone 모드로 실행시키고 ExceptionHandlerExceptionResolver라는 ControllerAdvice(전역 에러처리를 도와주는 Bean)를 추가한다.



**When()** 

Mock으로 만든 객체의 메소드들에 대해 반환해줄 값을 설정해주는 메소드.

```java
when(menuCategoryService.findCategories())
                .thenReturn(categories);
```

> anyInt() 등의 any.. 메서드, 예외 발생 등도 가능하다.



**HttpSatus.OK.value()**

```java
	assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
```

MockHttpServletResponse는 status를 int값으로 저장한다.



![image-20190424153139704](/assets/images/image-20190424153139704.png)

<center>response 변수의 타입인 MockHttpServletResponse</center>



HttpStatus Enum객체인 HttpStatus이고, HttpStatus enum들의 value() 메서드는 200 이라는 상태값을 integer로 출력한다.

![image-20190424153311326](/assets/images/image-20190424153311326.png)

<center>HttpStatus enum 의 value 메서드</center>



# 마침

> 다음 글에선 WebApplicationContext와 함께하는 MockMVC테스트 코드에 대해 다루도록 하겠습니다. 감사합니다.





**주석**

----



[^1]: **여러 빈들을 보관하는곳. 크게 Root와 Servlet WebApplicationContext로 나뉜다. 이 글에선 filter, ControllerAdvice 빈과 같은 주변 빈을 같이 로딩해주는 요소로 이해할 수 있다. ** 더 자세한 정보는 다음을 참조한다.  [참조1](<https://unordinarydays.tistory.com/131>) , [참조2](<https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#spring-web>)![mvc context hierarchy](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/mvc-context-hierarchy.png)




