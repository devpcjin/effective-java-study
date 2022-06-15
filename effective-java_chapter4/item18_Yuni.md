# Item 18: Favor Composition over Inheritance


## Types of Inheritance


1. implementation inheritance : one class extends the functionality of another class. - handled in the book and this post


2. interface inheritance : a class implements an interface or one interface extends another interface.


- Inheriting from classes across package is dangerous.
- The core is inheritance breaks encapsulation unless the superclass’s authors have designed it specifically for the purpose of being extended... 
* cause issues in the sub-classes without the sub-classes realizing it => breakage, unexpected data leakage


* example of the problem
```Java
@NoArgsConstructor
public class InstrumentedHashSet<E> extends HashSet<E> {
  @Getter
  private int addCount = 0;

  public InstrumentedHashSet(int initCap, float loadFactor) {
    super(initCap, loadFactor);
  }

  @Override
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }

  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(c);
  }
}
```


- gives 'addCount' of 6 not 3 since 'addAll' adds 3 and 'add' adds 3.


 - So use composition. This changes the inheritance is-a relationship to a has-a relationship.


```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
  @Getter
  private int addCount;

  public InstrumentedSet(Set<E> wrappedSet) {
    super(wrappedSet);
  }

  @Override
  public boolean add(E newElement) {
    addCount++;
    super.add(newElement);
  }

  @Override
  public boolean addAll(Collection<? extends E> newElements) {
    addCount += newElements.size();
    return super.addAll(newElements);
  }
}

@RequiredArgsConstructor
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> set;

  public void clear() {
    set.clear();
  }

  public boolean contains(Object o) {
    return set.contains(o);
  }

  public boolean isEmpty() {
    return set.isEmpty();
  }

  public int size() {
    return set.size();
  }

  ... repeated for every method in the Set interface.
}
```


- Took our one class using inheritance and turned it into 2 classes only using composition


- this is called 'Decorator pattern' as we are decorating an already existing object.
    
    
    * (+) Robust: Isolated from chnages in the individual concrete classes. 


    * (+) Flexibility: we take a Set of any type(HashSet) and operate on any of the sets. 

    * (-) Lengthy: small class with inheritance became 2 long classes.


- When B is-a B, inheritance is appropriate. if B just needs to have the behavior of A, composition is better.   




## How to use composition and forwarding technique


1. Create a Forwarding class
    * Composition : Give your class a private field that references an instance of the existing class (Set)


```Java
//your forwarding class
public class ForwardingSet<E> implements Set<E> {
 	private final Set<E> s;
 	...
}
```


    * Forwarding: Each instance method in the new class invokes the corresponding method on the contained instance of the existing class and returns the results


```Java
@Override
	public boolean add(E e) {
		return s.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		return s.addAll(c);
	}
```



2. Extend the forwarding class


```Java
public class InstrumentedSet<E> extends ForwardingSet<E> {

	// The number of attempted element insertions
	private int addCount = 0;
    
        // constructor 
	public InstrumentedSet(Set<E> s) {
		super(s);
	}

	@Override
	public boolean add(E e) {
	...your new implementation ...
	}
	
	....other overriden methods...
	
	//your new method
	public int getAddCount() {
		return addCount;
	}
```


3. Final look


```Java
	InstrumentedSet<String> s2 = new InstrumentedSet<>(new TreeSet<>());
	s2.addAll(Arrays.asList("Snap", "Crackle", "Snap"));
	System.out.println("Total: "  + s2.getAddCount());
```








### Reference
 - https://suleymanyildirim.org/java/item-18-favor-composition-over-inheritance-when-one-class-extends-another/


 - https://dev.to/kylec32/effective-java-tuesday-favor-composition-over-inheritance-4ph5
