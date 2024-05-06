###### Objectives

- Understand recursive types

- Read and write datatype definitions

- Understand & implement functions over recursive datatypes

- Understand immutable lists and know the standard operations on them

- Know and follow a recipe for writing programs with ADTs

    

###### Introduction

In this reading we'll look at *recursive datatype*s

- How to specify operations on them, how to implement them

- Our main example will be immutable lists

    

###### Recursive Datatype

- A *recursive datatype* is just like a *recursive function*, which is defined in terms of itself

- We'll see the same need for *base* & *recursive* cases, which will now appear: 
  
  - As different *variants* of the abstract type

    

###### Immutable Lists (1 / 6)

We'll start with a classic recursive datatype, the immutable list

- Immutability is powerful because of its safety and its potential for sharing
  
  - Less memory consumed & less time spent copying

- Let's see how to represent list data structures differently than in familiar ones
  
  - Like array or linked lists

    

###### Immutable Lists (2 / 6)

Let's define `ImList<E>` for an immutable list, with the following 4 operations:

![](C:\Users\yangs\AppData\Roaming\marktext\images\2024-04-19-14-52-53-image.png)

These 4 operations have a long and distinguished pedigree

- They are fundamental to the list-processing languages `Lisp` & `Scheme`

- They are widely used in functional programming, where `first` & `rest`:
  
  - Are sometimes called `head` & `tail` instead

    

###### Immutable Lists (3 / 6)

Let's get used to the operations a bit (lists are written with `[]`)

```scheme
empty() → [ ]  
cons(0, empty()) → [ 0 ]  
cons(0, cons(1, cons(2, empty()))) → [ 0, 1, 2 ]

x = cons(0, cons(1, cons(2, empty())))  → [ 0, 1, 2 ]  
first(x) → 0  
rest(x) → [ 1, 2 ]

first(rest(x)) → 1  
rest(rest(x)) → [ 2 ]  
first(rest(rest(x)) → 2  
rest(rest(rest(x))) → [ ]
```

    

###### Immutable Lists in Java (4 / 6)

- To implement this datatype in Java, we'll write an interface:
  
  ```java
  public interface ImList<E> {
      // todo: empty() returning ImList<E>
      public ImList<E> cons(E elt);
      public E first();
      public ImList<E> rest();
  }
  ```

- And we'll write 2 classes that implement this interface:
  
  - `Empty` represents the result of `empty` operation (an empty list)
  
  - `Cons` represents result of `cons` (an element glued with another list)

    

###### Immutable Lists in Java (5 / 6)

```java
public class Empty<E> implements ImList<E> {
    public Empty() {
    }
    public ImList<E> cons(E elt) {
        return new Cons<>(elt, this);
    }
    public E first() {
        throw new UnsupportedOperationException();
    }
    public E rest() {
        throw new UnsupportedOperationException();
    }
}
```

```java
public class Cons<E> implements ImList<E> {
    private final E elt; 
    private final ImList<E> rest;
    public Cons(E elt, ImList<E> rest) {
        this.elt = elt;
        this.rest = rest;
    }
    public ImList<E> cons(E elt) {
        return new Cons<>(elt, this);
    }
    public E first() {
        return elt;
    }
    public ImList<E> rest() {
        return rest;
    }
}
```

    

###### Immutable Lists in Java (6 / 6)

So we have methods for `cons`, `first`, `rest`, but where is `empty` operation?

- Why don't we specify `empty` signature in the interface like all other 3 methods?
  
  - Because no way `Cons` can implement it (non-empty list only for `Cons`) 

- One way to implement it is to have clients call the `Empty` class constructor
  
  - However, this sacrifices representation independence:
  
  - Clients have to know about the `Empty` class!

- A better way to do it is as a static factory method that takes no argument
  
  - And produces an instance of `Empty`

- We can directly put a static method in the `ImList` interface
  
  ```java
  public interface ImList<E> {
      public static <E> ImList<E> empty() { return new Empty<>(); } 
      ...
  }
  ```

    

###### Notes on Empty

