---
title: "[Spring/번역] Spring Boot RESTful Service를 위한 유효성 검사방법"
date: 2019-04-08 16:09:28 -0400
categories: Java/Spring
tags : Java/Spring
---

> 이글은 [IN 28 MINUTES](http://www.springboottutorial.com/)님의 게시글인  **"[Implementing Validation for RESTful Services with Spring Boot](http://www.springboottutorial.com/spring-boot-validation-for-rest-services)"** 을 번역한 글임을 밝힙니다.  
>


이 가이드는 Spring Boot 에서 REST API/Service에 대한 유효성 검사를 효율적으로하도록 도와줄것이다.  

# 학습내용

- 유효성 검사(validation) 란?

- 왜 유효성 검사가 필요한가?

- Hibernate Validator가 무엇인가?

- Bean Validation API가 무엇인가?

- 스프링 부트가 제공하는 기본 유효성 검사 기능은 무엇인가?

- 스프링부트에서 유효성 검사를 어떻게 수행하는가?

- Bean Validation API로 어떻게 유효성 검사를 수행하는가?  

# REST API 추천 강좌

[![Image](http://www.springboottutorial.com/images/Course-Go-Full-Stack-With-Spring-Boot-and-React.png)](https://www.udemy.com/full-stack-application-with-spring-boot-and-react/?couponCode=SBT-2019)

[![Image](http://www.springboottutorial.com/images/Course-Go-Full-Stack-With-SpringBoot-And-Angular.png)](https://www.udemy.com/full-stack-application-development-with-spring-boot-and-angular/?couponCode=SBT-2019)

[![Image](http://www.springboottutorial.com/images/Course-Master-Microservices-with-Spring-Boot-and-Spring-Cloud.png)](https://www.udemy.com/microservices-with-spring-boot-and-spring-cloud/?couponCode=SBT-2019)



## 10 Step Reference Courses

- [Spring Framework for Beginners in 10 Steps](https://courses.in28minutes.com/p/spring-framework-for-beginners)
- [Spring Boot for Beginners in 10 Steps](https://courses.in28minutes.com/p/spring-boot-for-beginners-in-10-steps)
- [Spring MVC in 10 Steps](https://www.youtube.com/watch?v=BjNhGaZDr0Y)
- [JPA and Hibernate in 10 Steps](https://courses.in28minutes.com/p/jpa-and-hibernate-tutorial-for-beginners-with-spring-boot)
- [Eclipse Tutorial for Beginners in 5 Steps](https://courses.in28minutes.com/p/eclipse-tutorial-for-beginners)
- [Maven Tutorial for Beginners in 5 Steps](https://courses.in28minutes.com/p/maven-tutorial-for-beginners-in-5-steps)
- [JUnit Tutorial for Beginners in 5 Steps](https://courses.in28minutes.com/p/junit-tutorial-for-beginners)
- [Mockito Tutorial for Beginners in 5 Steps](https://courses.in28minutes.com/p/mockito-for-beginner-in-5-steps)
- [Complete in28Minutes Course Guide](https://courses.in28minutes.com/p/in28minutes-course-guide)

  

# 예시 프로젝트 코드 구조

아래 파일들은 우리가 만들 프로젝트의 중요한 구성요소들이다. 몇가지 요소를 자세히 보자

- `SpringBoot2RestServiceApplication.java` - Spring Initializer로 생성된 스프링부트 어플리케이션 클래스. 어플리케이션의 시작점을 담당한다.
- `pom.xml` - 프로젝트를 구성하기위한 모든 의존성을 포함하는 파일. 이 프로젝트 에서는 Spring Boot Starter AOP를 사용한다.
- `Student.java` - Student JPA Entity
- `StudentRepository.java` - Student JPA Repository. Spring Data의 JpaRepository인터페이스를 사용하여 만든다.
- `StudentResource.java` - student 리소스에 대한 모든 서비스들을 보여주는 Spring Rest Controller
- `CustomizedResponseEntityExceptionHandler.java` - 글로벌 예외처리를 수행하고 예외 타입에 따라 response를 커스터마이징 하기위한 컴포넌트
- `ErrorDetails.java` - API에서 예외들을 던질때 사용하기 위한 Response Bean
- `StudentNotFoundException.java` - student가 탐색되지 않을때 던지는 에러
- `data.sql` - student table을 위한 초기데이터 파일. 스프링 부트는 엔티티들로부터 테이블이 생성된 이후에 이 스크립트 파일을 실행한다.

  

## Tools you will need

- Maven 3.0이상의 빌드 툴
- 선호하는 IDE. 본문은 Eclipse를 사용한다.
- JDK 1.8+

  

## Complete Maven Project With Code Examples

> [완성된 예시코드가 있는 Github 리파지토리](https://github.com/in28minutes/spring-boot-examples/tree/master/spring-boot-2-rest-service-validation)

  

# 1. 유효성 검사란?

우리는 보통 RESTful Service가 특정한 포맷의 request만을 수신할 것으로 예상한다. 또 이 request의 요소들이 특정한 데이터 타입과, 특정한 도메인 제약사항들을 가졌을 것이라고 생각한다.

만약, 이 제약사항을 만족하지 않는 request를 받는경우는 어떻게 될까?

잠시 생각해보라. 어떻게 해야할까?

 `Something went wrong.`.  이라는 메세지를 돌려주는 방법은 어떨까. 충분할까?

RESTful service의 핵심 디자인 원리중 하나는 아래와 같다.

> 사용자에 대해 고민하자. (Think about the consumer)

그래서, request의 일부가 제약사항을 어겼을때, 어떻게 해아할까?

적합한 error response를 리턴해줘야한다.

- 어떤점이 잘못됬는지 나타내는 메세지 : 어느 field가 에러를 가지고 있고, 올바른 field의 형태는 무엇인가, 어떻게 에러를 해결하는가
- 올바른 Reponse 상태코드 : Bad Request.
- Response에 중요한 정보는 포함시키면 안된다.  

  

## 유효성 에러에 대한 Response 상태코드

유효성 에러에 대한 권장 상태코드는 400 - BAD REQUEST 이다.  

  

## REST Resouce로 프로젝트를 간단하게 구성하기

[이전 기사](http://www.springboottutorial.com/spring-boot-crud-rest-service-with-jpa-hibernate)에서,CRUD 메서드를 포함한 리소스로 간단한 restful service를 생성했다.

> 예외 핸들링에 대해 공부하기 위해 같은 샘플코드를 사용할 것이다.  

---

# 2. 스프링 부트로 기본적인 Validation 하기

스프링 부트는 RESTful Service들을 위해 좋은 기본 기능들을 제공한다. 이 기본 예외 핸들링의 특징들을 빠르게 살펴보자.



## 잘못된 컨텐츠 타입

만일 Content-Type을 `application/xml` 로 사용하고, 어플리케이션에서 지원하지 않는 경우, 스프링 부트는 기본적으로  415 - Unsupported Media Type 응답상태를 반환합니다. 



## 타당하지 않은 JSON Content

body(본문)을 기다리는 메소드에 잘못된 JSON 컨텐르츠를 보내면 400 - Bad Request를 반환합니다.



## 타당하지만 누락된 요소가 있는 JSON

만약 타당한 JSON 구조에 속성/요소를 없거나/타당하지 않게 보내면,  어플리케이션은쓸수있는 아무데이터나 포함시켜서 request를 실행합니다.  

**아래의 request는 201 Created 상태코드로 실행됩니다.**

POST `http://localhost:8080/students`

비어있는 Request Content

```java
{
  
}
```

  

**아래의 request도 201 Created 상태코드로 실행됩니다.**

POST `http://localhost:8080/students`

Request Content

```java
{
    "name1": null,
    "passportNumber": "A12345678"
}
```

위 요청에 잘못된 속성인 name1이 있음을 알 수 있다.  



아래는 `http://localhost:8080/students` 에 GET요청을 보냈을때의 Response이다.

[ { “id”: 1, “name”: null, “passportNumber”: null }, { “id”: 2, “name”: null, “passportNumber”: “A12345678” }, { “id”: 10001, “name”: “Ranga”, “passportNumber”: “E1234567” }, { “id”: 10002, “name”: “Ravi”, “passportNumber”: “A1234568” } ]

id가 1과 2인 리소스에서 바르지 않은값이 null로 생성된 것을 확인 할 수 있다. 타당하지 않은 요소나 속성들은 무시된다.  

  

# 3. 유효성 검사 커스터마이징 하기. 

유효성을 커스터마이징 하기 위해서, 이글에서는 bean validation api의 구현체인 Hibernate Validator를 사용한다.

Spring Boot Starter Web을 사용하면 Hibernate Validator를 바로 이용 할 수 있다.

먼저, 유효성 검사를 구현하는 것부터 시작하자.



## Bean으로 제약조건 주기

몇가지 제약조건을 Student Bean에 추가하자. @Size 어노테이션을 사용하면 최소값과 유효성 에러가 났을때의 메세지를 구체화 할 수 있다.

```java
@Entity
public class Student {
  @Id
  @GeneratedValue
  private Long id;
  
  @NotNull
  @Size(min=2, message="Name should have atleast 2 characters")
  private String name;
  
  @NotNull
  @Size(min=7, message="Passport should have atleast 2 characters")
  private String passportNumber;
  
```

  

Bean Validation API는 이런 종류의 어노테이션을 많이 제공한다. 대부분은 이해하기 쉬운 것들이다.

- DecimalMax
- DecimalMin
- Digits
- Email
- Future
- FutureOrPresent
- Max
- Min
- Negative
- NegativeOrZero
- NotBlank
- NotEmpty
- NotNull
- Null
- Past
- PastOrPresent
- Pattern
- Positive
- PositiveOrZero

  

## Resource에서 유효성 검사를 하는 방법

간단하다. @Valid 어노테이션을 @RequestBody에 추가하라.

```java
public ResponseEntity<Object> createStudent(@Valid @RequestBody Student student) {
```

끝이다.

제약조건과 맞지않는 속성값을 가지고 request를 보내면, 404 BAD Request상태 코드가 리턴된다.

  

Request

```java
{
    "name": "",
    "passportNumber": "A12345678"
  }
```

하지만 문제는 무엇이 잘못되었는지에 대한 설명이 포함되어 있지 않다는 것이다.

- 사용자는 이것이 bad request라는 것을 안다.
- 하지만, 무엇을 잘못한 것인지는? 어떤 요소가 유효성검사를 통과하지 못했을까?어떻게 고쳐야 할까?

  

## 유효성 검사 Response를 커스터마이징 하기

간단한 에러 response bean을 정의해 보자.

```java
public class ErrorDetails {
  private Date timestamp;
  private String message;
  private String details;

  public ErrorDetails(Date timestamp, String message, String details) {
    super();
    this.timestamp = timestamp;
    this.message = message;
    this.details = details;
  }
```



이제 @ControllerAdvice 를 정의하여 유효성 검사 오류를 처리해 보자. `ResponseEntityExceptionHandler` 안의 handleMethodArgumentNotValid(MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatus status, WebRequest request) 메서드를 오버라이딩 함으로써 구현 할 수 있다.

```java
@ControllerAdvice
@RestController
public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

  @Override
  protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex,
      HttpHeaders headers, HttpStatus status, WebRequest request) {
    ErrorDetails errorDetails = new ErrorDetails(new Date(), "Validation Failed",
        ex.getBindingResult().toString());
    return new ResponseEntity(errorDetails, HttpStatus.BAD_REQUEST);
  } 
```


`ErrorDetails`를 사용하여 에러 response를 반환하려면, 아래와 같이 ControllerAdvice를 정의하면 된다.

```java
@ControllerAdvice
@RestController
public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

  @ExceptionHandler(StudentNotFoundException)
  public final ResponseEntity<ErrorDetails> handleUserNotFoundException(StudentNotFoundException ex, WebRequest request) {
    ErrorDetails errorDetails = new ErrorDetails(new Date(), ex.getMessage(),
        request.getDescription(false));
    return new ResponseEntity<>(errorDetails, HttpStatus.NOT_FOUND);
  }
```

제약조건을 위반한 속성을 가지고 request를 실행하면, 404 BAD Request 상태가 리턴된다.


Request

```java
{
    "name": "",
    "passportNumber": "A12345678"
  }
```


또한, 무엇이 잘못 되었는지 알려주는 Response Body도 받을 수 있다!

```json
{
  "timestamp": 1512717715118,
  "message": "Validation Failed",
  "details": "org.springframework.validation.BeanPropertyBindingResult: 1 errors\nField error in object 'student' on field 'name': rejected value []; codes [Size.student.name,Size.name,Size.java.lang.String,Size]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [student.name,name]; arguments []; default message [name],2147483647,2]; default message [Name should have atleast 2 characters]"
}
```

이제 여러분은 필요에 따라 유효성 검증 에러 메세지를 커스터마이징 할 수 있는 준비가 되었다. 행운을 빈다!











