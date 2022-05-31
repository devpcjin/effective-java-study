# Static Factory Method VS Constructor
- constructor :  the main purpose is to perform initialization of an object but not to create an object. 
    * Foo x = new Foo()
- Static Factory Method
    * Foo x = Foo.create()

- Class can provide Static factory method that returns instance of the class
* ex) brings basic type 'boolean' and transform Boolean object reference
```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

| Constructor | Static Factory Method |
| :---        |    :---     |
| The constructor doesn’t have a meaningful name, so they always restricted to the standard naming convention | The static factory method can have a meaningful name hence we can explicitly convey what this method does.        |
| Constructors can’t have any return type not even void.|Static factory methods can return the same type that implements the method, a subtype, and also primitives.        |
| Inside the constructor, we can only perform the initialization of objects. | Inside static factory method other than initialization if we want to perform any activity for every object creation like increasing count value for every object creation we can do this in the static factory method.        |
| C onstructor always creates a new object inside the heap, so it is not possible to return a cached instance of the class from a constructor. | But Factory methods can take advantage of caching i.e we can return the same instance of Immutable class from the factory method instead of always creating a new object.        |
| The first line inside every constructor should be either super() or this() | But inside the factory method, it is not necessary that the first line must be either super() or this()        |



## Static factory method 장점
1. 이름을 가질 수 있다.
    - constructor 자체만으로는 반환될 객체의 특성을 제대로 설명 X
    - 이름을 잘 지으면 객체의 특성 묘사 가능, '값이 소수인 BigInteger을 반환한다'를 더 잘 나타내는 것은?
        * constructor : BigInteger(int, int, Random)
        * static factory method : BigInteger.probablePrime
    - 한 클래스에 시그니처가 같은 생성자 여러 개 필요 시, 생성자를 정적 팩터리 메서드로 바꾸고 이름 부여


2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
    - immutable class는 미리 인스턴스를 만들어 놓거나 새로 생성한 인스턴스를 cache & recycle.
    - Boolean.valueOf(boolean) does not create any object.
    - similar to Flyeight pattern.
    - strict control over instances, called instance-controlled
        * 클래스를 singleton, noninstanciable로 만들 수 있다.
        * Flyweight pattern의 근간
        * 열거 타입 인스턴스가 하나만 만들어짐을 보장
    

3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
    - 반환할 객체의 class 선택 가능, flexibility
    - API 만들 때 활용시 구현 class 공개하지 않고 객체 반환 가능, API 작게 유지
    - interface based framework 핵심 기술


4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    * ex) EnumSet class는 public constructor 없이 static factory만 제공, 원소의 수에 따라 두 가지 하위 class중 하나의 인스턴스 반환(원소 64개 이하 : long 변수 하나로 관리하는 RegularEnumset의 인스턴스, 65개 이상 : long 배열로 관리하는 JumboEnumSet 인스턴스 반환)


5. 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    - service provider framework, such as JDBC(Java Database Connectivity)를 만드는 근간이 된다. 3가지 core components
        * Service interface  
        * Provider registration API
        * Service access API
            - 사용시 원하는 구현체의 조건 명시 가능. 아닐 시 기본 구현체 혹은 지원하는 구현체들 하나씩 돌아가며 반환
    

## Static factory method 단점
1. public이나 protected 생성자 가 필요하니 하위 클래스를 만들 수 없다.


2. 프로그래머가 찾기 어렵다.
    - API 설명에 드러나지 않음




## Naming rules for Static Factory Method


- from : 매개변수 하나 받아서 해당 타입의 인스턴스를 반환하는 type conversion method


- of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 method


- valueOf : more specific version of from and of


- instance / getInstance : 매개변수로 명시한 인스턴스 반환하나 같은 인스턴스임을 보장 X


- create / newInstance : works as instance or getInstance, but always create a new instance when return


- getType : same as getInstance but used in other class' factory method not the one to create.


- newType : same as newInstance but used in other class' factory method not the one to create.


- type : shorter viersion for getType & newType.
