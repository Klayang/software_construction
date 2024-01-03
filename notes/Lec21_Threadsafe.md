###### Race Conditions

- Race conditions arise when multiple threads share the same mutable variable:
  
  - Without coordinating what they’re doing

- This is unsafe, because the correctness of the program depends on: 
  
  - Timing of their low-level operations

    

###### How to Avoid Race Condition

4 ways to make variable access safe in shared-memory concurrency

- *Confinement*: don't share variables or data between threads

- *Immutability*: make the shared variables immutable (more in this reading)

- *Thread safe data type*: encapsulate the shared data in an existing thread-safe type

- *Synchronization*: keep threads from accessing shared variables at the same time
  
  - Used to build our own thread-safe type. Described in next reading

    

###### What Threadsafe means (1 / 2)

A data type or static method is *threadsafe* if:

- It behaves correctly when used from multiple threads

- Regardless of how those threads are executed and

- Without demanding additional coordination from the calling code

    

###### What Threadsafe means (2 / 2)

- *Without additional coordination* means the type cannot put preconditions: 
  
  - On its caller related to timing, like "can't call `get` while `set` in progress"

- e.g., `Iterator`, is not thread-safe when used with a mutable collection
  
  - The spec says you can't modify a collection when you iterate it
  
  - Except for using its own `remove` method

    

###### Strategy #1: Confinement (1 / 2)

Simple idea: keep the data confined to a single thread

- Local variables are always thread confined. A local variable is stored: 
  
  - In the stack, and each thread has its own stack

- But be careful, since a local variable may point to a global object
  
  - Shouldn't have references to that 1 object in different threads

    

###### Strategy #1: Confinement (2 / 2)

Confiment makes `n`, `i`, `result` thread-safe in the following code:

```java
public class Factorial {
    /**
     * Computes n! and prints it on standard output.
     * @param n must be >= 0
     */
    private static void computeFact(final int n) {
        BigInteger result = BigInteger.valueOf(1);
        for (int i = 1; i <= n; ++i) {
            System.out.println("working on fact " + n);
            result = result.multiply(BigInteger.valueOf(i));
        }
        System.out.println("fact(" + n + ") = " + result);
    }

    public static void main(String[] args) {
        new Thread(new Runnable() { // create a thread using an
            public void run() {     // anonymous Runnable
                computeFact(99);
            }
        }).start();
        computeFact(100);
    }
}
```

    

###### Avoid Global Variables (1 / 5)

- Unlike local variables, static variables are not automatically thread confined

- If static variables in your program, have to make sure only 1 thread accesses that

    

###### Avoid Global Variables (2 / 5)

Example: singleton design patter, which uses a private static variable:

```java
// This class has a race condition in it.
public class PinballSimulator {

    private static PinballSimulator simulator = null;
    // invariant: there should never be more than one PinballSimulator
    //            object created

    private PinballSimulator() {
        System.out.println("created a PinballSimulator object");
    }

    // factory method that returns the sole PinballSimulator object,
    // creating it if it doesn't exist
    public static PinballSimulator getInstance() {
        if (simulator == null) {
            simulator = new PinballSimulator();
        }
        return simulator;
    }
}
```

    

###### Avoid Global Variables (3 / 5)

- The class above has a race in the `getInstance` method. 2 threads can call it:
  
  - At the same time, and end up creating 2 instances, which we don't want

- In general, static variables are very risky for concurrency. They might be hiding:
  
  - Behind an innocuous function that seems to have no side-effects or mutations

    

###### Avoid Global Variables (4 / 5)

```java
// is this method threadsafe?
/**
 * @param x integer to test for primeness; requires x > 1
 * @return true if x is prime with high probability
 */
public static boolean isPrime(int x) {
    if (cache.containsKey(x)) return cache.get(x);
    boolean answer = BigInteger.valueOf(x).isProbablePrime(100);
    cache.put(x, answer);
    return answer;
}

private static Map<Integer,Boolean> cache = new HashMap<>();
```

    

###### Avoid Global Variables (5 / 5)

- It stores the answers from previous calls in case they’re requested again
  
  - Called *memoization*

- But now the `isPrime` method is not safe to call from multiple threads
  
  - Because `cache` is shared, and `HashMap` is not threadsafe

