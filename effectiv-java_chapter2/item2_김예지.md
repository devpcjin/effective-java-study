## 이해를 위해 필요한 개념

>#### <span style="color:violet">[정적 팩터리 메소드](https://velog.io/@nnakki/이펙티브-자바-1.-생성자-대신-정적-팩토리-메서드-고려하기)</span> 
>#### <span style="color:violet">[불변클래스(item 17)](https://velog.io/@nnakki/이펙티브-자바17.-변경-가능성-최소화하기Immutable-Class)</span>
>#### [완전한상태](https://codingdog.tistory.com/entry/%EC%83%9D%EC%84%B1%EC%9E%90%EC%99%80-%EA%B0%9D%EC%B2%B4%EC%9D%98-%EC%99%84%EC%A0%84%ED%95%9C-%EC%83%81%ED%83%9C)
#### 공변반환타이핑(convariant return typing)
- 리턴 타입은 서브클래스라는 범위 안에서 다양할 수 있다.
원래는 오버라이딩 이름이 같아야하고, 매개변수가 같아야하고, 반환타입도 같아야 했지만 
Java 1.5부터는 Primitive 타입이 아닌 SubClass 타입으로 오버라이딩이 가능하다.
```java
public class Main {
	/** Test를 반환하도록 추상함수 ret을 정의**/
    abstract static class Test {   
        int a = 1;
        public int getA() {return a;}     
        abstract public Test ret();
    }

	/**Test의 하위 클래스 T에서 ret을 구현할 때 반환형을 자기 자신인 T로 오버라이딩**/
    static class T extends Test {
        int b = 2;        
        public int getB() {return b;}
        
        @Override
        public T ret() {return new T();}
    }
    
    public static void main(String[] args) {
        T t = new T();
        t.getA();
        t.getB()
    }
}
```
상속관계에서 `T extends Test`라는 건 `T is Test` 가 성립한다. 
그렇기에 Test대신 하위타입인 T를 반환해도 문제가 없다
-> 메서드를 재정의할 때 재정의 된 메서드의 반환 유형이 재정이 된 메서드의 반환 유형의 하위 유형이 될 수 있음을 의미한다.
#### [명명된 선택적 매개변수(named optional parameters) in python](https://www.delftstack.com/ko/howto/python/python-optional-arguments-optional-parameters-python/#python%EC%97%90%EC%84%9C-%EC%9D%B8%EC%88%98%EB%A5%BC-%EC%84%A0%ED%83%9D%EC%A0%81%EC%9C%BC%EB%A1%9C-%EB%A7%8C%EB%93%A4%EA%B8%B0)
함수를 호출 할 때마다 함수가 인수를 받는지 여부에 따라 해당 함수에 일부 인수를 전달한다.
하지만 함수에 인수를 전달하는 것은 필수가 아니고, **함수는 인수를 사용하지 않거나 임의의 수의 인수를 사용할 수 있다**


--------------



## 점층적 생성자 패턴 (telescoping constructor pattern)

```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, 
						int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, 
							int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}

// 사용예제
ex) NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

>매개변수가 늘어나면 늘어날 수록 클라이언트 코드를 작성하거나 읽는 게 어려워진다. 
<span style="color:red">**확장이 어렵다**</span>


## 자바빈즈 패턴(JavaBeans pattern)
: 매개변수가 없는 생성자로 객체를 만든 후, Setter 메서드를 통해 원하는 매개변수 값을 설정하는 방식.
```java
public class NutritionFacts {

    // 기본값 초기화, 필수 = -1
    private int servingSize = -1;
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {};
    
    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}

// 사용예제
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setCSodium(35);
cocaCola.setCarbohydrate(27);
```
>하나의 객체를 만들기 위해 여러개의 메스드를 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태가 된다. 점층적 생성자에서는 매개변수가 유효한 지를 생성자에서만 확인하면 됐는데, 그게 불가능해지면서 불변으로도 만들 수 없어졌다.  
> <span style="color:red">**일관성이 깨지고, 불변으로 만들 수 없다.**</span>

---------------

## <span style="color:violet">빌더 패턴 (Builder pattern)</span>
점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성, 이점만을 취한 패턴
1. `필수매개변수`만 으로 생성자(혹은 정적 팩토리)를 호출해 빌더 객체를 얻는다. 
2. 빌더 객체가 제공하는 일종의 `setter 메소드`로 원하는 선택 매개변수를 설정한다.
3. 마지막으로 .build()를 호출해 불변인 객체를 얻는다
><span style="color:red">확장이 어렵다</span> 
-> **자바빈즈와 마찬가지로 setter 메소드를 사용하는 형식이므로 확장이 쉽다.**
<span style="color:red">불변으로 만들 수 없다</span> 
-> `final`** 선언을 통해 불변성을 지킨다.**
<span style="color:red">일관성이 깨진다</span> 
-> **빌더의 setter 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.**
(fluent API, method chaning)

```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
        this.carbohydrate = builder.carbohydrate;
    };

    static class Builder {
        
        //필수 매개변수
        private final int servingSize;
        private final int servings;
        
        //선택적 매개변수 - 기본값으로 초기화
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}

// 사용예제
new NutritionFacts.Builder(240, 8).calories(240).sodium(35).carbohydrate(27).build();
```

>클라이언트는 생성자(혹은 정적 팩터리 메서드)를 호출하여 인스턴스를 얻는 것이 아닌, 
class 내에 정의되어 있는 정적 맴버 클래스 Builder를 이용하여 인스턴스를 얻는다.

------

#### 빌더 패턴은 계층적으로 설계 된 클래스와 함께 사용하기 좋다. 
각 계층의 클래스에 관련 빌더를 멤버로 정의. 추상 클래스는 추상 빌더, 구체 클래스는 구체 빌더를 갖게한다. 
>Pizza.Builder 클래스는 재귀적 타입 한정을 이용하는 제네릭 타입이다. 여기에 추상 메서드인 self를 추가하여 하위 클래스에서는 형변환하지 않고도 메서드 연쇄를 지원할 수 있다.
```java
public abstract class Pizza{
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>>{
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping){
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        protected abstract T self();
    }

    Pizza(Builder<?> builder){
        toppings = builder.toppings.clone();
    }
}
```

>**Pizza를 상속받는 NyPizza**
```java
public class NyPizza extends Pizza{
    public enum Size {SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder>{
        private final Size size;

        public Builder(Size size){
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```
>**Pizza를 상속받는 Calzone **
  ```java
public class Calzone extends Pizza{
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder>{
        private boolean sauceInsize = false;

        public Builder sauceInsize(){
            sauceInsize = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInsize;
    }
}
```
각 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다. 실제 Pizza의 build는 Pizza를 반환하지만, 이를 Overriding한 NyPizza의 build는 NyPizza를, Calzone의 build는 Calzone을 반환한다


```java
NyPizza nyPizza = new NyPizza.Builder(NyPizza.Size.SMALL)
                             .addTopping(Pizza.Topping.SAUSAGE)
                             .addTopping(Pizza.Topping.ONION).build();
Calzone calzone = new Calzone.Builder()
                             .addTopping(Pizza.Topping.HAM)
                             .sauceInsize().build();
```

이러한 빌더 패턴은 가변인수 매개변수를 여러개 사용할 수 있다는 이점이 있는데, 이는 생성자로는 누릴 수 없는 사소한 이점 중 하나이다.
또한 빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.
그러나 단점으로는 객체를 만들기 위해서 빌더부터 만들어야 한다는 것이다. 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.

---------

<span style="color:violet">**생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다.
빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.**</span>

