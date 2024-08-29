###### Objectives

- Understand the properties of an equivalence relation

- Understand equality for immutable types, defined in terms of *AF*, and observation

- Differentiate between reference equality & object equality

- Differentiate between strict observational, and behavioral equality for mutables

- Understand the *object contract*, and implement equality correctly for types

    

###### Introduction

- In previous readings, we covered *data abstraction*, which is to create types, defined:
  
  - By their operations, rather than their representation

- For an *ADT*, the *AF* explains how to interpret a representation into a value
  
  - Of the abstract type

- In this reading, we turn to how *equality* of values in a data type is defined: the *AF*
  
  - Will give us a way to cleanly define the *equality* operation on an *ADT*

- The first part of this reading focuses on defining *equality* for immutable types, then:
  
  - We'll move to *mutable*s as well

    

###### Equivalence Relation

Start with a look at the mathematical properties that need to be satisfied by equality

- An equality operation `E` on type `T`, can be seen as a *binary relation* `E` ⊆ `T` x `T`
  
  - `x` and `y` are only equal when `(x, y) ⊆ E`

- For `E`, it has to be an *equivalence relation*, meaning it satisfies the following three:
  
  ![](C:\Users\yangs\AppData\Roaming\marktext\images\2024-04-16-11-50-35-image.png)

- We'll now replace `E` with `==`, and now these properties can also be written as:
  
  ![](C:\Users\yangs\AppData\Roaming\marktext\images\2024-04-16-11-52-36-image.png)

    

###### Equality of Immutable Types (1 / 2)

See equality in the context of *ADT*s, start with *immutable*s, which's defined in 2 ways:

- Using the *abstract function*. We say `a` equals to `b` if and only if `AF(a) = AF(b)`

- Using observation. Every operation we apply produces the same result for both
  
  - Consider the set expressions `{1, 2}` & `{2, 1}`, `||` & `∈` function the same
  
  - e.g., `|{1,2}| = 2` & `|{2,1}| = 2`; `1 ∈ {1,2}` is true, & `1 ∈ {2,1}` is true

    

###### Equality of Immutable Types (2 / 2)

"Operation of the *ADT*" means the operations that are part of the spec of the *ADT*

- Java has language features that allow a client to break the abstraction barrier
  
  - And observe between objects irrelevant to their abstract value

- e.g., `==` detects if two objects are actually stored at different places in memory
  
  - So does `System.identityHashcode()`, which computes a hash code
  
  - Based on an object's memory address

- But neither of these operations would be part of the spec of an *ADT* (e.g., set). Thus:
  
  - They're not considered when deciding if two objects are equal by observation

- The two definitions of equality – by abstraction function, and by observation: 
  
  - Should be consistent with each other, or something is wrong

    

###### == vs. `Equals()`

- Java has 2 operations for testing equality between objects, with different semantics
  
  - `==` tests *reference equality* (same storage in memory)
  
  - `equals()` tests *object equality* (contents, by abstract function of observation)

- Here are equality operations in several other languages:
  
  <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-04-16-13-59-56-image.png" title="" alt="" width="245">

    

###### == vs. `Equals()` Cont.

- Note that `==` unfortunately flips its meaning between Java and Python

- Here we're talking about equality for objects. For built-in primitives (e.g., `int`)
  
  - Can't use `equals`, and `==` does *object equality* testing

- When we define a new data type, it’s our responsibility to decide: 
  
  - What object equality means for values of the data type
  
  - And implement the `equals()` operation appropriately

    

###### Implementing `equals()`

 `equals()` method is defined by `Object`, and its default implementation looks like:

```java
public class Object {
    ...
    public boolean equals(Object that) {
        return this == that;
    }
}
```

- The default `equals()` is the same as *reference equality* (i.e., almost always wrong)

- So you have to **override** the `equals()` method, replacing it with your own

    

###### The Wrong Way to Implement `equals()` (1 / 3)

Here's our first try at overriding `equals()` for `Duration`

```java
public class Duration {
    private final int mins, secs;
    /** Make a duration lasting for m minutes and s seconds. */
    public Duration(int m, int s) {
        mins = m; secs = s;
    }
    /** @return length of this duration in seconds */
    public long getLength() {
        return (long)mins*60 + secs;
    }
```

```java
    public boolean equals(Duration that) {
        return this.getLength() == that.getLength();        
    }
}
```

    

###### The Wrong Way to Implement `equals()` (2 / 3)

A subtle problem here, why doesn't this work?

