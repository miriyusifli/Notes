### Item  1: Consider static factory methods instead of constructors
One advantage of static factory methods is that, unlike constructors, they have names. A second advantage of static factory methods is that, unlike constructors, they are not required to create a new object each time they’re invoked. A third advantage of static factory methods is that, unlike constructors, they can return an object of any subtype of their return type. A fourth advantage of static factories is that the class of the returned object can vary from call to call as a function of the input parameters. A fifth advantage of static factories is that the class of the returned object need not exist when the class containing the method is written.

The main limitation of providing only static factory methods is that classes without public or protected constructors cannot be subclassed. A second shortcoming of static factory methods is that they are hard for programmers to find. 

### Item  2: Consider a builder when faced with many constructor parameters
### Item  3: Enforce the singleton property with a private constructor or an enum type
Making a class a singleton can make it difficult to test its clients because it’s impossible to substitute a mock implementation for a singleton unless it implements an interface that serves as its type.

### Item  4: Enforce noninstantiability with a private constructor
Attempting to enforce noninstantiability by making a class abstract does not work. The class can be subclassed and the subclass instantiated. A class can be made noninstantiable by including a private constructor.

### Item  5: Prefer dependency injection to hardwiring resources
A simple pattern that satisfies this requirement is to pass the resource into the constructor when creating a new instance. This is one form of dependency injection.
### Item  6: Avoid creating unnecessary objects

```Java
// Performance can be greatly improved!
   static boolean isRomanNumeral(String s) {
       return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
}
+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
```

The problem with this implementation is that it relies on the String.matches method. While String.matches is the easiest way to check if a string matches a regular expression, it’s not suitable for repeated use in performance-critical situations. The problem is that it internally creates a Pattern instance for the regular expression and uses it only once, after which it becomes eligible for garbage collection. Creating a Pattern instance is expensive because it requires compiling the regular expression into a finite state machine.
To improve the performance, explicitly compile the regular expression into a Pattern instance (which is immutable) as part of class initialization, cache it, and reuse the same instance for every invocation of the isRomanNumeral method:
```Java
// Reusing expensive object for improved performance
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
        "^(?=.)M*(C[MD]|D?C{0,3})"
        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
    static boolean isRomanNumeral(String s) {
           return ROMAN.matcher(s).matches();
} }

```
Autoboxing blurs but does not erase the distinction between primitive and boxed primitive types. Prefer primitives to boxed primitives, and watch out for unintentional autoboxing.

### Item  7: Eliminate obsolete object references
Generally speaking, whenever a class manages its own memory, the pro- grammer should be alert for memory leaks. Whenever an element is freed, any object references contained in the element should be nulled out.

Another common source of memory leaks is caches. Once you put an object reference into a cache, it’s easy to forget that it’s there and leave it in the cache long after it becomes irrelevant.

A third common source of memory leaks is listeners and other callbacks.
If you implement an API where clients register callbacks but don’t deregister them explicitly, they will accumulate unless you take some action. One way to ensure that callbacks are garbage collected promptly is to store only weak references to them, for instance, by storing them only as keys in a WeakHashMap.

### Item  8: Avoid finalizers and cleaners
### Item  9: Prefer try-with-resources to try-finally
### Item  10: Obey the general contract when overriding equals
The easiest way to avoid problems is not to override the equals method, in which case each instance of the class is equal only to itself. This is the right thing to do if any of the following conditions apply:

- Each instance of the class is inherently unique. This is true for classes such as Thread that represent active entities rather than values. The equals imple- mentation provided by Object has exactly the right behavior for these classes.
- There is no need for the class to provide a “logical equality” test. For example, java.util.regex.Pattern could have overridden equals to check whether two Pattern instances represented exactly the same regular expression, but the designers didn’t think that clients would need or want this functionality. Under these circumstances, the equals implementation inherited from Object is ideal.
- A super class has already overridden equals,and the super class behavior is appropriate for this class. For example, most Set implementations inherit their equals implementation from AbstractSet, List implementations from AbstractList, and Map implementations from AbstractMap.
- The class is private or package-private, and you are certain that its equals method will never be invoked. If you are extremely risk-averse, you can over- ride the equals method to ensure that it isn’t invoked accidentally:
    ```Java
    @Override 
    public boolean equals(Object o) {
        throw new AssertionError(); // Method is never called
    }
    ```

So when is it appropriate to override equals? It is when a class has a notion of logical equality that differs from mere object identity and a superclass has not already overridden equals. 

One kind of value class that does not require the equals method to be overrid- den is a class that uses instance control (Item 1) to ensure that at most one object exists with each value. Enum types (Item 34) fall into this category. For these classes, logical equality is the same as object identity, so Object’s equals method functions as a logical equals method.

The equals method implements an equivalence relation. It has these properties:
- Reflexive:For any non-null reference value x, x.equals(x) must return true.
- Symmetric:For any non-null reference values x and y, x.equals(y)must return true if and only if y.equals(x) returns true.
- Transitive:Foranynon-nullreferencevaluesx,y,z,ifx.equals(y)returns true and y.equals(z) returns true, then x.equals(z) must return true.
- Consistent: For any non-null reference values x and y, multiple invocations of x.equals(y) must consistently return true or consistently return false, provided no information used in equals comparisons is modified.
- Foranynon-nullreferencevaluex,x.equals(null)mustreturnfalse.

Putting it all together, here’s a recipe for a high-quality equals method:
1. Use the == operator to check if the argument is a reference to this object. If so, return true. This is just a performance optimization but one that is worth doing if the comparison is potentially expensive.
2. Use the instanceof operator to check if the argument has the correct type. If not, return false. Typically, the correct type is the class in which the method occurs. Occasionally, it is some interface implemented by this class. Use an interface if the class implements an interface that refines the equals contract to permit comparisons across classes that implement the interface. Collection interfaces such as Set, List, Map, and Map.Entry have this property.
3. Cast the argument to the correct type. Because this cast was preceded by an instanceof test, it is guaranteed to succeed.
4. For each “significant” field in the class, check if that field of the argument matches the corresponding field of this object. If all these tests succeed, re- turn true; otherwise, return false. 

### Item  11: Always override hashCode when you override equals
Here is the contract:
- When the hashCode method is invoked on an object repeatedly during an execution of an application, it must consistently return the same value, provided no information used in equals comparisons is modified. This value need not remain consistent from one execution of an application to another.
- If two objects are equal according to the equals(Object) method,then calling hashCode on the two objects must produce the same integer result.
- If two objects are unequal according to the equals(Object)method,it is not required that calling hashCode on each of the objects must produce distinct results. However, the programmer should be aware that producing distinct results for unequal objects may improve the performance of hash tables.


A good hash function tends to produce unequal hash codes for unequal instances. Luckily it’s not too hard to achieve a fair approximation. Here is a simple recipe:
1. Declare an int variable named result, and initialize it to the hash code c for the first significant field in your object, as computed in step 2.a. (Recall from Item 10 that a significant field is a field that affects equals comparisons.)
2. For every remaining significant field f in your object, do the following: a. Compute an int hash code c for the field:
i. If the field is of a primitive type, compute Type.hashCode(f), where Type is the boxed primitive class corresponding to f’ s type.
ii. If the field is an object reference and this class’s equals method compares the field by recursively invoking equals, recursively invoke hashCode on the field. If a more complex comparison is required, compute a “canonical representation” for this field and invoke hashCode on the canonical representation. If the value of the field is null, use 0 (or some other constant, but 0 is traditional).
iii. If the field is an array, treat it as if each significant element were a separate field. That is, compute a hash code for each significant element by applying these rules recursively, and combine the values per step 2.b. If the array has no significant elements, use a constant, preferably not 0. If all elements are significant, use Arrays.hashCode.
b. Combine the hash code c computed in step 2.a into result as follows: result = 31 * result + c;
3. Return result.