- If multiple threads mutate the map at the same time, by calling `cache.put()`
  
  - The map can be corrupted like the bank account in concurrency reading

    

###### Confiment & Fields

```java
public class PinballSimulator {
    private final List<Ball> ballsInPlay;
    private final Flipper leftFlipper;
    private final Flipper rightFlipper;
    // ... other instance fields...
    // ... instance observer and mutator methods ...
}
```

- Instance variables aren't automatically thread confined, even if declared `private` 

- If we want to argue that the `PinballSimulator` ADT is threadsafe:
  
  - We cannot use confinement

- We don’t know whether, or how, clients created aliases to 1 `PinballSimulator` 
  
  - Instance, from multiple threads

- If they have, then concurrent calls to methods of this instance will make: 
  
  - Concurrent access to its fields and their values

    

###### Confinement & Annoymous Classes (1 / 5)

Using annoymous `Runnable` can complicate the confinement of local variables

```java
public class PinballSimulator {

    private int highScore;
    // ...

    public void simulate() {
        int numberOfLives = 3;
        List<Ball> ballsInPlay = new ArrayList<>();

        new Thread(new Runnable() {
            public void run() {
                ballsInPlay.add(new Ball());
            }
        }).start();
    }
}
```

- Though `ballsInPlay` is local in `simulate`, it's not thread confined

- And since `ArrayList` is not thread safe, the piece of code is not thread safe

    

###### Confinement & Annoymous Classes (2 / 5)

In the example above, we modify the object the variable points to, but not reassign it

```java
public class PinballSimulator {

    private int highScore;
    // ...

    public void simulate() {
        int numberOfLives = 3;
        List<Ball> ballsInPlay = new ArrayList<>();

        new Thread(new Runnable() {
            public void run() {
                numberOfLives = 1; // static error
            }
        }).start();
    }
}
```

- If we reassign a variable in the new thread, we'd get a static error:
  
  - Local variable defined in an enclosing scope must be final or effectively final

- That is, variables from the outer scope are only accessible if: 
  
  - They are *never reassigned*

- Neither of the following cases are allowed:
  
  - Reassigning outside locals in the `Runnable`
  
  - Reading them in the `Runnable` when they are reassigned

    

###### Confinement & Annoymous Classes (3 / 5)

- Can read variables in inner `Runnable`, even though they're not declared `final`

- But to keep the code easy to understand, always declare local variables as `final` 
  
  - That you intend to share with an inner class

    

###### Confinement & Annoymous Classes (4 / 5)

What happens if we try to reassign `highScore`(an instance variable) in `Runnable`?

```java
public class PinballSimulator {

    private int highScore;
    // ...

    public void simulate() {
        int numberOfLives = 3;
        List<Ball> ballsInPlay = new ArrayList<>();

        new Thread(new Runnable() {
            public void run() {
                highScore += 1000;
            }
        }).start();
    }
}
```

Unlike local variables, fields are neither confined nor unreassignable by default

    

###### Confinement & Annoymous Classes (5 / 5)

In a system where you create and start new threads: 

- Sharing fields and local variables with inner `Runnable` instances is common

- Need to use one of the upcoming strategies to ensure that: 
  
  - Sharing those non-confined references and objects is safe

    

###### Strategy #2: Immutability

The 2nd way: use unreassignable references & immutable data types

- Immutability tackles the shared-mutable-state cause of a race condition

- A type is thread safe it it has:
  
  - No mutator methods
  
  - All fields declared `private` & `final`
  
  - No representation exposure

    

###### Strategy #3: Using Thread-Safe Data Types

The 3rd way: stored shared mutable data in existing thread-safe data types

- When a type in is threadsafe, its documentation will explicitly state that:
  
  - Check [`StringBuffer`](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/lang/StringBuffer.html) & [`StringBuilder`](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/lang/StringBuilder.html) for detail

- It's become common in the Java API to find two mutable data types: 
  
  - That do the same thing, one threadsafe and the other not
  
  - Reason: threadsafe data types usually incur performance penalty 
    
    - Compared to an unsafe type

    

###### Thread-Safe Collections (1 / 7)

- The collection interfaces in Java (`List`, `Set`, `Map`), have basic implementations 
  
  - That are **not** threadsafe. e.g., `ArrayList`, `HashMap`, `TreeSet`, etc

