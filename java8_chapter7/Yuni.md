# Annotation
- Spring and Hibernate heavily rely on annotations
- ~ Java 8, annotations only on declaration. In Java 8, also in generics and casts
1. Inform the compiler about warnings and errors
2. Manipulate source code at compilation time
3. Modify or examine behavior at runtime






### @FunctionalInterface
- When we use a SAM interface to be used by lambdas
```java
@FunctionalInterface
public interface Adder {
    int add(int a, int b);
}
```
- Works as @Override method. Now, whether we use @FunctionalInterface or not, we can still use Adder in the same way:

```java
Adder adder = (a,b) -> a + b;
int result = adder.add(4,5);
```
 - but if we add a second method to Adder, the compiler will complain:
```java
@FunctionalInterface
public interface Adder { 
    // compiler complains that the interface is not a SAM
    
    int add(int a, int b);
    int div(int a, int b);
}
```
 - Even though it's legal to have more than one method on an interface, it isn't when that interface is being used as a lambda target. Without this annotation, the compiler would break in the dozens of places where Adder was used as a lambda. Now, it just breaks in Adder itself.






### @Native
 - only applicable to fields
 - the annotated field is a constant that may be referenced from the native code

```java
public final class Integer {
    @Native public static final int MIN_VALUE = 0x80000000;
    // omitted
}
```






## Meta-Annotations


### @Target
 - consumed with constructor and field declarations.
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD})
public @interface SafeVarargs {
}
```






### @Retention
 - some annotations used as hints for the compiler, while others are used at runtime.
 - tells where in our program's lifecycle our annotation applies
1. RetentionPolicy.SOURCE – visible by neither the compiler nor the runtime
2. RetentionPolicy.CLASS – visible by the compiler(default)
3. RetentionPolicy.RUNTIME – visible by the compiler and the runtime

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(TYPE)
public @interface RetentionAnnotation {
}
```






### @Inherited
 - subclass bound to a parent class
```java
@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface InheritedAnnotation {
}

@InheritedAnnotation
public class BaseClass {
}

public class DerivedClass extends BaseClass {
}
```
 - after extending the BaseClass, DerivedClass appears to have the same annotation at run time:
```java
@Test
public void whenAnnotationInherited_thenShouldExist() {
    DerivedClass derivedClass = new DerivedClass();
    InheritedAnnotation annotation = derivedClass.getClass()
      .getAnnotation(InheritedAnnotation.class);
 
    assertThat(annotation, instanceOf(InheritedAnnotation.class));
}
```






### @Repeatable
Before, we had to group annotations together into a single container annotation:
```java
@Schedules({
    @Schedule(time = "15:05"),
    @Schedule(time = "23:00")
})
void scheduledAlarm() {
}
```

- with @Repeatable, we can make an annotation repeatable:
```java
@Repeatable(Schedules.class)
public @interface Schedule {
    String time() default "09:00";
}
```






# Arrays.sort vs Arrays.parallelSort


## Arrays.sort()
- single-threaded with two variants
1. sort(array) – sorts the full array into ascending order
2. sort(array, fromIndex, toIndex) – sorts only the elements from fromIndex to toIndex

(-)


 - Performance degrades for large datasets
 - Multiple cores of the system aren't utilized




## Arrays.parallelSort()
 - uses a parallel sort-merge sorting algorithm. Breaks the array into sub-arrays that are themselves sorted and then merged.
 

(-)


 - Slower for smaller size arrays

### reference
 - https://www.baeldung.com/java-default-annotations
 - https://www.infoq.com/articles/Type-Annotations-in-Java-8/#:~:text=With%20Java%208%2C%20annotations%20can,of%20the%20new%20Java%20release.
 - https://www.baeldung.com/java-arrays-sort-vs-parallelsort