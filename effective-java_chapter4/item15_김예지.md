잘 설계 된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔하게 분리한다. 
오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다. 
<span style="color:violet">정보은닉, 캡슐화</span>라고 부르는 이 개념은 소프트웨어 설계의 근간이 되는 원리이다. 

## 정보은닉의 장점
### 1. 시스템 개발 속도를 높인다. 
여러 컴포넌트를 병렬로 개발할 수 있기 때문이다. 
ex) 회원 탈퇴 기능 
```java
public interface WithdrawalService {
  void withdrawalMember(Member member);
  List<Member> getMembers();
  WithdrawalProcessType getProcessType(); 
  default startProcess() {
    List<Member> members = getMembers();
    members.stream().foreach(this::withdrawalMember);
  }
}
```
위와 같은 회원탈퇴 로직에 관한 인터페이스를 제공하고, 일반회원 탈퇴, 해외회원 탈퇴, 비회원 탈퇴등의 로직이 상이하여 별도로 구현하는 경우 interface스펙에 맞추어 여러 개발자가 개발 할 수 있기 때문이다. 
### 2. 시스템 관리 비용을 낮춘다. 
각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 다른 컴포넌트로 교체하는 부담도 적다.
위에서 예시로 들은 일반회원 탈되, 해외회원 탈퇴, 비회원 탈퇴에서 비회원 탈퇴 로직에 변경이 생긴 경우 `WithdrawalService interface`를 구현한 비회원 탈퇴 서비스를 만들면 되기 때문이다.


### 3. 성능 최적화에 도움을 준다.
완성된 시스템을 프로파일링해 최적화할 컴포넌트를 정한 다음 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화 할 수 있다.
위에서 예시로 들은 일반회원 탈되, 해외회원 탈퇴, 비회원 탈퇴에서 비회원 탈퇴 중 비회원 탈퇴에 대해 처리대상이 많아 startProcess 메서드를 확장하여 내부적으로 withdrawalMember 메서드를 사용하지 않고 회원 리스트에 대해 쿼리를 이용해 Insert~ Select로 한방에 처리 할 수도 있다.

### 4. 소프트웨어 재사용성을 높인다.
외부에 의존하지 않고, 독자적으로 동작할 수 있는 컴포넌트라면, 그 컴포넌트와 함께 개발되지 않은 낯선 환경에서도 유용하게 쓰일 가능성이 크다.
예를들면 알림톡 서버 API를 호출 할 수 있는 인터페이스가 있는 경우 그 인터페이스를 그대로 사용할 수 있다.

### 5. 큰 시스템을 제작하는 난이도를 낮춰준다. 
시스템 전체가 완성되지 않은 상태에서도 개별 컴포넌트의 동작을 검증할 수 있다.
쉬운 예로 Mockito를 이용한 TestCase 작성이다.


---