The signature on `empty` uses an unfamiliar bit of generic type syntax

- The `E` in `ImList<E>` is a placeholder for the type of elements in an `ImList`

- But `empty` is a static method: it can't see instance fields, and thus can't see:
  
  - The instance type parameter

- You can read the declaration of `empty` as: for any `E`, `empty()` returns:
  
  - An `ImList<E>`

    

###### Two Classes Implementing One Interface

This design is different from what we've seen with `List`, `ArrayList`, `LinkedList`

- `List` is an *ADT*, and `ArrayList` & `LinkedList` are 2 alternative representations:
  
  - For that datatype

- For `ImList`, the 2 implementations `Empty` & `Cons` cooperate to function
  
  - You need them both

    

###### Recursive Datatype Definitions (1 / )

- `Cons` is an implementation of `ImList`, but it also uses `ImList` inside its rep

- So it recursively requires an implementation of `ImList` to fulfill its contract

    

###### Recursive Datatype Definitions (2 / )

To make this fact clearly visible, we'll write a *datatype definition*:

```c
ImList<E> = Empty + Cons(elt:E, rest:ImList<E>)
```

This is a recursive definition of `ImList` as a set of values. Here's what it means:

- `ImList` consists of values represented in 2 ways: either an `Empty` object only

- Or by a `Cons` object whose fields are an element `elt`, and an `ImList` `rest`

- Can write any `ImList` value as a term using this definition. `[1, 2, 3]` can be:
  
  ```c
  Cons(0, Cons(1, Cons(2, Empty)))
  ```

    

###### Recursive Datatype Definitions (3 / )

Formally, a datatype definition has:

- An *abstract datatype* on the left, defined by its rep on the right

- The rep consists of variants of the datatype, combined by a `+`

- Each variant is a class name with 0 or more fields, written with name & type
  
  - Separated by a `:`

    

###### Recursive Datatype Definitions (4 / )

