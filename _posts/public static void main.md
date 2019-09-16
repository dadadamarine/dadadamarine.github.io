

# [[Java\] public static void main(String [] args)](https://devbox.tistory.com/entry/Java-public-static-void-mainString-args)

<https://devbox.tistory.com/entry/Java-public-static-void-mainString-args?category=574549>



**public static void main(String[] args)**



**public**

main메서드를 public으로 설정해서, 전역적으로 이용 가능하게 한다. 
또한 public이라서 JVM이 현재클래스에 존재하지 않더라도, 클래스 외부에서 invoke할 수 있다.

- 현재 클래스 외부에서이 메소드를 호출 할 수 있음을 의미합니다.이 메소드는 현재 클래스에없는 Java 런타임 시스템에서 호출하기 때문에 필요합니다.

java.exe는 간단한 C코드로 작성된 어플리케이션인데, command line을 파싱하고, jvm안에 새 string array을 생성해서 인자들을 가지게 하고, main메서드를 포함한 클래스 이름을 분석한다.

>### 2. Why Java main method is public?
>
><https://howtodoinjava.com/java/basics/main-method/#internal>
>
>

JNI를 호출하여 main메서드를 찾고 파라미터를 넘기면서 invoke()한다. (이는 java의 reeflection과 유사하다.) -> java.c파일 즉 c코드에는 다음과 같은 코드가 있다.

```c

/* Get the application's main method */
mainID = (*env)->GetStaticMethodID(env, mainClass, "main", "([Ljava/lang/String;)V");
... ...
 
{/* Make sure the main method is public */
jint mods;
jmethodID mid;
jobject obj = (*env)->ToReflectedMethod(env, mainClass, mainID, JNI_TRUE);
... ...
 
/* Build argument array */
mainArgs = NewPlatformStringArray(env, argv, argc);
if (mainArgs == NULL) {
ReportExceptionDescription(env);
goto leave;
}
 
/* Invoke main method. */
(*env)->CallStaticVoidMethod(env, mainClass, mainID, mainArgs);
```



**static**
JVM은 main메서드를 invoking함으로써 자바 프로그램을 실행한다.

- static이란 메서드와 사용될 때, 메서드를 class method(!= instance method)로 만드는 키워드이다. JVM이 클래스의 인스턴스를 생성 하지 않고, 메서드를 실행 할 수 있게한다.
  이로써 단순히 main메서드를 호출하기 위해서 인스턴스를 생성함 으로써 발생 될 불필요한 메모리 낭비를 줄인다.

- static을 써야하는 이유는, 어떤 생성자가 호출되어야 하는지 모호해지기 때문이다. 

  이게 없다면, 자바프로그램이 빌드시마다 각자의 entry 메서드를 구체화해야한다. 클래스에 기본생성자가 없다면?, 파라미터가 있는 생성자를 호출해야 한다면 어떤 인자를 넘겨야하나? 등
  즉, Entry point에 해당하는 main메서드를 호출하기전에 인스턴스화 하는 경우, 너무 많은 케이스와 모호성이 있기때문에.

\- 메인 메서드는 항상 정적이어야 한다. 클래스는 메모리에 로딩된 다음에 사용이 가능하다. static이 붙은 클래스나 메서드, 변수는 컴파일시 자동으로 로딩된다. 메인 메서드는 클래스 로딩 없이 호출할 수 있어야 한다. 그렇기 때문에 static을 사용한다. --> static

**void**

메인 메서드가 끝나자 마자, 자바 프로그램이 종료된다. 따라서 JVM이 이 리턴 값으로 아무것도 하지 않으므로 다른 반환 타입을 선언 할 필요가 없다.



**main** 

main 메서드는 자바 어플리케이션의 entry 포인트이다.

자바 메인메서드의 이름이다. 이 이름은 JVM이 자바 프로그램의 시작점을 탐색할 때 사용하는 식별자이다.




**String[] args**

자바 커맨드라인 인자들을 저장한다. 이때 커맨드라인들은 Java.lang.String 클래스 타입의 배열로 전달된다. 이때 이 이름은 고정되어 있지 않아 사용자가 다른 이름을 사용해도 된다. 



\- String[] args 는 프로그램 실행시 매개변수를 보내서 실행할 수 있다는 것을 뜻한다. 1개를 사용할수도 있고 여러개를 사용할 수도 있기 때문에 배열을 사용한다.


