# Item 5 — Prefer dependency injection to hardwiring resources

## Dependency


 - In object oriented programming, if object 'A' contains 'B' as a data member, 'A' is dependent on 'B'. Also said, 'A' has a dependency on 'B'






## Coupling


 - The measurement of the dependency of an entity/object/software module over another. 
 A highly coupled design is undesirable. Loosely coupled software design is desirable since it enables easier testing of individual module by isolating a module.


 * example : buying a laptop VS a desktop
    - A laptop, a processor, gpu, psu are all soldered on the motherboard, making it un-upgradable. Laptop is tightly coupled.
    - A desktop has a great flexibilit in upgrading components. Thereby has a loose coupling.




## Dependency Injection


 - It is the process of liberating a class from the task of instantiating its own dependencies and delegating some other class to inject these dependencies on their behalf. 
 
 
 - Dependency injection ensures that loose coupling is achieved.




- Inappropriate use of static utility - inflexible & untestable!


 ```Java
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {} // Noninstantiable

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```




-  Inappropriate use of singleton - inflexible & untestable!


```Java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```




- Dependency injection provides flexibility and testability


```Java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```




  1. Constructor injection: the dependency injected as a parameter while object creation(Strict Dependencies).


  2. Setter injection : the dependency set using a setter for the dependency field(Optional dependencies).


  3. Method injection : if the dependency needs to change every time we use or has many forms and we are not sure which form will the dependency assume. (Multiple implementations of an interface).




## Example


1. Every cake is made of cake batter. This makes batter a necessary dependency.


2. A cake contains icing. However this is optional since we can savour plain cakes as well. This makes our dependency optional.


3. In case a cake contains topping, depending upon a user's request, we must be able to supply different toppings to the cake. Some may like fruit toppings, and some may love chocochips on their cake. Thus this dependency is varying in nature.




```Java
class Cake {
    
    private  CakeBatter cakeBatter;
    private  Icing icing;
    private  Topping topping;
    
}
```


1. Adding constructor injection to the cake class: This is trivial and since cakeBatter is a strict dependency we shall pass an instance of cakeBatter while creating a Cake object. Thus the code will look something like this:
```Java
class Cake {

    private  CakeBatter cakeBatter;
    private  Icing icing;
    private  Topping topping;
    
    
    //Constructor Injection of cakeBatter in the Cake Object
    Cake(CakeBatter cakeBatter) {
        this.cakeBatter = cakeBatter;
    }

}
```


2. Adding setter injection for the icing: This is also trivial and the code would look something like this:
```Java
class Cake {

    private  CakeBatter cakeBatter;
    private  Icing icing;
    private  Topping topping;


    //Constructor Injection of cakeBatter in the Cake Object
    Cake(CakeBatter cakeBatter) {
        this.cakeBatter = cakeBatter;
    }
    
    //Setter Injection of the Icing in the Cake Object
    void setIcing(Icing icing) {
        this.icing = icing;
    }
}
```


3. Adding method injection for the topping:
 - Suppose we have an interface topping containing the method addTopping. This interface is implemented by two classes FruitTopping and ChocolateTopping.
```Java
interface  Topping {
    void addTopping();
}

class FruitTopping implements   Topping {
    @Override
    public void addTopping() {
        System.out.println("Adding fruit toppings.");
    }
}

class ChocolateTopping implements  Topping {
    @Override
    public void addTopping() {
        System.out.println("Adding choco chips topping");
    }
}
```


- To set this topping dependency our method in the Cake class would look something like this.
```Java
class Cake {

    private  CakeBatter cakeBatter;
    private  Icing icing;
    private  Topping topping;


    //Constructor Injection of cakeBatter in the Cake Object
    Cake(CakeBatter cakeBatter) {
        this.cakeBatter = cakeBatter;
    }

    //Setter Injection of the Icing in the Cake Object
    void setIcing(Icing icing) {
        this.icing = icing;
    }
    
    //Method Injection of the Topping in the Cake Object
    void addToppingToCake(Topping topping) {
        this.topping = topping;
        topping.addTopping();
    }
}
```


- To send a particular implementation of this topping, our code will be:
```Java
//Adding fruit topping in delegated class
void addFruitTopping(Cake cake) {
    Topping topping = new FruitTopping();
    cake.addToppingToCake(topping);
}

//Adding chocolate topping in delegated class 
void addChocolateTopping(Cake cake) {
    Topping topping = new ChocolateTopping();
    cake.addToppingToCake(topping);
}
```




## Reference
 - https://github-wiki-see.page/m/saurabhojha/Effective-java/wiki/Item-5:-Prefer-dependency-injection-to-hardwiring-resources
  - Bloch, J. (2018). Effective Java Third Edition. USA : Addison-Wesley.