```java
Duration d1 = new Duration (1, 2);
Duration d2 = new Duration (1, 2);
Object o2 = d2;
d1.equals(d2) → true
d1.equals(o2) → false
```

- Though `d2` & `o2` refer to the same object, still get different results from `equals`

- It's because `Duration` has **overloaded** the `equals` method, not **overriden** it

- We actually have 2 `equals` in `Duration`, one implicitly inherited, and a new one:
  
  ```java
  public class Duration extends Object {
      // explicit method that we declared:
      public boolean equals(Duration that) {
          return this.getLength() == that.getLength();
      }
      // implicit method inherited from Object:
      public boolean equals(Object that) {
          return this == that;
      }
  }
  ```

    

###### The Wrong Way to Implement `equals()` (3 / 3)

To avoid such mistake, Java comes up with the annotation `@override`

- Which you'd use whenever your itention is to override a method in the superclass

- With this annotation, Java compiler will if this method with the same signature:
  
  - Exists in the superclass, and gives you errors if it does not (i.e., mistake)

    

###### instanceof

The `instanceof` operator tests if an object is an instance of a particular type

- It's used as dynamic type checking, not the static type checking we prefer

- In general, using `instanceof` in object-oriented programming is a bad smell

- The prohibition includes other ways of inspecting runtime types (e.g., `getClass`)

- In a future reading, we’ll see examples of when you might use `instanceof`, and:
  
  - How to write alternatives that are safer from bugs and ready for change

    

###### The Object Contract (1 / 2)

The spec of `Object` class is so important that it's referred to as *the `Object` Contract*

- The contract can be found in the method specifications for the `Object` class

- We'll focus on the contract for `equals`. When you override the `equals` method
  
  - You must adhere to its general contract (states as below)

    

###### The Object Contract (2 / 2)

- `equals` must define an *equivalence relation* - reflexive, symmetric, transitive

- `equals` must be consistent: repeated calls to that yield the same result

