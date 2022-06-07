## 필요한 개념
[[Java]hashcode( )](https://velog.io/@nnakki/Java해쉬테이블의-이해와-구현)

[[이펙티브 자바]10. equals는 일반 규약을 지켜서 재정의하라
](https://velog.io/@nnakki/이펙티브-자바10.-equals는-일반-규약을-지켜서-재정의하라)

---

## hashcode 규약  

>1. equals비교에 사용되는 정보가 변경되지 않았다면, 객체의 hashcode 메서드는 몇번을 호출해도 항상 일관된 값을 반환해야 한다.
>2. equals메서드 통해 두 개의 객체가 같다고 판단했다면, 두 객체는 똑같은 hashcode 값을 반환해야 한다.
>3.  equals메서드가 두 개의 객체를 다르다고 판단했다 하더라도, 두 객체의 hashcode가 서로 다른 값을 가질 필요는 없다. (Hash Collision)
>   

hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 2번.
즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야한다. 


## hashCode() 재정의 예시

```java
@RunWith(JUnit4.class)
public class HashCodeTest {

    @Test
    public void hashcode_재정의() {
        HashMap<ExtendedPhoneNumber, String> map = new HashMap<>();
        map.put(new ExtendedPhoneNumber(707, 867, 5307), "제니");
        System.out.println("Instance 1 hashcode : " + new ExtendedPhoneNumber(707, 867, 5307).hashCode());
        System.out.println("Instance 2 hashcode : " + new ExtendedPhoneNumber(707, 867, 5307).hashCode());
        Assert.assertEquals(map.get(new ExtendedPhoneNumber(707, 867, 5307)), "제니");
    }

    @Test
    public void hashcode_재정의안함() {
        HashMap<PhoneNumber, String> map = new HashMap<>();
        map.put(new PhoneNumber(707, 867, 5307), "제니");
        System.out.println("Instance 1 hashcode : " + new PhoneNumber(707, 867, 5307).hashCode());
        System.out.println("Instance 2 hashcode : " + new PhoneNumber(707, 867, 5307).hashCode());
        Assert.assertNotEquals(map.get(new PhoneNumber(707, 867, 5307)), "제니");
    }

    @NoArgsConstructor
    @AllArgsConstructor
    public static class PhoneNumber {
        protected int firstNumber;
        protected int secondNumber;
        protected int thirdNumber;

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof PhoneNumber)) {
                return false;
            }

            PhoneNumber p = (PhoneNumber) o;
            return this.firstNumber == p.firstNumber &&
                    this.secondNumber == p.secondNumber &&
                    this.thirdNumber == p.thirdNumber;
        }
    }

    @NoArgsConstructor
    public static class ExtendedPhoneNumber extends PhoneNumber {

        public ExtendedPhoneNumber(int firstNumber, int secondNumber, int thirdNumber) {
            super(firstNumber, secondNumber, thirdNumber);
        }

        @Override
        public int hashCode() {
            int c = 31;
            int hashcode = Integer.hashCode(firstNumber);
            hashcode = c * hashcode + Integer.hashCode(secondNumber);
            hashcode = c * hashcode + Integer.hashCode(thirdNumber);
            return hashcode;
        }
    }
}
```

### CASE1. 재정의 하지 않은 경우
PhoneNumber 클래스에서는 객체 동치성 체크를 위해 equals메서드를 재정의했다. 하지만 hashcode 메서드는 재정의 하지 않았다.
new를 통해 새로운 객체를 만들 때마다 늘 다른 hashcode를 리턴하여, 실제 HashMap에 key로 PhoneNumber클래스를 삽입하더라도 hashcode가 다르기 때문에 다른 버킷에가서 데이터를 조회 하려고해서 결과적으로 null이 나와 기댓값인 "제니"를 만족하지 못했다.
> Instance 1 hashcode : 1686369710
>
> Instance 2 hashcode : 194706439

### CASE2. 재정의 한 경우
ExtendPhoneNumber 클래스에서는 PhoneNumber를 상속 받은 뒤, hashCode 메서드를 재정의하였다.
(equals메서드는 PhoneNumber의 equals를 그대로 사용하였다.)
두 개의 ExtendPhoneNumber 객체에 대해 같은 hashcode를 리턴하고 있었다.
그렇기 때문에 HashMap에 삽입한 ExtendPhoneNumber 객체에 대해 new ExtendPhoneNumber()로 조회를 하니 같은 hashCode의 버킷을 조회하여 "제니"라는 데이터를 얻어올 수 있었다.
> Instance 1 hashcode : 711611
>
> Instance 2 hashcode : 711611

----

## hashCode() 작성법
### 최악의 hashcode()
```java
@Override
public int hashCode() {
  return 42;
}
```
이 코드는 동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법한 해시코드 처럼 보인다.
하지만, 모든 객체에 대해 똑같은 해시코드를 반환하니 모든 객체가 같은 해시테이블 버킷에 담겨 연결리스트(Linked List)처럼 동작하게 된다. 평균 수행시간이 O(1)에서 O(n)으로 느려져서, 성능이 매우 낮아질 뿐더러 버킷에 대한 overflow가 발생하는 경우 데이터가 누락될 수도 있다.

### 좋은 해시함수 만들기
##### 이상적인 해시함수는 서로 다른 인스턴스에 대해 다른 해시코드를 반환해야한다.
```java
@Override
public int hashCode() {
    int c = 31;
    //1. int변수 result를 선언한 후 첫번째 핵심 필드에 대한 hashcode로 초기화 한다.
    int result = Integer.hashCode(firstNumber);

    //2. 기본타입 필드라면 Type.hashCode()를 실행한다
    //Type은 기본타입의 Boxing 클래스이다.
    result = c * result + Integer.hashCode(secondNumber);

    //3. 참조타입이라면 참조타입에 대한 hashcode 함수를 호출 한다.
    //4. 값이 null이면 0을 더해 준다.
    result = c * result + address == null ? 0 : address.hashCode();

    //5. 필드가 배열이라면 핵심 원소를 각각 필드처럼 다룬다.
    for (String elem : arr) {
      result = c * result + elem == null ? 0 : elem.hashCode();
    }

    //6. 배열의 모든 원소가 핵심필드이면 Arrays.hashCode를 이용한다.
    result = c * result + Arrays.hashCode(arr);

    //7. result = 31 * result + c 형태로 초기화 하여 
    //result를 리턴한다.
    return result;
}
```
---

## hashCode()를 편하게 만들어주는 모듈
  - Objects.hash()
    - 내부적으로 AutoBoxing이 일어나 성능이 떨어진다.
- Lombok의 @EqualsAndHashCode
- Google의 @AutoValue

## hashCode()를 재정의할 때 고려할 점
- 불변 객체에 대해서는 hashcode 생성비용이 많이 든다면, hashcode를 캐싱하는 것도 고려하자
  - 스레드 안전성까지 고려해야 한다.
- 성능을 높인답시고 hashcode를 계산할 떄 핵심필드를 생략해서는 안된다.
  - 속도는 빨라지겠지만, hash품질이 나빠져 해시테이블 성능을 떨어뜨릴 수 있다 (Hashing Collision)
- hashcode 생성규칙을 API사용자에게 공표하지 말자
  - 그래야 클라이언트가 hashcode값에 의지한 코드를 짜지 않는다.
  - 다음 릴리즈 시, 성능을 개선할 여지가 있다.
  
---
