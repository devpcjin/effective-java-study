# 주제
> clone 재정의는 주의해서 진행하라

# Cloneable
![](https://velog.velcdn.com/images/dbtlwns/post/2cf4a2c5-a617-47ae-af7b-e53fd6fd2770/image.png)
Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다. 위 사진을 보면 아무 메소드도 정의되어 있지 않다. 그냥 껍데기이고 clone에 의해 복사할 수 있다라는 것만 나타낸다.
>믹스인은 다른 클래스에서 사용할 목적으로 만든 클래스를 지칭한다.
믹스인을 구현한 클래스에 원래의 주된 타입 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 줄 수 있음(ex Comparable)

### 동작 방식
cloneable은 아무 메서드도 없지만 clone의 동작방식을 결정한다.
cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며 그렇지 않은 클래스의 인스턴스를 호출하면 CloneNotSupportedException을 던진다. (인터페이스를 이례적으로 사용한 예시다. 따라하기 금지!!)
![](https://velog.velcdn.com/images/dbtlwns/post/d6b4e27f-bd0d-48ed-9f01-da7d9496c682/image.png)
![](https://velog.velcdn.com/images/dbtlwns/post/190d170e-9306-43b8-9a96-216412e459fe/image.png)

### 단점 
![](https://velog.velcdn.com/images/dbtlwns/post/d3227b20-0d84-4517-83a4-230d3fff07b9/image.png)

* 하지만 의도한 목적을 제대로 이루지 못했다. 가장 큰 문제는 clone메서드가 선언된 곳이 Cloneable이 아닌 Object이고 이 마저도 protected이다. 그래서 외부객체에서는 접근할 수가 없다.

![](https://velog.velcdn.com/images/dbtlwns/post/cb290576-2078-4cf0-9811-8444a480914a/image.png)


### 일반 규약
>
1. x.clone()!=x
2. x.clone().getClass() == x.getClass()
3. x.clone().equals(x) //필수조건은 아닌데 보통 참

### 특징
* 얕은 복사가 진행된다. 즉 equals에 대해서는 참을 리턴하고 ==에 대해서는 거짓을 리턴한다.
* clone 대상 클래스에 대해 새로운 객체를 new로 생성한다.
* 모든 필드들에 대해 초기화를 진행
* 강제성이 없다는 점을 빼면 생성자 체이닝(생성자내에 다른 생성자)과 비슷하다. clone메서드가 super.clone이 아닌 생성자를 호출해 얻은 인스턴스를 반환해도 문제가 없다.
* clone이 재정의된 클래스의 하위클래스엥서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어져 하위클래스의 clone이 제대로 동작하지 않게 된다.

# 재정의
> 제대로 동작하는 clone메서드를 가진 상위 클래스를 상속해 Cloneable을 구현하고 싶다!!!

### 불변 객체를 참조
super.clone메서드를 호출한다. 여기서 얻은 객체는 원본의 완벽한 복제본이다. 모든 필드가 기본타입이거나 불변 객체를 참조한다면 이 객체는 완벽히 우리가 원하는 상태다.
아래는 가변 상태를 참조하지 않는 클래스용 clone메서드 이다.
![](https://velog.velcdn.com/images/dbtlwns/post/6095f451-cdf6-47ca-811b-5557bd5e69f6/image.png)

이 처럼 사용하면 이 메서드를 사용하는 곳에서 형변환을 해줄 필요가 없다. try-catch로 묶었지만 절대 실패하지 않는다. 그럼 굳이 try-catch블럭으로 감싼 이유는 Object의 clone메서드가 검사 예외인 CloneNotSupportedException을 던지도록 선언되었기 때문이다. 거추장스러운 코드는 CloneNotSupportedException은 unchecked exception이었어야 한다고 말해준다.

### 가변 객체를 참조
**재앙이다.**
```java
class Stack{
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack(){
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    // 여러 메서드
}
```
위와 같은 Stack클래스를 예로 들어보자.
clone메서드가 super.clone의 결과를 그대로 반환한다면 어떻게 될까?
size필드는 올바른 값을 갖지만 elements필드는 원본 Stack과 같은 인스턴스를 참조할 것 이다.
즉 같은 인스턴스를 공유해 불변식을 해친다.
>clone 메서드는 사실상 생성자와 같은 효과를 낸다. clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

### 재귀적 호출
![](https://velog.velcdn.com/images/dbtlwns/post/e335441e-a135-4905-a972-8bd7cc7ab236/image.png)
배열을 복제할 때는 clone메서드를 사용하라고 권장한다. 배열의 clone은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 배열은 clone기능을 제대로 사용하는 유일한 예이다.
Array필드가 final이라면 Array.clone을 통한 복제는 할 수 없다.
Cloneable 아키텍처는 가변객체를 참조하는 필드를 final로 선언하라는 말과 충돌한다.(공유 가능하다면 상관없음)
복제 할 수 있는 클래스를 만들기 위해 일부 필드에서 final을 제거해야 할 수도 있다.

### 해시 테이블용 clone 메서드
해시 테이블 내부는 버킷 배열이고 각 버킷은 키-밸류 쌍을 담는 연결리스트의 첫 번째 엔트리를 참조한다.
```java
class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
}
```
Stack처럼 단순히 버킷 배열의 clone을 재귀적으로 호출한다.
![](https://velog.velcdn.com/images/dbtlwns/post/c689942f-017e-4f47-a6d9-e96fb8f15c7c/image.png)

복제본은 자신만의 버킷 배열을 갖지만 이 배열은 원본과 같은 연결 리스트를 참조한다.
즉 연결리스트까지 복사해야 한다. Entry 클래스에 deepCopy를 구현하여 보강했다.
이거면 될까?
![](https://velog.velcdn.com/images/dbtlwns/post/6d99e3bb-b64a-48ca-9913-8da2fa36dec2/image.png)
Entry의 deepCopy 메서드는 자신이 가리키는 연결리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다. 간단하며 버킷이 길지 않다면 잘 동작한다. 하지만 연결 리스트를 복제하는 방법으로는 좋지않다. 스택 오버플로우의 위험이 있다. 이 문제를 해결하기 위해 deepCopy대신 반복자를 써서 순회하는 방향으로 수정해야 한다.
![](https://velog.velcdn.com/images/dbtlwns/post/a0e1fdc9-214a-4e8e-bcfd-895f708785e1/image.png)

# 기타
### 고수준 API와 저수준
HashTable을 예로 들때 buckets필드를 새로운 버킷배열로 모두 초기화하고 put메서드를 사용해 둘의 내용을 똑같게 해주면 된다. 이처럼 고수준 API를 사용하는 방법이 있다. 하지만 저수준에서 처리하는 방식보다 느리다.

### 생성자에서 재정의 될 수 있는 메서드를 호출하지 말자
clone이 하위 클래스에서 재정의한 메서드를 호출하면 하위 클래스는 복사 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 있다.

### CloneNotSupportedException
Object의 clone메서드는 CloneNotSupportedException을 던지지만 재정의한 메서드는 그렇지 않다. public인 메서드에서는 throws절을 없애던가 예외 전환을 해줘야 한다.

clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 할 수도 있다.
```java
@Override
protected final Object clone() throws CloneNotSupportedException {
	throw new CloneNotSupportedException();
}
```

### 스레드 동기화
Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone메서드 역시 적절히 동기화 해줘야 한다.

### 결론
Cloneable을 구현하는 모든 클래스는 clone을 재정의 해줘야한다. 접근제한자는 public으로 하며 반환타입은 클래스 자신으로 변경한다. super.clone을 호출한 후 필요한 필드를 전부 **복사** 해준다.

# 진짜 결론
복사 생성자
```java
public Yum(Yum yum){...};
```

복사 팩토리 메서드
```java
public static Yum newInstance(Yum yum){...};
```
>위 두가지 방식이 Cloneable,clone에 비해 장점이 많다.

* 안전하다.
* 규약에 기대지 않는다.
* final과 충돌하지 않는다.
* CheckedException 처리할 필요가 없다.
* 형변환이 필요없다.
* 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.
  * 즉 HashSet 객체 hs를 TreeSet ts로 복제할 수 있다.

>
***복제기능이 필요하다면 생성자와 팩토리를 이용하는게 최고다. 단 배열은 clone메서드 방식이 깔끔하다.***
