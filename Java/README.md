# Book: Java - The Complete Reference 11th Edition

## Abstraction
An essential element of object-oriented programming is *abstraction*. Humans manage complexity through abstraction. For example, people do not think of a car as a set of tens of thousands of individual parts. They think of it as a well- defined object with its own unique behavior. 

## Encapsulation
*Encapsulation* is the mechanism that binds together code and the data it manipulates, and keeps both safe from outside interference and misuse. One way to think about encapsulation is as a protective wrapper that prevents the code and data from being arbitrarily accessed by other code defined outside the wrapper.

## Inheritance
Inheritance is the process by which one object acquires the properties of another object. You can only specify one superclass for any subclass that you create. Java does not support the inheritance of multiple superclasses into a single subclass. However, no class can be a superclass of itself. Although a subclass includes all of the members of its superclass, it cannot access those members of the superclass that have been declared as private.

When a reference to a subclass object is assigned to a superclass reference variable, you will have access only to those parts of the object defined by the superclass. This is why plainbox can’t access weight even when it refers to a BoxWeight object.

Thus, super( ) always refers to the superclass immediately above the calling class. This is true even in a multileveled hierarchy. Also, super( ) must always be the first statement executed inside a subclass constructor.


## Polymorphism
Polymorphism (from Greek, meaning “many forms”) is a feature that allows one interface to be used for a general class of actions.

## Compiling the Program
The Java bytecode is the intermediate representation of your program that contains instructions the Java Virtual Machine will execute.

In this case, main( ) must be declared as public, since it must be called by code outside of its class when the program is started. The keyword static allows main( ) to be called without having to instantiate a particular instance of the class. This is necessary since main( ) is called by the Java Virtual Machine before any objects are made.

## Identifiers
Identifiers are used to name things, such as classes, variables, and methods. An identifier may be any descriptive sequence of uppercase and lowercase letters, numbers, or the underscore and dollar-sign characters. (The dollar-sign character is not intended for general use.) They must not begin with a number.

## The Primitive Types
- **Integers:** This group includes byte, short, int, and long, which are for whole-valued signed numbers. Variables of type *byte* are especially useful when you’re working with a stream of data from a network or file. 
- **Floating-point:** numbers This group includes float and double, which represent numbers with fractional precision.
- **Characters:** This group includes char, which represents symbols in a character set, like letters and numbers.
- **Boolean:** This group includes boolean, which is a special type for representing true/false values.


The primitive types represent single values—not complex objects. Although Java is otherwise completely object-oriented, the primitive types are not. They are analogous to the simple types found in most other non–object-oriented languages. The reason for this is efficiency. Making the primitive types into objects would have degraded performance too much.

### Float vs double
The type float specifies a single-precision value that uses 32 bits of storage. Single precision is faster on some processors and takes half as much space as double precision, but will become imprecise when the values are either very large or very small.

Double precision, as denoted by the double keyword, uses 64 bits to store a value. Double precision is actually faster than single precision on some modern processors that have been optimized for high-speed mathematical calculations.

## Variables
The variable is the basic unit of storage in a Java program. A variable declared within a block is called a *local variable*. Objects declared in the outer scope will be visible to code within the inner scope. However, the reverse is not true. Variables are created when their scope is entered, and destroyed when their scope is left. Although blocks can be nested, you cannot declare a variable to have the same name as one in an outer scope.

## Java’s Automatic Conversions
When one type of data is assigned to another type of variable, an automatic type conversion will take place if the following two conditions are met:
- The two types are compatible.
- The destination type is larger than the source type.

 ```Java
 byte b;
 int i = 257;
 b = (byte) i; //i is 1
 ```

When the value 257 is cast into a byte variable, the result is the remainder of the division of 257 by 256 (the range of a byte), which is 1 in this case.

```Java
byte a = 40;
byte b = 50;
byte c = 100;
int d = a * b / c;
```

The result of the intermediate term a * b easily exceeds the range of either of its byte operands. To handle this kind of problem, Java automatically promotes each byte, short, or char operand to int when evaluating an expression. This means that the subexpression a*b is performed using integers —not bytes. Thus, 2,000, the result of the intermediate expression, 50 * 40, is legal even though a and b are both specified as type byte.


Java defines several type promotion rules that apply to expressions. They are as follows:
- All byte, short, and char values are promoted to int, as just described. 
- If one operand is a long, the whole expression is promoted to long.
- If one operand is a float, the entire expression is promoted to float.
- If any of the operands are double, the result is double.

## Introducing Type Inference with Local Variables
Beginning with JDK 10, it is now possible to let the compiler infer the type of a local variable based on the type of its initializer, thus avoiding the need to explicitly specify the type. 
```Java
double avg = 10.0;
```
Using type inference, this declaration can now also be written like this:
```Java
var avg = 10.0;
```

In both cases, avg will be of type double. In the first case, its type is explicitly specified. In the second, its type is inferred as double because the initializer 10.0 is of type double. As mentioned, var was added as a context-sensitive identifier. When it is used as the type name in the context of a local variable declaration, it tells the compiler to use type inference to determine the type of the variable being declared based on the type of the initializer.

