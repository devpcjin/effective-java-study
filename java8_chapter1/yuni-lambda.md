# Lambda

# Understanding of Lambda
- A Lambda expression implements a functional interface

We just send the action itself, we had contained the expression in an object before.

an anonymous method that implements a functional interface (can only be used with a functional interface)

Abstract method = has no implementation in an interface interface has ONE abstract method => functional interface(=SAM interface) could contain other kind of method like static or default method. More than 1 method, lambda cannot be used

Class, provide an implementation, some class, implement printable interface, and then provide implementation of the method. And Then, create a new object, pass in the object ,

aBlockOfCode = **public void perform**() { System.out.println("Hello World!"); }


public = is accessible with the variable aBlockOfCode, not needed perform = will use the variable name aBlockOfCode, not needed void = compiler can easily tell what the return is, not needed Parameter, actual method implementation is needed

that leaves out,

aBlockOfCode = () { System.out.println("Hello World!"); }

Finally,

aBlockOfCode = () -> System.out.println("Hello World!");