The Objects class has a static method that takes an arbitrary number of objects and returns a hash code for them. This method, named hash, lets you write one-line hashCode methods whose quality is comparable to those written accord- ing to the recipe in this item. Unfortunately, they run more slowly because they entail array creation to pass a variable number of arguments, as well as boxing and unboxing if any of the arguments are of primitive type. This style of hash function is recommended for use only in situations where performance is not critical. Here is a hash function for PhoneNumber written using this technique:
```Java
// One-line hashCode method - mediocre performance
@Override
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```
If a class is immutable and the cost of computing the hash code is significant, you might consider caching the hash code in the object rather than recalculating it each time it is requested. If you believe that most objects of this type will be used as hash keys, then you should calculate the hash code when the instance is created. Otherwise, you might choose to lazily initialize the hash code the first time hashCode is invoked.

In summary, you must override hashCode every time you override equals, or your program will not run correctly. 

### Item 12: Always override toString
### Item 13: Override clone judiciously
So what does Cloneable do, given that it contains no methods? It determines the behavior of Object’s protected clone implementation: if a class implements Cloneable, Object’s clone method returns a field-by-field copy of the object; otherwise it throws CloneNotSupportedException. This is a highly atypical use of interfaces and not one to be emulated. Normally, implementing an interface says something about what a class can do for its clients. In this case, it modifies the behavior of a protected method on a superclass.

But if a final class has a clone method that does not invoke super.clone, there is no reason for the class to implement Cloneable, as it doesn’t rely on the behavior of Object’s clone implementation. 

Immutable classes should never provide a clone method.

Java supports covariant return types. In other words, an overriding method’s return type can be a subclass of the overridden method’s return type.

If an object contains fields that refer to mutable objects, the simple clone implementation shown earlier can be disastrous.

The Cloneable architecture is incompatible with normal use of final fields referring to mutable object.

A better approach to object copying is to provide a copy constructor or copy factory. The copy constructor approach and its static factory variant have many advantages over Cloneable/clone: they don’t rely on a risk-prone extralinguistic object creation mechanism; they don’t demand unenforceable adherence to thinly documented conventions; they don’t conflict with the proper use of final fields; they don’t throw unnecessary checked exceptions; and they don’t require casts. As a rule, copy functionality is best provided by constructors or factories. A nota- ble exception to this rule is arrays, which are best copied with the clone method.

### Item 14: Consider implementing Comparable
The general contract of the compareTo method is similar to that of equals:
- The implementor must ensure that sgn(x.compareTo(y))==-sgn(y. compareTo(x)) for all x and y. (This implies that x.compareTo(y) must throw an exception if and only if y.compareTo(x) throws an exception.)
- The implementor must also ensure that the relation is transitive: (x. compareTo(y) > 0 && y.compareTo(z) > 0) implies x.compareTo(z) > 0.
- Finally, implementor must ensure that x.compareTo(y)==0impliesthat sgn(x.compareTo(z)) == sgn(y.compareTo(z)), for all z.
- It is strongly recommended, but not required, that (x.compareTo(y) == 0) == (x.equals(y)). Generally speaking, any class that implements the Comparable interface and violates this condition should clearly indicate this fact. The recommended language is “Note: This class has a natural ordering that is inconsistent with equals.”


Writing a compareTo method is similar to writing an equals method, but there are a few key differences. Because the Comparable interface is parameter- ized, the compareTo method is statically typed, so you don’t need to type check or cast its argument. If the argument is of the wrong type, the invocation won’t even compile. If the argument is null, the invocation should throw a NullPointer- Exception, and it will, as soon as the method attempts to access its members.

In a compareTo method, fields are compared for order rather than equality. To compare object reference fields, invoke the compareTo method recursively. If a field does not implement Comparable or you need a nonstandard ordering, use a Comparator instead. You can write your own comparator or use an existing one, as in this compareTo method for CaseInsensitiveString in Item 10:
```Java
// Single-field Comparable with object reference field
   public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    
        public int compareTo(CaseInsensitiveString cis) {
           return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
}
       ... // Remainder omitted
   }

```

If a class has multiple significant fields, the order in which you compare them is critical. Start with the most significant field and work your way down.In Java 8, the Comparator interface was outfitted with a set of comparator construction methods, which enable fluent construction of comparators. These comparators can then be used to implement a compareTo method, as required by the Comparable interface. Many programmers prefer the conciseness of this approach, though it does come at a modest performance cost: sorting arrays of PhoneNumber instances is about 10% slower on my machine. When using this approach, consider using Java’s static import facility so you can refer to static comparator construction methods by their simple names for clarity and brevity. Here’s how the compareTo method for PhoneNumber looks using this approach:
```Java
// Comparable with comparator construction methods
   private static final Comparator<PhoneNumber> COMPARATOR =
           comparingInt((PhoneNumber pn) -> pn.areaCode)
             .thenComparingInt(pn -> pn.prefix)
             .thenComparingInt(pn -> pn.lineNum);
   public int compareTo(PhoneNumber pn) {
       return COMPARATOR.compare(this, pn);
}
```
### Item  15: Minimize the accessibility of classes and members
It is wrong for a class to have a public static final array field, or an accessor that returns such a field. If a class has such a field or accessor, clients will be able to modify the con- tents of the array. 

### Item 16: Inpublicclasses,use accessor methods,not public fields
### Item 17: Minimize mutability
To make a class immutable, follow these five rules:
1. Don’t provide methods that modify the object’s state (known as mutators).
2. Ensure that the class can’t be extended. This prevents careless or malicious subclasses from compromising the immutable behavior of the class by behaving as if the object’s state has changed. Preventing subclassing is generally accomplished by making the class final, but there is an alternative that we’ll discuss later.
3. Make all fields final. This clearly expresses your intent in a manner that is en- forced by the system. Also, it is necessary to ensure correct behavior if a refer- ence to a newly created instance is passed from one thread to another without synchronization, as spelled out in the memory model [JLS, 17.5; Goetz06, 16].
4. Make all fields private. This prevents clients from obtaining access to mutable objects referred to by fields and modifying these objects directly. While it is technically permissible for immutable classes to have public final fields containing primitive values or references to immutable objects, it is not recommended because it precludes changing the internal representation in a later release (Items 15 and 16).
5. Ensure exclusive access to any mutable components. If your class has any fields that refer to mutable objects, ensure that clients of the class cannot obtain references to these objects. Never initialize such a field to a client-provided object reference or return the field from an accessor. Make defensive copies (Item 50) in constructors, accessors, and readObject methods (Item 88).

Immutable objects are inherently thread-safe; they require no synchroni- zation. They cannot be corrupted by multiple threads accessing them concur- rently. This is far and away the easiest approach to achieve thread safety. 

The major disadvantage of immutable classes is that they require a separate object for each distinct value.

