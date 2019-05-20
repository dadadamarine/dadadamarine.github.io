---
title: "[Spring] Column unique 제약조건의 에러처리 삽질기"
date: 2019-04-19 02:25:28 -0400
categories: Java/Spring
---



# 개요

Spring의 Entity 에서 Column(unique = true) 제약조건에 대한 핸들링을 하고싶었다.

![image-20190421171034376](/assets/images/image-20190421171034376.png)



같은 userId로 회원가입 했을때 unique 제약조건에에 대한 에러 핸들링을 처리하고 싶다.



> 결론부터 말하자면 DataIntegrityViolationException / ConstraintViolationException 으로 Controller Advice 에서 또는 비즈니스 로직에서 try catch 구문으로 처리하시면 됩니다.





# 본문



### 1. 테스트 코드 작성

먼저 어떤 에러가 발생하는지 알기위해 중복되는 아이디로 계정을 생성하는 Acceptance Test를 실행시켰다.

```
@Test
public void create_not_unique_userId() throws Exception {
    AccountRegistrationDTO account = AccountRegistrationDTO
            .builder("test@google.com", "!Password1234", "!Password1234", "name")
            .phoneNumber("010-1234-1234")
            .email("test@google.com")
            .type(AccountType.MEMBER)
            .build();

    ResponseEntity<String> response = sendPost("/member", account, String.class);

    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
    assertThat(accountRepository.findByUserId(account.getUserId()).isPresent()).isEqualTo(true);
}
```



**결과**

![image-20190421171423439](/assets/images/image-20190421171423439.png)

Mysql Connector에서 MySQLIntegrityConstraintViolationException이 발생한것을 볼 수 있다.



**필자 생각**

필자는 MySQLIntegrityConstraintViolationException로 예외처리를 할경우 sql의 종류에 종속적인 코드가 될것이라고 판단하였고.

클래스의 extends를 타고올라가서 Connector에서 상속받아 구현한 클래스 보다 추상화레벨이 높은 클래스 SQLIntegrityConstraintViolationException 를 사용하기로 하였다.

![image-20190421171659036](/assets/images/image-20190421171659036.png)

<center> SQLIntegrityConstraintViolationException의 모습 </center>



ControllerAdvice를 이용하기전에 먼저 tryCatch구문으로 구현해 보기로 한다.



### 2. 에러처리 구현

**구현**

![image-20190421172003595](/assets/images/image-20190421172003595.png)

뭐..? 뻔히 테스트에서 에러가 로그에찍혔었는데 해당 에러가 발생하지 않는다고..?

- 혹시나 해서 MySQLIntegrityConstraintViolationException로도 catch 해보았지만 마찬가지

- Test에 expected를 써볼까..?

  ![image-20190421172308293](/assets/images/image-20190421172308293.png)

  <center>expected를 써보아도 테스트가 깨진다.</center>



흠.. 구글링을 해보자

![image-20190421173340430](/assets/images/image-20190421173340430.png)

<center> DataIntegrityViolationException을 사용한다.</center>



출처 : <https://stackoverflow.com/questions/27582757/catch-duplicate-entry-exception>



…?? 왜지 대체왜지..? 왜 보지도 못했던 Exception으로 에러를 처리하니까 catch가 되는거지? 

하는 생각에 DataIntegrityViolationException를 에러에서 찾아보았다.

![image-20190421174109336](/assets/images/image-20190421174109336.png)

있다!!..?



내가 봤던 예외의 바로 위에서 던져진 예외였다.![image-20190421174147852](/assets/images/image-20190421174147852.png)



**내용**

```java
2019-04-21 17:36:05.532 ERROR 10016 --- [o-auto-1-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [
Request processing failed; 
    nested exception is org.springframework.dao.DataIntegrityViolationException: could not execute statement; SQL [n/a]; constraint [UK_h6dr47em6vg85yuwt4e2roca4]; 
    nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement
] with root cause

```

에러의 내용으로 봤을때 dispatcherServlet에서 던져진 에러다.  또한 hibernate의 ConstraintViolationException도 같이 던져진다.



**필자생각**

Spring 초심자의 생각의 흐름에 따라 이런 결론에 도달했다.

> test에서 expected로도 검사가 안되고, catch로도 검사가 안된다. 그렇다면 스프링 내부(dispatcherServlet)에서 내가 검사하고 싶었던MySQLIntegrityConstraintViolationException 를 먼저 검사해서 가공된 예외를 던지는게 아닐까? 



### 3. 예외를 가공해서 던져주는 코드 탐색

dispatcherServlet에서 던지는 예외 vs POJO에서 검출되는 예외의 차이 에 대한 학습을 할 필요성을 느꼈다. 어떤 구조로 예외가 가공되어 던져지는지 확인해보자.



에러발생의 객체인 dispatcherServlet과 Servlet을 뜯어본다.



**Servlet**

![image-20190421175836765](/assets/images/image-20190421175836765.png)

Servlet은 역시 인터페이스이다. 그리고 빈으로 관리되고 있다.

구현체로 가보자.

![image-20190421175911565](/assets/images/image-20190421175911565.png)

<center>옆의 콩버튼을 통해 빈으로 관리되고있는 Servlet인터페이스의 구현체를 찾아갈 수 있다.</center>



