# 더 자바, Java 8

## 메소드 레퍼런스

메소드 참조는 메소드를 참조해서 매개변수의 정보 및 리턴 타입을 알아내어, 람다식에 불필요한 매개 변수를 제거하는 것이 목적
람다식은 종종 기본 메소드를 단순히 호출만 하는 경우가 많다.

- ex) 두 개의 값을 받아 큰 수를 리턴하는 Math 클래스의 max() 정적 메소드를 호출하는 람다식

  ```java
  (left, right) -> Math.max(left, right);
  ```

  이 때 랍다식은 단순히 두 개의 값을 Math.max() 로 전달하는 역할만 하기 때문에, 코드가 불필요하게 길어졌다. 이럴 때 메소드 참조를 사용한다.

  ```java
  Math :: max;    //(타입::인스턴스메소드)_임의의 객체 Math의 인스턴스 메소드 max() 참조
  ```

​		메소드 참조 역시 람다식과 마찬가지로, 인터페이스의 익명 구현 객체로 생성되므로, 타겟 타입인 인터페이스의 추상 메소드가 어떤 매개 변수를 가지고, 
​		리턴 타입이 무엇인가에 따라 달라진다.

- ex) IntBinaryOperator 인터페이스는 두 개의 int 매개 값을 받아, int 값을 리턴하므로 Max::max 메소드를 참조 대입할 수 있다.

  ```java
  IntBianryOperator operator = Math :: max;
  ```

  -----------------------------

### ▶정적 메소드와 인스턴스 메소드 참조

정적(static) 메소드르 참조한 경우 클래스 이름 뒤에 :: 기호를 붙이고 정적 메소드 이름을 기입 ` 클래스 :: 메소드`

인스턴스(instance) 메소드일 경우는 `참조변수 :: 메소드`
  : 객체를 생성한 다음, 참조한 변수 뒤에 :: 인스턴스 기호를 붙이고, 인스턴스 메소드 이름을 기술하면 된다. 

```java
  public class Calculator{
      
      //정적 메소드
      public static int staticMethod(int x, int y){
          return x+y;
      }
      
      //인스턴스 메소드
      public int instanceMethod(int x, int y){
          return x+y;
      }
  }
```

  

#### <span style="color:Blue">**정적 및 인스턴스 메소드 참조**</span>

  ```java
  import java.util.function.IntBinaryOperator;
  
  public class MethodReferencesExample{
      public static void main(String[] args){
          InitBinaryOperator operator;
          
          //정적메소드참조
          operator = (x,y) -> Calculator.staticMethod(x,y);
          operator = Calculator :: staticMethod;
          
          //인스턴스메소드 참조
          Cacluator cal = new Calculator();
          operator = (x,y) -> cal.instanceMethod(int x, int y)
          operator = cal :: instanceMethod;
          
          //실행식
          system.out.println(operator.applyAsInt(x,y))
      }
	}
  ```

  ​	

  #### <span style="color:Blue">**매개변수의 메소드 참조**</span>

  메소드는 람다식 외부의 클래스 멤버일 수도 있고, 람다식에서 제공되는 매개 변수의 멤버일 수도 있다. 
  ​	(위의 예제는 람다식 외부의 클래스 멤버인 메소드를 호출한 경우)
  ​	여기에 더해서 다음과 같이 **람다식에서 제공되는 a 매개변수의 메소드**를 호출해서 **b 매개변수의 매개값**으로 사용하는 경우도 있다.
```java
		(a,b) -> {a.instanceMethod(b)}
```
이를 메소드 참조로 표현하면, 
```java
클래스 :: 메소드
a의클래스이름 :: instanceMethod
```
정적 메소드의 참조와 동일하지만, a의 인스턴스 메소드가 실행되게 되므로 전혀 다른 코드가 실행된다.

```java
public class ArgumentMethodReferencesExample{
    public static void main(Stringp[] args){
        ToIntBiFunction<String,String> function;
        
        function = (a,b) -> a.compareToIgnoreCase(b);
        print(function.applyAsInt("Java8", "JAVA8"))				// 결과?
        
        function - String :: compareToIgnoreCase;
        print(function.applyAsInt("Java8", "JAVA8"))				// 결과?
	    }
	}
```
cf) String의 인스턴스 메소드인 compareToIgonoreCase()는 a.compareToIgonreCase(b)로 호출되면, a가 b보다 먼저 오면 음수를, 동일하면 0을, 나중에 오면 양수를 리턴함). 

