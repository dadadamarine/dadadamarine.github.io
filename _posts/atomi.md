동기화 문제

가시성 문제, 경쟁상태 문제



**가시성 문제**

변수의 가시성 문제는 아래와 같다.

변수의 값은 CPU 메모리와 메인 메모리에 저장된다.

이 값을 CPU 메모리인지 메인 메모리에서 가져오는 지 알 수가 없다는 문제가 변수의 가시성 문제이다.

아래 메모리 구조를 보면 이해하길 바란다.

![메모리 구조](https://t1.daumcdn.net/cfile/tistory/21166D4E588D52070F)



이렇게 CPU 메모리에서 값을 읽어들인다면 매우 안전하지 못한다.



그래서 사용되는 것이 CAS 알고리즘이다. **현재 쓰레드에 저장된 값과 메인 메모리에 저장된 값을 비교하여 일치하는 경우 새로운 값으로 교체하고, 일치 하지 않는 다면 실패하고 재시도를 한다.** 이렇게 처리되면 CPU캐시에서 잘못된 값을 참조하는 가시성 문제가 해결되게 된다.



참고로 synchronized 블락의 경우 synchronized블락 진입전 후에 메인 메모리와 CPU 캐시 메모리의 값을 동기화 하기 때문에 문제가 없도록 처리한다. 

출처: https://javaplant.tistory.com/23 [자바공작소]







**AtomicInteger**

아래는 직접 AtomicInterger를 Decompile 하여 요약한 내용이다.

```
public class AtomicInteger extends Number implements java.io.Serializable {
	
    private volatile int value;

    public final int incrementAndGet() {
        int current;
        int next;
        do {
            current = get();
            next = current + 1;
	} while (!compareAndSet(current, next)); 
	return next;
    }
	
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }	
}
```



### CAS(Compare And Swap) 방식**

Atomic은 CAS 방식에 기반하여 동기화 문제를 해결합니다.

그렇다면 CAS란 무엇일까요? **CAS란 변수의 값을 변경하기 전에 기존에 가지고 있던 값이 내가 예상하던 값과 같을 경우에만 새로운 값으로 할당하는 방법입니다.**



여기서 눈여겨 볼 접은 volatile int value; 입니다.

volatile 변수는 가시성 문제를 해결하기 위하여 사용됩니다 .

**volatile키워드가 붙어있는 객체는 CPU캐시가 아닌 메모리에서 값을 참조합니다.**

그러면 ? 굳이 volatile을 사용하여 메모리에서 직접 값을 참조하는데 CAS알고리즘적용이 필요한가요 ??

네 .필요합니다 .

volatile 키워드는 오직 한개의 쓰레드에서 쓰기작업을할때, 그리고 다른 쓰레드는 읽기작업만을 할떄 안정성을 보장합니다.

하지만 AtomicInteger는 여러 쓰레드에서 읽기/쓰기작업을 병행합니다.

그래서 CAS 알고리즘을 사용하여 2중 안전을 기하는 방법을 사용합니다.