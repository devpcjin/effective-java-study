## 불필요한 객체, 자주 사용되는 객체의 재사용

##### 자주 사용되는 객체가 있다면, 매번 생성하지 말고 객체 하나를 재사용하자

```java
Boolean trueObject = new Boolean(true);
Boolean falseObject = new Boolean(false);
```
위와 같이 코드를 작성한다면, Boolean객체를 사용할 때 마다 Boolean객체는 항상 새롭게 생성된다. Boolean에서의 true와 false는 Boolean객체 내에서 `정적 필드 변수`로 존재하기 때문에 굳이 별도의 객체를 생성하지 않고, 기존의 만들어진 객체를 그대로 재활용해도 된다.
```java
Boolean trueObject = Boolean.TRUE;
Boolean falseObject = Boolean.FALSE;
```
혹인 Boolean객체 내의 valueOf() `정적 팩토리 메소드`를 사용해 객체를 재활용해도 된다.
```java
public static Boolean valueOf(boolean b){
	return (b? Boolean.TRUE : Boolean.FALSE);
}

Boolean trueObject = Boolean.valueOf(true);
Boolean falseObject = Boolean.valueof(false);
```
이와 유사하게 String 객체도 캐싱을 사용하게 된다. 
```java
String a = new String("hi");
String b = new String("hi");
String c = new String("hi");
```
![](https://velog.velcdn.com/images/nnakki/post/88e1701b-cf3c-4f54-9c71-ffe1cd6a990a/image.png)


문자열 변수 a,b,c는 모두 "hi"라는 문자열을 가지게된다. 하지만, 이 세 문자열이 참조하는 주소는 모두 다르다. 각각의 변수는 모두 다른 영역을 참조하게 되는데, 동일한 문자열을 이처럼 여러개 중복 생성하는것은 메모리 낭비다. 

```java
String s = "hi";
```

이렇게 리터럴로 선언을 해 두면 컴파일시점에서 상수 풀(constant pool)에 해당 String 인스턴스를 저장하며 같은 JVM안에서 동일한 리터럴을 발견하면 동일한 인스턴스를 사용하게 함으로써 여러 문자열이 hi라는 리터럴을 바라보게 되어도 모두 동일한 인스턴스를 바라보게 됨으로써 모든 코드가 같은 객체를 재사용한다는게 보장된다. 

> <span style="color:violet">기능적으로 동일한 객체를 새로 만드는 대신 객체 하나를 재사용하는 것이 대부분 적절하다. 재사용하면 더 빠르고 유려하다. 불변객체(아이템17)는 항상 재사용할 수 있다.</span>

----

## 무거운 객체
만드는 데 시간이 오래걸리거나 메모리를 많이 잡아먹는 '비싼 객체'를 반복해서 사용해야 하는 경우, 캐시해두고 재사용할 수 있는지의 여부를 고려하는 것이 조금 더 중요하다.


String.matches는 내부적으로 Pattern 객체를 만들어 쓰는데 그 객체를 만들려면 정규 표현식으로 유한 상태 기계로 컴파일 하는 과정이 필요하다. 즉 비싼 객체다.


```java
static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

>mathces 함수의 내부를 보면 아래와 같이 Pattern 객체를 통해 regex 문자열을 compile하는 것을 알수있따. 또한 compile함수가 객체를 내부적으로 생성하는 것 또한 볼 수 있다. 
- `compile()`,`matcher()`는 Pattern 클래스
- `matches()`는 Matcher클래스, Matcher객체는 Pattern객체의 `matches()`를 호출해서 얻는다.) 
##### matches가 계속 호출되면 Pattern객체를 지속적으로 생성한다.
[참고](https://hbase.tistory.com/160)
```java
public static boolean mathces(String regex, CharSequence input){
	Pattern p = Pattern.compile(regex);	//주어진 정규표현식으로 패턴생성
	Matcher m = p.matcher(input);	//Matcher객체생성
    return m.matches();		//대상 문자열이 패턴과 일치할 경우 true반환
}

public static Pattern compile(String regex){
	return new Pattern(regex, 0);	//Pattern객체생성
}
```
**`String.matches`가 가장 쉽게 정규 표현식에 매치가 되는지 확인하는 방법이긴 하지만 성능이 중요한 상황에서 반복적으로 사용하기에 적절하지 않다.**

성능을 개선하려면 Pattern 객체를 만들어 재사용하는 것이 좋다.


```java
public class RomanNumber {
    private static final Pattern ROMAN = 
		Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
하지만 이 코드도 문제가 있는데, isRomanNumeral 메소드가 호출되지 않는다면, ROAM이라는 필요없이 만든셈이 된다. 게으른 초기화(lazily initializing)(아이템83)를 사용해서 최적화 할 수 있지만 추천하진 않는다. 보통 지연 초기화는 측정 가능한 성능 개선 없이 구현을 복잡하게 만든다.(아이템67)

## 어댑터

불변 객체인 경우에 안정하게 재사용하는 것이 매우 명확하다 하지만 몇몇 경우에 분명하지 않은 경우가 있다. 오히려 반대로 보이기도 한다. 어댑터를 예로 들면, 어댑터는 인터페이스를 통해서 뒤에 있는 객체로 연결해주는 객체라 여러개 만들 필요가 없다.

Map 인터페이스가 제공하는 keySet은 Map이 뒤에 있는 Set 인터페이스의 뷰를 제공한다. keySet을 호출할 때마다 새로운 객체가 나올거 같지만 사실 같은 객체를 리턴하기 때문에 리턴 받은 Set 타입의 객체를 변경하면, 결국에 그 뒤에 있는 Map 객체를 변경하게 된다.

```java
public class UsingKeySet {

    public static void main(String[] args) {
        Map<String, Integer> menu = new HashMap<>();
        menu.put("Burger", 8);
        menu.put("Pizza", 9);

        Set<String> names1 = menu.keySet();
        Set<String> names2 = menu.keySet();

        names1.remove("Burger");
        System.out.println(names2.size()); // 1
        System.out.println(menu.size()); // 1
    }
}
```
>객체가 불변이라면 재사용해도 문제가 없다. 
하지만 불변이 보장되지 않는 상황도 있는데, 어댑터 패턴(link) 을 생각해보면 실제 객체를 연결해주는 제 2의 인터페이스 역할을 하는 어댑터같은 경우 사용자는 이 어댑터를 사용할 때 뒷 단에서 매번 같은 인스턴스가 반환될 지 동일한 내용에 대해서 동일한 인스턴스를 반환해줄지 알 수 없다.


## 오토박싱(auto boxing)
기본타입과 래퍼클래스(Wrapper Class)간에 자동으로 상호변환해주는 기술인 오토박싱(언박싱)은 기본타입과 래퍼클래스간의 구분을 흐리게 해준다. 하지만 이런 점 때문에 프로그래머가 혼용해서 써도 에러가 발생하지 않으니 성능상의 문제가 발생할 수 있다.
**오토박싱은 프리미티브 타입과 박스 타입의 경계가 안보이게 해주지만 그렇다고 그 경계를 없애주진 않는다.**

```java
public class AutoBoxingExample {

    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        Long sum = 0l;
        for (long i = 0 ; i <= Integer.MAX_VALUE ; i++) {
            sum += i;
        }
        System.out.println(sum);
        System.out.println(System.currentTimeMillis() - start);
    }
}
```

위 코드에서 `sum` 변수의 타입을 `Long`으로 만들었기 때문에 불필요한 Long 객체를 2의 31 제곱개 만큼 만들게 되고 대략 6초 조금 넘게 걸린다. 타입을 프리미티브 타입으로 바꾸면 600 밀리초로 약 10배 이상의 차이가 난다.

**불필요한 오토박싱을 피하려면 박스 타입 보다는 프리미티브 타입을 사용해야 한다.**


------

### 오해하지 말아야 할 부분
불필요한 객체 생성을 피하라는 것을 단순하게 객체 생성의 비용이 크니까 피해야 한다고 오해해서는 안된다. 
요즘 JVM에서 불필요하게 생성된 작은 객체들을 생성및 회수하는 일은 별로 부담되는 작업이 아니다. 
데이터베이스를 연결과 같이 비용이 몹시 높아 재사용하는 편이 명확히 좋은편인 경우가 아니라면 객체 풀을 만들어서 객체들을 모아놓는 것은 코드를 헷갈리게 하고, 메모리 사용량을 늘려 성능을 떨어뜨린다. 
무엇보다, 이렇게 객체를 방어적으로 복사하는(defensive copy) 방식은 피해가 발생했을 때 객체를 반복 생성했을 때보다 훨씬 크다.  반복 생성의 부작용은 코드 형태나 성능에만 영향을 주지만, 방어적 복사가 실패했을 경우에는 버그와 보안문제로 직행한다. 

> 이번 아이템으로 인해 객체를 만드는 것은 비싸며 가급적이면 피해야 한다는 오해를 해서는 안된다. 특히 방어적인 복사(Depensive copying)를 해야 하는 경우에도 객체를 재사용하면 심각한 버그와 보안성에 문제가 생긴다. 객체를 생성하면 그저 스타일과 성능에 영향을 줄 뿐이다.