**DispatcherServletAutoConfiguration**


Servlet의 configuration에 도착하였고.

![image-20190421180020496](/assets/images/image-20190421180020496.png)

![image-20190421180103995](/assets/images/image-20190421180103995.png)

그렇다. dispatcherServlet을 구현체로 사용하고 있다.

dispatcherServlet클래스로 가보자.



**dispatcherServlet**

```java
public class DispatcherServlet extends FrameworkServlet {

...

@Nullable
    private List<HandlerExceptionResolver> handlerExceptionResolvers;
    
}
```

handlerExceptionResolvers를 리스트로 관리하고있다.



**dispatcherServlet.initHandlerExceptionResolvers()**

```java
 private void initHandlerExceptionResolvers(ApplicationContext context) {
        this.handlerExceptionResolvers = null;
        if (this.detectAllHandlerExceptionResolvers) {
            Map<String, HandlerExceptionResolver> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerExceptionResolver.class, true, false);
            if (!matchingBeans.isEmpty()) {
                this.handlerExceptionResolvers = new ArrayList(matchingBeans.values());
                AnnotationAwareOrderComparator.sort(this.handlerExceptionResolvers);
            }
        } else {
            try {
                HandlerExceptionResolver her = (HandlerExceptionResolver)context.getBean("handlerExceptionResolver", HandlerExceptionResolver.class);
                this.handlerExceptionResolvers = Collections.singletonList(her);
            } catch (NoSuchBeanDefinitionException var3) {
            }
        }

        if (this.handlerExceptionResolvers == null) {
            this.handlerExceptionResolvers = this.getDefaultStrategies(context, HandlerExceptionResolver.class);
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("No HandlerExceptionResolvers found in servlet '" + this.getServletName() + "': using default");
            }
        }

    }
```

여기서 빈으로 등록된 HandlerExceptionResolver를 리스트에서 추가해준다.



**dispatcherServlet.processHandlerException()**

```java
@Nullable
    protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) throws Exception {
        ModelAndView exMv = null;
        if (this.handlerExceptionResolvers != null) {
            Iterator var6 = this.handlerExceptionResolvers.iterator();

            while(var6.hasNext()) {
                HandlerExceptionResolver handlerExceptionResolver = (HandlerExceptionResolver)var6.next();
                exMv = handlerExceptionResolver.resolveException(request, response, handler, ex);
                if (exMv != null) {
                    break;
                }
            }
        }

        if (exMv != null) {
            if (exMv.isEmpty()) {
                request.setAttribute(EXCEPTION_ATTRIBUTE, ex);
                return null;
            } else {
                if (!exMv.hasView()) {
                    String defaultViewName = this.getDefaultViewName(request);
                    if (defaultViewName != null) {
                        exMv.setViewName(defaultViewName);
                    }
                }

                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Handler execution resulted in exception - forwarding to resolved error view: " + exMv, ex);
                }

                WebUtils.exposeErrorRequestAttributes(request, ex, this.getServletName());
                return exMv;
            }
        } else {
            throw ex;
        }
    }
```

handlerExceptionResolvers 리스트를 돌면서 예외 resolve를 진행한다.



**필자생각**

> 그렇다면, default로 리스트에 등록된 ExceptionResolver중에 우리가 찾는 녀석 (여러 에러들을 catch해서 가공시켜 던져주는 녀석)이 있는게 아닐까.
>
> 리스트를 찍어보자.
>
> private인 리스트를 출력하기위해 get메서드도 찾아보고, DispatcherServlet을 상속받는 클래스도 만들어보고.. 여러 삽질을 한 자바초보. 처음부터 디버깅하는게 빨랐을거같다.



**dispatcherServlet.processHandlerException()** 디버깅

![image-20190421182934865](/assets/images/image-20190421182934865.png)

![image-20190421183016174](/assets/images/image-20190421183016174.png)



**ExceptionHandlerExceptionResolver**

![image-20190421183625163](/assets/images/image-20190421183625163.png)

소스를 보니 여러 에러처리 관련된 객체인듯 하다.



**ResponseStatusExceptionResolver**

![image-20190421183723495](/assets/images/image-20190421183723495.png)

message.property사용 관련된 객체로 보인다.



**DefaultHandlerExceptionResolver**

![image-20190421183850229](/assets/images/image-20190421183850229.png)

![image-20190421183925926](/assets/images/image-20190421183925926.png)

기본 에러 404 등등을 관리하는 객체로 보인다.



가장 필자가 찾는것과 적합해보이는 ResponseStatusExceptionResolver를 읽어보니..

![image-20190421184151293](/assets/images/image-20190421184151293.png)

무수히 많은 Resolver들을 등록하고있다.



더 코드를 뜯어보면서 찾는건 학습 가성비가 떨어지므로 질문을 하기로한다. 꼭 sql exception을 다른 에러로 변환해주는 코드를 눈으로 확인하고싶다.



# 결론

처음 MySQLIntegrityConstraintViolationException의 검출로 시작했던 학습이

Dispatcher Servlet의 기능과 속성들을 파악하는 것으로 이어졌다.

이후 spring 에러처리에 대한 학습이 마무리 되면 정리해서 블로깅을 다시 하도록 하자.

