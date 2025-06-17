# Introduction
The goal of good software design?

The measure of design quality is simply the measure of the effort required to meet the needs of the customer. If that effort is low, and stays low throughout the lifetime of the system, the design is good. If that effort grows with each new release, the design is bad. It’s as simple as that.

The only way to reverse the decline in productivity and the increase in cost because of bad architecture is to get the developers to stop thinking like the overconfident Hare and start taking responsibility for the mess that they’ve made. The developers may think that the answer is to start over from scratch and redesign the whole system—but that’s just the Hare talking again. The same overconfidence that led to the mess is now telling them that they can build it better if only they can start the race over. The reality is less rosy:

**_Their overconfidence will drive the redesign into the same mess as the original project._**

# Paradigm Overview
## Structured Programming
Structured programming imposes discipline on direct transfer of control.

It is a programming paradigm that enforces a disciplined approach to control flow, avoiding the chaos of unstructured code, such as excessive use of goto statements. It relies on three core constructs to structure a program’s logic:

**Sequence:** Statements are executed in a defined order, one after another. For example, in a function, each line of code runs sequentially unless altered by other control structures.

**Selection:** Conditional statements, like `if-then-else` or `switch`, allow the program to choose between different paths based on a condition. This enables decision-making without jumping arbitrarily to different parts of the code.

**Iteration:** Loops, such as `for`, `while`, or `do-while`, allow repetitive execution of a block of code until a condition is met, providing a controlled way to handle repetition.


## Object-Oriented Programming
Object-oriented programming imposes discipline on indirect transfer of control.

Historically, in software systems, the flow of control (the order in which functions are called at runtime) dictated the source code dependencies (which modules depend on others at compile time). If module A calls a function in module B, A depends on B in the source code.

This tight coupling creates rigid systems where high-level modules (business logic) are tied to low-level details (e.g., specific implementations). Polymorphism, enabled by object-oriented programming (OOP), allows the flow of control and source code dependencies to be separated. With polymorphism, a high-level module can call a function through an interface (or abstraction), without depending on the specific implementation of that function. As an example, you can rearrange the source code dependencies of your system so that the database and the user interface (UI) depend on the business rules, rather than the other way around.

![](/Books/Programming/Clean_Architecture/img/figure-5.3.png)

## Functional Programming**
Functional programming imposes discipline upon assignment.

It is a programming with pure functions, which are functions that: 
- Always produce the same output for the same input (referential transparency).
- Have no side effects (e.g., they don’t modify external state, like global variables or I/O).


Here is several benefits of adopting Functional Programming principles, even in a language like C:

- Predictability: Pure functions make the core logic (entities and use cases) predictable and easier to reason about, reducing bugs.
- Testability: Pure functions in the use case layer can be tested without mocking external systems, aligning with Martin’s emphasis on testability.
- Concurrency: Immutability eliminates many concurrency issues (e.g., race conditions due to shared mutable state), which Martin notes as a growing concern in modern systems.
- Modularity: By isolating side effects, FP supports Clean Architecture’s goal of creating modular systems where business logic is independent of frameworks and external systems.


# Design principles
## SRP: The Single Principle
```
A module should be responsible to one, and only one, actor.
```
Let's explain the SRP by examining the symptoms of violating it.

### Symptom 1: Accidental Duplication
This occurs when code responsible to different actors is placed in close proximity. A common example given is an Employee class in a payroll application with methods like calculatePay(), reportHours(), and save(). These three methods are responsible to different actors: calculatePay() serves the accounting department (reporting to the CFO), reportHours() serves human resources (reporting to the COO), and save() serves database administrators (reporting to the CTO). By putting the source code for these methods in a single Employee class, the developers couple these actors together. This coupling means that actions by one team (e.g., the CFO's) can affect something depended upon by another team (e.g., the COO's).

### Symptom 2: Merges
Source files containing methods responsible to different actors are prone to frequent merge conflicts. For instance, if the CTO's DBAs decide to make a schema change impacting the save() method, and the COO's HR clerks decide they need a format change for the hours report impacting the reportHours() method, both teams might modify the same Employee source file. When the changes from both teams are merged, conflicts arise. The way to avoid this issue is to separate code that supports different actors

### Soltion 1: Separating Data from Functions

One method is to create a simple data structure, like an EmployeeData class, that holds the data but has no methods. Different classes (e.g., one for pay calculation, one for hours reporting, one for saving) can then share access to this data structure. Each of these new classes contains only the source code necessary for its specific function and is not aware of the other classes. This approach prevents accidental duplication.The sources clarify that each of these functional classes is likely to contain many private methods related to its specific responsibility; the class acts as a scope that hides these private members.