- A recursive datatype definition is when the abstract type (what's on the left):

- Appears in its own definition (as the type of a field on the right). Binary tree:
  
  ```c
  Tree<E> = Empty + Node(e:E, left:Tree<E>, right:Tree<E>)
  ```

- When you write a recursive datatype, document its definition as a comment
  
  ```java
  public interface ImList<E> {
      // Datatype definition:
      //   ImList<E> = Empty + Cons(elt:E, rest:ImList<E>)
  ```

    

###### Functions Over Recursive Datatypes （1 / )

The way of thinking about datatypes - as a recursive definition: 

- Of an abstract datatype with concrete variants, is appealing not only because:

- It can handle recursive & unbounded structures (lists, trees), but also because:

- It provides a convenient way to describe operations over the datatype, as:
  
  - Functions with one case per variant

    

###### Functions Over Recursive Datatypes (2 / )

First, notice how datatype definition maps to the abstract interface & concrete variants

<img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-05-03-11-55-37-image.png" title="" alt="" width="628">

    

###### Functions Over Recursive Datatypes (3 / )

- Now let's add a new operation, by thinking about it as a recursive function: 
  
  - With one case per variant

- The operation is `size`, which can be defined as `size: ImList -> int`. Then:
  
  ```c
  size(Empty) = 0
  size(Cons(elt:E, rest:ImList)) = 1 + size(rest)
  ```

- The function is recursive. Can execute the `size` as a series of reduction steps:
  
  ```c
  size(Cons(0, Cons(1, Empty)))  
     = 1 + size(Cons(1, Empty))  
     = 1 + (1 + size(Empty))  
     = 1 + (1 + 0)  
     = 1 + 1  
     = 2
  ```

- And the cases from the definition can be translated directly into java methods:
  
  ```java
  public interface ImList<E> {
      public int size();
  }
  ```
  
  ```java
  public class Empty<E> implements ImList<E> {
      public int size() { return 0; }
  }
  ```
  
  ```java
  public class Cons<E> implements ImList<E> {
      public int size() { return 1 + rest.size(); }
  }
  ```

    

###### Functions Over Recursive Datatypes (4 / )

This pattern, called *interpreter pattern*, does operation over a recursive datatype by:

- Declaraing the operation in the abstract datatype interface

- Implement the operation (recursively) in each concrete variant

    

###### Let's Implement More: `isEmpty()`

```java
public interface ImList<‍E> { ...
    public boolean isEmpty();
}
```

```java
class Empty<‍E> implements ImList<‍E> {
    @Override
    public boolean isEmpty() { return true; }
}
```

```java
class Cons<‍E> implements ImList<‍E> {
    @Override
    public boolean isEmpty() { return false; }
}
```

    

###### Let's Implement More: `append()`

```java
public interface ImList<‍E> { ...
    public ImList<E> append(ImList<E> other);
}
```

```java
class Empty<‍E> implements ImList<‍E> {
    @Override
    public ImList<E> append(ImList<E> other) { return other; }
}
```

```java
class Cons<‍E> implements ImList<‍E> {
    @Override
    public ImList<E> append(ImList<E> other) {
        return new Cons<>(elt, rest.append(other));
        // or return rest.append(other).cons(elt);
    }
}
```

    

###### Turning the Rep (1 / 3)

- `size` might be called fairly often, but our current impelmentation takes `O(n)`

- Can make it better by a change to the rep, that caches the size of the list: 
  
  - When we compute it for the first time, so that subsequent calls cost `O(1)`

    

###### Turning the Rep (2 / 3)

```java
public class Cons<E> implements ImList<E> {
    private final E elt;
    private final ImList<E> rest;
```

```java
    // rep invariant:
    //     elt != null, rest != null, size >= 0
```

```java
    public int size() {
        if (size == 0) size = 1 + rest.size();
        return size;
    }
}
```

    

###### Turning the Rep (3 / 3)

- This is a immutable datatype, and yet it has a mutable rep

- It modifies its own `size` field, to cache the result of an expensive operation

- This is an example of a *beneficent mutation*, a state change that doesn't change:
  
  - The abstract value represented by the object, so the type is still immutable

    

###### Rep Independence & Rep Exposure Revisited (1 / 4)

Does our Java implementation of `ImList` still have rep independence?

- The `Empty` constructor should be called behind static `ImList.empty()`

- And `Cons` constructor should be called behind method `ImList::cons()`

- Both'd not be called directly by clients. Can hide by making them *package-private*
  
  - i.e., declared with no access modifiers, so that: 
  
  - Classes outside of `ImList`'s package cannot see or use them

    

###### Rep Independence & Rep Exposure Revisited (2 / 4)

We have much freedom to change our implementation

- We added a `size` field to the internal rep of `Cons`

- We could even have an extra array in there to make `get` run faster

- This may get expensive in space, but we're free to consider the tradeoffs

    

###### Rep Independence & Rep Exposure Revisited (3 / 4)

What about an operation like `isEmpty` above?

- Does that break *rep independence* by revealing the concrete variant to clients?
  
  - i.e., is it the same as checking `instanceof Empty`?

- No, the spec of `isEmpty` is not *return true iff this is an instance of Empty*
  
  - Instead, like any operation of an abstract type, we specify it abstractly
  
  - e.g., *return true iff this list contains no elements*

- Lesson here: any time you write an *ADT*, the specs must not talk about the rep
  
  - The concrete variants of a recursive *ADT* are its reps, which:
  
  - Cannot be mentioned in the specs

    

###### Rep Independence & Rep Exposure Revisited (4 / 4)

Is there *rep exposure* since `Cons.rest()` returns a reference to its internal list?

- A client add elements to `rest` of the list? 2 `Cons`'s invariants'd be threatened:
  
  - Immutability, and that the cached `size` is always correct

- But there’s no risk of rep exposure, because the internal list is immutable
  
  - Nobody can threaten the rep invariant of `Cons`

    

###### Null vs. Empty (1 / 2)

Tempting to get rid of the `Empty` class, and just use `null`? Resist that tempation

- Using an object, rather than a null reference, to signal the base-case / endpoint
  
  - Is an example of a design pattern called *sentinel object*s

- The enormous advantage that is that It acts like an object in the datatype
  
  - So you can call methods on it

- We can't call `size` on an empty list, if it's represented by a `null`. The code'll be:
  
  ```java
  if (list != null) n = list.size();
  ```

- What's above is cluttered, obscure, and easy to forget. Better with this:
  
  ```java
  n = list.size();
  ```

    

###### Null vs. Empty (2 / 2)

Keep `null` values out of your data structures, and your life will be happier

    

###### Declared Type vs. Actual Type (1 / )

- Now that we're using interfaces & classes,  it's worth taking a moment to reinforce:
  
  - How Java's type-checking works. All statically-checked languages work this way

- 2 terms in type check: *compile time* before program runs & *run time* as it executes
  
  - We saw this distinction when we discussed [dynamic dispatch](https://web.mit.edu/6.031/www/sp21/classes/12-interfaces-generics-enums/#overriding_and_dynamic_dispatch)

- At *compile time*, every variable has a *declared type* (a.k.a., *static type*), in declaration
  
  - The compiler uses the declared types of variables (and method return values)
  
  - To deduce static types for every expression in the program
  
  - e.g., after declaration `String s`, `s` has *static type* `String`, `s.charAt(0)`:
    
    - Has *static type* `char`, `s.length()` has *static type* `int`

    

###### Declared Type vs. Actual Type (2 / )

At runtime, every object has an actual type, a.k.a., *dynamic type*

- Imbued in it, by the constructor that created the object

- e.g.,  `new String()` makes an object whose actual type is `String`

- `new Empty()` makes an object whose actual type is `Empty`

- `new ImList()` is forbidden by Java, because `ImList` is an interface
  
  - It has no object values of its own, and no constructors

    

###### Another Example: Boolean Formulas

Another useful sort of recursive datatype is for boolean formulas

```c
(P ∨ Q) ∧ (¬P ∨ R)
```

- We can give a datatype definition suitable for representing all formulas:
  
  ```c
  Formula = Variable(name:String)
            + Not(formula:Formula)
            + And(left:Formula, right:Formula)
            + Or(left:Formula, right:Formula)
  ```

- `(P ∨ Q) ∧ (¬P ∨ R)` would be:
  
  ```c
  And( Or(Variable("P"), Variable("Q")),
       Or(Not(Variable("P")), Variable("R")) )
  ```

    

###### Backtracking Search with Immutability (1 / 4)

<img title="" src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-05-03-17-54-25-image.png" alt="" width="479">

We started out this reading with immutable lists

- Which permits lots of sharing between different list instances

- This sharing is limited though: only the ends of lists can be shared
  
  - If 2 lists are identical at the beginning but then diverge
  
  - They have to be stored separately, as additions come from the head

    

###### Backtracking Search with Immutability (2 / 4)

Backtracking search is a great application for immutable, sharable lists

- A search through a space generally proceeds by making one choice after another
  
  - And when a choice leads to a dead end, you backtrack

- Mutable data structures are typically not a good approach for backtracking
  
  - If you use a mutable `Map`, to keep track of the current bindings
  
  - Then you have to undo those bindings every time you backtrack
  
  - That’s error-prone and painful compared to what you do with immutables
    
    - When you backtrack, you just throw the map away!

    

###### Backtracking Search with Immutability (3 / 4)

- But immutable data structures with no sharing aren’t a great idea either
  
  - Because the space used to track where you are will grow quadratically 
  
  - If you have to make a complete copy every time you take a new step
  
  - Have to hold on to all the previous stuff on your path, in case back up

- Immutable lists have the nice property that each step taken on the path: 
  
  - Can share all info from previous steps, just by adding to the front
  
  - When you have to backtrack, stop using the current step’s state
    
    - But you still have references to the previous step’s state

    

###### Backtracking Search with Immutability (4 / 4)

Finally, a search that uses immutable data structures is ready to be parallelized

- Can delegate multiple processors to search multiple paths at once

- Without having to deal with the problem that they'll step on each other
  
  - In a shared mutable data structure
