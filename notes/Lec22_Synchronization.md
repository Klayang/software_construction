###### Objectives

- Understand how a lock is used to protect shared mutable data

- Be able to recognize deadlock and know strategies to prevent it

- Know the monitor pattern & be able to apply it to a data type

    

###### Introduction (1 / 2)

- *Thread safety*: a data type behaves correctly when used in multiple threads
  
  - Regardless of how those threads are executed

- General principle: the correctness of a concurrent program should not depend on:
  
  - Any accidents of timing

    

###### Introduction (2 / 2)

To achieve that, we enumerated 4 strategies for making code safe for concurrency

1. *Confinement*: don’t share data between threads. Keep it accessible from 1 thread

2. *Immutability*: make shared data immutable, by using final & immutable types

3. Using existing thread-safe data types: use those that already do coordination

4. *Synchronization*: prevent threads from accessing the shared data at the same time
   
   - This is what we used to implement a thread-safe type

    

###### Synchronization (1 / 3)

*Lock*s are 1 type of synchronization technique

- A lock is an abstraction that allows at most 1 thread to own it at a time

- *Holding a lock* is how one thread informs other threads: 
  
  - “I’m working with this thing, don’t touch it right now.”

    

###### Synchronization (2 / 3)

Locks have 2 operations:

- **`acquire`** allows a thread to take ownership of a lock. If a thread tries to: 
  
  - Acquire a lock currently owned by another thread, it *blocks* until: 
    
    - The other thread releases the lock
  
  - At that point, it'll contend with any other threads that're also trying to acquire
    
    - At most one thread can own the lock at a time

- **`release`** relinquishes ownership of the lock, allowing another thread:
  
  - To take ownership of it

    

###### Synchronization (3 / 3)

The lock also tells compiler & processor that you use shared memory concurrently

- So that registers & caches will be written back correctly to shared storage

