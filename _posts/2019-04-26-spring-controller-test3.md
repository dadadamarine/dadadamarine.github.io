---
title: "[Spring/Project] Annotationed Parameter가 있는 Controller에 대한 단위테스트 작성법"
date: 2019-04-26 05:25:28 -0400
categories: Java/Spring
---



> 이전 Controller 단위테스트 코드와 설명 2 글에서 통합테스트가 아닌 Controller에 대한 단위테스트로 방향을 전환한후 후, 어떻게 어노테이션이 달린 파라미터가 있는 컨트롤러 메서드에 대한 테스트를 작성할 것인지에 대한 방법을 적은 글입니다.



# 개요

```java
// ApiMenuCategoryController

@PostMapping("")
public ResponseEntity<MenuCategory> create(@ManagerAccount Account manager, @RequestBody MenuCategoryDTO menuCategoryDTO) {
    MenuCategory createdCategory = menuCategoryService.create(menuCategoryDTO);
    return makeCreatedResponseEntity(createdCategory);
}
```

이렇게 컨트롤러에 @ManagerAccount 라는 어노테이션이 붙은 경우, 어떻게 컨트롤러에 대한 단위테스트를 구현 할 수 있을까? 에 관한 글입니다.



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

이 단위테스트에서는 1번방법인 MockitoJUnitRunner를 이용하여

`ManagerAccountHandlerMethodArgumentResolver`를 Mocking하여 Auth를 통과한 컨트롤러 로직만을 단위테스트 할 예정입니다.



**테스트 코드**

```java
@RunWith(MockitoJUnitRunner.class)
public class ApiMenuCategoryControllerTest {
    private static Logger log = LoggerFactory.getLogger(ApiMenuCategoryControllerTest.class);

    public static final String URI_MENU_CATEGORY = "/api/menuCategory";

    private MockMvc mockMvc;

    @Mock
    private MenuCategoryService menuCategoryService;

    @Spy
    private ManagerAccountHandlerMethodArgumentResolver managerArgumentResolver;

    private JacksonTester<MenuCategoryDTO> jsonMenuCategoryDTO;

    @InjectMocks
    private ApiMenuCategoryController apiMenuCategoryController;

    private List<MenuCategory> categories = new ArrayList<>();

    @Before
    public void setup() {
        JacksonTester.initFields(this, ObjectMapper::new);

        mockMvc = MockMvcBuilders.standaloneSetup(apiMenuCategoryController)
                .setControllerAdvice(new ExceptionHandlerExceptionResolver())
                .setCustomArgumentResolvers(managerArgumentResolver)
                .build();

        MenuCategory fstCategory = new MenuCategory(null, "카테고리1");
        fstCategory.setId(1l);
        fstCategory.addChild(new MenuCategory(1l, "카테고리1의 하위 카테고리"));
        categories.add(fstCategory);
        categories.add(new MenuCategory(null, "카테고리2"));
    }

    @Test
    public void deleteCategoryTest() throws Exception {
        //given
        when(menuCategoryService.deleteById(1l))
                .thenReturn(new MenuCategory());
        when(managerArgumentResolver.supportsParameter((MethodParameter) notNull()))
                .thenReturn(true);
        when(managerArgumentResolver.resolveArgument(
                (MethodParameter) notNull()
                , (ModelAndViewContainer) notNull()
                , (NativeWebRequest) notNull()
                , (WebDataBinderFactory) notNull()
        )).thenReturn(manager);

        //when
        MockHttpServletResponse response = mockMvc.perform(
                delete(URI_MENU_CATEGORY + "/{id}", 1l))
                .andReturn().getResponse();

        //then
        log.debug(response.getContentAsString());
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
    }

}
```

 

해당 테스트는 ManagerAccountHandlerMethodArgumentResolver를 Spy로 등록하여 어노테이션 체크인 supportsParameter() 메서드는 원래대로 동작합니다.