The performance problem is magnified if you perform a multistep operation that generates a new object at every step, eventually discarding all objects except the final result. There are two approaches to coping with this problem. The first is to guess which multistep operations will be commonly required and to provide them as primitives. If a multistep operation is provided as a primitive, the immutable class does not have to create a separate object at each step. Internally, the immutable class can be arbitrarily clever. For example, BigInteger has a pack- age-private mutable “companion class” that it uses to speed up multistep operations such as modular exponentiation. It is much harder to use the mutable companion class than to use BigInteger, for all of the reasons outlined earlier. Luckily, you don’t have to use it: the implementors of BigInteger did the hard work for you.
The package-private mutable companion class approach works fine if you can accurately predict which complex operations clients will want to perform on your immutable class. If not, then your best bet is to provide a public mutable companion class. The main example of this approach in the Java platform libraries is the String class, whose mutable companion is StringBuilder (and its obsolete predecessor, StringBuffer).
Now that you know how to make an immutable class and you understand the pros and cons of immutability, let’s discuss a few design alternatives. Recall that to guarantee immutability, a class must not permit itself to be subclassed. This can be done by making the class final, but there is another, more flexible alternative. Instead of making an immutable class final, you can make all of its constructors private or package-private and add public static factories in place of the public constructors (Item 1). To make this concrete, here’s how Complex would look if you took this approach:
```Java
// Immutable class with static factories instead of constructors
public class Complex {
    private final double re;
    private final double im;
    private Complex(double re, double im) { this.re = re;
           this.im = im;
       }
    public static Complex valueOf(double re, double im) { return new Complex(re, im);
    }
       ... // Remainder unchanged
}
```

To its clients that reside outside its package, the immutable class is effectively final because it is impossible to extend a class that comes from another package and that lacks a public or protected constructor. Besides allowing the flexibility of multiple implementation classes, this approach makes it possible to tune the performance of the class in subsequent releases by improving the object-caching capabilities of the static factories.

One caveat should be added concerning serializability. If you choose to have your immutable class implement Serializable and it contains one or more fields that refer to mutable objects, you must provide an explicit readObject or readResolve method, or use the ObjectOutputStream.writeUnshared and ObjectInputStream.readUnshared methods, even if the default serialized form is acceptable. Otherwise an attacker could create a mutable instance of your class. This topic is covered in detail in Item 88.

Classes should be immutable unless there’s a very good reason to make them mutable.

### Item 18: Favor composition over inheritance
Unlike method invocation, inheritance violates encapsulation [Snyder86]. In other words, a subclass depends on the implementation details of its superclass for its proper function. The superclass’s implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched. As a consequence, a subclass must evolve in tandem with its superclass, unless the superclass’s authors have designed and documented it specifically for the purpose of being extended.

```Java
 // Broken - Inappropriate use of inheritance!
public class InstrumentedHashSet<E> extends HashSet<E> {
    // The number of attempted element insertions
    private int addCount = 0;
    public InstrumentedHashSet() {}
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
    public int getAddCount() {
        return addCount;
    }
}
```
Internally, HashSet’s addAll method is imple- mented on top of its add method, although HashSet, quite reasonably, does not document this implementation detail. The addAll method in Instrumented- HashSet added three to addCount and then invoked HashSet’s addAll implemen- tation using super.addAll. This in turn invoked the add method, as overridden in InstrumentedHashSet, once for each element. It would be slightly better to override the addAll method to iterate over the specified collection, calling the add method once for each element. This would guarantee the correct result whether or not HashSet’s addAll method were implemented atop its add method because HashSet’s addAll implementation would no longer be invoked

A related cause of fragility in subclasses is that their superclass can acquire new methods in subsequent releases. Suppose a program depends for its security on the fact that all elements inserted into some collection satisfy some predicate. This can be guaranteed by subclassing the collection and overriding each method capable of adding an element to ensure that the predicate is satisfied before adding the element. This works fine until a new method capable of inserting an element is added to the superclass in a subsequent release. Once this happens, it becomes possible to add an “illegal” element merely by invoking the new method, which is not overridden in the subclass. This is not a purely theoretical problem. Several security holes of this nature had to be fixed when Hashtable and Vector were ret- rofitted to participate in the Collections Framework.

Luckily, there is a way to avoid all of the problems described above. Instead of extending an existing class, give your new class a private field that references an instance of the existing class. This design is called composition because the exist- ing class becomes a component of the new one. 
```Java
// Wrapper class - uses composition in place of inheritance
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    public InstrumentedSet(Set<E> s) {
        super(s);
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
    public int getAddCount() {
        return addCount;
    }
}
// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) {
        this.s = s; 
    }
    public void clear() { 
        s.clear();            
    }
    public boolean contains(Object o) { 
        return s.contains(o); 
    }
    public boolean isEmpty()
    public int size()
    public Iterator<E> iterator()
    public boolean add(E e)
    public boolean remove(Object o)
    public boolean containsAll(Collection<?> c) { 
        return s.containsAll(c); 
    }
    public boolean addAll(Collection<? extends E> c) { 
        return s.addAll(c);      
    }
    public boolean removeAll(Collection<?> c) { 
        return s.removeAll(c);   
    }
    public boolean retainAll(Collection<?> c) { 
        return s.retainAll(c);   
    }
    public Object[] toArray() { 
        return s.toArray();  
    }
    public <T> T[] toArray(T[] a) { 
        return s.toArray(a); 
    }
    @Override public boolean equals(Object o) { 
        return s.equals(o);  
    }
    @Override public int hashCode() { 
        return s.hashCode(); 
    }
    @Override public String toString() { 
        return s.toString(); 
    }
}
```
Inheritance is appropriate only in circumstances where the subclass really is a subtype of the superclass. In other words, a class B should extend a class A only if an “is-a” relationship exists between the two classes. If you are tempted to have a class B extend a class A, ask yourself the question: Is every B really an A? If you cannot truthfully answer yes to this question, B should not extend A. If the answer is no, it is often the case that B should contain a private instance of A and expose a different API: A is not an essential part of B, merely a detail of its implementation. If you use inheritance where composition is appropriate, you needlessly expose implementation details. For example, if p refers to a Properties instance, then p.getProperty(key) may yield different results from p.get(key): the for- mer method takes defaults into account, while the latter method, which is inher- ited from Hashtable, does not.

To summarize, inheritance is powerful, but it is problematic because it violates encapsulation. It is appropriate only when a genuine subtype relationship exists between the subclass and the superclass. Even then, inheritance may lead to fragility if the subclass is in a different package from the superclass and the superclass is not designed for inheritance. To avoid this fragility, use composition and forwarding instead of inheritance, especially if an appropriate interface to implement a wrapper class exists. Not only are wrapper classes more robust than subclasses, they are also more powerful.

### Item 19: Design and document for inheritance or else prohibit it
The superclass constructor runs before the subclass constructor, so the overriding method in the subclass will get invoked before the subclass constructor has run.
Note that it is safe to invoke private methods, final methods, and static meth- ods, none of which are overridable, from a constructor. The Cloneable and Serializable interfaces present special difficulties when designing for inheritance. If you do decide to implement either Cloneable or Serializable in a class that is designed for inheritance, you should be aware that because the clone and readObject methods behave a lot like constructors, a similar restriction applies: neither clone nor readObject may invoke an overridable method, directly or indirectly. In the case of readObject, the overriding method will run before the subclass’s state has been deserialized. In the case of clone, the overriding method will run before the subclass’s clone method has a chance to fix the clone’s state. Finally, if you decide to implement Serializable in a class designed for inheritance and the class has a readResolve or writeReplace method, you must make the readResolve or writeReplace method protected rather than private. If these methods are private, they will be silently ignored by subclasses.

