---
title: "[Spring/Project] @WebMvcTest ComponentScan으로 인한 에러 해결기"
date: 2019-04-24 02:25:28 -0400
categories: Java/Spring
---



# 개요

전의 글에서 2번에 해당하는 @WebMvcTest를 이용한 테스트 작성중에 에러가 발생하였다. 



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



아래와 같은 에러가 발생하였고 이 에러가 발생한 원인과 해결방법의 과정을 공유하고자 한다.

```java
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'accountService': Unsatisfied dependency expressed through field 'accountRepository'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'codesquad.domain.AccountRepository' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```



### 테스트 코드

```java
@RunWith(SpringRunner.class)
@WebMvcTest(ApiMenuCategoryController.class)
public class ApiMenuCategoryControllerWithApplicationContextTest {
    private static Logger log = LoggerFactory.getLogger(ApiMenuCategoryControllerWithApplicationContextTest.class);

    public static final String URI_MENU_CATEGORY = "/api/menuCategory";

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private MenuCategoryService menuCategoryService;

    private JacksonTester<MenuCategoryDTO> jsonMenuCategoryDTO;

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
        assertThat(response.getStatus()).isEqualTo(HttpStatus.BAD_REQUEST.value());
    }
}
```

해당 테스트는 ApiMenuCategoryController에 대한 단위테스트를 목적으로 작성되었다. 하지만 WebMvcTest + Application의 ComponentScan의 영향으로 다른  Controller들을 빈으로 등록하다가 에러가 발생한다.

```java
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'accountService': Unsatisfied dependency expressed through field 'accountRepository'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'codesquad.domain.AccountRepository' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}

```

<center>AccountService 안에서 @Autowired로 주입된 AccountRepository때문에 에러가 발생함 </center>



**@ComponentScan**

Application파일의 어노테이션의 @ComponentScan이 문제가 되는 것을 검색등을 통해 확인하였다.

```java
@SpringBootApplication
@EnableJpaAuditing
@ComponentScan({"codesquad","support"})
public class BaeminchanApplication {

    @Autowired
    public static void main(String[] args) {
        SpringApplication.run(BaeminchanApplication.class, args);
    }
}
```

참조

1. <https://stackoverflow.com/questions/47054716/componentscan-in-application-class-breaks-webmvctest-and-springboottest>



**Spring Document**

`@WebMvcTest`어노테이션은 일부의 빈들을 스캔하는데. 이 중 WebMvcConfigurer은 스캔하지만, @Configuration은 포함하지 않는다. 



> To test whether Spring MVC controllers are working as expected, use the `@WebMvcTest` annotation. `@WebMvcTest` auto-configures the Spring MVC infrastructure and limits scanned beans to `@Controller`, `@ControllerAdvice`, `@JsonComponent`, `Converter`, `GenericConverter`, `Filter`, `WebMvcConfigurer`, and `HandlerMethodArgumentResolver`. Regular `@Component` beans are not scanned when using this annotation. 



**문제 해결**

이를 활용하여 Application에 설정한 `@EnableJpaAuditing`, `@ComponentScan`을 다른 config파일로 분리한다.

이렇게 하면 WebMvcTest 중에는 ComponentScan어노테이션이 포함되지 않게 되고, 어플리케이션 동작에는 @Configuration파일을 스캔함으로써 해당 어노테이션인 `@EnableJpaAuditing` ,`@ComponentScan({"codesquad","support"})`을 스캔한다. 



```java
@Configuration
@EnableJpaAuditing
@ComponentScan({"support"})
public class JpaEnableConfiguration {
}
```



**실행 결과**

테스트가 실패 하였지만, 깨지는 오류는 해결된 것을 볼 수 있다.

![image-20190426154248783](/assets/images/image-20190426154248783-6262586.png)

### 