반면 resolveArgument() 메서드를 Mocking 하여 원래 진행하던 권한체크를 넘기고 바로 Account를 리턴하여 컨트롤러 로직이 동작하도록 작성하였습니다.



**ManagerAccountHandlerMethodArgumentResolver 코드**

```java
public class ManagerAccountHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter methodParameter) {
        return methodParameter.hasParameterAnnotation(ManagerAccount.class);
    }

    @Override
    public Object resolveArgument(MethodParameter methodParameter,
                                  ModelAndViewContainer modelAndViewContainer,
                                  NativeWebRequest nativeWebRequest,
                                  WebDataBinderFactory webDataBinderFactory) throws Exception {
        Account account = SessionUtils.getUserFromSession(nativeWebRequest);
        if (!account.isManager()) {
            throw new UnAuthorizedException("You're not manager!");
        }
        return account;
    }
}
```



**실행결과**

![image-20190426202049695](/assets/images/image-20190426202049695.png)

~~하지만 예상과는다르게 SessionUtils의 isLogin메서가 실행되었고, NullPointerException이 발생했습니다. 이는 resolveArgument() 메서드에 대한 mocking이 되지 않았단 뜻입니다.~~ 

처음엔 이렇게 예상했었으나, 에러가 발생한 지점이 Test코드인 것으로 보아, ArgumentResolver을 Mocking하는 When메서드에서 에러가 났다고 생각할 수 있습니다.





> @Spy를 @Mock 으로 바꾼후. supportsParameter()메서드도 Mocking해주니 정상적으로 작동하였다.

```java
@Test
public void deleteCategoryTest() throws Exception {
    //given
    when(menuCategoryService.deleteById(1l))
        .thenReturn(new MenuCategory());
    when(managerArgumentResolver.resolveArgument(
        any(), any(), any(), any()
    )).thenReturn(manager);
    when(managerArgumentResolver.supportsParameter(any())).thenReturn(true);

    //when
    MockHttpServletResponse response = mockMvc.perform(
        delete(URI_MENU_CATEGORY + "/{id}", 1l))
        .andReturn().getResponse();

    //then
    log.debug(response.getContentAsString());
    assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
}
```

**결과**

![image-20190426204115776](/assets/images/image-20190426204115776.png)



> 하지만 @Spy로 다시 변경할경우

![image-20190426204450219](/assets/images/image-20190426204450219.png)

위의 디버깅 포인트까지 넘어가지도 못하고 when()에서 에러가 발생한다.



**필자 생각**

이는 @Spy 어노테이션이 동작할때 Mocking 하기 전 기존의 코드들도 모두 정상적으로 동작해야 한다는 것을 뜻합니다.

하지만 서블릿이 온전하지 못한 상태에서 resolveArgument() 메서드의 파라미터로 null이 전달되게 되고, 이는 spy빈이 제대로 동작하지 못한다는 뜻입니다. 

그래서 when 메서드에서 이 스파이빈은 정상적으로 동작하지 않는다는 에러를 내보내는 것으로 생각됩니다. 



> 이 내용은 확실히 공부한 뒤에 보강해서 업로드 하겠습니다.



# 다른 방식의 테스트 작성 및 코드설명

테스트와 관련된 공부를 하다가 Mocking을 다르게 구현할 수 있는 방법을 찾았습니다.

거기다가 이 방법이 더 옳은방법이라는 설득력도 충분한 글이었습니다.

해당 글을 소개함과 함께 현재 테스트 코드를 이 방식으로 적용하겠습니다.