### Item 20: Prefer interfaces to abstract classes

Existing classes can easily be retrofitted to implement a new interface. If you want to have two classes extend the same abstract class, you have to place it high up in the type hierarchy where it is an ancestor of both classes. Unfortunately, this can cause great collateral damage to the type hierarchy, forcing all descendants of the new abstract class to subclass it, whether or not it is appropriate. Although many interfaces specify the behavior of Object methods such as equals and hashCode, you are not permitted to provide default methods for them. Also, interfaces are not permitted to contain instance fields or nonpublic static members (with the exception of private static methods). Finally, you can’t add default methods to an interface that you don’t control.

### Item 21: Designinterfacesforposterity
It is not always possible to write a default method that maintains all invariants of every conceivable implementation. In the presence of default methods, existing implementations of an inter- face may compile without error or warning but fail at runtime.

Using default methods to add new methods to existing interfaces should be avoided unless the need is critical, in which case you should think long and hard about whether an existing interface implementation might be broken by your default method implementation. 

### Item 22: Useinterfacesonlytodefinetypes
One kind of interface that fails this test is the so-called constant interface. Such an interface contains no methods; it consists solely of static final fields, each exporting a constant. The constant interface pattern is a poor use of interfaces. That a class uses some constants internally is an implementation detail. Implementing a constant interface causes this implementation detail to leak into the class’s exported API. It is of no consequence to the users of a class that the class implements a constant interface. In fact, it may even confuse them. If you want to export constants, there are several reasonable choices. If the constants are strongly tied to an existing class or interface, you should add them to the class or interface. For example, all of the boxed numerical primitive classes, such as Integer and Double, export MIN_VALUE and MAX_VALUE constants. If the constants are best viewed as members of an enumerated type, you should export them with an enum type (Item 34). Otherwise, you should export the constants with a noninstantiable utility class (Item 4). Here is a utility class version of the PhysicalConstants example shown earlier:
```Java
 // Constant utility class
package com.effectivejava.science;

public class PhysicalConstants {
    private PhysicalConstants() { }  // Prevents instantiation
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23; 
    public static final double BOLTZMANN_CONST =1.380_648_52e-23; 
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

### Item 23: Prefer class hierarchies to tagged classes
Occasionally you may run across a class whose instances come in two or more flavors and contain a tag field indicating the flavor of the instance.  In short, tagged classes are verbose, error-prone, and inefficient. A tagged class is just a pallid imitation of a class hierarchy.

### Item 24: Favor static member classes over nonstatic
There are four kinds of nested classes: static member classes, nonstatic member classes, anonymous classes, and local classes. 
A common use of private static member classes is to represent components of the object represented by their enclosing class. For example, consider a Map instance, which associates keys with values. Many Map implementations have an internal Entry object for each key-value pair in the map. To recap, there are four different kinds of nested classes, and each has its place. If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of a mem- ber class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.

### Item25: Limit source files to a single top-level class

### Item 26: Don’t use raw types
```Java
// Raw collection type - don't do this!
// My stamp collection. Contains only Stamp instances.
private final Collection stamps = ... ;
```

If you use this declaration today and then accidentally put a coin into your stamp collection, the erroneous insertion compiles and runs without error (though the compiler does emit a vague warning). You don’t get an error until you try to retrieve the coin from the stamp collection:

```Java
// Raw iterator type - don't do this!
for (Iterator i = stamps.iterator(); i.hasNext(); )
Stamp stamp = (Stamp) i.next(); // Throws ClassCastException stamp.cancel();
```
If you use raw types, you lose all the safety and expressiveness benefits of generics. While you shouldn’t use raw types such as List, it is fine to use types that are parameterized to allow insertion of arbitrary objects, such as List<Object>. Just what is the difference between the raw type List and the parameterized type List<Object>? Loosely speaking, the former has opted out of the generic type system, while the latter has explicitly told the compiler that it is capable of holding objects of any type. While you can pass a List<String> to a parameter of type List, you can’t pass it to a parameter of type List<Object>. There are subtyping rules for generics, and List<String> is a subtype of the raw type List, but not of the parameterized type List<Object> (Item 28). As a consequence, you lose type safety if you use a raw type such as List, but not if you use a parameterized type such as List<Object>. To make this concrete, consider the following program:
```Java
 // Fails at runtime - unsafeAdd method uses a raw type (List)!
public static void main(String[] args) {
    List<String> strings = new ArrayList<>(); 
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0); // Has compiler-generated cast
}

private static void unsafeAdd(List list, Object o) { 
    list.add(o);
}
```
This program compiles, but because it uses the raw type List, you get a warning. And indeed, if you run the program, you get a ClassCastException when the program tries to cast the result of the invocation strings.get(0), which is an Integer, to a String. This is a compiler-generated cast, so it’s normally guaranteed to succeed, but in this case we ignored a compiler warning and paid the price.
If you replace the raw type List with the parameterized type List<Object> in the unsafeAdd declaration and try to recompile the program, you’ll find that it no longer compiles but emits the error message. 

```
Test.java:5: error: incompatible types: List<String> cannot be
         converted to List<Object>
             unsafeAdd(strings, Integer.valueOf(42));
                 ^
```                 

What is the difference between the unbounded wildcard type Set<?> and the raw type Set? Does the question mark really buy you anything? Not to belabor the point, but the wildcard type is safe and the raw type isn’t. You can put any element into a collection with a raw type, easily corrupting the collection’s type invariant (as demonstrated by the unsafeAdd method on page 119); you can’t put any ele- ment (other than null) into a Collection<?>. Attempting to do so will generate a compile-time error message like this:

```
WildCard.java:13: error: incompatible types: String cannot be
   converted to CAP#1
       c.add("verboten");
             ^
     where CAP#1 is a fresh type-variable:
       CAP#1 extends Object from capture of ?