## 제어 메커니즘
- 클래스, 인터페이스, 멤버의 접근성(접근 허용 범위)를 명시한다. 
<span style="color:violet">**각 요소의 접근성은 그 요소가 선언된 위치와 접근 제한자로 정해진다.**</span>
>**Java의 접근제한자**
![](https://velog.velcdn.com/images/nnakki/post/cdaf9c89-e3a1-456e-9a86-e1bec3ec4813/image.png)
`private`: 멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다.
>
>`package-private`: 멤버가 소속된 패키지 안의 모든 클래스에서 접근 할 수 있다.
>
>`protected`: package-private의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다.
>
>`public`: 모든 곳에서 접근 할 수 있다.

---
## 모든 클래스와 멤버의 접근성을<span style="color:red"> 가능한 한 좁히기</span>
##### 소프트웨어가 올바르게 작동하는 한 항상 가장 낮은 접근 수준을 부여하자.
- `public`으로 선언하는 경우 공개  API로 사용 - 영원히 하위호환을 관리해줘야 함.
- `package-private`로 사용하는 경우 해당 패키지 안에서만 사용 가능 - 다음 릴리즈에서 내부 로직을 변경해도 괜찮음
### 클래스 접근 제한자 
##### 톱레벨 수준(파일명 = 클래스명)이 같은 수준에서의 접근제한자는 public과 package-private만 사용 할 수 있다.
- `private` :  멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다. 
- `package-private` : 멤버가 소속 된 패키지 안의 모든 클래스에서 접근할 수 있다. 

### 이너클래스 사용하기

> -  한 클래스에서만 사용하는 package-private 톱레벨 클래스나 인터페이스는 사용하는 클래스 안에 `private static으로 중첩`해보자
 - public일 필요가 없는 클래스의 접근 수준을 package-private로 해보자 - 다른 패키지에서 사용하지 못하게 막아야 한다.

```java
public class Food {
  private String name;
  private int price;
  private List<Shop> shops;
}
```

```java
public class Shop {
  private String name;
  private String owner;
}
```

```java
public class Food {
  private String name;
  private int price;
  private List<Shop> shops;

  private static class Shop {
    private String name;
    private String owner;
  }
}
```

### private, package-private은 해당 클래스의 구현에 해당하므로 공개 api에 영향을 미치지 않는다.
- 일단 처음에는 모든 멤버는 private으로 만들어야 한다.
- 같은 패키지에서 접근해야 하는 멤버가 있는 경우 package-private로 변경
- 단, Serialize를 구현한 클래스의 경우 공개 API에 의도치 않게 공개 될 수도 있다.
- 필드의 접근 권한을 package-private에서 protected로 바꾸는 순간 필드에 접근 할 수 있는 대상 범위가 늘어나니 주의해야 한다.

### 메서드를 재정의 할 경우에는 접근 수준을 상위 클래스에서보다 좁게 설정 할 수 없다.
- 상위 클래스에서 접근제한자가 protected인데 하위클래스에서 갑자기 package-private이나 private로 변경할 수 없다.
- 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다는 규칙(리스코프 치환원칙)을 위반하기 때문이다.
- 단 인터페이스를 구현하는 경우에는 클래스의 메서드는 모두 public으로 해야 한다.

### 코드를 테스트 하려는 목적으로 클래스, 인터페이스를 접근 범위를 넓히는 것을 주의하라
- public 클래스의 private -> package-private으로 바꾸는거는 괜찮다. 하지만 그이상의 경우 공개 API에 문제가 될 수 있다.

## public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.
필드가 가변객체를 참조하거나(Collections이나 배열), final이 아닌 인스턴스 필드를 public으로 선언하면 불변식을 보장할 수 없다. public 가변 필드를 갖는 클래스는 일반적으로 thread safe 하지않다. 내부 구현을 바꾸고 싶어도 public 필드를 없애는 방식으로는 리팩터링이 불가하다.

하지만 상수라면 관례대로 public static final 필드로 공개해도 좋다. 필드명 네이밍은 관례 상 `대문자 + _(언더바)`로 구성한다. 그리고 반드시 불변 객체를 참조하도록 한다.

### 클래스에서 public static final 배열 필드를 두지말아라
```java
public static final Thing[] VALUES = {...};
```
이런 경우 VALUES에 대한 참조를 변경할 수는 없지만, 배열내의 내용을 변경하는 것은 가능하다.

#### CASE1. Thing 배열을 private로 만들고 public 불변 리스트를 추가한다.
```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```
#### CASE2. Thing 배열을 private로 만들고 public 메서드를 추가한다.
```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final Thing[] values() {
  return PRIVATE_VALUES.clone(); //방어적 복사본
}
```
## Java9에 추가된 module ([Project Jigsaw](https://infoscis.github.io/2017/03/24/First-steps-with-java9-and-jigsaw-part-1/))
Java9 부터는 Module시스템이라는 개념이 도입 되었다.

패키지는 클래스의 묶음인 것 처럼, 모듈(Modules)은 패키지의 묶음이다. 모듈은 관례 상, 자신이 속하는 패키지 중 공개(export) 할 것을 선언한다.

관례상 module-info.java 에 선언한다. 모듈을 정의하기 위해 3가지 정보를 작성해야 한다.
>
> 1. `module 명 - module 명칭`
>2. `공개할 package (export)`
    - 공개한 package의 public type에 접근제한자만 외부에서 접근할 수 있다.
    - 만약 export에 적지 않은 패키지의 클래스 중 public 접근제한자가 있어도 접근을 할 수 없다.
>3. `module내 package의 클래스를 사용하기 위해 종속되는 package (require)`
    - 다른 모듈의 대한 require를 작성하면 다른 모듈에서 export하는 클래스의 public type에 접근이 가능하다
>
![](https://velog.velcdn.com/images/nnakki/post/e5745509-1bd4-48b5-be49-da92da203103/image.png)

