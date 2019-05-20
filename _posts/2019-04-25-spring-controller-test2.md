---
title: "[Spring/Project] Controller 단위테스트 코드와 설명 2"
date: 2019-04-25 02:25:28 -0400
categories: Java/Spring
---







> 필자는 이 테스트가 단위테스트에 적합하지 않다고 판단해 글 마지막 부분에서 이 테스트 내용들을 AcceptanceTest로 전환하기로 결정하였습니다. 
>
> 또한 이 컨트롤러에 대한 단위테스트는 1번 방법으로 HandlerMethodArgumentResolver 를 Mocking하여 구현하고.
>
> 인증에 대한 단위테스트는 HandlerMethodArgumentResolverTest에 대한 단위테스트로 따로 작성하는게 옳다고 생각합니다.



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

이 글에선 Parameter에 Annotation이 있는 메서드를 테스트 하기 위한 2번 방법을 사용합니다.



> 코드상에 WebMvcTest를 동작시키면서 에러가 발생하였고 이에 관한 내용은 [@WebMvcTest ComponentScan으로 인한 에러 해결기](<https://dadadamarine.github.io/java/spring/project/spring-web-mvc-test-error/>) 에 있습니다.





## @ManagerAccount를 이용한 UnAuthorized 테스트



**테스트 코드**

```java

@RunWith(SpringRunner.class)
@WebMvcTest(value = ApiMenuCategoryController.class, secure = false)
public class ApiMenuCategoryControllerWithApplicationContextTest {
    private static Logger log = LoggerFactory.getLogger(ApiMenuCategoryControllerWithApplicationContextTest.class);

    public static final String URI_MENU_CATEGORY = "/api/menuCategory";

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private MenuCategoryService menuCategoryService;

    private JacksonTester<MenuCategoryDTO> jsonMenuCategoryDTO;
    private JacksonTester<MenuCategory> jsonMenuCategory;

    private List<MenuCategory> categories = new ArrayList<>();

    @Before
    public void setup() {
        JacksonTester.initFields(this, ObjectMapper::new);

        MenuCategory fstCategory = new MenuCategory(1l, "카테고리1");
        MenuCategory subCategory = new MenuCategory(2l, "카테고리1의 하위 카테고리");
        fstCategory.addChild(subCategory);
        subCategory.setParent(fstCategory);
        categories.add(fstCategory);
        categories.add(new MenuCategory(3l, "카테고리2"));
    }

    @Test
    public void create_test_not_manager_fail() throws Exception {
        //when
        MockHttpServletResponse response = mockMvc.perform(
                post(URI_MENU_CATEGORY)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(jsonMenuCategoryDTO.write(
                                new MenuCategoryDTO(0l, "카테고리2의 하위 카테고리", 3l))
                                .getJson()))
                .andReturn().getResponse();

        //then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.FORBIDDEN.value());
    }

    @Test
    public void delete_test_not_manager_fail() throws Exception {
        //when
        MockHttpServletResponse response = mockMvc.perform(
                delete(URI_MENU_CATEGORY + "/{id}", 1l))
                .andReturn().getResponse();

        //then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.FORBIDDEN.value());
    }

}
```



**컨트롤러 코드**

```java

@RestController
@RequestMapping("/api/menuCategory")
public class ApiMenuCategoryController {

    @Autowired
    MenuCategoryService menuCategoryService;

    @GetMapping("")
    public List<MenuCategory> getCategories() {
        return menuCategoryService.findCategories();
    }

    @PostMapping("")
    public ResponseEntity<MenuCategory> create(@ManagerAccount Account manager, @RequestBody MenuCategoryDTO menuCategoryDTO) {
        MenuCategory createdCategory = menuCategoryService.create(menuCategoryDTO);
        return makeCreatedResponseEntity(createdCategory);
    }

    @DeleteMapping("/{id}")
    public MenuCategory delete(@ManagerAccount Account manager, @PathVariable Long id) {
        return menuCategoryService.deleteById(id);
    }
}

```



**동작 순서 설명**

1. **WebMvcTest 이므로, 설정한 컨트롤러와 함께 HandlerMethodArgumentResolver이 등록된다.**

![image-20190426164916982](/Users/dadadamarine/Desktop/study/blog/dadadamarine.github.io/_posts/assets/images/image-20190426164916982.png)

<center>이 테스트의 핵심이 되는 ManagerAccountHandlerMethodArgumentResolver이 등록됨.</center>



2. **테스트 코드의 실행과 컨트롤러 매핑**

```java
// ApiMenuCategoryControllerWithApplicationContextTest

@Test
    public void create_test_not_manager_fail() throws Exception {
        //when
        MockHttpServletResponse response = mockMvc.perform(
                post(URI_MENU_CATEGORY)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(jsonMenuCategoryDTO.write(
                                new MenuCategoryDTO(0l, "카테고리2의 하위 카테고리", 3l))
                                .getJson()))
                .andReturn().getResponse();

        //then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.FORBIDDEN.value());
    }
```



이를 통해 @ManagerAccount가 파라미터로 붙은 컨트롤러에 대해 권한체크를 수행한다.

```java
// ApiMenuCategoryController

@PostMapping("")
    public ResponseEntity<MenuCategory> create(@ManagerAccount Account manager, @RequestBody MenuCategoryDTO menuCategoryDTO) {
        MenuCategory createdCategory = menuCategoryService.create(menuCategoryDTO);
        return makeCreatedResponseEntity(createdCategory);
    }

```



3. **HandleMethodArgumentResolver 동작**

[`document `](<https://docs.spring.io/spring/docs/3.1.x/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html>)

파라미터 값을 전달하기전에 resolveArgument 메서드를 수행하게 되고

```java
public class ManagerAccountHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter methodParameter) {
        return methodParameter.hasParameterAnnotation(ManagerAccount.class);
    	// methodParameter 가 이 Resolver로 처리가능한지 여부를 검사하여 true/false 리턴
        // 현재 @ManagerAccount Account manager로 파라미터가 @ManagerAccount라는 파라미터 어노테이션을 가지고 있으므로 true를 반환하고, resolveArgument를 실행한다.
    }

    @Override
    public Object resolveArgument(MethodParameter methodParameter,
                                  ModelAndViewContainer modelAndViewContainer,
                                  NativeWebRequest nativeWebRequest,
                                  WebDataBinderFactory webDataBinderFactory) throws Exception {
        // methodParameter를 주어진 request의 인자값으로 변환한다.
        // 이 메서드에서 파라미터 형태의 실제 객체를 리턴할경우 return 값이 컨트롤러 메소드의 인자인 manager로 들어가게 됨.
        Account account = SessionUtils.getUserFromSession(nativeWebRequest);
        if (!account.isManager()) {
            throw new UnAuthorizedException("You're not manager!");
        }
        return account;
    }
}
```

세션의 유저를 받아서 ManagerType계정이 아닐경우 에러를 던지고, 맞을 경우만 계정을 반환해서 파라미터의 manager변수에 매핑해준다.

  

resolverArgument 메서드의 첫번재 줄에서 SessionUtils.getUserFromSession 메서드를 실행한다.

```java
// SessionUtils

public static Account getUserFromSession(NativeWebRequest nativeWebRequest) {
        if (!isLogin(nativeWebRequest)) {
            throw new UnAuthorizedException("not login");
        }
        return (Account) nativeWebRequest.getAttribute(USER_SESSION_KEY, NativeWebRequest.SCOPE_SESSION);
    }
```

세선이 없을경우 UnAuthorizedException를 던지고 ControllerAdvice에서 이를 처리한다.

세션이 있을경우 세션의 유저를 반환한다.



4. **ControllerAdvice 동작**

```java
// ExceptionResponseEntityExceptionHandler : @ControllerAdvice

@ExceptionHandler(UnAuthorizedException.class)
@ResponseStatus(value = HttpStatus.FORBIDDEN)
public void unAuthorized() {
    log.debug("UnAuthorizedException is happened!");
}
```

Status.FORBIDDEN으로 response가 테스트코드 쪽으로 전달된다.



**결과**

해당 테스트로 @ManagerAccount가 정상적으로 동작함을 확인 할 수 있었다.

![image-20190426161711389](/Users/dadadamarine/Desktop/study/blog/dadadamarine.github.io/_posts/assets/images/image-20190426161711389.png)



**어떻게 NativeWebRequest에서 세션정보를 가져오는 것인가?**

+ NativeWebRequest란?

  + ttpServletRequest 의 요청 정보를 대부분 그대로 갖고 있는, 서블릿 API 에 종속적이지 않은 오브젝트 타입입니다.

    출처: <https://springsource.tistory.com/13> [Rednics Blog]





## ~~Interceptor를 이용한 권한 통과 테스트~~

**Web Basic authentication**

![img](https://docs.microsoft.com/en-us/aspnet/web-api/overview/security/basic-authentication/_static/image1.png)

<center> Basic Authentication의 흐름</center>

[참조 자료](<https://docs.microsoft.com/en-us/aspnet/web-api/overview/security/basic-authentication>) , [HTTP 기본인증 (Basic authentication)](<https://hamait.tistory.com/416>)

Basic authentication은 HTTP프로토콜이 제공하는 자체적인 인증 기능이다. 이는 HTTP헤더 내의 제어 헤더의 내용으로 '인증' 기능을 제공하는 것이다. 

이 내용을 응용한다. 테스트상에서  request의 Authrization 헤더를 설정하고, 인증정보를 전송하면 스프링에선 Intercepter에서 Session에 유저의 정보를 등록 (로그인) 시켜줄 수 있다. 즉 컨트롤러 실행전에 로그인을 실행해 주는것이다. 이는 따로 로그인을 하지 않아도 로그인 된채로 컨트롤러를 실행 시킬수 있음을 의미한다.



설정해야 하는 내용은 다음과 같다.

1. **테스트에서 BasicAuthentication을 헤더에 담아 전송하는 `when`에 해당하는 테스트 코드 작성**
2. **BasicAuthentication 헤더정보를 가지고 컨트롤러 실행전에 Session에 정보를 등록해줄 BasicAuthenticationIntercepter생성**



## 단위 테스트에 대한 개념정리

위의 내용들은 컨트롤러 단위테스트가 아닌 통합테스트의 방식으로 생각된다.

컨트롤러의 단위테스트는 컨트롤러 로직을 테스트하는 개념이므로 ManagerAccountHandlerMethodArgumentResolver을 Mocking해서 컨트롤러의 로직만 테스트했어야했다.

이 인증 로직은 MockitoJUnitRunner를 활용 HandlerMethodArgumentResolverTest라는 테스트 코드를 작성함으로 써 따로 단위테스트가 가능하다. 



# 결론

해당 테스트는 단위테스트의 범위를 조금 벗어난 spring의 주변요소와 함께하는 컨트롤러에 대한 단위테스트였다.

이 테스트를 통해서 @AdminUser라는 어노테이션으로 로그인 되지 않은 유저에 대해 권한 제어가 정상적으로 이루어 짐을 확인 할 수 있었다.

다음은 **@AdminUser 어노테이션의 기능을 Mocking해서 컨트롤러의 비즈니스 로직에 대한 단위테스트**를 구현한다.

그 후 **로그인한 유저에 대한 AcceptanceTest**도 구현하고 테스트를 마무리한다.