# 객체 생성과 파괴

## private 생성자나 열거 타입으로 싱글턴임을 보증하라

### 싱글턴
- 인스턴스를 오직 하나만 생성할 수 있는 클래스
    - 무상태 객체, 설계상 유일해야 하는 시스템 컴포넌트

- 싱글턴 패턴이 필요한 이유
    - 예시
        ![](https://injae-kim.github.io/assets/Dev/2020-08-06-singleton-pattern-usage/%EC%8A%AC%EB%9D%BC%EC%9D%B4%EB%93%9C1.JPG) [출처 : Injae's devlog](https://injae-kim.github.io/dev/2020/08/06/singleton-pattern-usage.html)
        - 1대만 존재하는 프린터를 여러 사람이 공유하는 상황
        - 프린터를 사용하려는 사람들이 프린터를 각자 생성해서 사용하는것은 불가능

- 

### 싱글턴을 만드는 방식
1. public static 멤버가 final 필드
    ```java
    public class Elvis{
        public static final Elvis INSTANCE = new Elvis();
        private Elvis() { ... }

        public void leaveTheBuilding() { ... }
    }
    ```
    - private 생성자는 `public static final` 필드인 `Elvis.INSTANCE`를 초기화 할 때 딱 한 번만 호출

    - public이나 protected 생성자가 없어 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장

    ```java
    @Test
        public void singletonTest(){
            Elvis elvis1 = Elvis.INSTANCE;
            Elvis elvis2 = Elvis.INSTANCE;

            assertSame(elvis1, elvis2); //SUCCESS
        }
    ```

    - 예외
        - 리플렉션 API인 `AccessibleObject.setAccessible`을 사용해 private 생성자를 호출 할 수 있음
        ```java
        Constructor<Elvis> constructor = (Constructor<Elvis>) elvis2.getClass().getDeclaredConstructor();
        constructor.setAccessible(true);

        Elvis elvis3 = constructor.newInstance();
        assertNotSame(elvis2, elvis3); // FAIL
        ```
        - 리플렉션 API : Class 객체가 주어지면 해당 클래스의 인스턴스를 생성하거나 메소드를 호출거나, 필드에 접근할 수 있음
    
    - 해결
        - 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던짐
        ```java
        private Elvis() {
            if(INSTANCE != null){
                throw new RuntimeException("생성자를 호출할 수 없습니다!");
            }
        }
        ```

2.  정적 팩터리 메서드를 public static 멤버로 제공
    ```java
    public class Elvis {
        private static final Elvis INSTANCE = new Elvis();
        private Elvis() { ... }
        public static Elvis getInstance() { return INSTANCE; }

        public void leaveTheBuilding() { ... }
    }
    ```
    - `Elvis.getInstance()`는 항상 같은 객체의 참조를 반환
        - 제 2의 Elvis 인스턴스는 만들어 지지 않음
    
    - 리플렉션을 통한 예외는 똑같이 적용


    ```java
    @Test
        public void singletonTest(){
            Elvis elvis1 = Elvis.INSTANCE;
            Elvis elvis2 = Elvis.INSTANCE;

            assertSame(elvis1, elvis2); //SUCCESS
        }
    ```
    - 장점
        - API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있음
            - `getInstance()`호출부 수정 없이 내부에서 private static이 아닌 새 인스턴스를 생성해 주면 됨
        - 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있음
            - 제네릭 싱글턴 팩터리 : 제네릭으로 타입설정 가능한 인스턴스를 만들어두고, 반환 시에 제네릭으로 받은 타입을 이용해 타입을 결정하는 것
([출처 : 제이크서 블로그](https://jake-seo-dev.tistory.com/13?category=909023))
        - 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있음
            - Supplier : get 메서드만을 가지고 아무 type이나 리턴할 수 있는 인터페이스
            ```java
            Supplier<Elvis> elvisSupplier = Elvis::getInstance;
            Elvis elvis = elvisSupplier.get();
            // 메서드에 대한 레퍼런스를 `Supplier`타입으로 만듦.
            ```

-  두가지 방식의 문제점

    - 각 클래스를 직렬화한 후 역직렬화할 때 새로운 인스턴스를 만들어서 반환
        - 역직렬화는 기본 생성자를 호출하지 않고 값을 복사해서 새로운 인스턴스를 반환
            - 직렬화 : 객체를 직렬화하여 전송 가능한 형태로 만드는 것을 의미
            - 역직렬화 : 직렬화된 파일 등을 역으로 직렬화하여 다시 객체의 형태로 만드는 것

    - 해결
        - 역직렬화할 때마다 새로운 인스턴스가 만들어지는 것을 예방하려면 `readResolve`메서드를 추가
        ```java
        private Obejct readResolve() {
            return INSTANCE;
        }
        // 진짜 Elvis는 반환하고 가짜 Elvis는 GC에 맡긴다
        ```
    
3. 원소가 하나인 열거 타입을 선언하는 방식
-  대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법
    ```java
    public enum Elvis {
        INSTANCE; 
        
        public String getName() {
            return "Elvis";
        }

        public void leaveTheBuilding() { ... }
    }
    ```

    ```java
    String name = Elvis.INSTANCE.getName();
    ```

    - Elvis 타입의 인스턴스는 `INSTANCE`하나 뿐 더이상 만들 수 없음
    - 복잡한 직렬화 상황이나 리플렉션 공격에도 제 2의 인스턴스가 생기는 일을 완벽히 차단

    - 단, 만들려는 싱글턴이 Enum외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없음
        - 열거타입이 다른 인터페이스를 구현하도록 선언할 수는 있음