### Solution 2: Facade Pattern
Another solution involves keeping the most important method in the original class and using it as a Facade to delegate to other functions or classes responsible for the lesser concerns

![](/Books/Programming/Clean_Architecture/img/figure-7.4.png)

The EmployeeFacade contains very little code. It is responsible for instantiating and delegating to the classes with the functions.

Some developers prefer to keep the most important business rules closer to the data. This can be done by keeping the most important method in the original Employee class and then using that class as a Facade for the lesser functions.

![](/Books/Programming/Clean_Architecture/img/figure-7.5.png)

## OCP: The Open-Closed Principle
A software artifact should be open for extension but closed for modification.

![](/Books/Programming/Clean_Architecture/img/figure-8.3.png)

The Interactor is in the position that best conforms to the OCP. Changes to the Database, or the Controller, or the Presenters, or the Views, will have no impact on the Interactor.

Why should the Interactor hold such a privileged position? Because it contains the business rules. The Interactor contains the highest-level policies of the application. All the other components are dealing with peripheral concerns. The Interactor deals with the central concern.

Notice how this creates a hierarchy of protection based on the notion of “level.” Interactors are the highest-level concept, so they are the most protected. Views are among the lowest-level concepts, so they are the least protected. Presenters are higher level than Views, but lower level than
the Controller or the Interactor.

------
## LSP: Liskov Substiooub Principle
Child class can be more, but cannot be less.

## ISP: The Interface Segregation Principle 
It states that no client should be forced to depend on methods it does not use.

On the architectural level, Consider, for example, an architect working on a system, S. He wants to include a certain framework, F, into the system. Now suppose that the authors of F have bound it to a particular database, D. So S depends on F. which depends on D.


![](/Books/Programming/Clean_Architecture/img/figure-10.3.png)

## DIP: The Dependency Inversion Principle
The DIP states that source code dependencies must point only toward abstractions, not to concretions

Every change to an abstract interface corresponds to a change to its concrete implementations. Conversely, changes to concrete implementations do not always, or even usually, require changes to the interfaces that they implement. Therefore interfaces are less volatile than implementations. 

Indeed, good software designers and architects work hard to reduce the volatility of interfaces. They try to find ways to add functionality to implementations without making changes to the interfaces.

This implication boils down to a set of very specific
coding practices:
- **Don’t refer to volatile concrete classes:** It means that your code should not directly name or import concrete classes that are likely to change often. Instead, it should interact with them through stable abstract interfaces or abstract classes
- **Don’t derive from volatile concrete classes:** Deriving directly from a concrete class tightly couples your class to the implementation details and dependencies of the parent concrete class. This makes your class vulnerable to changes in the volatile parent, which can be difficult to manage
- **Don’t override concrete functions:** Overriding a concrete function in a derived class means your class is still fundamentally linked to the concrete base class containing that function. the recommended approach is to make the function abstract in the base class or an interface and then provide multiple concrete implementations of that abstract function in different derived classes
- **Never mention the name of anything concrete and volatile:** 
The underlying concern is that mentioning the name of a concrete, volatile entity in your source code creates a direct dependency on that entity. This dependency can cause your code to be affected (e.g., needing recompilation) when the volatile entity changes, even if your code doesn't directly use the specific part that changed. Avoiding such direct references to volatile concretions helps create a system where high-level policies (which should be stable) are independent of low-level, volatile details.

If you apply DIP principle properly, the source code dependencies are inverted against the flow of control—which is why we refer to this principle as
Dependency Inversion.

# Component Coupling
![](/Books/Programming/Clean_Architecture/img/figure-14.2.png)

Consider what happens when we want to test the Entities component. To our chagrin, we find that we must build and integrate with Authorizer and Interactors. This level of coupling between components is troubling, if not intolerable.

It is always possible to break a cycle of components and reinstate the dependency graph as a DAG. There are two primary mechanisms for
doing so:

- Apply the Dependency Inversion Principle (DIP). In the case in Figure 14.3, we could create an interface that has the methods that User needs. We could then put that interface into Entities and inherit it into Authorizer. This inverts the dependency between Entities and Authorizer, thereby breaking the cycle.
![](/Books/Programming/Clean_Architecture/img/figure-14.3.png)

- Create a new component that both Entities and Authorizer depend on. Move the class(es) that they both depend on into that new component.
![](/Books/Programming/Clean_Architecture/img/figure-14.4.png)