>  이동욱 님의 [@SpyBean @MockBean 의도적으로 사용하지 않기](https://jojoldu.tistory.com/320)

 

테스트 코드 내부에 원래 argumentResolver를 상속하는 MockManagerArgumentResolver를 만든후에 원래 메서드를 @Override하는 방법입니다.

```java
//ApiMenuCategoryControllerTest
static class MockManagerArgumentResolver extends ManagerAccountHandlerMethodArgumentResolver {

        @Override
        public Object resolveArgument(MethodParameter methodParameter,
                                      ModelAndViewContainer modelAndViewContainer,
                                      NativeWebRequest nativeWebRequest,
                                      WebDataBinderFactory webDataBinderFactory) throws Exception {
            return new Account();
        }
    }
```



```java
@RunWith(MockitoJUnitRunner.class)
public class ApiMenuCategoryControllerTest {
    private static Logger log = LoggerFactory.getLogger(ApiMenuCategoryControllerTest.class);

    public static final String URI_MENU_CATEGORY = "/api/menuCategory";

    private MockMvc mockMvc;

    @Mock
    private MenuCategoryService menuCategoryService;

    //@Mock
    //private ManagerAccountHandlerMethodArgumentResolver managerArgumentResolver;

    private MockManagerArgumentResolver mockManagerArgumentResolver = new MockManagerArgumentResolver();

    private JacksonTester<MenuCategoryDTO> jsonMenuCategoryDTO;

    private Account manager;

    @InjectMocks
    private ApiMenuCategoryController apiMenuCategoryController;

    private List<MenuCategory> categories = new ArrayList<>();

    @Before
    public void setup() {
        JacksonTester.initFields(this, ObjectMapper::new);

        mockMvc = MockMvcBuilders.standaloneSetup(apiMenuCategoryController)
                .setControllerAdvice(new ExceptionHandlerExceptionResolver())
				//.setCustomArgumentResolvers(managerArgumentResolver)
                .setCustomArgumentResolvers(mockManagerArgumentResolver)
                .build();

        MenuCategory fstCategory = new MenuCategory(null, "카테고리1");
        fstCategory.setId(1l);
        fstCategory.addChild(new MenuCategory(1l, "카테고리1의 하위 카테고리"));
        categories.add(fstCategory);
        categories.add(new MenuCategory(null, "카테고리2"));

        manager = new Account("manager@gmail.com", "!Password1234", "manager", "manager@gmail.com", AccountType.MANAGER);
    }


    @Test
    public void createCategoryTest() throws Exception {
        //given
        when(menuCategoryService.create(any(MenuCategoryDTO.class)))
                .thenReturn(new MenuCategory());

        //when
        MockHttpServletResponse response = mockMvc.perform(
                post(URI_MENU_CATEGORY)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(jsonMenuCategoryDTO.write(
                                new MenuCategoryDTO(0l, "카테고리2의 하위 카테고리", 3l))
                                .getJson()))
                .andReturn().getResponse();

        //then
        log.debug(response.getContentAsString());
        assertThat(response.getStatus()).isEqualTo(HttpStatus.CREATED.value());
    }

    static class MockManagerArgumentResolver extends ManagerAccountHandlerMethodArgumentResolver {

        @Override
        public Object resolveArgument(MethodParameter methodParameter,
                                      ModelAndViewContainer modelAndViewContainer,
                                      NativeWebRequest nativeWebRequest,
                                      WebDataBinderFactory webDataBinderFactory) throws Exception {
            return new Account("managerByMock@gmail.com", "!Password1234", "manager", "manager@gmail.com", AccountType.MANAGER);
        }
    }
}
```

 

**실행결과**

![image-20190426211507115](/assets/images/image-20190426211507115.png)

Mocking한 return값인 `managerByMock@Gmail.com`이 파라미터로 전달된 것을 확인 할 수 있습니다.



이 방법을 사용하면 Spring 프레임워크의 기능을 덜 사용하여 POJO에 가까운 테스트 코드를 작성할 수 있습니다.















> 19.04.26
>
> <https://jojoldu.tistory.com/239> 에대한 학습을 더 한후에 글 추가하기.
> 해당 내용은 비슷한 상황에 spy 어노테이션이 동작하는것을 테스트한 내용이다.