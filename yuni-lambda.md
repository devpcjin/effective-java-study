# Lambda

# Understanding of Lambda
- an anonymous method that implements a functional interface
  (can only be used with a functional interface)
  
  Abstract method = has no implementation in an interface
  interface has ONE abstract method => functional interface(=SAM interface) could contain other kind of method like static or default method
  
  More than 1 method, lambda cannot be used

- Functional programming: Passing code/function as parameter
- Class, provide an implementation, some class, implement printable interface, and then provide implementation of the method. And Then, create a new object, pass in the object ,

- just pass in the print method with printable method
- public is not needed, name of the method is also not needed, we just need to have what it does. return type is also not needed. Compiler can decide what to return. Parameter, actual method implementation is needed
 () -> {
   System.out.println("Meow")

  }

We just send the action itself, we had contained the expression in an object before.

Functions as Value