#### <span style="color:Blue">**생성자 참조**</span>
: 생성자를 참조한다는 것은 단순한 객체 생성을 의미한다. 단순히 메소드 호출로 구성 된 람다식을 메소드 참조로 대치할 수 있듯이, 단순히 객체를 생성하고 리턴하도록 구성된 람다식은 생성자 참조로 대치할 수 있다.  **생성자 참조의 람다식은 단순히 객체를 생성만 한 후 리턴한다.**
```java
(a,b) -> {return new 클래스 (a,b);}
```
이를 생성자 참조로 표현하면
```
클래스 :: new
```
(+) 만약 생성자가 오버로딩 되어 여러 개가 있을 경우, 컴파일러는 함수적 인터페이스의 추상 메소드와 동일한 매개 변수 타입과 개수를 가진 생성자를 찾아 실행하고, 해당 생성자가 존재하지 않으면 컴파일 오류를 발생시킨다.

```java
public class ConstructorREferencesExample{
    public static void main(String[] args){
        Function<String, Member> function1 = Member :: new;
        Member member1 = function1.apply("angel");									  // 결과?
        
        BiFunction<String, String, Member> function2 = Member :: new;
        Member member2 = function2.apply("신천사", "angel")							// 결과? 
    }
}
```

```java
public class Member{
	private String name;
    private String id;
    
    public Member(){system.out.println("Member() 실행");}
    public Member(String id){system.out.println("Member(String id) 실행")}
    public Member(String id){system.out.println("Mebmer(String id, String name) 실행")}
}
```



-----------

### ▶한 눈에 다시보기

```java
public class Greeting {
    private String name;

    //생성자
    public Greeting() {}
    public Greeting(String name) {this.name = name;}
    public String getName() {return name;}
	//인스턴스메소드
    public String hello(String name) {return "hello " + name;}
    //정적메소드
    public static String hi(String name) {return "hi " + name;}
}
```

```java
public class App {
    public static void main(String[] args) {
        //인스턴스 메소드 사용을 위한 객체생성
        Greeting greeting = new Greeting();

        // static 메소드 사용
        UnaryOperator<String> hi = Greeting::hi;    
        System.out.println(hi.apply("naki"));							// hi naki 

        // Greeting 클래스의 hello 메소드 사용(인스턴스)
        UnaryOperator<String> hello = greeting::hello;    
        System.out.println(hello.apply("naki"));						// hello naki
 		
        // 생성자 참조
        Supplier<Greeting> newGreeting = Greeting::new;  
        newGreeting.get();
		
        // 매개변수가 있는 생성자
        Function<String, Greeting> nakiGreeting = Greeting::new;    
        Greeting naki = nakiGreeting.apply("naki");
        System.out.println(naki.getName());								//naki
    }
}
```
```java
//임의 객체의 인스턴스 메소드 참조
// before
String[] names = {"naki", "megu", "maru"};
Arrays.sort(names, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        return 0;
    }
});

//after
String[] names = {"naki", "megu", "maru"};
Arrays.sort(names, String::compareToIgnoreCase);
System.out.println(Arrays.toString(names));								//[maru, megu, naki]
```


- `//before`의 Comarator<T>가 함수형 인터페이스기 때문에 람다식으로 사용이 가능하다.

  사용 가능한 람다식의 방법들 중, <span style="color:violet">임의의 객체</span>의 <span style="color:blue">인스턴스 메소드</span>를 참조하는 형식으로 
  <span style="color:violet">String 객체</span>의 <span style="color:blue">compareToIgnoreCase 메소드</span>를 참조했다고 보면 

- `Comparator.java`

- ![image-20220510134446060](C:\Users\yezi\AppData\Roaming\Typora\typora-user-images\image-20220510134446060.png) 

- `String.java`

  ![image-20220510133820962](C:\Users\yezi\AppData\Roaming\Typora\typora-user-images\image-20220510133820962.png) 

  


## 인터페이스 기본 메소드와 스태틱 메소드

- ### 인터페이스

 **자바에서 인터페이스는 <u>객체의 사용 방법을 정의</u>한 타입**으로 객체의 교환성을 높여주기 때문에 다형성을 구현하는데 매우 중요한 역할을 수행한다. (코드가 인터페이스의 의 메소드를 호출하면, 인터페이스는 객체의 메소드를 호출한다.) 

인터페이스는 하나의 객체가 아니라 여려 객체들과 사용이 가능하므로 어떤 객체를 사용하느냐에 따라서 실행 내용과 리턴값이 달라질 수 있다. 따라서 개발 코드 측면에서는 코드 변경 없이 실행 내용과 리턴값을 다양화할 수 있다는 장점을 가진다.

