# 주제
> finalizer와 cleaner 사용을 피하라

# 소멸자
finalizer와 cleaner는 자바에서 제공하는 소멸자이다.
자바에서 제공하지만 쓰면 안된다. 가장 치명적인 단점인 예측불가라는 단점이 있다.
자바에서는 GC가 객체를 회수하며 비메모리 자원은 try-with-resources와 try-finally로 해결한다.

# 단점
1. 제때 실행되어야 하는 작업은 절대 할 수 없다.(수행시점)
unreachable한 객체가 GC가 동작하여 메모리를 회수하는 시점을 정확하게 예측할 수 없다.
언제 실행될지는 전적으로 GC 알고리즘에 달렸지만 GC 구현 마다 다르고 finalizer나 cleaner의 수행 시점에 의존하는 프로그램의 동작 또한 마찬가지다.

2. 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.(수행여부)
종료 작업을 전혀 수행하지 못한 채 중단될 수도 있다. 프로그램 생애주기와 상관없는 상태를 영구적으로 수정하는 작업에서는 사용해서는 안된다.
System.gc 메서드에 현혹되면 안된다. System.gc를 호출해도 GC가 그때 발생한다는 보장이 없다.

3. 예외처리
finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다. 잡지못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있고 다른 스레드가 훼손된 객체를 사용하려다 예측불가능한 결과를 가져올 수 있다.

4. finalizer와 cleaner는 심각한 성능 문제도 동반한다.(성능)
try-with-resource를 사용해 객체를 수거하는 방식에 비해 성능이 확 떨어진다.

5. finalizer 공격에 노출되어 심각한 보안문제를 일으킬 수도 있다.
finalizer는 생성자나 직렬화 과정에서 예외가 발생하면 생성되다만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 한다. finalizer는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다.
객체 생성을 막으려면 생성자에서 예외를 던지면 되지만 finalizer가 있다면 그렇지 않다.
final이 아닌 클래스를 finalizer 공격으로 부터 방어하려면 아무일도 하지않는 finalize()메서드를 만들고 final로 선언하자.

# 대안
파일이나 스레드 등 종료해야 할 자원을 담고 있는 객체 클래스에서 finalizer와 cleaner를 대신할 묘안은 try-with-resources이다. AutoCloseable을 구현해주고 try-with-resources를 사용해 예외가 발생해도 제대로 종료될 수 있다.

# cleaner와 finalizer의 쓰임새
### 1. 자원의 소유자가 close메서드를 호출하지 않는 것에 대한 안전망
즉시 호출된다고 보장은 없지만 자원 회수를 늦게라도 해주므로 안전망 역할을 할 수 있다. 예를 들어, 
### 2. 네이티브 피어와 연결된 객체에
네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다. 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못한다. 그 결과 자바 피어를 회수 할 때 네이티브 객체까지 회수하지 못한다. 성능저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 앞서 설명한 close메서드를 사용해야 한다.

# cleaner 사용법
>
Cleaner를 안전망으로 사용하는 방법

```java
class Room implements AutoCloseable{
    private static final Cleaner cleaner = Cleaner.create();
    private static class State implements Runnable{
        int numJunkPiles; // 방(Room)안의 쓰레기 수

        public State(final int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }
        @Override
        public void run() {
            System.out.println("방청소");
            numJunkPiles = 0;
        }
    }

    private final State state; //방의 상태. cleanable 과 공유한다.

    private final Cleaner.Cleanable cleanable; //cleanable 객체. 수거 대상이 되면 방을 청소한다.

    public Room(final int numJunkFiles) {
        this.state = new State(numJunkFiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```
* Room의 close메서드를 호출할 때 run이 호출됨
* GC가 Room을 회수할 때까지 close가 호출되지 않는다면 cleaner가 (may be)State의 run메서드를 호출함
* State인스턴스는 Room인스턴스를 절대로 참조하면 안됨 - 순환참조 발생
* Room의 cleaner는 단지 안전망이다. try-with-resources를 이용하면 전혀 문제가 없다.
```java
    public static void main(String[] args) throws Exception {
        try(Room room = new Room(1)){
            System.out.println("안녕~~~");
        }
    }
```
>안녕
방청소
이렇게 출력된다. 위 결과는 잘 짜여진 예이다.

```java
    public static void main(String[] args) throws Exception {
        new Room(99);
        System.out.println("아무렴");
//      System.gc();
    }
```
>아무렴
이렇게 출력된다. 아무렴 다음 방청소가 출력될수도 있지만 안될수도 있다.
그건 보장을 못한다. System.gc()를 추가하면 나올수도 있고 안나올수도 있다.
결국 안전하지 않다.