```
### Item27: Eliminate unchecked warnings
Eliminate every unchecked warning that you can. If you can’t eliminate a warning, but you can prove that the code that provoked the warning is typesafe, then (and only then) suppress the warning with an @SuppressWarnings("unchecked") annotation. Always use the SuppressWarnings annotation on the smallest scope possible. 

### Item 28: Prefer lists to arrays
```Java
// Fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException
```

```Java
// Won't compile!
List<Object> ol = new ArrayList<Long>(); // Incompatible types 
ol.add("I don't fit in");
```
Either way you can’t put a String into a Long container, but with an array you find out that you’ve made a mistake at runtime; with a list, you find out at compile time. Of course, you’d rather find out at compile time. it is illegal to create an array of a generic type, a parameterized type, or a type parameter. Therefore, none of these array creation expressions are legal: new List<E>[], new List<String>[], new E[]. All will result in generic array creation errors at compile time.
```Java
// Why generic array creation is illegal - won't compile! 
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42); // (2)
Object[] objects = stringLists; // (3)
objects[0] = intList; // (4)
String s = stringLists[0].get(0); // (5)
```
Let’s pretend that line 1, which creates a generic array, is legal. Line 2 creates and initializes a List<Integer> containing a single element. Line 3 stores the List<String> array into an Object array variable, which is legal because arrays are covariant. Line 4 stores the List<Integer> into the sole element of the Object array, which succeeds because generics are implemented by erasure: the runtime type of a List<Integer> instance is simply List, and the runtime type of a List<String>[] instance is List[], so this assignment doesn’t generate an ArrayStoreException. Now we’re in trouble. We’ve stored a List<Integer> instance into an array that is declared to hold only List<String> instances. In line 5, we retrieve the sole element from the sole list in this array. The compiler automatically casts the retrieved element to String, but it’s an Integer, so we get a ClassCastException at runtime. In order to prevent this from happening, line 1 (which creates a generic array) must generate a compile-time error.

### Item 29: Favor generic types
### Item 30: Favor generic methods
### Item 31: Use bounded wildcards to increase API flexibility
### Item 32: Combine generics and varargs judiciously
### Item 33: Consider type safe heterogeneous containers
### Item 34: Use enums instead of int constants
There is no easy way to translate int enum constants into printable strings. If you print such a constant or display it from a debugger.  Because clients can neither create instances of an enum type nor extend it, there can be no instances but the declared enum constants. To associate data with enum constants, declare instance fields and write a constructor that takes the data and stores it in the fields. Enums are by their nature immutable, so all fields should be final (Item 17). Fields can be public, but it is better to make them private and provide public accessors. Luckily, there is a better way to associate a different behavior with each enum constant: declare an abstract apply method in the enum type, and override it with a concrete method for each constant in a constant-specific class body. Such meth- ods are known as constant-specific method implementations:
```Java
 // Enum type with constant-specific method implementations
public enum Operation {
    PLUS {public double apply(doublex,doubley){returnx+y;}}, 
    MINUS {public double apply(double x, double y){return x - y;}}, 
    TIMES {public double apply(double x, double y){return x * y;}}, 
    DIVIDE{public double apply(double x, double y){return x / y;}};
     
    public abstract double apply(double x, double y);
}
```
In the unlikely event that you do forget, the compiler will remind you because abstract methods in an enum type must be over- ridden with concrete methods in all of its constants. 

What you really want is to be forced to choose an overtime pay strategy each time you add an enum constant. Luckily, there is a nice way to achieve this. The idea is to move the overtime pay computation into a private nested enum, and to pass an instance of this strategy enum to the constructor for the PayrollDay enum. The PayrollDay enum then delegates the overtime pay calculation to the strategy enum, eliminating the need for a switch statement or constant-specific method implementation in PayrollDay. While this pattern is less concise than the switch statement, it is safer and more flexible:

```Java
// The strategy enum pattern
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
       
    private final PayType payType;
       
    PayrollDay(PayType payType) { 
        this.payType = payType; 
    }
       
    PayrollDay() { 
        this(PayType.WEEKDAY); 
    } 
       
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
       
    // The strategy enum type
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            } 
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
};
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        } 
    }
}
```
Use enums any time you need a set of con- stants whose members are known at compile time. It is not necessary that the set of constants in an enum type stay fixed for all time.

### Item35: Use instance fields instead of ordinals
All enums have an ordinal method, which returns the numerical position of each enum constant in its type. You may be tempted to derive an associated int value from the ordinal. While this enum works, it is a maintenance nightmare. If the constants are reordered, the numberOfMusicians method will break.

```Java
// Abuse of ordinal to derive an associated value - DON'T DO THIS
   public enum Ensemble {
       SOLO,   DUET,   TRIO, QUARTET, QUINTET,
       SEXTET, SEPTET, OCTET, NONET,  DECTET;
    
    public int numberOfMusicians() { return ordinal() + 1; } }
```
Never derive a value associated with an enum from its ordinal; store it in an instance field instead. you are best off avoiding the ordinal method entirely.

### Item 36: Use EnumSet instead of bit fields
### Item 37: Use EnumMap instead of ordinal indexing
### Item 38: Emulate extensible enums with interfaces
That said, there is at least one compelling use case for extensible enumerated types, which is operation codes, also known as opcodes. An opcode is an enumer- ated type whose elements represent operations on some machine, such as the Operation type in Item 34, which represents the functions on a simple calculator. Sometimes it is desirable to let the users of an API provide their own operations, effectively extending the set of operations provided by the API. Luckily, there is a nice way to achieve this effect using enum types. The basic idea is to take advantage of the fact that enum types can implement arbitrary inter- faces by defining an interface for the opcode type and an enum that is the standard implementation of the interface. For example, here is an extensible version of the Operation type from Item 34:
```Java
// Emulated extensible enum using an interface
public interface Operation {
    double apply(double x, double y);
}
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; } 
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; } 
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };
    
    private final String symbol;
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    } 
}