- This avoids the problem of [reordering](https://web.mit.edu/6.031/www/sp21/classes/20-concurrency/#reordering), ensuring that the owner of a lock: 
  
  - Is always looking at up-to-date data

    

###### Bank Account Example

The bank has several cash machines, all of which can read and write the same object

- Without any coordination between concurrent reads & writes, [things went wrong](https://web.mit.edu/6.031/www/sp21/classes/20-concurrency/#interleaving)

- We can add a lock that protects each account. Before they can access an account: 
  
  - Cash machines must first acquire the lock on that account

    

###### Lock

Locking is a *convention* — a protocol for good behavior with a shared object

- All participating threads with access to the same shared memory object: 

- Have to carefully acquire and release the appropriate lock

- If a badly-written client fails to acquire or release the right lock: 
  
  - Then the system isn’t actually threadsafe

    

###### Deadlock (1 / 2)

However, sometimes problems show up when using with locks

- Because locks require threads to wait, it's possible to get to a situation where:
  
  - Two threads are waiting for each other, thus neither make progress

- *Deadlock*s occur when concurrent modules're stuck waiting for each other to do sth
  
  - May involve more than 2 modules, e.g., `A` waiting for `B`, `B` for `C`, `C` for `A`
  
  - The essential feature of a *deadlock* is a *cycle of dependencies* like this

    

###### Deadlock (2 / 2)

Can also have *deadlock*s without using any locks

- A Message-passing system can experience *deadlock*s when message buffers fill up

- If a client fills up the server's buffer with requests, and **block**s: 
  
  - Wating to add another request

- The server may fill up the client's buffer with results & **block** itself: 
  
  - At the same time

- So the client is waiting for the server, and the server waiting for the client
  
  - And neither can make progress

    

###### Developing A Threadsafe ADT (1 / 4)

Say we develope a multi-user editor, like Google Docs, allowing shared edition 

- Need a mutable type to represent text in the doc: a string with `insert` & `delete`

- We'll define interface `EditBuffer` as the text rep, with different implementations
  
  ```java
  /** 
   * An EditBuffer represents a threadsafe mutable
   * string of characters in a text editor.
   */
  public interface EditBuffer {
      /**
       * Modifies this by inserting a string.
       * @param position position to insert at 
       *      (requires 0 <= position <= current buffer length)
       * @param insertion string to insert
       */
      public void insert(int position, String insertion);
  
      /**
       * Modifies this by deleting a substring
       * @param position, starting position of substring to delete 
       *      (requires 0 <= position <= current buffer length)
       * @param len, length of substring to delete
       *      (requires 0 <= len <= current buffer length - position)
       */
      public void delete(int position, int len);
  
      /**
       * @return length of text sequence in this edit buffer
       */
      public int length();
  
      /**
       * @return content of this edit buffer
       */
      public String toString();
  }
  ```

    

###### Developing A Threadsafe ADT (2 / 4)

A very simple rep for the datatype would just be a `String`

```java
public class SimpleBuffer implements EditBuffer {
    private String text;
    // Rep invariant:
    //   true
    // Abstraction function: 
    //   represents the sequence text[0],...,text[text.length()-1]
```

The downside is that every `insert` or `delete` requires copy of the entire string

    

###### Developing A Threadsafe ADT (3 / 4)

A more interesting rep & used by many text editors is called a *gap buffer*

- It's a `char` array with extra space, which can show up anywhere in the buffer

- Whenever an `insert` or `delete` needs to be done, the datatype first moves:
  
  - The gap to the specified location, then does the `insert` or `delete`

- If gap is already there, nothing needs to be copied. An `insert` consumes the gap
  
  - And a `delete` enlarges the gap

- *Gap buffer*s are well-suited for string edited by user with a cursor, since: 
  
  - `Insert`s & `delete`s tend to be focused around the cursor
  
  - Thus ,the gap rarely moves after the 1st one

    

###### Developing A Threadsafe ADT (4 / 4)

```java
/** 
 * GapBuffer is a non-threadsafe EditBuffer that is optimized
 * for editing with a cursor, which tends to make a sequence of
 * inserts and deletes at the same place in the buffer.
 */
public class GapBuffer implements EditBuffer {
    private char[] a;
    private int gapStart;
    private int gapLength;
    // Rep invariant:
    //   0 <= gapStart <= a.length
    //   0 <= gapLength <= a.length - gapStart
    // Abstraction function: 
    //   represents the sequence a[0],...,a[gapStart-1],
    //                           a[gapStart+gapLength],...,a[a.length-1]
```

    

###### Steps to Develop A Datatype (1 / 2)

Recall our [recipe for designing & implementing an *ADT*](https://web.mit.edu/6.031/www/sp21/classes/19-programming-with-adts/#recipes_for_programming):

1. **Specify**. Define the operations (method signatures & specs, as in an interface)

2. **Test**. Develop test cases for the operations

3. **Rep.** Choose a rep. We chose 2 for `EditBuffer`, and it's often good to:
   
   - Implement a simple, brute-force rep first
   
   - Write down *rep invariant* & *abstraction function*. Implement `checkRep`

    

###### Steps to Develop A Datatype (2 / 2)

4. **Synchronize.** Make an argument that your rep is threadsafe
   
   - Write it down as a comment in your class, right by the rep invariant
   
   - So that a maintainer knows how you designed thread safety into the class

5. **Iterate**. Your choice of operations may make it hard to write a threadsafe type 
   
   - You might discover this in different steps. If that’s the case: 
   
   - Go back and refine the set of operations your *ADT* provides

    

###### Locking (1 / 2)

*Lock*s are so commonly used thata java provides it as a built-in language feature

- In java, every object has a lock implicitly associated with it - a `String`, array, or:
  
  - Even a humble object

- Thus, bare `Object`s are often used for explicit locking:
  
  ```java
  Object lock = new Object();
  ```

- Can't call `acquire` & `release` on java's intrinsic locks. Instead, use`synchronized`
  
  ```java
  synchronized (lock) { // thread blocks here until lock is free
      // now this thread has the lock
      balance = balance + 1;
  } // exiting the block releases the lock
  ```

    

###### Locking (2 / 2)

`Synchronized` regions like the above provide *mutual exclusion*

- only one thread at a time can be in that region guarded by a given object’s lock

- Back in sequential programming world, with only one thread running at a time
  
  - With respect to other `synchronized` regions referring to the same object

    

###### Locks Guard Access to Data (1 / 3)

- Locks are used to guard a shared data variable, like the account balance

- If all accesses to a data variable are guarded (surrounded by a `synchronized`): 
  
  - By the same lock object, then those accesses will be guaranteed: 
  
  - To be *atomic*, i.e., uninterrupted by other threads

    

###### Locks Guard Access to Data (2 / 3)

- Locks only provide *mutual exclusion* with other threads that acquire the same lock

- Given every object in java has an implicitly associated lock, you might think:
  
  - Owning that lock automatically prevents other threads accessing it

- That's not the case. Say a thread `t` acquires an object's lock with 
  
  ```java
  synchronized(obj) {...}
  ```
  
  - It does 1 thing and that thing only - preventing other threads from:
  
  - Entering their own `synchronized(obj)` block, until `t` finishes the block

    

###### Locks Guard Access to Data (3 / 3)

Even while `t` is in its `synchronized` block:

- Another thread can dangerously mutate `obj`, simply by neglecting to use `syn()`

- In order to use an object lock for synchronization, have to explicitly & carefully 
  
  - Guard every such access with a `synchronized` block or method keyword

    

###### Monitor Pattern (1 / 5)

- The most convenient lock is the instance itself, i.e., `this`. As a simple approach:

- Can guard the entire rep by wrapping all accesses inside `synchronized(this)`

    

###### Monitor Pattern (2 / 5)

```java
/** SimpleBuffer is a threadsafe EditBuffer with a simple rep. */
public class SimpleBuffer implements EditBuffer {
    private String text;
    ...
    public SimpleBuffer() {
        synchronized (this) {
            text = "";
            checkRep();
        }
    }
    public void insert(int position, String insertion) {
        synchronized (this) {
            text = text.substring(0, position) + insertion +
                   text.substring(position);
            checkRep();
        }
    }
    public void delete(int position, int len) {
        synchronized (this) {
            text = text.substring(0, position) + 
                   text.substring(position+len);
            checkRep();
        }
    }
    public int length() {
        synchronized (this) {
            return text.length();
        }
    }
    public String toString() {
        synchronized (this) {
            return text;
        }
    }
}
```

    

###### Monitor Pattern (3 / 5)

This approach is called *the monitor pattern*

- A *monitor* is a class whose methods are mutually exclusive, so that only 1 thread
  
  - Can be inside an instance of the class at a time

- Syntactic sugar: If you add the keyword `synchronized` to a method signature
  
  - Java'll act as if you wrote `synchronized (this)` around the body
  
  - Here's an equal but simplified way to implement `synchronized` buffer

    

###### Monitor Pattern (4 / 5)

```java
/** SimpleBuffer is a threadsafe EditBuffer with a simple rep. */
public class SimpleBuffer implements EditBuffer {
    private String t;
    ...
    public SimpleBuffer() {
        t = "";
        checkRep();
    }
    public synchronized void insert(int position, String insertion) {
        t = t.substring(0, position) + insertion + t.substring(position);
        checkRep();
    }
    public synchronized void delete(int position, int len) {
        t = t.substring(0, position) + t.substring(position+len);
        checkRep();
    }
    public synchronized int length() {
        return t.length();
    }
    public synchronized String toString() {
        return t;
    }
}
```

    

###### Monitor Pattern (5 / 5)

Constructors doesn’t have a `synchronized` keyword. Java forbids it, syntactically. Why? 

- As we say `synchronized`, we refer to only 1 thread accessing an object at a time

- When multiple constructors are being called, they're constructing their own objects
  
  - So there's no concern that different threads accessing the same object

    

###### Thread Safety Argument with Synchronization (1 / 2)

Since we protect `SimpleBuffer`'s rep with a lock, can write a thread safety argument

```java
/** SimpleBuffer is a threadsafe EditBuffer with a simple rep. */
public class SimpleBuffer implements EditBuffer {
    private String text;
    // Rep invariant: 
    //   true
    // Abstraction function: 
    //   represents the sequence text[0],...,text[text.length()-1]
    // Safety from rep exposure:
    //   text is private and immutable
    // Thread safety argument:
    //   all accesses to text happen within SimpleBuffer methods,
    //   which are all guarded by SimpleBuffer's lock
```

    

###### Thread Safety Argument with Synchronization (2 / 2)

Note that the encapsulation of the class (no rep exposure), matters for the argument

```java
public String text;
```

- If `text` is `public` like the above, clients outside `SimpleBuffer` can access it:

- Without first acquiring the lock, thus making `SimpleBuffer` no longer safe

    

###### Locking Discipline

A locking discipline ensures that synchronized code is threadsafe, satisfying 2 rules:

- All shared mutable variable must be guarded by locks. Every access to the variable
  
  - Must be inside a synchronized block, that acquires the lock

- If an invariant involves multiple shared mutable variables (may in different objects)
  
  - Then all the variables involved must be guarded by the *same* lock

    

###### Atomic Operations

```java
/** 
 * Modifies b by replacing the first occurrence of p with r
 * If p is not found in b, then has no effect.
 * @return true if and only if a replacement was made
 */
public static boolean findReplace(EditBuffer b, String p, String r) {
    int i = b.toString().indexOf(p);
    if (i == -1) return false;
    b.delete(i, p.length());
    b.insert(i, r);
    return true;
}
```

- The function made 3 *atomic* calls `b`, but itself as a whole isn't threadsafe 

- Because other threads might mutate `b` while `findReplace` is working
  
  - Causing it to delete the wrong region or put `r` in the wrong place

    

###### Giving Clients Access to A Lock (1 / )

To prevent the case above, `findReplace` needs to synchronize the use of `b`

- Useful to make your datatype's lock available to clients, so that they can use it:
  
  - To implement higher-level *atomic* operations with your datatype

- One approach to the problem with `findReplacement` is to document that:
  
  - Clients can use the `EditBuffer`'s lock to synchronize with each other

    

###### Giving Clients Access to A Lock (2 / )

The following code first declared the buffer's lock can be grabbed, then grab it

```java
/**
 * An EditBuffer represents a threadsafe mutable string of characters
 * in a text editor. Clients may synchronize with each other using the
 * EditBuffer object itself. 
 */
public interface EditBuffer {...}
```

```java
public static boolean findReplace(EditBuffer b, String p, String r) {
    synchronized (buf) {
        int i = b.toString().indexOf(p);
        if (i == -1) return false;
        b.delete(i, p.length());
        b.insert(i, r);
        return true;
    }
}
```

    

###### Sprinkling Synchronized Everywhere? (1 / 3)

Is thread safety simply putting `synchronized` in every method? Unfortunately not

- 1st, no need, since it costs a lot on performance
  
  - Java leaves many of its mutable datatypes unsynchronized for this issue
  
  - Therefore, when you don't need synchronization, don't use it

- 2nd, not enough. Dropping `synchronized` onto a method without thinking:
  
  - Might have you acquire the incorrect lock for guarding the shared data

    

###### Sprinkling Synchronized Everywhere? (2 / 3)

Suppose we had dropped `synchronized` onto the declaration of `findPlace`

```java
public static synchronized boolean findReplace(EditBuffer b, ...) {
```

- This'd acquire a static lock for the whole class that `findReplace` happens to be in
  
  - Rather than an instance object lock

- As a result only 1 thread could call `findReplace` at a time - even if other threads:
  
  - Want to operate on different buffers, which is safe, they'll still get blocked

- So we’d suffer much loss in performance, because only 1 user of the whole editor 
  
  - Can do a *find-and-replace* at a time, even if they’re all editing different docs

    

###### Sprinkling Synchronized Everywhere? (3 / 3)

The `synchronized` keyword is not a panacea. Thread safety requires a discipline: 

- Using confinement, immutability, or locks to protect shared data

- And that discipline needs to be written down, or maintainers won’t know

    

###### Designing A Datatype for Concurrency (1 / 4)

`findReplace`'s problem can be interpreted another way:

- The `EditBuffer` interface isn't that friendly to multiple simultaneous clients

- It relies on `int` indexes to specify `insert` & `delete` locations, which are:
  
  - Extremely brittle to other mutations

- If somebody else inserts or deletes before the index position, then: 
  
  - The index becomes invalid

    

###### Designing A Datatype for Concurrency (2 / 4)

So if we’re designing a datatype specifically for use in a concurrent system

- We need to think about providing operations that: 

- Have better-defined semantics when they're interleaved

    

###### Designing A Datatype for Concurrency (3 / 4)

e.g., better to pair `EditBuffer` with a `Position` datatype, representing:

- A cursor in the buffer, or even a `Selection` representing a range

- Once obtained, a `Position` can hold its location in the text against: 
  
  - The wash of `insert` & `delete`

- If some other thread deleted all the text around the `Position`, the `Position`
  
  - Can inform a subsequent client about it (with an exception)
  
  - And allow the client to decide what to do

    

###### Designing A Datatype for Concurrency (4 / 4)

As another example, consider the `ConcurrentMap` interface in java

- It extends the existing `Map` interface, adding a few methods commonly needed:
  
  - As *atomic* operations on a shared mutable map

- These methods and their corresponding unsynchronized operations are as follows:
  
  ```java
  map.putIfAbsent(key,value); 
  // is an atomic version of
  if (!map.containsKey(key)) map.put(key, value);
  ```
  
  ```java
  map.replace(key, value); 
  // is an atomic version of
  if (map.containsKey(key)) map.put(key, value);
  ```
