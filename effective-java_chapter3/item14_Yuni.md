# Item 14: Consider implementing Comparable


 - compareTo method is not declared in Object but is a sole method in Comparable interface.


  - order & equality comparisons (once implement the interface, sorting is simple by using Arrays.sort(myArray))

  * example
  ```java
    public interface Comparable<T> {
    int compareTo(T t);
    }
  ```
  * compare objects' order, return negative number, 0, or positive number if the object is less than, equal to, or greater than the provided object.


  * throw a ClassCastException if the provided object type is not compatible to be compared with the object.


## compareTo method general contract
- sgn means signum function
1. For all x and y, sgn(x.compareTo(y)) == -sgn(y.compareTo(x))


2. Related to the above, x.compareTo(y) should only throw an exception if y.compareTo(x) also throws an exception.


3. Same as the equals function, compareTo should be transitive. Therefore, if x.compareTo(y) > 0 && y.compareTo(z) > 0 then x.compareTo(z) > 0. This should also work with < and ==


* If compareTo returns 0 are they equal?
    - Not always...


```java
public static void main(String[] args) {

        BigDecimal a = new BigDecimal("1.0");
        BigDecimal b = new BigDecimal("1.00");

        Set<BigDecimal> hs = new HashSet<>();
        hs.add(a);
        hs.add(b);

        System.out.println(hs.size()); // 2

        Set<BigDecimal> ts = new TreeSet<>();
        ts.add(a);
        ts.add(b);

        System.out.println(ts.size()); // 1

    }
```


- HashSet uses euqals method returning 2 elements, TreeSet use compareTo method returning 1 element.




### How can we write compareTo method?


1. Determine the order of significance of the fields of the class.


2. Compare the fields by either recursively calling compareTo methods for reference types or use one of the built in BoxedTime.compare() methods such as Double.compare().


3. Once you find a field that is unequal return the value for that field (or if there are no differences return 0).

```java
    public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0) {
        result = Short.compare(lineNumber, pn.lineNumber);
        }
    }
    return result;
    }
```


* to get rid of the deep indentation using Java 8 alternative way :
```java
    private static final Comparator<PhoneNumber> COMPARATOR = 
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
    }
```


### Reference
- https://dev.to/kylec32/effective-java-tuesday-consider-implementing-comparable-3m9j