// Emulated extension enum
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        } 
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y; }
        };
       
    private final String symbol;
       
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override 
    public String toString() {
        return symbol;
    } 
}
```
You can now use your new operations anywhere you could use the basic oper- ations, provided that APIs are written to take the interface type (Operation), not the implementation (BasicOperation).

### Item 39: Prefer annotations to naming patterns
### Item 40: Consistently use the Override annotation
Luckily, the compiler can help you find non-overriden methods which are intented to be overriden, but only if you help it by telling it that you intend to override. Use the Override annotation on every method decla- ration that you believe to override a superclass declaration. 

### Item 41: Use marker interfaces to define types
First and foremost, marker interfaces define a type that is implemented by instances of the marked class; marker annotations do not. The existence of a marker interface type allows you to catch errors at compile time that you couldn’t catch until runtime if you used a marker annotation. Java’s serialization facility (Chapter 6) uses the Serializable marker inter- face to indicate that a type is serializable. The ObjectOutputStream.writeObject method, which serializes the object that is passed to it, requires that its argument be serializable. Had the argument of this method been of type Serializable, an attempt to serialize an inappropriate object would have been detected at compile time (by type checking). Another advantage of marker interfaces over marker annotations is that they can be targeted more precisely.

So when should you use a marker annotation and when should you use a marker interface? Clearly you must use an annotation if the marker applies to any program element other than a class or interface, because only classes and inter- faces can be made to implement or extend an interface. If the marker applies only to classes and interfaces, ask yourself the question “Might I want to write one or more methods that accept only objects that have this marking?” If so, you should use a marker interface in preference to an annotation. This will make it possible for you to use the interface as a parameter type for the methods in question, which will result in the benefit of compile-time type checking. If you can convince your- self that you’ll never want to write a method that accepts only objects with the marking, then you’re probably better off using a marker annotation. If, addition- ally, the marking is part of a framework that makes heavy use of annotations, then a marker annotation is the clear choice.


In summary, marker interfaces and marker annotations both have their uses. If you want to define a type that does not have any new methods associated with it, a marker interface is the way to go. If you want to mark program elements other than classes and interfaces or to fit the marker into a framework that already makes heavy use of annotation types, then a marker annotation is the correct choice. If you find yourself writing a marker annotation type whose target is ElementType.TYPE, take the time to figure out whether it really should be an annotation type or whether a marker interface would be more appropriate.

### Item 42: Prefer lambdas to anonymous classes
### Item 43: Prefer method references to lambdas
For example, consider this snippet, which is presumed to occur in a class named GoshThisClassNameIsHumongous: 
```Java
service.execute(GoshThisClassNameIsHumongous::action);
```
The lambda equivalent looks like this:
```Java
service.execute(() -> action());
```
The snippet using the method reference is neither shorter nor clearer than the snippet using the lambda, so prefer the latter. 
In summary, method references often provide a more succinct alternative to lambdas. Where method references are shorter and clearer, use them; where they aren’t, stick with lambdas.

### Item 44: Favor the use of standard functional interfaces
The Operator interfaces represent functions whose result and argument types are the same. The Predicate interface represents a function that takes an argument and returns a boolean. The Function interface represents a function whose argument and return types differ. The Supplier interface represents a function that takes no arguments and returns (or “supplies”) a value. Finally, Consumer represents a function that takes an argument and returns nothing, essentially consuming its argument. The six basic functional interfaces are summarized below:
| Interface         | Function Signature  | Example             |
|-------------------|---------------------|---------------------|
| UnaryOperator<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add     |
| Predicate<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T,R>     | R apply(T t)        | Arrays::asList      |
| Supplier<T>       | T get()             | Instant::now        |
| Consumer<T>       | void accept(T t)    | System.out::println |

### Item 45: Use streams judiciously
A stream pipeline consists of a source stream followed by zero or more intermediate operations and one terminal operation. Each intermediate operation transforms the stream in some way, such as mapping each element to a function of that element or filtering out all elements that do not satisfy some condition. Intermediate operations all transform one stream into another, whose element type may be the same as the input stream or different from it. The terminal operation performs a final computation on the stream resulting from the last intermediate operation, such as storing its elements into a collection, returning a certain element, or printing all of its elements. Stream pipelines are evaluated lazily: evaluation doesn’t start until the terminal operation is invoked.

### Item 46: Prefer side-effect-free functions in streams
Intermediate operation returns stream.
Terminal operaiton returns non-stream such as collection,int and etc.

### Item 47: Prefer Collection to Stream as a return type
If an API returns only a stream and some users want to iterate over the returned sequence with a for-each loop, those users will be justifiably upset. It is especially frustrating because the Stream interface contains the sole abstract method in the Iterable interface, and Stream’s specification for this method is compatible with Iterable’s. The only thing preventing programmers from using a for-each loop to iterate over a stream is Stream’s failure to extend Iterable.

Do not store a large sequence in memory just to return it as a collection. If the sequence you’re returning is large but can be represented concisely, con- sider implementing a special-purpose collection. For example, suppose you want to return the power set of a given set, which consists of all of its subsets. The power set of {a, b, c} is {{}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}}. If a set has n elements, its power set has 2n. Therefore, you shouldn’t even consider storing the power set in a standard collection implementation. It is, however, easy to implement a custom collection for the job with the help of AbstractList. The trick is to use the index of each element in the power set as a bit vector, where the nth bit in the index indicates the presence or absence of the nth element from the source set. In essence, there is a natural mapping between the binary numbers from 0 to 2n − 1 and the power set of an n-element set. Here’s the code:

```java
// Returns the power set of an input set as custom collection
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException("Set too big " + s); return new AbstractList<Set<E>>() {
        @Override 
        public int size() {
            return 1 << src.size(); // 2 to the power srcSize
        }
        @Override
        public boolean contains(Object o) {
            return o instanceof Set && src.containsAll((Set)o);
        }
        
        @Override
        public Set<E> get(int index) {
            Set<E> result = new HashSet<>();
            for (int i = 0; index != 0; i++, index >>= 1)
                if ((index & 1) == 1)
                result.add(src.get(i));
            return result;
        }
    }; 
}
}
```
Note that PowerSet.of throws an exception if the input set has more than 30 elements. This highlights a disadvantage of using Collection as a return type rather than Stream or Iterable: Collection has an int-returning size method, which limits the length of the returned sequence to Integer.MAX_VALUE, or 231 − 1. The Collection specification does allow the size method to return 231 − 1 if the collection is larger, even infinite, but this is not a wholly satisfying solution.


In summary, when writing a method that returns a sequence of elements, remember that some of your users may want to process them as a stream while others may want to iterate over them. Try to accommodate both groups. If it’s fea- sible to return a collection, do so. If you already have the elements in a collection or the number of elements in the sequence is small enough to justify creating a new one, return a standard collection such as ArrayList. Otherwise, consider implementing a custom collection as we did for the power set. If it isn’t feasible to return a collection, return a stream or iterable, whichever seems more natural. If, in a future Java release, the Stream interface declaration is modified to extend Iterable, then you should feel free to return streams because they will allow for both stream processing and iteration.

### Item 48: Use caution when making streams parallel
```Java
// Stream-based program to generate the first 20 Mersenne primes
   public static void main(String[] args) {
       primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
           .filter(mersenne -> mersenne.isProbablePrime(50))
           .limit(20)
           .forEach(System.out::println);
}
   static Stream<BigInteger> primes() {
       return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```
On my machine, this program immediately starts printing primes and takes 12.5 seconds to run to completion. Suppose I naively try to speed it up by adding a call to parallel() to the stream pipeline. What do you think will happen to its performance? Will it get a few percent faster? A few percent slower? Sadly, what happens is that it doesn’t print anything, but CPU usage spikes to 90 percent and stays there indefinitely (a liveness failure). The program might terminate eventu- ally, but I was unwilling to find out; I stopped it forcibly after half an hour.
What’s going on here? Simply put, the streams library has no idea how to par- allelize this pipeline and the heuristics fail. Even under the best of circumstances, parallelizing a pipeline is unlikely to increase its performance if the source is from Stream.iterate, or the intermediate operation limit is used. This pipe- line has to contend with both of these issues. Worse, the default parallelization strategy deals with the unpredictability of limit by assuming there’s no harm in processing a few extra elements and discarding any unneeded results. In this case, it takes roughly twice as long to find each Mersenne prime as it did to find the pre- vious one. Thus, the cost of computing a single extra element is roughly equal to the cost of computing all previous elements combined, and this innocuous-looking pipeline brings the automatic parallelization algorithm to its knees. The moral of this story is simple: Do not parallelize stream pipelines indiscriminately. The performance consequences may be disastrous. As a rule, performance gains from parallelism are best on streams over ArrayList, HashMap, HashSet, and ConcurrentHashMap instances; arrays; int ranges; and long ranges. What these data structures have in common is that they can all be accurately and cheaply split into subranges of any desired sizes, which makes it easy to divide work among parallel threads. The nature of a stream pipeline’s terminal operation also affects the effective- ness of parallel execution. If a significant amount of work is done in the terminal operation compared to the overall work of the pipeline and that operation is inher- ently sequential, then parallelizing the pipeline will have limited effectiveness. The best terminal operations for parallelism are reductions, where all of the elements emerging from the pipeline are combined using one of Stream’s reduce methods, or prepackaged reductions such as min, max, count, and sum. The short- circuiting operations anyMatch, allMatch, and noneMatch are also amenable to parallelism. The operations performed by Stream’s collect method, which are known as mutable reductions, are not good candidates for parallelism because the overhead of combining collections is costly.

In summary, do not even attempt to parallelize a stream pipeline unless you have good reason to believe that it will preserve the correctness of the computation and increase its speed. The cost of inappropriately parallelizing a stream can be a program failure or performance disaster. If you believe that parallelism may be justified, ensure that your code remains correct when run in parallel, and do careful performance measurements under realistic conditions. If your code remains correct and these experiments bear out your suspicion of increased performance, then and only then parallelize the stream in production code.

### Item 49: Check parameters for validity
The Objects.requireNonNull method, added in Java 7, is flexible and convenient, so there’s no reason to perform null checks manually anymore.
In Java 9, a range-checking facility was added to java.util.Objects. This facility consists of three methods: checkFromIndexSize, checkFromToIndex, and checkIndex.

### Item 50: Make defensive copies when needed
You must program defensively, with the assumption that clients of your class will do their best to destroy its invariants.

```Java
 // Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;
/**
* @param start the beginning of the period
* @param end the end of the period; must not precede start * @throws IllegalArgumentException if start is after end
* @throws NullPointerException if start or end is null
*/
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                   start + " after " + end);
        this.start = start;
        this.end   = end;
    }
    
    public Date start() {
           return start;
    }
