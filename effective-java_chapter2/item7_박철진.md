# item 7. 다 쓴 객체 참조를 해제하라

## Garbage Colloctor
- 자바처럼 `GC(가비지 컬렉터)`를 갖춘 언어로 넘어오면 프로그래머의 삶이 평안해짐
    - 다쓴 객체를 알아서 회수해감
- **하지만** 메모리 관리에 더 이상 신경을 쓰지 않아도 된다는 것은 오해


## 아래의 코드에서 메모리 누수가 일어나는 위치는 어디 일까?

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
- `pop()`동작을 수행할 때 꺼내진 객체들을 가비지 컬렉터가 회수하지 않음
    - 해당 Stack이 객체들의 다 쓴 참조 (obsolete reference)를 여전히 가지고 있기 때문
        - index가 size보다 작은 `활성영역`의 원소들을 제외하고는 모두 다 쓴 참조에 해당
            - Obsolete reference : 말 그대로 다시 쓰지 않을 참조
        - 스택에서 빼내도 스택이 차지하고 있는 메모리는 줄지 않음

- GC 언어에서는 메모리 누수를 찾기 어려움
    - 객체 참조 하나를 살려두면 GC는 해당 객체 뿐 아니라 그 객체가 참조하는 모든 객체를 회수 하지 못함
    - 결국에는 성능에 악영향

- 메모리 누수가 야기하는 문제
    - GC활동과 메모리 사용량이 늘어나 결국 성능 저하
    - 심할 경우 디스크 페이징이나 `OutOfMemoryError`를 일으켜 프로그램이 예기치 못하게 종료됨

## 어떻게 해결할 수 있을까?

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```
- 해당 참조를 다 썼을 때 **`null` 처리 (참조 해제)**
    - null 처리한 참조를 실수로 사용할 때 `NullPointerException`을 던지며 종료

## null 처리는 항상 옳을까?
- 모든 객체를 쓰자마자 일일이 null 처리하는 것은 바람직하지 않고 프로그램을 필요 이상으로 지저분하게 만듦
    - > 객체 참조를 null 처리 하는 일은 예외적인 경우여야한다.
        - 가장 좋은 방법은 해당 참조를 담은 변수를 유효 범위(Scope) 박으로 밀어내는 것
    - 변수의 범위를 가능한 최소가 되게 정의했다면 자연스럽게 이루어짐
        ```java
        Object pop() {

            Object num = 1234;

            num = null;
        }
        ```
        - `num`은 scope가 `pop()` 내부에서 형성되어 있으므로 scope 밖으로 나가면 무의미한 레퍼런스 변수가 되어 굳이 null처리할 필요가 없음


## 그렇다면 null 처리는 언제?
- Stack 클래스는 왜 메모리 누수에 취약할까?
    - 스택이 자기 메모리를 직접 관리하기 때문
        - elements배열로 저장소 풀을 만들어 원소들을 관리
        - 배열의 활성 영역에 속한 원소들은 사용하고 비활성 영역은 쓰이지 않음

- 그렇다면 문제는 왜 발생할까?
    - GC는 위의 사실을 알 길이 없음
    - GC에게는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체

> 그러므로 프로그래머는 비활성 영역이 되는 순간 null 처리해서 해당 객체를 더는 쓰지 않을 것임을 GC에 알려야한다.

> 자기 메모리를 직접접 관리하는 클래스라면 프로그래머는 항상 메모리 누수에 주의 해야한다.

- 원소를 다 사용한 즉시 그 원소가 참조한 객체를 다 null 처리

---

## 캐시, 메모리 누수를 일으키는 주범
객체의 레퍼런스를 캐시에 넣어 놓고, 그 객체를 다 쓴 뒤로도 한참을 놔두는 것 또한 메모리 누수를 야기한다.

### 예시
```java
public class CacheSample {
	public static void main(String[] args) {
		Object key = new Object();
		Object value = new Object();

		Map<Object, List> cache = new HashMap<>();
		cache.put(key, value);
		...
	}
}
```
- key의 사용이 없어지더라도 cache가 key의 레퍼런스를 가지고 있으므로, GC의 대상이 될 수 없음

### WeakHashMap을 사용하여 해결
```java
public class CacheSample {
	public static void main(String[] args) {
		Object key = new Object();
		Object value = new Object();

		Map<Object, List> cache = new WeakHashMap<>();
		cache.put(key, value);
		...
	}
}
```
- WeakHashMap은 key값을 모두 Weak Reference로 감싸 Strong Reference가 없어지면 GC의 대상이 됨


[(Java)참조 유형 (Strong Reference/ Soft Reference/ Weak Reference/ Phantom References)](https://lion-king.tistory.com/entry/Java-%EC%B0%B8%EC%A1%B0-%EC%9C%A0%ED%98%95-Strong-Reference-Soft-Reference-Weak-Reference-Phantom-References)

---


## 메모리 누수의 세번째 주범, 콜백(callback) / 리스너(listener)
- 콜백을 등록만 하고 명확히 해지하지 않으면, 뭔가 조치해주지 않은 한 콜백은 계속 쌓임
- 이때 콜백을 약한 참조 (Weak Reference)로 저장하면 GC가 즉시 수거
    - `WeakHashMap`에 키로 저장해서 사용 


## 핵심 정리
> **메모리 누수는 겉으로 잘 드러나지 않아** 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 **예방법**을 익혀두는 것이 매우 중요하다.