The preceding example uses var to declare only simple variables, but you can also use var to declare an array. For example:

```Java
var myArray = new int[10]; // This is valid.
```

Furthermore, you cannot use brackets on the left side of a var declaration.

```Java
var[] myArray = new int[10]; // Wrong
```
It is important to emphasize that var can be used to declare a variable only when that variable is initialized.

```Java
var counter; // Wrong! Initializer required.
```

Also, remember that var can be used only to declare local variables. It cannot be used when declaring instance variables, parameters, or return types, for example.

- Only one variable can be declared at a time
- A variable cannot use null as an initializer and the variable being declared cannot be used by the initializer expression. 
- Although you can declare an array type using var, you cannot use var with an array initializer.

## Operators
```Java
x = 42;
y = x++;
```
The value of x is obtained before the increment operator is executed, so the value of y is 42. Of course, in both cases x is set to 43.

If you use the || and && forms, rather than the | and & forms of these operators, Java will not bother to evaluate the right-hand operand when the outcome of the expression can be determined by the left operand alone. 

```Java
if (denom != 0 && num / denom > 10)
```
Since the short-circuit form of AND (&&) is used, there is no risk of causing a run-time exception when denom is zero. 

## Switch

```Java
switch (expression) { 
    case value1:
    // statement sequence break;
    case value2:
    // statement sequence break;
    case valueN :
    // statement sequence break;
    default:
    // default statement sequence
}
```
For versions of Java prior to JDK 7, expression must resolve to type *byte, short, int, char, or an enumeration*. Beginning with JDK 7, expression can also be of type *String*. 

In summary, there are three important features of the switch statement to note:
- The switch differs from the if in that switch can only test for equality, whereas if can evaluate any type of Boolean expression. That is, the switch looks only for a match between the value of the expression and one of its case constants.
- No two case constants in the same switch can have identical values. Of course, a switch statement and an enclosing outer switch can have case constants in common.
- A switch statement is usually more efficient than a set of nested ifs.

When it compiles a switch statement, the Java compiler will inspect each of the case constants and create a “jump table” that it will use for selecting the path of execution depending on the value of the expression. Therefore, if you need to select among a large group of values, a switch statement will run much faster than the equivalent logic coded using a sequence of if-elses. The compiler can do this because it knows that the case constants are all the same type and simply must be compared for equality with the switch expression. The compiler has no such knowledge of a long list of if expressions.

## Iteration Statements
Java permits you to include multiple statements in both the initialization and iteration portions of the for. Each statement is separated from the next by a comma.

```Java
for (int i = 0, j = 10; i < 10 && j > 0; i++,j--) {
    System.out.println("i = " + i + " :: " + "j= " + j);
}
```

You can intentionally create an infinite loop (a loop that never terminates) if you leave all three parts of the for empty.

```Java
for ( ; ; ){
    //...
}
```

### For-Each
A for-each style loop is designed to cycle through a collection of objects, such as an array, in strictly sequential fashion, from start to finish. In Java, the for-each style of for is also referred to as the enhanced for loop. There is one important point to understand about the for-each style loop. Its iteration variable is “read-only” as it relates to the underlying array.

```Java
for (int x[] : nums){
  for (int y : x){
      System.out.println("Valus is " + y);
  }
}
```

```Java
for (var y : nums){
      System.out.println("Valus is " + y);
}
```

## Jump statements
```Java
out:for(int i=1; i<=100; i++){
  System.out.println("outer");
  
  for(int j=1; j<=100; j++){
    System.out.println("nested");
    if(j==2){
    // break; this will exit from inner for loop only
    break out; // this will exit from both for loops
    }
  }
}
```

Keep in mind that you cannot break to any label which is not defined for an enclosing block.

```Java
one:for(int i=1; i<=100; i++){
  System.out.println("outer");
}

two:for(int i=1; i<=100; i++){
  if(j==2){
      break one; //WRONG
  }
}
```
Since the loop labeled one does not enclose the break statement, it is not possible to transfer control out of that block.

A *continue* statement causes control to be transferred directly to the conditional expression that controls the loop.

## Class
A class is a template for an object, and an object is an instance of a class.

The data, or variables, defined within a class are called *instance variables*.

Collectively, the methods and variables defined within a class are called *members* of the class.

First, you must declare a variable of the class type. This variable does not define an object. Instead, it is simply a variable that can refer to an object. Second, you must acquire an actual, physical copy of the object and assign it to that variable. You can do this using the new operator. The new operator dynamically allocates (that is, allocates at run time) memory for an object and returns a reference to it. This reference is, essentially, the address in memory of the object allocated by new. This reference is then stored in the variable.