```
At first glance, this class may appear to be immutable and to enforce the invariant that the start of a period does not follow its end. It is, however, easy to violate this invariant by exploiting the fact that Date is mutable:
```Java
// Attack the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); end.setYear(78); // Modifies internals of p!
```

As of Java 8, the obvious way to fix this problem is to use Instant (or Local- DateTime or ZonedDateTime) in place of a Date because Instant (and the other java.time classes) are immutable (Item 17). Date is obsolete and should no lon- ger be used in new code. 

To protect the internals of a Period instance from this sort of attack, it is essential to make a defensive copy of each mutable parameter to the construc- tor and to use the copies as components of the Period instance in place of the originals:

```Java
// Repaired constructor - makes defensive copies of parameters
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end   = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException(this.start + " after " + this.end);
}
```
With the new constructor in place, the previous attack will have no effect on the Period instance. Note that defensive copies are made before checking the validity of the parameters (Item 49), and the validity check is performed on the copies rather than on the originals. While this may seem unnatural, it is necessary. It protects the class against changes to the parameters from another hread during the window of vulnerability between the time the parameters are checked and the time they are copied. In the computer security community, this is known as a time-of-check/time-of-use or TOCTOU attack.

### Item 51: Design method signatures carefully
1. Choose method names carefully.
2. Don’t go overboard in providing convenience methods.
3. Avoid long parameter lists.
4. For parameter types, favor interfaces over classes
5. Prefer two-element enum types to boolean parameters

### Item 52: Use over loading judiciously
```Java
 // Broken! - What does this program print?
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }
    public static String classify(List<?> lst) {
        return "List";
    }
    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }
    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
    };
    for (Collection<?> c : collections)
        System.out.println(classify(c));
    }
} 
```
You might expect this program to print Set, followed by List and Unknown Collection, but it doesn’t. It prints Unknown Collection three times. Why does this happen? Because the classify method is overloaded, and the choice of which overloading to invoke is made at compile time. For all three iterations of the loop, the compile-time type of the parameter is the same: Collection<?>. The runtime type is different in each iteration, but this does not affect the choice of overloading. Because the compile-time type of the parameter is Collection<?>, the only applicable overloading is the third one, classify(Collection<?>), and this overloading is invoked in each iteration of the loop.The correct version of an overridden method is chosen at runtime, based on the runtime type of the object on which the method is invoked. Compare this to overloading, where the runtime type of an object has no effect on which overloading is executed; the selection is made at compile time, based entirely on the compile-time types of the parameters.

There may be times when you feel the need to violate the guidelines in this item, especially when evolving existing classes. For example, consider String, which has had a contentEquals(StringBuffer) method since Java 4. In Java 5, CharSequence was added to provide a common interface for StringBuffer, StringBuilder, String, CharBuffer, and other similar types. At the same time that CharSequence was added, String was outfitted with an overloading of the contentEquals method that takes a CharSequence.There may be times when you feel the need to violate the guidelines in this item, especially when evolving existing classes. For example, consider String, which has had a contentEquals(StringBuffer) method since Java 4. In Java 5, CharSequence was added to provide a common interface for StringBuffer, StringBuilder, String, CharBuffer, and other similar types. At the same time that CharSequence was added, String was outfitted with an overloading of the contentEquals method that takes a CharSequence. While the resulting overloading clearly violates the guidelines in this item, it causes no harm because both overloaded methods do exactly the same thing when they are invoked on the same object reference. The programmer may not know which overloading will be invoked, but it is of no consequence so long as they behave identically. The standard way to ensure this behavior is to have the more specific overloading forward to the more general:
```Java
// Ensuring that 2 methods have identical behavior by forwarding
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```

### Item 53: Use varargs judiciously
### Item 54: Return empty collections o rarrays, not nulls
### Item 55: Return optionals judiciously
Never return a null value from an Optional-returning method. ontainer types, including collections, maps, streams, arrays, and optionals should not be wrapped in optionals. You should declare a method to return Optional<T> if it might not be able to return a result and clients will have to perform special processing if no result is returned.

Returning an optional that contains a boxed primitive type is prohibitively expensive compared to returning a primitive type because the optional has two levels of boxing instead of zero. Therefore, the library designers saw fit to provide analogues of Optional<T> for the primitive types int, long, and double. These optional types are OptionalInt, OptionalLong, and OptionalDouble. They contain most, but not all, of the methods on Optional<T>. Therefore, you should never return an optional of a boxed primitive type, with the possible exception of the “minor primitive types,” Boolean, Byte, Character, Short, and Float.

It is almost never appropriate to use an optional as a key, value, or element in a collection or array.

### Item 56: Write doc comments for all exposed API elements
To document your API properly, you must precede every exported class, interface, constructor, method, and field declaration with a doc comment. To document your API properly, you must precede every exported class, interface, constructor, method, and field declaration with a doc comment.

### Item 57: Minimize the scope of local variables
The most powerful technique for minimizing the scope of a local variable is to declare it where it is first used. 

### Item 58: Prefer for-each loops to traditional for loops
As of Java 7, you should no longer use Random. For most uses, the random number generator of choice is now ThreadLocalRandom. It produces higher quality random numbers, and it’s very fast. On my machine, it is 3.6 times faster than Random. For fork join pools and parallel streams, use SplittableRandom.

### Item 60: Avoid float and double if exact answers are required
Use BigDecimal, int, or long for monetary calculations. There are, however, two disadvantages to using BigDecimal: it’s a lot less convenient than using a primitive arithmetic type, and it’s a lot slower. The latter disadvantage is irrelevant if you’re solving a single short problem, but the former may annoy you. An alternative to using BigDecimal is to use int or long, depending on the amounts involved, and to keep track of the decimal point yourself. In this example, the obvious approach is to do all computation in cents instead of dollars. Here’s a straightforward transformation that takes this approach:

```Java
public static void main(String[] args) {
    int itemsBought = 0;
    int funds = 100;
    for (int price = 10; funds >= price; price += 10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + " items bought.");
    System.out.println("Cash left over: " + funds + " cents");
   }