특히 자바 8에서 인터페이스는 매우 중요한데, **자바 8의 람다식이 함수적 인터페이스의 구현 객체를 생성**하기 때문이다. 

> 👉 개발 코드는 객체 내부의 구조를 알 필요가 없고, 인터페이스의 메소드만 알고 있으면 된다.
> 👉 **사용이유?** 개발 코드를 수정하지 않고, 사용하는 객체를 변경할 수 있도록 하기 위함.


-----------------------


### ▶Default Method

디폴트 메소드는 인터페이스에 선언되지만 사실은 객체(구현객체)가 가지고 있는 인스턴스 메소드라고 생각해야 한다. **디폴트 메소드의 허용 이유는 기존의 인터페이스를 확장하여 새로운 기능을 추가하기 위함이다.**

```java
public interface Foo{
    void printName();
}
```

```java
public class DefaultFoo implements Foo{
    @Override
    public void printName(){
        system.out.println("DefaultFoo")
    }
}
```

- 이렇게 인터페이스를 선언하고, 인터페이스를 implements 하면 인터페이스 내부에 선언한 메소드를 

  @Override의 형태로 구현해야하는데, 

  ```java
  public interface Foo{
      void printName();
      void printUpperName();
  }
  ```

   이렇게 interface에 메소드를 그냥 추가하게 되면, 추가하기 전에 Foo를 구현한 모든 클래스는 컴파일 오류가 뜬다. (printUpperName에 대한 내용도 추가 구현을 해줘야하기 때문에)

  이렇듯, 모든 클래스에 제공을 할 만한 기능을 추가하는 경우, 단순한 메소드의 형태가 아니라 default의 형태로 추가 해줘야함

  ```java
  public interface Foo{
      void printName();
      
      default void printNameUpperCase(){
          system.out.println(getName().toUpperCase)
      }
  }
  ```

  ```java
  public class DefaultFoo implements Foo{
      String name;
      
      public DefaultFoo(String name){
          this.name = name;
      }
      
      @Override
      public void printName(){
          system.out.println(this.name);
      }
      
      @Override
     	public String getName(){
          return this.name;
      }   
  }
  ```

  ```java
  public class App{
      public static void main(String[] args){
          Foo foo = new DefaultFoo("naki");
          foo.printName();											//naki
         	foo.printNameUpperCase();									//NAKI
          //DefaultFoo에 printNameUpperCase()를 구현하진 않았지만,
  		//Foo interface에 default 메소드로 구현되어 있으므로 사용 가능
      }
  }
  ```

  🚨주의 : default 메소드는 구현체가 모르게 추가된 기능이므로 리스크를 동반함.

  ​				제공하는 기본 구현체에 대한 설명을 **주석을 달아주는 게 좋음** 

  ​				default 메소드 역시 @Override를 통해 구현하는 객체쪽에서 **재정의가 가능**

  ```java
  public interface Foo{
      void printName();
      
      /**
      * @implSpec
      * 이 구현체는 getName()으로 가져온 문자열을 대문자로 바꿔 출력한다.
      **/
      default void printNameUpperCase(){
          system.out.println(getName().toUpperCase)
      }
  }
  ```

-----

### ▶Static Method

정적 메소드는 디폴트 메소드와 마찬가지로 자바8에서 추가된 인터페이스의 새로운 멤버다. 
형태는 클래스의 정적 메소드와 완전히 동일. 정적 메소드는 public 특성을 갖기 때문에 public을 생략하더라도 컴파일 과정에서 자동으로 public이 붙게 된다. 

```java
public interface Foo{
    void printName();
    
    static void printAnything(){
    	system.out.println("Foo");
    }
}
```

- 인터페이스에 바로 static method를 선언하면

  ```java
  public class App{
      public static void main(String[] args){
      	//따로 클래스에서 재정의하지 않아도, 바로 인터페이스를 참조해서 갖다 쓸 수 있음.
          Foo.printAnything();
      }
  }
  ```

  

## 자바 8 API의 기본 메소드와 스태틱 메소드

#### Iterable의 기본 메소드

- forEach()
- spliterator()

#### Collection의 기본 메소드

- stream() / parallelStream()
- removeIf(Predicate)
- spliterator()

#### Comparator의 기본 메소드 및 스태틱 메소드

- reversed()
- thenComparing()
- static reverseOrder() / naturalOrder()
- static nullsFirst()  nullsLast()
- static comparing()
