###### Subtypes (1 / 3)

A type is a set of values

- `List` in Java is a type, if we think about all possible `List` values:
  
  - None of them are objects of `List` (since it cannot be instantiated)

- Instead, those values are objects of class implementing `List` 
  
  - e.g., `ArrayList`, `LinkedList`, etc

- A *subtype* is a subset of the *supertype*. When we say *B is a subtype of A*, we mean:
  
  - *Every B is an A*, or *every B satisfies the spec of A*

    

###### Subtypes (2 / 3)

`B` is only a subtype of *A* if *B*'s specification is at least as strong as `A`'s

- When we declare a class that implements an interface, java compiler: 
  
  - Enforces part of the requirement automatically
  
  - It ensures every method in `A` appears in `B`, with a compatible type signature

- But the compiler cannot check we haven’t weakened the spec in other ways:
  
  - Strengthening the precondition on some inputs to a method
  
  - Weakening a postcondition
  
  - Weakening a guarantee that the interface abstract type advertises to clients

    

###### Subtypes (3 / 3)

If you declare a subtype, must ensure subtype’s spec is at least as strong as supertype’s

    

###### Why Interfaces

A more detailed explanation can be found here: [why interfaces]([Reading 12: Defining ADTs with Interfaces, Generics, Enums, and Functions](https://web.mit.edu/6.031/www/sp21/classes/12-interfaces-generics-enums/#why_interfaces))

- Documentation for both the compiler and for human

- Allowing performance trade-offs

- Methods with intentionally undetermined specifications

- Multiple views of one class

- More and less trustworthy implementations

    

###### Subclassing (1 / 3)

- Implementing an interface is 1 way to make a subtype. Another way is *subclassing*
  
  - It means defining a class as an extension, or subclass of another class

- The subclass inherits the instance methods of its superclass, **including bodies**
  
  - And the subclass can overwrite any method body with its own

- The subclass also **inherits the fields** (attributes) of its superclass, and can add:
  
  - New instance methods & fields

- The boldfaced part above shows how *subclassing* differs from: 
  
  - Implementing an interface

    

###### Subclassing (2 / 3)

- Like implementing an interface, *subclassing* should imply *subtyping*
  
  - If `A` is a subclass of `B`, `A` has to have a at least as strong spec as `B`

- Unlike implementing an interface, a subclass inherits not only the spec: 
  
  - But also the *rep* of its superclass

- At first this seems like a great idea. Reusing implementation ought to: 
  
  - Make our code more DRY

    

###### Subclassing (3 / 3)

- But after years of experience API designers have revealed subtle challenges to: 
  
  - Making subclassing safe from bugs and ready for change

- If you use subclass, which in turn inherits representation, the following happens:
  
  - Rep exposure between superclass and all its subclasses
  
  - Rep dependence between superclass and all its subclasses
  
  - Superclass and subclass can inadvertently break each other's rep invariant

- Designing for safe subclassing means the superclass must now offer 2 contracts: 
  
  - One for interaction with clients, and one for subclasses
  
  - These issues simply do not arise with interfaces

    

###### Overriding & Dynamic Dispatch

- We can't get away from thinking about *subclassing* when we write classes in Java
  
  - Every class is a subclass of `Object`, and inherits methods like`toString()`

- When a method has multiple implementations, then Java has to determine:
  
  - Which to use when the method is called, called *dispatching* to the method

- Java's Java’s rule is *dynamic dispatch* – using implementation of *dynamic type* 
  
  - Instead of the static type of the reference that points to the object

    

###### Generic Types (1 / 3)

Generic type: a type whose spec is in terms of a placeholder type to be filled later

- Suppose we want to implement the generic `Set<E>` interface, can either write:
  
  - A non-generic implementation that replaces `E` with a specific type
  
  - Or a generic implementation that keeps the placeholder

- Let's start with a generic interface & non-generic implementation example
  
  ```java
  public class CharSet implements Set<Character>
  ```
  
  <img src="file:///C:/Users/yangs/screenshots/set.PNG" title="" alt="" width="544">
  
  - Wherever the interface mentions `E`, the `CharSet` replace it with `Character`

    

###### Generic Types (2 / 3)

Note the rep for `CharSet` is not suited for that of arbitrary-type elements

- A `String` rep cannot represent a `Set<Integer>` without careful work to:

- Define a new rep invariant & abstraction function that handle multi-digit nums

    

###### Generic Types (3 / 3)

Now let's look at the generic interface & generic implementation example:

![](C:\Users\yangs\screenshots\hashmap.PNG)

- Like the code above, we can write code blind to the actual type that clients'd use

- A generic implementation can only rely on details of a placeholder type `E` 
  
  - That are explicitly included in the generic interface specification

- `HashSet` can call method that implement `Object` like `hashCode` & `equals`
  
  - But cannot call methods that are only declared for specific type like `String`

    

###### Concept Test

```java
/** Represents a set that can grow but never shrink. */
/*1*/ public interface GrowingSet<Z> extends Set<E> {
/*2*/     public boolean contains(Z z);
          // ...
      }
```

Here there's an error at line 1: `Set<E>` should be `Set<Z>`

- `GrowingSet` is a generic interface with its own type parameter, and it's allowed to:
  
  - Name its type parameter whatever it wants

- Same as that a function can name its args anything it wants: 
  
  - `f(int n)` can be changed to `f(int p)` or `f(int k)` 
  
  - Without affecting clients of `f`

- So introducing `Z` is fine, but this type needs to be used to instantiate `Set`
  
  - `E` is unknown in this code, 

    

###### Enumerations (1 / 5)

Sometimes an ADT has a small, finite set of immutable values

- It makes sense to define all the values as named constants, called an *enumeration*
  
  ```java
  public enum Month { JANUARY, FEBRUARY, MARCH, ..., DECEMBER };
  ```

- The `enum` defines a new type `Month`, in the same way as `class` & `interface`

- Also defines a set of values in all-caps as they're`public static final` constants

- Can now write code like this:
  
  ```java
  Month thisMonth = MARCH;
  ```

    

###### Enumerations (2 / 5)

- For the `enum` above, Java'd assign numbers to elements as their default rep value

- The simplest operation you'd do with `enum`, is testing quality between values:
  
  ```java
  if (day.equals(SAT) || day.equals(SUN)) System.out.println("weekend");
  ```

- May also see code like this. If the rep value is of an object, it'd be wrong:
  
  ```java
  if (day == SAT || day == SUN) {System.out.println("weekend");}
  ```

- But with `enum`, only 1 object in memory representing values in the *enumeration*
  
  - No way for a client to create more (no *constructor*s). So `==` is fine (better)

- With `==`, we can fail fast if anything wrong, since it does a static check that: 
  
  - Both sides have the same enum type, whereas `equals()` delays until runtime

    

###### Enumerations (3 / 5)

Using an enumeration can feel like you’re using primitive `int` constants

- Java even supports using them in `switch` statements
  
  ```java
  switch (direction) {
      case NORTH: return "polar bears";
      case SOUTH: return "penguins";
  }
  ```

- Note Java only allow primitive integer types & wrappers, and `String`s, in `switch`
  
  - `byte`, `char`, `short`, `int`, `Byte`, `Character`, `Short`, `Integer`

- But unlike `int` values, *enumeration*s have more static checking:
  
  ```java
  Month firstMonth = MONDAY; 
  // static error: MONDAY has type DayOfWeek, not type Month 
  ```

    

###### Enumerations (4 / 5)

- An `enum` declaration can contain all the usual fields & methods that a `class` can
  
  - Here's an example `enum `called [`Month`](https://web.mit.edu/6.031/www/sp21/classes/12-interfaces-generics-enums/#enumerations)

- All `enum` types also have some automatically-provided operations
  
  - `ordinal` is the index of value in the enum (`JANUARY.ordinal` returns 0)
  
  - `compareTo()` compares two values based on their ordinal numbers.
  
  - `name()` returns name of value as a string (`MAY.name()` returns `"MAY"`)
  
  - `toString()` has the same behavior as `name()`

    

###### Enumerations (5 / 5)

- An `enum` could be in a file by itself, like a class or interface

- But often, an `enum` is a subsidiary to another module (like `Month` in `Date`)

- It'd make sense to define the `enum` as a public declaration inside `Date`
  
  - Now external clients can refer to it using `Date.Month`

    

###### ADTs in Non-OOP Languages

One more way to define an ADT is global function operating on an opaque type

- Rarely seen in OOP languages, but appears in C, here's some file I/O in C
  
  ```c
  FILE* f = fopen("out.txt", "w"); // open a file for writing
  fputs("hello", f); // write to the file
  fclose(f);  // close the file
  ```

- In the code above, `FILE` is an ADT. The functions (`fopen`, `fputs`, `flose`) are:
  
  - Operations on type `File`

    

###### ADTs in Java

We’ve now completed our [Java toolbox of ADT concepts](https://web.mit.edu/6.031/www/sp21/classes/10-abstract-data-types/#realizing_adt_concepts_in_java)

<img src="file:///C:/Users/yangs/screenshots/ADTs.PNG" title="" alt="" width="455">

    
