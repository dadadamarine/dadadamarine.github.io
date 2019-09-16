# Abstract vs Interface

### 추상 클래스(abstract class)

**하나 이상의 추상 메소드(abstract method)를 포함하는 클래스이다.** 

추상 메소드는 선언만 있고 본체는 없는 함수이며 선언부에 '**abstract**' 라는 키워드를 붙인다.



### **인터페이스**(**interface**)

자바 프로그래밍 언어에서 **클래스들이 구현해야 하는 동작을 지정**하는데 사용되는 추상형이다.



---

**책 오브젝트 중**

>  **역할**의 개념을 적용하면 Movie가 구체적인 클래스는 알지 못한 채 오직 역할에 대해서만 결합되도록 의존성을 제한할 수 있다.
>
>  역할을 사용하면 객체의 구체적인 타입을 **추상화**할 수 있다. 
>
>  자바에서는 일반적으로 역할을 구현하기 위해 `추상 클래스`나 `인터페이스`를 사용한다.
>  역할을 대체할 클래스들 사이에서 구현을 공유해야 할 필요가 있다면 추상클래스를 사용하면 된다.
>  구현을 공유할 필요 없이 역할을 대체하는 객체들의 책임만 정의하고 싶다면 인터페이스를 사용하면 된다.



**Geeks for Geeks에 따르면**

**Abstraction:** Hiding the internal implementation of the feature and only showing the functionality to the users. i.e. what it works (showing), how it works (hiding). Both [abstract class](https://www.geeksforgeeks.org/abstract-classes-in-java/) and [interface](http://quiz.geeksforgeeks.org/interfaces-in-java/) are used for abstraction.

추상화 : 내부 구현을 숨기고, 유저에게 기능만 보여준다. 추상 클래스와 인터페이스가 이를 담당한다.







그렇다면 Java8에서 default 추가이후 왜 추상클래스가 필요한가? 를 알아보기 위해 default가 추가된 이유를 알아보자



### Default method

<https://blog.powerumc.kr/473>

원래 객체지향 언어에서 Interface는 그 시그너처와 선언이 변하지 않는다는 것을 전제로 하여 다형성(polymophism)을 정의하는 객체간의 규약이다. 그러나 Java8 Interface는 Interface를 업그레이드하는 개념을 도입하였다.

> Now users of your code can choose to continue to use the old interface or to upgrade to the new interface.
> Alternatively, you can define your new methods as default methods



**Side effect**

-> 이제 구현코드의 포함으로 인해 interface를 "규약 또는 약속"으로 정의하기가 모호해졌다.

-> 다시 다중 상속으로 인한 모호성 문제 (c++의 해결책과 유사)

즉, 에러가 난다. 구현시 명시적으로 재정의 해야함

```java
public interface OperateCar {
    default public int startEngine(EncryptedKey key) {
        // Implementation
    }
}
public interface FlyCar {
    default public int startEngine(EncryptedKey key) {
        // Implementation
    }
}
 
public class FlyingCar implements OperateCar, FlyCar {
    public int startEngine(EncryptedKey key) {
        FlyCar.super.startEngine(key);
        OperateCar.super.startEngine(key);
    }
}

```





[**default 추가이후 디폴트 메소드 vs 추상클래스**](https://wedul.site/320)

차이점

인터페이스는 private 값을 가지지 못한다.(오직 public, abstract, default, static 상태만 가질 수 있다.)

인터페이스는 생성자를 가질 수 없지만 추상클래스는 생성자를 가질 수 있다. ()

new AbstractClass(); ->

AbstractClass ab= new Class();

new Class()

Implements(구현) vs extends(상속)이라는 이름의 차이

다중 구현 가능 여부 

> 즉 추상클래스는 상속받아 기능을 확장하는것, 인터페이스는 구현한 객체의 같은 동작을 보장하기 위함이라는 목적은 유지가 된다고 볼 수 있을 것 같다.





다음중 하나라도 해당할 경우 추상 클래스 사용

1. 반드시 필요한 일부 코드를 공유해야 하는 경우
2. Static 또는 final이 아닌 **필드** 선언할 필요가 있고, 메서드를 통해 이 필드를 수정하거나 접근 할 필요가 있을때.
3. 즉 공통 메서드나 공통 필드를 가져야 할때, 혹은 publc이외의 접근제어자가 필요할때 추상 클래스를 사용한다.



다음 중 하나라도 해당할 경우 인터페이스를 사용

1. 완전한 추상화. 즉 이 인터페이스를 구현하는 클래스는 반드시 인터페이스에 선언된 메서드 모드를 구현해야한다.
2. 여러 인터페이스를 상속해야 할때
3. **특정 데이터 타입의 행위는 구체화 하지만, 누가 이 행위를 하는지는 상관이 없을 때.**



------

**추상클래스 와 상속의 사용 예시**

추상클래스는 객체의 추상적인 상위 개념으로 공통된 개념을 표현할 때 쓴다. 

위에 예를 들어, 동물에는 공통적으로 이름과 색깔이 있고, 달린다는 공통된 행동이 있다. 하지만, 각기 요구하는 객체구현 방식마다 객체가 달라지겠지만, 공통된 속성을 동물이라는 추상클래스에 추상메소드로 구현하여 하위 개념인 말이라는 객체에 상속시켜 구현하면 객체의 재사용성과 객체를 표현하는 것이 더 명확해진다고 생각한다. 

그리고 동물마다 차이가 생기는 태울 수(탈 수) 있는가, 없는가를 나누기 위해 RideInterface를 구분하여 동물 중에서 태울 수 있는 동물들에게 Implements 시키면 객체를 좀더 잘 표현할 수 있을거라고 생각한다.



AbstractAnimal{

  이름

}



Inter(달린다. )





------

**정의!**

<https://brunch.co.kr/@kd4/6>

인터페이스와 추상 클래스는 존재 목적이 다르다. 

`추상 클래스`는 **그 추상 클래스를 상속받아서 기능을 이용하고, 확장** 시키는 데 있다. 

` 인터페이스`는 함수의 껍데기만 있는데, 그 이유는 그 **함수의 구현을 강제하기 위해서** 입니다. 구현을 강제함 으로써 구현 **객체들의 같은 동작을 보장 할 수 있다.**

상속 : 슈퍼 클래스의 기능을 이용하거나 확장하기 위해 사용. 다중 상속의 모호성때문에 하나만 상속받을 수 있다.

인터페이스 : 해당 인터페이스를 구현한 객체들에 대해 동일한 동작을 약속하기 위해 존재한다.



> 다중상속의 모호성 : 상속하는 클래스들이 같은 메서드를 가질때. 문제발생

3. 