- Fortunately, java provides another set of wrapper methods 
  
  - To make collections threadsafe, while still mutable

- These wrappers effectively make each method of the collection *atomic* 
  
  - With respect to the other methods

    

###### Thread-Safe Collections (2 / 7)

An *atomic* action effectively happens all at once

- It doesn’t interleave its internal operations with those of other actions

- And none of the effects of the action are visible to other threads 
  
  - Until the entire action is complete, so it never looks partially done

    

###### Thread-Safe Collections (3 / 7)

Now we see a way to fix the `isPrime` method we had earlier:

```java
private static Map<Integer,Boolean> cache =
                Collections.synchronizedMap(new HashMap<>());
```

    

###### Thread-Safe Collections (4 / 7)

A few points here:

1. *Don't circumvent the wrapper*. Throw away references to the underlying: 
   
   - Non-thread-safe collection, and access only through the schronized wrapper

2. *Iterators are still not thread-safe*. Even though method calls on the collection itself: 
   
   - (`get()`, `put()`, `add()`, etc.) are now threadsafe
   
   - Iterators created from the collection are still not threadsafe
   
   - So you can’t use `iterator()`, or the for loop syntax:
     
     ```java
     List<String> list = ...;
     for (String s: list) { ... }
     // not threadsafe, even if list is a synchronized list wrapper
     ```
   
   - The solution to this iteration problem will be to acquire the collection’s lock:
     
     - When you need to iterate over it, which we’ll talk about in a future reading

    

###### Thread-Safe Collections (5 / 7)

3. *Atomic operations aren't enough to prevent races*
   
   - Even though you use the synchronized collection, you may still have races
   
   - Consider this code, which checks whether a list has any element & gets that
     
     ```java
     if ( ! list.isEmpty()) { String s = list.get(0); ... }
     ```
   
   - Even if you make `list` into a synchronized one, the code still has races, since:
     
     - Another thread may call `remove` between the `isEmpty()` & `get()` call

    

###### Thread-Safe Collections (6 / 7)

So the `isPrime` method has potential races, let's dig in:

```java
if (cache.containsKey(x)) return cache.get(x);
boolean answer = BigInteger.valueOf(x).isProbablePrime(100);
cache.put(x, answer);
```

1. Say `containsKey(x)` returns true, another thread mutates `cache` before `get(x)`
   
   - Not harmful because we never remove items from the cache
   
   - Once it contains a result for `x`, it will continue to do so

2. Say `containsKey(x)` returns false, another thread mutates the cache before `put` 
   
   - Both of them should call `put(x, answer)` with the same value for `answer` 
   
   - So it doesn’t matter which one wins the race – the result will be the same

    

###### Thread-Safe Collections (7 / 7)

- In `isPrime` example, the code is thread-safe after we research it

- However, we can tell the concurrency is hard from the process, because of:
  
  - The need to make these kinds of careful arguments about safety: 
  
  - Even when you’re using threadsafe data types

    

###### Wrapper Implementations

- Wrapper implementations delegate all their real work to a specified collection
  
  - But add extra functionality on top of what this collection offers

- These implementations are annoymous. Rather than providing a public class:
  
  - The library provides a static factory method

    

###### Synchronization Wrappers (1 / 3)

These wrappers add automatic synchronization (*thread-safe*) to any collection (6)

<img src="file:///C:/Users/yangs/screenshots/synchronized.PNG" title="" alt="" width="522">

        

###### Synchronization Wrappers (2 / 3)

- When iterating over a collection, the user must manually synchronize it

- Because iteration is accomplished via multiple calls into the collection, which:
  
  - Cannot be composed into a single atomic call

- The following is the idiom to iterate over a wrapper-synchronized collection:
  
  ```java
  Collection<Type> c = Collections.synchronizedCollection(myCollection);
  synchronized(c) {
      for (Type e : c)
          foo(e);
  }
  ```

    

###### Synchronization Wrappers (3 / 3)

- When iterating over any collection views of a synchronized `Map`, synchronize on:
  
  - The `Map` itself, instead of those collection views

- Here is an example to show the operation:
  
  ```java
  Map<KeyType, ValType> m = 
          Collections.synchronizedMap(new HashMap<KeyType, ValType>());
      ...
  Set<KeyType> s = m.keySet();
      ...
  // Synchronizing on m, not s!
  synchronized(m) {
      while (KeyType k : s)
          foo(k);
  }
  ```

    