## Stability
How can we measure the stability of a component? One way is to count the number of dependencies that enter and leave that component. These counts will allow us to calculate the positional stability of the component.
- **Fan-in**: Incoming dependencies. This metric identifies the number of classes outside this component that depend on classes within the component.
- **Fan-out**: Outgoing depenencies. This metric identifies the number of classes inside this component that depend on classes outside the component.
- **Instability:** I = Fan-out / (Fan-in + Fan-out). This metric has the range [0, 1]. I = 0 indicates a maximally stable component. I = 1 indicates a maximally unstable component. The Fan-in and Fan-out metrics1 are calculated by counting the number of classes outside the component in question that have dependencies with the classes inside the component in question.

The Stable Dependencies Principle (SDP) says that the I metric of a component should be larger than the I metrics of the components that it depends on. That is, I metrics should decrease in the direction of dependency.

## Not All Components Should Be Stable
If all the components in a system were maximally stable, the system would be unchangeable. This is not a desirable situation. Indeed, we want to design our component structure so that some components are unstable and some are stable. The changeable components are on top and depend on the stable component at the bottom. 

![](/Books/Programming/Clean_Architecture/img/figure-14.9.png)

Flexible is a component that we have designed to be easy to change. We want Flexible to be unstable. However, some developer, working in the component named Stable, has hung a dependency on Flexible. This violates the SDP because the I metric for Stable is much smaller than the I metric for Flexible. As a result, Flexible will no longer be easy to change. A change to Flexible will force us to deal with Stable and all its dependents.

We can fix this by employing the DIP. We create an interface class called US and put it in a component named UServer. We make sure that this interface declares all the methods that U needs to use. We then make C implement this interface as shown in Figure 14.11. This breaks the dependency of Stable on Flexible, and forces both components to depend on UServer. UServer is very stable (I = 0), and Flexible retains its necessary instability (I = 1). All the dependencies now flow in the direction of decreasing I.

![](/Books/Programming/Clean_Architecture/img/figure-14.11.png)

# Architecture
The architecture of a software system is the shape given to that system by those who build it. The form of that shape is in the division of that system into components, the arrangement of those components, and the ways in which those components communicate with each other. The purpose of that shape is to facilitate the development, deployment, operation, and maintenance of the software system contained within it.

```
The strategy behind that facilitation is to leave as many options open as possible, for as long as possible.
```

Good architects carefully separate details from policy, and then decouple the policy from the details so thoroughly that the policy has no knowledge of the details and does not depend on the details in any way. Good architects design the policy so that decisions about the details can be delayed and deferred for as long as possible.

For example:
- It is not necessary to choose a database system in the early days of development, because the high-level policy should not care which kind of database will be used. Indeed, if the architect is careful, the high-level policy will not care if the database is relational, distributed, hierarchical, or just plain flat files.
- It is not necessary to choose a web server early in development, because the high-level policy should not know that it is being delivered over the web. If the high-level policy is unaware of HTML, AJAX, JSP, JSF, or any of the rest of the alphabet soup of web development, then you don’t need to decide which web system to use until much later in the project. Indeed, you don’t even have to decide if the system will be delivered over the web.
- It is not necessary to adopt REST early in development, because the high-level policy should be agnostic about the interface to the outside world. Nor is it necessary to adopt a micro-services framework, or a SOA framework. Again, the high-level policy should not care about these things.
- It is not necessary to adopt a dependency injection framework early in development, because the high-level policy should not care how dependencies are resolved.

# Independence
As we previously stated, a good architecture must support:
- The use cases and operation of the system.
- The maintenance of the system.
- The development of the system.
- The deployment of the system.


- **Use cases:** The use cases of that system will be plainly visible within the structure of that system. Developers will not have to hunt for behaviors, because those behaviors will be first-class elements visible at the top level of the system. Those elements will be classes or functions or modules that have prominent positions within the architecture, and they will have names that clearly describe their function.
- **Operation:** In the context of software architecture, it refers to how the system functions and performs in a production environment. As strange as it may seem, this decision is one of the options that a good architect leaves open. A system that is written as a monolith, and that depends on that monolithic structure, cannot easily be upgraded to multiple processes, multiple threads, or micro-services should the need arise. By comparison, an architecture that maintains the proper isolation of its components, and does not assume the means of communication between those components, will be much easier to transition through the spectrum of threads, processes, and services as the operational needs of the system change over time.
- **Development:** Any organization that designs a system will produce a design whose structure is a copy of the organization’s communication structure. A system that must be developed by an organization with many teams and many concerns must have an architecture that facilitates independent actions by those teams, so that the teams do not interfere with each other during thedevelopment.
- **Deployment:** A good architecture does not rely on dozens of little configuration scripts and property file tweaks. It does not require manual creation of directories or files that must be arranged just so. A good architecture helps the system to be immediately deployable after build.