```

### Item 61: Prefer primitive types to boxed primitives
There are three major differences between primitives and boxed primitives. First, primitives have only their values, whereas boxed primitives have identities distinct from their values. In other words, two boxed primitive instances can have the same value and different identities. Second, primitive types have only fully functional values, whereas each boxed primitive type has one nonfunctional value, which is null, in addition to all the functional values of the corresponding primi- tive type. Last, primitives are more time- and space-efficient than boxed primitives. All three of these differences can get you into real trouble if you aren’t careful. Applying the == operator to boxed primitives is almost always wrong.

### Item 62: Avoid strings where other types are more appropriate
Strings are poor substitutes for other value types. To summarize, avoid the natural tendency to represent objects as strings when better data types exist or can be written. Used inappropriately, strings are more cumbersome, less flexible, slower, and more error-prone than other types. Types for which strings are commonly misused include primitive types, enums, and aggregate types.

### Item 63: Beware the performance of string concatenation
Using the string concatenation operator repeatedly to concatenate n strings requires time qua- dratic in n.

### Item 64: Refer to objects by their interfaces
If appropriate interface types exist, then parameters, return values, variables, and fields should all be declared using interface types. If you get into the habit of using interfaces as types, your program will be much more flexible. It is entirely appropriate to refer to an object by a class rather than an interface if no appropriate interface exists. If there is no appropriate interface, just use the least specific class in the class hierarchy that provides the required functionality.

### Item 65: Prefer interfaces to reflection
### Item 66: Use native methods judiciously
### Item 67: Optimize judiciously
More computing sins are committed in the name of efficiency (without neces- sarily achieving it) than for any other single reason—including blind stupidity. (—William A. Wulf )


Strive to write good programs rather than fast ones. Strive to avoid design decisions that limit performance. Consider the performance consequences of your API design decisions. Measure perfor- mance before and after each attempted optimization. 

### Item 68: Ad here to generally accepted naming conventions
### Item 69: Use exceptions only for exceptional conditions
Exceptions are, as their name implies, to be used only for exceptional conditions; they should never be used for ordinary control flow. A well-designed API must not force its clients to use exceptions for ordinary control flow. 

### Item 70: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
use checked exceptions for conditions from which the caller can reasonably be expected to recover. Use runtime exceptions to indicate programming errors. All of the unchecked throwables you implement should subclass RuntimeException.

### Item 71: Avoid unnecessary use of checked exceptions
In summary, when used sparingly, checked exceptions can increase the reliability of programs; when overused, they make APIs painful to use. If callers won’t be able to recover from failures, throw unchecked exceptions. If recovery may be possible and you want to force callers to handle exceptional conditions, first consider returning an optional. Only if this would provide insufficient information in the case of failure should you throw a checked exception.

### Item 72: Favor the use of standard exceptions
### Item73: Throw exceptions appropriate to the abstraction
It is disconcerting when a method throws an exception that has no apparent connection to the task that it performs. This often happens when a method propagates an exception thrown by a lower-level abstraction. Not only is it disconcerting, but it pollutes the API of the higher layer with implementation details. If the implementation of the higher layer changes in a later release, the exceptions it throws will change too, potentially breaking existing client programs. To avoid this problem, higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstraction. This idiom is known as exception translation:
```Java
// Exception Translation
try {
    ... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException e) {
    throw new HigherLevelException(...);
}
```
### Item 74: Document all exceptions thrown by each method
Always declare checked exceptions individually, and document precisely the conditions under which each one is thrown using the Javadoc @throws tag. Use the Javadoc @throws tag to document each exception that a method can throw, but do not use the throws keyword on unchecked exceptions. If an exception is thrown by many methods in a class for the same reason, you can document the exception in the class’s documentation comment rather than documenting it individually for each method.

### Item 75: Include failure-capture information in detail messages
To capture a failure, the detail message of an exception should contain the values of all parameters and fields that contributed to the exception. Do not include passwords, encryption keys, and the like in detail messages.

### Item 76: Strive for failure atomicity
After an object throws an exception, it is generally desirable that the object still be in a well-defined, usable state, even if the failure occurred in the midst of performing an operation. This is especially true for checked exceptions, from which the caller is expected to recover. Generally speaking, a failed method invocation should leave the object in the state that it was in prior to the invocation. A method with this property is said to be failure-atomic.
There are several ways to achieve this effect. The simplest is to design immutable objects (Item 17). If an object is immutable, failure atomicity is free. If an operation fails, it may prevent a new object from getting created, but it will never leave an existing object in an inconsistent state, because the state of each object is consistent when it is created and can’t be modified thereafter.
For methods that operate on mutable objects, the most common way to achieve failure atomicity is to check parameters for validity before performing the operation (Item 49). This causes most exceptions to get thrown before object modification commences. For example, consider the Stack.pop method in Item 7:
```Java
 public Object pop() {
       if (size == 0)
           throw new EmptyStackException();
       Object result = elements[--size];
       elements[size] = null; // Eliminate obsolete reference
       return result;
}
```

If the initial size check were eliminated, the method would still throw an exception when it attempted to pop an element from an empty stack. It would, however, leave the size field in an inconsistent (negative) state, causing any future method invocations on the object to fail. Additionally, the ArrayIndexOutOf- BoundsException thrown by the pop method would be inappropriate to the abstraction (Item 73).

### Item 77: Don’t ignore exceptions
### Item 78: Synchronize access to shared mutable data
### Item 79: Avoid excessive synchronization
### Item 80: Prefer executors,tasks,and streams to threads
### Item 81: Prefer concurrency utilities to wait and notify
### Item 82: Document thread safety
### Item 83: Use lazy initialization judiciously
### Item 84: Don’t depend on the thread scheduler
### Item 85: Prefer alternatives to Java serialization
A fundamental problem with serialization is that its attack surface is too big to protect, and constantly growing: Object graphs are deserialized by invoking the readObject method on an ObjectInputStream. This method is essentially a magic constructor that can be made to instantiate objects of almost any type on the class path, so long as the type implements the Serializable interface. In the process of deserializing a byte stream, this method can execute code from any of these types, so the code for all of these types is part of the attack surface. If you can’t avoid serialization and you aren’t absolutely certain of the safety of the data you’re deserializing, use the object deserialization filtering added in Java 9 and backported to earlier releases (java.io.ObjectInputFilter). This facility lets you specify a filter that is applied to data streams before they’re deserialized. It operates at the class granularity, letting you accept or reject certain classes. 

The best way to avoid serialization exploits is never to deserialize anything. There is no reason to use Java serialization in any new system you write. The leading cross-platform structured data representations are JSON. If you can’t avoid serialization and you aren’t absolutely certain of the safety of the data you’re deserializing, use the object deserialization filtering added in Java 9 and backported to earlier releases (java.io.ObjectInputFilter). This facility lets you specify a filter that is applied to data streams before they’re deserialized. It operates at the class granularity, letting you accept or reject certain classes. Prefer whitelisting to blacklisting.
### Item 86: Implement Serializable with great caution
A major cost of implementing Serializable is that it decreases the flexibility to change a class’s implementation once it has been released. When a class implements Serializable, its byte-stream encoding (or serialized form) becomes part of its exported API. Once you distribute a class widely, you are generally required to support the serialized form forever, just as you are required to support all other parts of the exported API. A second cost of implementing Serializable is that it increases the likeli- hood of bugs and security holes (Item 85). 
Classes designed for inheritance (Item19) should rarely implement Serializable, and interfaces should rarely extend it. Violating this rule places a substantial burden on anyone who extends the class or implements the interface.
### Item 87: Consider using a custom serialized form
### Item 88: Write readObject methods defensively
### Item 89: For instance control, prefer enum types to readResolve
### Item90: Consider serialization proxies instead of serialized instances
In summary, consider the serialization proxy pattern whenever you find yourself having to write a readObject or writeObject method on a class that is not extendable by its clients. This pattern is perhaps the easiest way to robustly serialize objects with nontrivial invariants.







