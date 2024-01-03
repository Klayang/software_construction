###### Objectives

- 2 concurrency models: *message passing* & *shared memory*

- Concurrent processes & threads, and time slicing

- The danger of *race condition*s

    

###### Concurrency (1 / 3)

This means multiple computations happening at the same time, and it's everywhere:

- Multiple computers in a network

- Multiple applications running on one computer

- Multiple processors in a computer (today, often multiple cores on a chip)

    

###### Concurrency (2 / 3)

Actually *concurrency* is essential in modern programming:

- Websites must handle multiple simultaneous users

- Mobile apps need to do some of their processing on servers (*in the cloud*)

- GUIs always require background work that does not interrupt the user
  
  - e.g., IDEA compiles your Java code while you’re still editing it

    

###### Concurrency (3 / 3)

Processor clock speeds are no longer increasing (Moore Law expired)

- Instead, we’re getting more cores with each new generation of chips

- In the future, in order to get a computation to run faster, we’ll have to: 
  
  - Split up a computation into concurrent pieces

    

###### 2 Models for Concurrent Programming (1 / 2)

2 common models. Here comes the 1st one: *shared memory*

- Concurrent modules interact by reading & writing shared objects in memory

- `A` & `B` are concurrent, blues are private to `A`/`B` (but oranges are shared)
  
  <img title="" src="file:///C:/Users/yangs/Downloads/shared_memory.PNG" alt="" width="317"> 

- Examples of this shared-memory model (that could explain the figure above):
  
  - `A` & `B` are 2 processors in the same pc, sharing the same physical memory
  
  - `A` & `B` are 2 programs running on the same computer, sharing a filesystem
    
    - With files they can read and write
  
  - `A` & `B` are 2 threads in the same Java program sharing the same Java objects

    

###### 2 Models for Concurrent Programming (2 / 2)

- *Message passing*: modules send messages to each other through a channel
  
  <img src="file:///C:/Users/yangs/Downloads/message_passing.PNG" title="" alt="" width="200">

- Modules send off messages, and incoming messages to each module: 
  
  - Are queued up for handling