```Java
 Box mybox; // declare reference to object 
 mybox = new Box(); // allocate a Box object
 ```

 ```Java
 Box b1 = new Box(); 
 Box b2 = b1;
 ```

 You might think that b2 is being assigned a reference to a copy of the object referred to by b1. That is, you might think that b1 and b2 refer to separate and distinct objects. However, this would be wrong. Instead, after this fragment executes, b1 and b2 will both refer to the same object. The assignment of b1 to b2 did not allocate any memory or copy any part of the original object. It simply makes b2 refer to the same object as does b1. Thus, any changes made to the object through b2 will affect the object to which b1 is referring, since they are the same object. Although b1 and b2 both refer to the same object, they are not linked in any other way. For example, a subsequent assignment to b1 will simply unhook b1 from the original object without affecting the object or affecting b2.

![alt text](img/img1.png " ")

```Java
Box b1 = new Box();
Box b2 = b1;
// ...
b1 = null;
```
Here, b1 has been set to null, but b2 still points to the original object.

### Constructors
Constructors look a little strange because they have no return type, not even void. This is because the implicit return type of a class’ constructor is the class type itself.

When you do not explicitly define a constructor for a class, then Java creates a default constructor for the class.

When using the default constructor, all non- initialized instance variables will have their default values, which are zero, null, and false, for numeric types, reference types, and boolean, respectively.

### The this Keyword
*this* can be used inside any method to refer to the current object.

### Garbage collection
Garbage collection only occurs sporadically (if at all) during the execution of your program. It will not occur simply because one or more objects exist that are no longer used.

### Overriding methods

In a class hierarchy, when a method in a subclass has the same name and type signature as a method in its superclass, then the method in the subclass is said to override the method in the superclass. Method overriding occurs only when the names and the type signatures of
the two methods are identical. If they are not, then the two methods are simply overloaded.

### Overloading Methods
return types do not play a role in overload resolution.

```Java
class OverloadDemo {
    void test(){
        //...
    }

    void test(int a){
        //...
    }
    
    double test(double a){
        //...
    }
}
```

some cases, Java’s automatic type conversions can play a role in overload resolution.

```Java
class OverloadDemo {
    void test(){
        //...
    }

    void test(int a, int b){
        //...
    }
    
    double test(double a){
        //...
    }

    public static void main(String args[]){
        test(88); //invokes test(double)
        test(132.23); //invokes test(double)

    }
}
```

### Argument Passing
The first way is *call-by-value*. This approach copies the value of an argument into the formal parameter of the subroutine. Therefore, changes made to the parameter of the subroutine have no effect on the argument. The second way an argument can be passed is *call-by-reference*. In this approach, a reference to an argument (not the value of the argument) is passed to the parameter. Inside the subroutine, this reference is used to access the actual argument specified in the call. This means that changes made to the parameter will affect the argument used to call the subroutine.

- When you pass a primitive type to a method, it is passed by value.
- When you pass an object to a method, the situation changes dramatically, because objects are passed by what is effectively call-by-reference.

Since all objects are dynamically allocated using new, you don’t need to worry about an object going out-of-scope because the method in which it was created terminates. The object will continue to exist as long as there is a reference to it somewhere in your program. When there are no references to it, the object will be reclaimed the next time garbage collection takes place.

## Recursion
When a method calls itself, new local variables and parameters are allocated storage on the stack, and the method code is executed with these new variables from the start. As each recursive call returns, the old local variables and parameters are removed from the stack, and execution resumes at the point of the call inside the method. Recursive methods could be said to “telescope” out and back.
Recursive versions of many routines may execute a bit more slowly than the iterative equivalent because of the added overhead of the additional method calls. A large number of recursive calls to a method could cause a stack overrun.

## Understanding static
When objects of its class are declared, no copy of a static variable is made. Instead, all instances of the class share the same static variable.

Methods declared as static have several restrictions:

- They can only directly call other static methods of their class.
- They can only directly access static variables of their class.
- They cannot refer to this or super in any way.

If you need to do computation in order to initialize your static variables, you can declare a static block that gets executed exactly once, when the class is first loaded. The following example shows a class that has a static method, some static variables, and a static initialization block:

```Java
class UseStatic(){
    static int a,b;
    
    static {
        b = 4;
        a = b * 5;
    }
}
```
As soon as the UseStatic class is loaded, all of the static statements are run.

## Introducing final
A field can be declared as *final*. Doing so prevents its contents from being modified, making it, essentially, a constant. You can do this in one of two ways: First, you can give it a value when it is declared. Second, you can assign it a value within a constructor. The first approach is probably the most common. Declaring a parameter *final* prevents it from being changed within the method. Declaring a local variable *final* prevents it from being assigned a value more than once. The keyword *final* can also be applied to methods, but its meaning is substantially different than when it is applied to variables. This additional usage of *final* is explained in the next chapter, when inheritance is described.

## Introducing Nested and Inner Classes
Thus, if class B is defined within class A, then B does not exist independently of A. A nested class has access to the members, including private members, of the class in which it is nested. However, the enclosing class does not have access to the members of the nested class.

There are two types of nested classes: *static* and *non-static*.

A static nested class is one that has the static modifier applied. Because it is static, it must access the non-static members of its enclosing class through an object. That is, it cannot refer to non-static members of its enclosing class directly. Because of this restriction, static nested classes are seldom used.

The most important type of nested class is the *inner* class. An inner class is a non-static nested class.

## Varargs: Variable-Length Arguments

