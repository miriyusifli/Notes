# BeforeAll in Junit5
By default, both JUnit 4 and 5 create a new instance of the test class before running each test method. This provides a clean separation of state between tests.

There are times when we need an object to exist across multiple tests. Let's imagine we would like to read a large file to use as test data. Since it might be time-consuming to repeat that before every test, we might prefer to read it once and keep it for the whole test fixture.

JUnit 4 addresses this with its `@BeforeClass` annotation:

```Java
private static String largeContent;

@BeforeClass
public static void setUpFixture() {
    // read the file and store in 'largeContent'
}
```

We should note that we have to make the variables and the methods annotated with JUnit 4's `@BeforeClass` static.

JUnit 5 provides a different approach. It provides the @BeforeAll annotation which is used on a static function, to work with static members of the class. However, `@BeforeAll` can also be used with an instance function and instance members if the test instance lifecycle is changed to per-class. The `@TestInstance` annotation lets us configure the lifecycle of JUnit 5 tests.

`@TestInstance` has two modes. One is `LifeCycle.PER_METHOD` (the default). The other is `Lifecycle.PER_CLASS`. The latter enables us to ask JUnit to create only one instance of the test class and reuse it between tests.

Let's annotate our test class with the `@TestInstance` annotation and use the `Lifecycle.PER_CLASS` mode:

```Java
@TestInstance(Lifecycle.PER_CLASS)
class TweetSerializerUnitTest {

    private String largeContent;

    @BeforeAll
    void setUpFixture() {
        // read the file
    }

}
```

As we can see, none of the variables or functions are static. We are allowed to use an instance method for `@BeforeAll` when we use the PER_CLASS lifecycle.

**How this is learned:** While writing a unit test, an injected field was needed to be used in a non-static method which had `@BeforeAll`. By default, a method which has `@BeforeAll` annotation, should be static. But, I couldn't make it static, because a non-static field cannot be used in a static method. On the other hand, injected fields cannot be static. To solve this issue, I had to make the method that has `@BeforeAll` non-static. To do that, I used `@TestInstance` with the `Lifecycle.PER_CLASS` option. So, I was able to make the method non-static and use the injected field inside it. 


# Useful Annotations
- @Rule
- @Container - testContainers in Junit
- @ConditionalOnExpression and @ConditionalOnProperty



# What is the difference between @MockBean vs @Mock
@MockBean is a Spring annotation that is used to create a mock of a bean that is already defined in the application context. It allows you to replace a real bean with a mock object in a Spring test. The mock object can be used to stub out the behavior of the real bean and test the code that depends on it.

@Mock is a JUnit annotation that is used to create a mock object for a class or interface. It is used in conjunction with a mocking library like Mockito to create a mock object that can be used in a test. Unlike @MockBean, @Mock does not affect the application context or the beans that are defined in it. It is used to create mock objects for classes that are not managed by Spring.

In summary, @MockBean is a Spring annotation that is used to replace a real bean with a mock object in a Spring test, while @Mock is a JUnit annotation that is used to create a mock object for a class or interface, and it is not related to the Spring's application context.

# References
## Strong reference
The strong reference is the most common type of Reference that we use in our day to day programming:
```Java
Integer prime = 1;
```
The variable prime has a strong reference to an Integer object with value 1. Any object which has a strong reference pointing to it is not eligible for GC.

## Soft Reference
Simply put, an object that has a SoftReference pointing to it won't be garbage collected until the JVM absolutely needs memory.

Let's see how we can create a SoftReference in Java:

```Java
Integer prime = 1;  
SoftReference<Integer> soft = new SoftReference<Integer>(prime); 
prime = null;
```
The prime object has a strong reference pointing to it.

Next, we are wrapping prime strong reference into a soft reference. After making that strong reference null, a prime object is eligible for GC but will be collected only when JVM absolutely needs memory.

## Weak reference
The objects that are referenced only by weak references are garbage collected eagerly; the GC won't wait until it needs memory in that case.

We can create a WeakReference in Java in the following way:
```Java
Integer prime = 1;  
WeakReference<Integer> soft = new WeakReference<Integer>(prime); 
prime = null;
```
When we made a prime reference null, the prime object will be garbage collected in the next GC cycle, as there is no other strong reference pointing to it.

References of a WeakReference type are used as keys in WeakHashMap.