- `x.equals(null)` always returns `false` (`x` can't be `null`, otherwise exceptions)

- `hashcode` must produce the same result for 2 objects deemed equal by `equals`

    

###### Breaking Hash Tables (1 / 4)

- To know the contract relating to the `hashCode`, need idea of how hash tables work

- `HashSet` & `HashMap`, use a hash table data structure, and depend on `hashCode`:
  
  - To be implemented correctly for the objects stored in the set/map

    

###### Breaking Hash Tables (2 / 4)

Here's how a hash table works

- It has an array with an initial size, when an item is inserted, we compute its hash
  
  - And convert the hash code into an index, then insert it

- The *rep invariant* of a hash table includes that an item can be found by:
  
  - Starting from the slot determined by its hash code

- When 2 items are hashed the same, collision occurs, at which `equals`'d used:
  
  - To determine whether the 2 are equal, and put in different buckets if not

    

###### Breaking Hash Tables (3 / 4)

`Object`'s default `hashCode` is consistent with its default `equals`    

```java
public class Object {
  ...
  public boolean equals(Object that) { return this == that; }
  public int hashCode() { return /* the memory address of this */; }
}
```

- Immutable objects need a different implementation of `hashCode`. If we don't do it:

- We're breaking the *object contract*, here's an example with `Duration`:
  
  ```java
  Duration d1 = new Duration(1, 2);
  Duration d2 = new Duration(1, 2);
  d1.equals(d2) → true
  d1.hashCode() → 2392
  d2.hashCode() → 4823
  ```
  
  - Equal objects give different hash codes. Need a fix!

    

###### Breaking Hash Tables (4 / 4)

A simple & drastic way is for `hashCode` to always return a constant

- (We don't say inequal objects have to return different hash codes in the contract)

- Now every object's hash code is the same, of course equal ones give the same
  
  - A disastrous effect on performance, shouldn't be applied 

- Instead, compute a hash code for each component of the object used in `equals`
  
  - Usually by calling their `hashCode`, and combine them together

- As a general rule: **Always override `hashCode` when you override `equals`**
  
  - Use `@override`, then you can find it when you mistype as `hashcode`

    

###### Equality of Mutable Types (1 / 3)

What about equality of mutable objects?

- Equality must still be an equivalence relation, and it'd respect *AF* & *observation*

- However, there is a new possibility: by calling a mutator before the *observation*
  
  - We may change its state and create a difference between the two objects

    

###### Equality of Mutable Types (2 / 3)

So let's refine our definition and allow 2 notions of equality based on *observation*:

- *Observational equality* means 2 references cannot be distinguished now
  
  - The `equals` should only call *observers* & *producers* on these objects:
  
  - And compares the results of the operations. This tests if the 2 objects:
  
  - Look the same at the current state of program

- *Behavioral equality* means 2 references cannot be distinguished now or future
  
  - Even if a *mutator* is called to change the state, they still look the same
  
  - This tests if the 2 will behave the same, in this & all future states

    

###### Equality of Mutable Types (3 / 3)

- For immutables, *obseravtional* & *behavioral equivalence* are the same, as for them:
  
  - There are no *mutators*

- For mutables, can be very tempting to use observational equality when designing
  
  - Java uses *observational equality* for most of its mutables (e.g., `List`)

    

###### Breaking a HashSet's Rep Invariant (1 / 3)

- 2 objects that are *observationally equal* may stop being equal after a mutate

- The fact allows us to easily break the rep invariants of other collection

- Suppose we make a `List`, and then drop it to a `HashSet`:
  
  ```java
  List<String> list = new ArrayList<>();
  list.add("a");
  
  Set<List<String>> set = new HashSet<List<String>>();
  set.add(list);
  ```

- We can check the the set contains the list we put in it, and it does:
  
  ```java
  set.contains(list) → true
  ```

- But now we mutate the list, and it no longer appears in the set:
  
  ```java
  list.add("goodbye");
  set.contains(list) → false!
  ```

- When we try to iterate the set, we find the list, but the set says it's not there
  
  ```java
  for (List<String> l : set) { 
      set.contains(l) → false! 
  }
  ```

    

###### Breaking a HashSet's Rep Invariant (2 / 3)

What's going on?

- Mutation affects result of `equals` & `hashCode`, when the list is first put into set:
  
  - It's stored in the bucket corresponding to its `hashCode` result at that time

- When the list is subsequently mutated, its `hashCode` changes, but the set doesn't:
  
  - Realize the list should be moved to another bucket
  
  - So it can never be found again

- When `equals` & `hashCode` can be affected by mutation, we can easily:
  
  - Break the rep invariant of a hash table that uses the object as a key

    

###### Breaking a HashSet's Rep Invariant (3 / 3)

- Here's a telling quote from the spec of `java.util.Set`:
  
  ```java
  Note: Great care must be exercised if mutable objects are used 
      as set elements. The behavior of a set is not specified if
      the value of an object is changed in a manner that affects 
      equals comparisons while the object is an element in the set
  ```

- The Java library is unfortunately inconsistent about its interpretation of `equals` 
  
  - For mutable classes

- Collections like `List`, `Set` & `Map` use *observational equality*. But other mutables
  
  - Like `StringBuilder`, and arrays, use *behavioral equality*

    

###### The Final Rule for `equals` & `hashCode`

- The lesson we learn is that `equals()` should implement behavioral equality
  
  - Which is to not override `equals` & `hashCode` (just compare the memory)

- For a mutable type that needs a notion of observational equality, it's better:
  
  - To define a completely new operation (e.g., `similar()`)

    

###### Autoboxing & Equality (1 / 2)

One more instructive pitfall in Java

- We’ve talked about primitive types and their object type equivalents
  
  - `int` and `Integer`

- For `Integer`, `equals` compares *object equality*, and `==` compares reference
  
  ```java
  Integer x = new Integer(3);
  Integer y = new Integer(3);
  x.equals(y); → true
  x == y; → false
  ```

- But for primitive types like `int`, `==` compares the value than the reference
  
  ```java
  (int)x == (int)y // returns true
  ```

- So can't use `Integer` interchangeably with `int`. The fact that Java automatically
  
  - Converts between `int` & `Integer` (i.e., *autoboxing* & *autounboxing*):
  
  - Can lead to subtle bugs

    

###### Autoboxing & Equality (2 / 2)

You have to be aware what the compile-types of your expressions are. Check this:

```java
Map<String, Integer> a = new HashMap<>(), b = new HashMap<>();
String c = "c";
a.put(c, 130); // put ints into the maps
b.put(c, 130);
a.get(c) == b.get(c) → ?? // what do we get out of the maps? 
// are they equal?
```

    

###### Summary

- Equality should be an equivalence relation (reflexive, symmetric, transitive)

- Equality and hash code must be consistent with each other

- The abstraction function is the basis for equality in immutable data types

- Reference equality is the basis for equality in mutable data types

    