- Examples of this message-passing model:
  
  - `A` & `B` are 2 pcs in a network, communicating by network connections
  
  - `A` & `B` are a web browser and a server, `A` asks `B` for a web page 
    
    - Then `B` sends the web page data back to `A`
  
  - `A` & `B` might be an instant messaging client and server
  
  - `A` & `B` might be two programs running on the same pc: 
    
    - Whose input & output have been connected by a [pipe](https://en.wikipedia.org/wiki/Anonymous_pipe)(e.g.,`ls | grep`)

    

###### Processes, Threads, Time-Slicing (1 / 7)

The concurrent modules come in 2 different kinds: processes & threads

- **Process**: an instance of a running program, isolated from other processes

- A process has its own private section of the machine's memory

- The process abstraction is a *virtual computer*. It makes the program feel like:
  
  - It has entire machine to itself, with fresh memory, just to run that app

    

###### Processes, Threads, Time-Slicing (2 / 7)

- Just like computers connected across a network, processes normally: 
  
  - Share no memory between them

- Sharing memory between processes is *possible* on most OSs
  
  - But requires special effort

    

###### Processes, Threads, Time-Slicing (3 / 7)

- By contrast, a new process is automatically ready for message passing
  
  - Since it's created with standard input & output streams, which are:
  
  - The `System.out` and `System.in` streams you’ve used in Java

- whenever you start any program on your pc – it starts a fresh process: 
  
  - To contain the running program

    

###### Processes, Threads, Time-Slicing (4 / 7)

**Thread**: a locus of control inside a running program

- Think of it as a place in the program that's being run, plus the stack of method calls
  
  - Which leads to that place (so the thread can go up the stack when to `return`)

- As a process represents a virtual computer, the thread abstraction represents: 
  
  - A *virtual processor* (i.e., CPU, that is, *ALU* + registers)

- Making a new thread simulates making a fresh processor inside the virtual pc:
  
  - Represented by the processor

- This new virtual processor runs the same program & shares the same memory: 
  
  - As other threads in the process

    

###### Processes, Threads, Time-Slicing (5 / 7)

- Threads are automatically ready for shared memory, because threads share: 
  
  - All the memory in the process. It takes special effort: 
  
  - To get *thread-local* memory that’s private to a single thread

- It’s also necessary to set up message-passing explicitly, by: 
  
  - Creating & using queue data structures

- Whenever you run a Java program, the program (process) starts with one thread
  
  - Which calls `main()` as its 1st step. This thread is referred to as the *main thread*

    

###### Processes, Threads, Time-Slicing (6 / 7)

How can you have many concurrent threads with only 1 processors on your pc?

- When threads are more than processors, concurrency is simulated by **time slicing**
  
  - Which means the processor switches between threads

- The left figure shows how `T1`, `T2` & `T3` might be time-sliced on 2 processors:
  
  <img src="file:///C:/Users/yangs/Downloads/time_slicing.PNG" title="" alt="" width="274">

-  The right figure shows how this looks from each thread’s point of view
  
  - Sometimes a thread is actively running on a processor, and sometimes: 
  
  - It is suspended waiting for its next chance to run on some processor

    

###### Processes, Threads, Time-Slicing (7 / 7)

- Time slicing happens unpredictably and nondeterministically

- Meaning that a thread may be paused or resumed at any time

    

###### Starting A Thread in Java (1 / 2)

- Can start new threads by making an instance of `Thread` and telling it to `start()`

- Should provide code for the new thread to run. The code should be in a class that:
  
  - Implements `Runnable`. The new thread will call `run()` method in that class

- Here the main thread will print sth, then call another thread to print sth else:
  
  ```java
  public class HelloRunnable implements Runnable {
      @Override
      public void run() {
          System.out.println("hello from thread created by the main");
      }
  }
  ```
  
  ```java
  public class Main {
      public static void main(String[] args) {
          System.out.println("Hello from the main thread");
          Thread thread = new Thread(new HelloRunnable());
          thread.start();
      }
  }
  ```
  
  ![](C:\Users\yangs\screenshots\hello_world_concurrency.PNG)

    

###### Starting A Thread in Java (2 / 2)

A common idiom to do, with an annoymous `Runnable`

```java
main() {
    new Thread(new Runnable() {
        public void run() {
            System.out.println("Hello from a thread!");
        }
    }).start();
}
```

    

###### Annoymous Class

- Usually when we implement an interface, we do so by declaring (creating) a class

- An annoymous class declares an unnamed class that implements an interface 
  
  - And immediately creates the one any only instance of that class

- Here's an example. The annoymous class here implements `Comparator`
  
  ```java
  SortedSet<String> strings = new TreeSet<>(new Comparator<String>() {
      @Override public int compare(String s1, String s2) {
          if (s1.length() == s2.length()) return s1.compareTo(s2);
          return s1.length() - s2.length();
      }
  });
  strings.addAll(List.of("yolanda", "zach", "alice", "bob"));
  // strings is { "bob", "zach", "alice", "yolanda" }
  ```

    

###### Using an anonymous Runnable to start a thread

Take a look at the annoymous version to start a thread. Can do this with lambda expr:

```java
new Thread(() => System.out.println("Hello")).start();
```

- A lambda expression can be used to: 
  
  - Implement a functional interface in an anonymous way
  
  - At the same time, creating an instance of the interface 

- Lambda expressions can be used only with interfaces: 
  
  - That declare a single method

    

###### Threads & Exceptions (1 / 2)

- Suppose we run this JUnit test:
  
  ```java
  @Test public void testOops() { throw new Error("oops"); }
  ```
  
  - Is a stack trace for `Error: oops` printed? Yes!
  
  - Will the test fail? Yes! The default behavior for JUnit tests – if exception thrown 
    
    - And not caught by the test code itself, then the test fails

- Now suppose we run this test:
  
  ```java
  @Test public void testThreadOops() {
      new Thread(() -> { throw new Error("thread oops"); }).start();
  }
  ```
  
  - Is a stack trace for `Error: thread oops` printed? Yes!
  
  - Will the test fail? No! The exception is thrown on the newly-created thread
    
    - But the test method `testThreadOops` returns normally

    

###### Threads & Exceptions (2 / 2)

When a JUnit test creates new threads, we should have it wait for them: 

- To run and communicate back their success or failure somehow
  
  - So that the test can pass or fail accordingly

- Various ways to do that, like `join()`, or threadsafe mutable data types
  
  - Or socket connections, which we’ll encounter in future readings

    

###### Shared Memory Example (1 / 5)

A bank has cash machines. All can read & write the same account objects in memory  

<img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2023-10-26-14-20-09-image.png" title="" alt="" width="427">

    

###### Shared Memory Example (2 / 5)

let’s simplify the bank down to a single account: 

```java
public class Acount {
    // suppose all the cash machines share a single bank account
    private static int balance = 0;
    // what customers could do
    public static void deposit() { balance = balance + 1; }
    public static void withdraw() { balance = balance - 1; }
}
```

- With a dollar balance stored in the `balance` variable

- And 2 operations `deposit` & `withdraw` that add or remove a dollar

    

###### Shared Memory Example (3 / 5)

- Each transaction can only be: a 1-dollar deposit followed by a 1-dollar withdrawal
  
  - So it should leave the balance in the account unchanged

- Throughout the day, each cash machine in our network is processing: 
  
  - A sequence of `deposit`/`withdraw` transactions

    

###### Shared Memory Example (4 / 5)

Each time we start a transaction, we're actually just calling the following method:

```java
public static void cashMachine() {
    new Thread(new Runnable(){
        public void run() { transaction(); }
        private void transaction() {
            for (int i = 0; i < TRANSACTIONS_PER_MACHINE; ++i) {
                Account.deposit();
                Account.withdrawl();
            }
        }
    }).start();
}
```

    

###### Shared Memory Example (5 / 5)

- No matter how many cash machines (processors) were running
  
  - Or how many Xacts we processed, should expect `balance` to still be 0

- But if we run this code, we discover frequently that the `balance` is *not* 0: 
  
  - If more than 1 thread created by `cashMachine()` is running at the same time

    

###### Interleaving (1 / 4)

Here's one thing that can happen

- Suppose 2 threads (`A` & `B`), are both working on a `deposit` at the same time

- Although 1 statement, can break down into 3 low-level processor instructions:
  
  <img src="file:///C:/Users/yangs/screenshots/3_step.PNG" title="" alt="" width="220">

    

###### Interleaving (2 / 4)

- When `A` & `B` run concurrently, low-level instructions *interleave* with each other
  
  - i.e., the sequence of execution can intersperse `A`’s & `B`’s code arbitrarily

- Here's one possible interleaving:
  
  <img src="file:///C:/Users/yangs/screenshots/interleaving1.PNG" title="" alt="" width="409">
  
  - This one is fine. After 2 `deposit`s, the `balance` is set to 2

    

###### Interleaving (3 / 4)

What about this situation below?

<img src="file:///C:/Users/yangs/screenshots/interleaving2.PNG" title="" alt="" width="403">

- The balance is now 1 – A’s dollar was lost! 

- `A` & `B` both read the `balance`, computed separate final `balance`s 
  
  - And then **raced** to store back the new balance 
  
  - which failed to take the other’s deposit into account

    

###### Interleaving (4 / 4)

The question that bothered me when I first came across this scenario is that:

- Why do we get `balance` first? Why don't we just add 1 to it?

- The thing is that `balance` is a value stored in *RAM*. To modify it:
  
  - First have to fetch it to register in CPU
  
  - Then operate ALU, at last store it back to the *RAM*

    

###### Race Condition

The case above is an example of a *race condition* 

- It means the correctness of the program depends on relative timing of events:
  
  - In concurrent computations within `A` & `B`

- Some interleaving of events may be ok, since they are consistent with:
  
  - What a single, nonconcurrent process would produce
  
  - But some others can produce wrong answers

    

###### Tweaking the Code Won't help (1 / 2)

All these versions of the bank-account code exhibit the same race condition:

```java
// version 1
private static void deposit()  { balance = balance + 1; }
private static void withdraw() { balance = balance - 1; }

// version 2
private static void deposit()  { balance += 1; }
private static void withdraw() { balance -= 1; }

// version 3
private static void deposit()  { ++balance; }
private static void withdraw() { --balance; }
```

    

###### Tweaking the Code Won't help (2 / 2)

- Can’t tell how the processor is going to execute, just from looking at Java code
  
  - can’t tell what the atomic operations will be
  
  - It isn’t atomic just because it’s one line of Java 
  
  - Doesn’t touch `balance` only once just because it occurs only once in the line

- Java compiler, and in fact the processor itself, makes no commitments: 
  
  - About what low-level operations it will generate from your code. 
  
  - In fact, a typical modern Java compiler produces exactly the same code: 
    
    - For all three of these versions

    

###### Reordering (1 / 4)

What's stated above is bad enough. However, it's even worse than that

- What we just said: the race condition can be explained in terms of
  
  - Different interleavings of sequential operations

- But in fact, when you’re using multiple variables & multiple processors: 
  
  - Can’t count on changes to those variables appearing in the same order

    

###### Reordering (2 / 4)

Here's an example:

```java
// shared memory
private boolean ready = false;
private int answer = 0;
```

```java
// computeAnswer runs in one thread
private void computeAnswer() {
    // ... calculate for a long time ...
    answer = 42;
    ready = true;
}
```

```java
// useAnswer runs in a different thread
private void useAnswer() {
    // busy-wait for computeAnswer to say it's done
    while (!ready) {
        Thread.yield();
    }
    if (answer == 0) throw new RuntimeException("answer wasn't ready!");
}
```

    

###### Reordering (3 / 4)

The `yield()` basically means that: 

- The thread is not doing anything particularly important

- If any other threads or processes need to be run, they should run

- Otherwise, the current thread will continue to run

    

###### Reordering (4 / 4)

The code above seems to be correct (though it's *busy waiting*), but actually no:

- Because modern compilers & processors do a lot to make the code fast

- `answer` & `ready` are fetched from memory, modified, then stored back
  
  - The order in which one is stored back first is actually uncertain

- Here’s what might be going on under the covers (expressed in Java syntax)
  
  ```java
  private void computeAnswer() {
      // ... calculate for a long time ...
  
      boolean tmpr = ready;
      int tmpa = answer;
  
      tmpa = 42;
      tmpr = true;
  
      ready = tmpr;
      // <-- what happens if useAnswer() interleaves here? Exception!
      // ready is set, but answer isn't.
      answer = tmpa;
  }
  ```

    

###### New Threads and Termination

JUnit calls `System.exit()` when all the tests are done. Consider:

```java
@Test public void testSlowOops() {
    new Thread(() -> {
        // ...
        // really long complicated time-consuming computation here
        // ...
        throw new Error("slow oops");
    }).start();
}
```

- Is a stack trace for `Error: slow oops` printed? No! It's already terminated

- Does the test fail? No! The error is thrown (though never) in a different thread

- `System.exit`: terminating all threads that are currently being running

    

###### Message Passing Example (1 / 4)

Now let’s look at the *message-passing* approach to our bank account example

- Note now both each cash machine & each account is a module

- Modules interact by sending messages to each other. Incoming requests: 
  
  - Are placed in a queue to be handled one at a time

    

###### Message Passing Example (2 / 4)

<img src="file:///C:/Users/yangs/screenshots/message_passing.PNG" title="" alt="" width="496">

    

###### Message Passing Example (3 / 4)

Unfortunately, *message passing* doesn't eliminate the possibility of race conditions

- Say each account supports `get-balance` & `withdraw` operations

- 2 users, at cash machines `A` & `B`, try to withdraw from the same account

- They check `balance` first to make sure overdraft never happens:
  
  ```java
  get-balance: if balance >= 1 then withdraw 1
  ```

    

###### Message Passing Example (4 / 4)

The problem is again interleaving

<img src="file:///C:/Users/yangs/screenshots/message_interleaving.PNG" title="" alt="" width="255">

- But this time interleaving of the *messages* sent to the bank account
  
  - Rather than the *instructions* executed by `A` & `B`

- If you have different messages that try to do one thing together:
  
  - Then they are likely to mess things up with concurrency
  
  - That's the reason we define transactions in database theory

- One lesson is to carefully choose operations of a message-passing model
  
  - `withdraw-if-sufficient-funds` is better than just `withdraw`

    

###### Concurrency is Hard to Test & Debug (1 / 3)

Very hard to discover & reproduce race conditions using testing

- These bugs are called [*heisenbugs*](https://en.wikipedia.org/wiki/Heisenbug), other deterministics are called *bohrbug*

- A heisenbug may even disappear when you try to `println` or use a debugger!
  
  - Since printing & debugging are so much slower than other operations
  
  - That they dramatically change the timing & the interleaving of operations

    

###### Concurrency is Hard to Test & Debug (2 / 3)

Inserting a simple print statement into the `cashMachine()`:

```java
public static void cashMachine() {
    new Thread(new Runnable() {
        public void run() { 
            for (int i = 0; i < TRANSACTIONS_PER_MACHINE; ++i) {
                deposit(); // put a dollar in
                withdraw(); // take it back out
                System.out.println(balance); // makes the bug disappear!
            }
        }
    }).start();
}
```

- Suddenly the balance is always 0, as desired, and the bug appears to disappear

- But it’s only masked, not truly fixed. A change in timing somewhere else: 
  
  - May suddenly make the bug come back

    

###### Concurrency is Hard to Test & Debug (3 / 3)

- Concurrency is hard to get right. Part of this reading is to scare you a bit

- Over the next several readings, we’ll see principled ways to design
  
  - Concurrent programs so that they are safer from these kinds of bug

    