###### Unmodifiable Wrappers (1 / 2)

- Unlike synchronization wrappers, which add functionality to the collection
  
  - The unmodifiable wrappers take functionality away

- They take away the ability to modify the collection by: 
  
  - Intercepting all the operations that would modify the collection 
  
  - And throwing an `UnsupportedOperationException`

    

###### Unmodifiable Wrappers (2 / 2)

Each of the six core `Collection` interfaces has one static factory method:

<img src="file:///C:/Users/yangs/screenshots/unmodifiable.PNG" title="" alt="" width="635">

    

###### Concurrent Modification Exception

Iterators make an effort to warn you when you’re using them unsafely

```java
List<String> subjects = new ArrayList<>(List.of("6.045", "6.031"));
for (String subject : subjects) {
    if (subject.startsWith("6.")) {
        subjects.remove(subject);
    }
}
```

- The code above throws a `ConcurrentModificationException` because: 

- The iterator, implicitly in the `for` loop, detects that the list it is iterating over: 
  
  - Has had an element removed from it

- Q: What does *concurrent* refer to in the name of this exception?
  
  A: calls to iterator are interleaved with calls to `remove()` in the same thread

    

###### How to Make A Safety Argument

- If you want to convince people that your concurrent program is correct
  
  - Make an explicit argument that it’s free from races & write it down

- A safety argument needs to catalog all the threads that exist in your program
  
  - And the data that they use, and argue which of the 4 techniques you use: 
  
  - To protect against races for each data object or variable: 
    
    - Confinement, immutability, threadsafe data types, or synchronization

- When you use the last two, also need to argue all accesses to the data: 
  
  - Are appropriately atomic. We gave an argument for `isPrime` above

    

###### Thread Safety Arguments for Data Types (1 / 2)

Let’s see examples of how to make thread safety arguments for a data type

- Remember our 4 approach. We won't use *confinement*, since you have to know:
  
  - What threads exist in the system & what objects they've been given access to

- If the data type creates its own set of threads, you can talk about confinement:
  
  - With respect to those threads

- Otherwise, the threads are comming in from the outside, carrying clent calls
  
  - And the data type may have no guarantees about: 
  
  - Which threads have references to what

- Usually we use confinement at a higher level, talking about the system as a whole 
  
  - And arguing why we don’t need thread safety for some of our modules
  
  - Because they won’t be shared across threads by design

    

###### Thread Safety Arguments for Data Types (2 / 2)

- So we’ll focus on arguments that appeal to immutability and threadsafe data types

- Have to avoid rep exposure & document argument for safety from rep exposure 
  
  - Rep exposure is bad for any type, since it threatens the type’s rep invariant
  
  - It’s similarly fatal to thread safety

    

###### Good Safety Arguments

![](C:\Users\yangs\screenshots\mystring.PNG)

    

###### Bad Safety Arguments (1 / 2)

<img src="file:///C:/Users/yangs/screenshots/graph.PNG" title="" alt="" width="461"> 

`Graph` relies on synchronized set & map to help it implement its rep 

- Which prevents some race conditions, but not all

    

###### Bad Safety Arguments (2 / 2)

There might be code like this:

```java
public void addEdge(Node from, Node to) {
    if ( ! edges.containsKey(from)) {
        edges.put(from, Collections.synchronizedSet(new HashSet<>()));
    }
    edges.get(from).add(to);
    nodes.add(from);
    nodes.add(to);
}
```

- Thread `A` finds no such a key in `edges`, so it enters the `if` block

- At the same time, thread `B` also finds no key in `edges`, and keeps executing
  
  - Put key in `edges`, add node to the value set that the key maps to

- Now `A` goes back, and put a new set mapped to the key in `edges`
  
  - The previous key-value pair is overwritten. Race condition!

    

###### Serializability

What we demand from a threadsafe data type is that: 

- When clients call its atomic operations concurrently, the results are consistent: 
  
  - With *some* sequential ordering of the calls

- Say 2 atomic calls `A` & `B`. The result should either be `A; B` or `B; A`
  
  - May interleave in actuality but result should be either 1 out of 2
  
  - This property is called *serializability* (used a lot in database theory)

    