A good architecture will allow a system to be born as a monolith, deployed in a single file, but then to grow into a set of independently deployable units, and then all the way to independent services and/or micro-services. Later, as things change, it should allow for reversing that progression and sliding all the way back down into a monolith.


# Policy and Level
Software systems are statements of policy. Indeed, at its core, that’s all a computer program actually is. A computer program is a detailed description of the policy by which inputs are transformed into outputs. Part of the art of developing a software architecture is carefully separating those policies from one another, and regrouping them based on the ways that they change. Policies that change for the same reasons, and at the same times, are at the same level and belong together in the same component. Policies that change for different reasons, or at different times, are at different levels and should be separated into different components.

## Level
A strict definition of “level” is “the distance from the inputs and outputs.” The farther a policy is from both the inputs and the outputs of the system, the higher its level. The policies that manage input and output are the lowest-level policies in the system.

The following diagram depicts a simple encryption program that reads characters from an input device, translates the characters using a table, and then writes the translated characters to an output device. An architecture for this system is shown in the class diagram in Figure 19.2. Note the dashed border surrounding the Encrypt class, and the CharWriter and CharReader interfaces. All dependencies crossing that border point inward. This unit is the highest-level element in the system.

![](/Books/Programming/Clean_Architecture/img/figure-19.2.png)

ConsoleReader and ConsoleWriter are shown here as classes. They are low level because they are close to the inputs and outputs. When changes are made to the input and output policies, they are not likely to affect the encryption policy.

Higher-level policies—those that are farthest from the inputs and outputs—tend to change less frequently, and for more important reasons, than lower-level policies. Lower-level policies—those
that are closest to the inputs and outputs—tend to change frequently, and with more urgency, but for less important reasons. Another way to look at this issue is to note that lower-level components
should be plugins to the higher-level components. 

# Business Rules
Strictly speaking, business rules are rules or procedures that make or save the business money. Very strictly speaking, these rules would make or save the business money, irrespective of whether they were implemented on a computer. They would make or save money even if they were executed manually. The fact that a bank charges N% interest for a loan is a business rule that makes the bank money. It doesn’t matter if a computer program calculates the interest, or if a clerk with an abacus calculates the interest. We shall call these rules **Critical Business Rules**, because they are critical to the business itself, and would exist even if there were no system to automate them. Critical Business Rules usually require some data to work with. For example, our loan requires a loan balance, an interest rate, and a payment schedule. We shall call this data **Critical Business Data**. This is the data that would exist even if the system were not automated. The critical rules and critical data are inextricably bound, so they are a good candidate for an object. We’ll call this kind of object an **Entity**.


## Entities
An Entity is an object within our computer system that embodies a small set of critical business rules operating on Critical Business Data. The Entity object either contains the Critical Business Data or has very easy access to that data. The interface of the Entity consists of the functions that implement the Critical Business Rules that operate on that data.

## Use cases
Use cases contain the rules that specify how and when the Critical Business Rules within the Entities are invoked. Use cases control the dance of the Entities. Notice also that the use case does not describe the user interface other than to informally specify the data coming in from that interface, and the data going back out through that interface. From the use case, it is impossible to tell whether the application is delivered on the web, or on a thick client, or on a console, or is a pure service. They describe the application-specific rules that govern the interaction between the users and the Entities. How the data gets in and out of the system is irrelevant to the use cases.

Entities have no knowledge of the use cases that control them. This is another example of the direction of the dependencies following the Dependency Inversion Principle. High-level concepts, such as Entities, know nothing of lower-level concepts, such as use cases. Instead, the lower-level use cases know about the higher-level Entities. Why are Entities high level and use cases lower level? Because use cases are specific to a single application and, therefore, are closer to the inputs and outputs of that system. Entities are generalizations that can be used in many different applications, so they are farther from the inputs and outputs of the system. Use cases depend on Entities; Entities do not depend on use cases.

## Request and Response Models
The use case class accepts simple request data structures for its input, and returns simple response data structures as its output. These data structures are not dependent on anything. They do not derive from standard framework interfaces such as HttpRequest and HttpResponse. They know nothing of the web, nor do they share any of the trappings of whatever user interface might be in place.

You might be tempted to have these data structures contain references to Entity objects. You might think this makes sense because the Entities and the request/response models share so much data. Avoid this temptation! The purpose of these two objects is very different. Over time they will change for very different reasons, so tying them together in any way violates the Common Closure and Single Responsibility Principles. The result would be lots of tramp data, and lots of conditionals in your code.

