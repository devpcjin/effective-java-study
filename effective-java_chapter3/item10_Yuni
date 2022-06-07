# Item 10 - Obey the general contract when overriding equals


### When NOT to use equals - each instance of the class is equal only to itself


1. Each instance of the class is inherently unique.


2. You don’t care whether the class provides a “logical equality” test.


3. A superclass has already overridden equals, and the superclass behavior is appropriate for this class.


4. The class is private or package-private, and you are certain that its quals method will never be invoked. 




### How to override the equals method?


    - Reflexive : x = x


    - Symeetric : x = y & y = x

    ```java
    // Broken - violates symmetry!
    public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        if (s == null)
            throw new NullPointerException();

        this.s = s;
    }
    }
    ```
 - the equals method in CaseInsensitiveString knows about ordinary strings, the equals method in String is oblivious to case- insensitive strings

    ```java
    // Broken - violates symmetry!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String) // One-way interoperability!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    
    CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
    String s = "polish";
    ```


- equals contract violated, you don't know how other objects will behave, to eliminate the problem : 

    ```java
    @Override public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString && 
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
    ```


    - Transitive : x = y & y = z => x = z


    ```java
        public class Point {
            private final int x;
            private final int y;
            public Point(int x, int y) {
                this.x = x;
                this.y = y;
            }
            @Override public boolean equals(Object o) {
                if (!(o instanceof Point))
                    return false;
                Point p = (Point)o;
                return p.x == x && p.y == y;
            }
        }
    ```

 - extend this class:  
    ```java
        public class ColorPoint extends Point {
            private final Color color;
            public ColorPoint(int x, int y, Color color) {
                super(x, y);
                this.color = color;
            }
        }
    ```


```java
    // Broken - violates symmetry!
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```
- This leeads to a problem in below case :


>Point p = new Point(1, 2);
 ColorPoint cp = new ColorPoint(1, 2, Color.RED);

>p.equals(cp) = true;

>cp.equals(p) = false;


- can solve this problem by making ColorPoint.equals ignore color


    ```java
    // Broken - violates transitivity!
        @Override public boolean equals(Object o) {
            if (!(o instanceof Point))
                return false;
            // If o is a normal Point, do a color-blind comparison
            if (!(o instanceof ColorPoint))
                return o.equals(this);
            // o is a ColorPoint; do a full comparison
            return super.equals(o) && ((ColorPoint)o).color == color;
        }
    ```
 - Satisfies symmetry at the expense of transivity

    >ColorPoint p1 = new ColorPoint(1, 2, Color.RED);


    >Point p2 = new Point(1, 2);


    >ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

    Here, p1 = p2 & p2 = p3, but p1 != p3

- java.sql.Timestamp extends java.util.Date and adds a nanoseconds field and that violates symmetry.

```java

```



    - Consistent : x = y always


    - non-nullity : return FALSE when compared to null


```java
// Class with a typical equals method 
public final class PhoneNumber { 
       private final short areaCode, prefix, lineNum;

public PhoneNumber(int areaCode, int prefix, int lineNum) {               
       this.areaCode = rangeCheck(areaCode, 999, "area code"); 
       this.prefix = rangeCheck(prefix, 999, "prefix"); 
       this.lineNum = rangeCheck(lineNum, 9999, "line num"); 
}

private static short rangeCheck(int val, int max, String arg) {
       if (val < 0 || val > max) throw new IllegalArgumentException(arg + ": " + val); 
       return (short) val; 
}

@Override 
public boolean equals(Object o) {

       if (o == this) return true; 
       if (!(o instanceof PhoneNumber)) return false; 
       PhoneNumber pn = (PhoneNumber)o; 
       return pn.lineNum == lineNum && pn.prefix == prefix
&& pn.areaCode == areaCode;

} ... // Remainder omitted

}
```


## 4 steps-recipe for a high-quality equals

    1. Use == operator to check if the argument is a reference to this object. If so return true: if(o == this) return true;


    2. Use instanceof operator to check if the argument has the correct type. If not return false: if(!( o instanceof Type)) return false;


    3. Cast the argument to the correct type:Type type = (Type) o;


    4. For each significant field in the class, check if that field matches the corresponding field of this object. If all these tests succeed return true otherwise return false.


```java
public class Location {
    private final int id;
    private final double latitude;
    private final double longitude;
    private final float altitude;

    public Location(int id, double latitude, double longitude, float altitude) {
        this.id = id;
        this.latitude = latitude;
        this.longitude = longitude;
        this.altitude = altitude;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Location)) return false;
        Location location = (Location) o;
        return id == location.id &&
                Double.compare(location.latitude, latitude) == 0 &&
                Double.compare(location.longitude, longitude) == 0 &&
                Float.compare(location.altitude, altitude) == 0;
    }
}
```

## Final notes

 - Always override hashCode when you override equals.


 - Don't try to be too clever. 


 - Don't substitute another type for Object in the equals declaration. 
