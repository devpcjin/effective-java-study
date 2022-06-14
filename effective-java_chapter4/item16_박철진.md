# item 16. public 클래스에서 public 필드가 아닌 접근자 메서드를 사용하라

## 퇴보한 클래스
- 인스턴스 필드들을 모아놓는 일 외에는 아무 목적도 없는 퇴보한 클래스
    ```java
    class Point {
        public double x;
        public double y;
    }
    ```
    - 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못함
        - API를 수정하지 않고는 내부 표현을 바꿀 수 없음
        - 불변식을 보장할 수 없음
        - 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없음

- 필드를 모두 private으로 바꾸고 `public 접근자(getter)`를 추가
    ```java
    class Point {
        private double x;
        private double y;

        public Point(double x, double y){
            this.x = x;
            this.y = y;
        }

        public double getX() { return x; }
        public double getY() { return y; }

        public void setX(double X) { this.x = x; }
        public void setY(double y) { this.y = y; }
    }
    ```
    - 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공
        - public 클래스가 필드를 공개하면 내부 표현 방식을 마음대로 바꿀 수 없음

- package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 문제 없음
    - package-private : 같은 패키지 및 같은 클래스에서만 접근 가능
    - private 중첩 클래스 : 내부 클래스가 private 형태로 된 클래스
    > 같은 패키지 안에서 어떤 특정 이유 때문에 사용하던가 탑 레벨 클래스에서만 접근하기 때문에 위에서 언급한 단점들이 나타날 이유가 없어 문제될 것이 없음


## public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례
- `java.awt.package`의 Point, Dimension 클래스
    - 심각한 성능 문제 발생 - 오늘날까지도 해결되지 않음

## 불변 필드를 노출한 public 클래스

```java
package effectivejava.chapter4.item16;

// 코드 16-3 불변 필드를 노출한 public 클래스 - 과연 좋은가? (103-104쪽)
public final class Time {
    private static final int HOURS_PER_DAY    = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    }

    // 나머지 코드 생략
}
```

- public 클래스 필드가 불변이라면 직접 노출할 떄의 단점이 조금은 줄어들지만, 여전히 좋은 코드가 아님
- public 필드에 final 키워드를 추가해 불변으로 만들면 불변식은 보장할 수 있게 되지만 여전히 API를 변경하지 않고는 표현 방식을 바꿀 수 없고 필드를 읽을 때 부수작업을 수행할 수 없다는 단점은 변하지 않음


## 핵심정리

> public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 `package-private클래스`나 `private 중첩 클래스` 에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.