# Clean architecture
Good architecture produces systems that have the following characteristics:
- Independent of frameworks. The architecture does not depend on the existence of some library of feature-laden software. This allows you to use such frameworks as tools, rather than forcing you to cram your system into their limited constraints.
- Testable. The business rules can be tested without the UI, database, web server, or any other external element.
- Independent of the UI. The UI can change easily, without changing the rest of the system. A web UI could be replaced with a console UI, for example, without changing the business rules.
- Independent of the database. You can swap out Oracle or SQL Server for Mongo, BigTable, CouchDB, or something else. Your business rules are not bound to the database.
- Independent of any external agency. In fact, your business rules don’t know anything at all about the interfaces to the outside world.

![](/Books/Programming/Clean_Architecture/img/figure-22.1.png)

Basically, source code dependencies must point only inward, toward higher-level policies. Nothing in an inner circle can know anything at all about something in an outer circle.

![](/Books/Programming/Clean_Architecture/img/figure-22.2.png)

The diagram shows a typical scenario for a web-based Java system using a database. The web server gathers input data from the user and hands it to the Controller on the upper left. The Controller packages that data into a plain old Java object and passes this object through the InputBoundary to the UseCaseInteractor. The UseCaseInteractor interprets that data and uses it to control the dance of the Entities. It also uses the DataAccessInterface to bring the data used by those Entities into memory from the Database. Upon completion, the UseCaseInteractor gathers data from the Entities and constructs the OutputData as another plain old Java object. The OutputData is then passed through the OutputBoundary interface to the Presenter. The job of the Presenter is to repackage the OutputData into viewable form as the ViewModel, which is yet another plain old Java object. The ViewModel contains mostly Strings and flags that the View uses to display the data. Whereas the OutputData may contain Date objects, the Presenter will load the ViewModel with corresponding Strings already formatted properly for the user. The same is true of Currency objects or any other business-related data.

The InputBoundary interface is defined within the inner layer (Use Cases layer), and the UseCaseInteractor implements it. UseCaseInteractor calls an interface (the OutputBoundary) which is defined within its own inner layer (the Use Cases layer). The Presenter, residing in the outer Interface Adapters layer, implements this OutputBoundary interface.

# Services: Great and small

## The Decoupling Fallacy
One of the big supposed benefits of breaking a system up into services is that services are strongly decoupled from each other. However, they can still be coupled by shared resources within a processor, or on the network. What’s more, they are strongly coupled by the data they share. For example, if a new field is added to a data record that is passed between services, then every service that operates on the new field must be changed. The services must also strongly agree about the interpretation of the data in that field. Thus those services are strongly coupled to the data record and, therefore, indirectly coupled to each other. Clearly, then, this benefit is something of an illusion.

# The Test Boundary
You can think of the tests as the outermost circle in the architecture. Nothing within the system depends on the tests, and the tests always depend inward on the components of the system.

# Frameworks are resources
The relationship between you and the framework author is extraordinarily asymmetric. You must make a huge commitment to the framework, but the framework author makes no commitment to you whatsoever. 
What are the risks? Here are just a few for you to consider.
- The architecture of the framework is often not very clean. Frameworks tend to violate he Dependency Rule. They ask you to inherit their code into your business objects—your Entities! They want their framework coupled into that innermost circle. Once in, that framework isn’t coming back out. The wedding ring is on your finger; and it’s going to stay there.
- The framework may evolve in a direction that you don’t find helpful. You may be stuck upgrading to new versions that don’t help you. You may even find old features, which you made use of, disappearing or changing in ways that are difficult for you to keep up with.
- A new and better framework may come along that you wish you could switch to.
  
## What is the solution?

Treat the framework as a detail that belongs in one of the outer circles of the architecture. Don’t let it into the inner circles. If the framework wants you to derive your business objects from its base classes, say no! Derive proxies instead, and keep those proxies in components that are plugins to your business rules. For example, maybe you like Spring. Spring is a good dependency injection framework. Maybe you use Spring to auto-wire your dependencies. That’s fine, but you should not sprinkle `@autowired` annotations all throughout
your business objects. Your business objects should not know about Spring. 

Instead, you can use Spring to inject dependencies into your Main component. It’s OK for Main to know about Spring since Main is the dirtiest, lowest-level component in the architecture. 

When faced with a framework, try not to marry it right away. See if there aren’t ways to date it for a while before you take the plunge. Keep the framework behind an architectural boundary if at all possible, for as long as possible. Perhaps you can find a way to get the milk without buying the cow.