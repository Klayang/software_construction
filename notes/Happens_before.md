Q1: why do we have the 2 rules in *happens-before*?

###### Overview

- *Happens-before* is a concept, a phenomenon, or simply a set of rules that:
  
  - Define the basis for reordering of instructions by a compiler or CPU

- It's not any keyword or object in Java, but a discipline put into place so that:
  
  - In a multi-threading environment, instructions that might be reordered: 
  
  - Does not result in a code that produces incorrect result

    

###### Java Memory Model (1 / 2)

Let's first learn where the need for *happens-before* originates from

- The *Java Memory Model* (`JMM`) defines how the storage & exchange of data
  
  - Take place between threads & hardware with single or multiple threads

- Some points to keep in mind as follows:
  
  - Every CPU core has its own set of registers 
  
  - Every CPU can execute more than 1 thread at a time
  
  - Every CPU has its own set of cache
  
  - A thread executes on a CPU core but its data is stored on `RAM` 
    
    - Where the local variables lie inside the *Thread Stack* 
    
    - And the objects lie inside the *Heap*

    

###### Java Momery Model (2 / 2)

<img src="https://media.geeksforgeeks.org/wp-content/uploads/20210628164332/jmm.png" title="" alt="Lightbox" width="351">

    

###### Possible Issue Caused by JMM

While working with multiple threads that share a variable:

- When one thread updates a shared variable’s value, the update has to be done: 
  
  - To the register, then cache, and finally the `RAM`

- When another thread requires to read that shared variable, It may read the value 
  
  - From the `RAM` where the correct value might have not been stored

- This is known as *update visibility*, and can be solved by use of `synchrnized` block
  
  - And volatile variables

    

###### Instruction Reordering (1 / 2)

- During compilation or runtime, the compiler or CPU may reorder the instructions:
  
  - To run them in parallel to increase the throughput & performance

- e.g., we have these 3 instructions:
  
  ```java
  FullName = FirstName + LastName;      // Statement 1
  UniqueId = FullName + TokenNo;        // Statement 2
  
  Age = CurrentYear - BirthYear;        // Statement 3
  ```
  
  - Cannot run `1` & `2` in parallel since `2` needs the output of `1` 
  
  - But `1` & `3` can run parallel since they're independent of each other

- Thus, the instructions could be reordered in the following way:
  
  ```java
  FullName = FirstName + LastName;      // Statement 1
  Age = CurrentYear - BirthYear;       // Statement 3
  
  UniqueId = FullName + TokenNo;       // Statement 2
  ```

    

###### Instruction Reordering (2 / 2)

However, if reordering is performed in a multi-threaded application where:

- Threads share some variables, it may cost us the correctness of the program
  
  - It could potentially be *race condition* or *update invisibility* as we mentioned

- Java provides us with some solutions, so we'll learn what they are
  
  - And finally *happens-before* will be introduced there

    

###### Volatile (1 / 7)

For a variable declared as `volatile`:

```java
private volatile count;
```

- Every write to the field will be written/flushed directly to `RAM` (bypassing cache)
  
  - Every read of that field is read directly from the `RAM`

- This means shared variable `count` always corresponds to its most recent value
  
  - Which'd prevent *update invisibility* since now the threads: 
  
  - Will always use the correct value of a shared variable

    

###### Volatile (2 / 7)

There are some more important points that the `volatile` dictates:

- At the time you write to a `volatile` variable, all the non-`volatile` variables
  
  - That are visible to that thread will also get written/flushed to the `RAM`
  
  - i.e., their most recent values will be stored in the `RAM` with `volatile` ones

- At the time you read a `volatile` variable, all the non-`volatile` variables 
  
  - That are visible to that thread will also get refreshed from the `RAM`
  
  - i.e. their most recent values will be assigned to them

    

###### Volatile (3 / 7)

What's above is called the *visibility guarantee* of a `volatile` variable

- All of this looks fine, unless the CPU decides to reorder your instructions

- Let's understand this by consider the following program, basically is:
  
  - Inputs a fresh assignment submitted by a student & collects it

    

###### Volatile (4 / 7)

```java
class ClassRoom {
    // Declaring and initializing variables
    private int numOfAssignSubmitted = 0;
    private int numOfAssignCollected = 0;
    private Assignment assgn = null;
    // Volatile shared variable
    private volatile boolean newAssignment = false;
    // 2 methods
}
```

```java
// Method 1, used by Thread 1
public void submitAssignment(Assignment assign) {
    this.assign = assign; // 1
    this.numOfAssgnSubmitted++; // 2
    this.newAssignment = true; // 3
}
```

```java
// Method 2, used by Thread 2
public Assignment collectAssignment() {
    while (!newAssignment) {// Wait until a new assignment is submitted}
    Assignment collectedAssgn = this.assign;
    this.numOfAssignCollected++;
    this.newAssignment = false;
    return collectedAssgn;
}
```

    

###### Volatile (5 / 7)

- The volatile variable `newAssignment` is shared between thread `1` & `2`
  
  - Which are running concurrently

- Since all other variables are visible to thread `1` & `2` alongwith `newAssignment`
  
  - The read-write operations will be done directly using the `RAM`

    

###### Volatile (6 / 7)

If we focus on the `submitAssignment` method:

- Statement `1`, `2`, `3` are independent of each other

- Hence your CPU might think, "*why not reorder them*?"

- So let us assume the CPU reordered the statements in this way:
  
  ```java
  this.newAssignment = true; // 3
  this.assign = assign;   // 1
  this.numOfAssgnSubmitted++; // 2
  ```

    

###### Volatile (7 / 7)

Our goal: collect a new fresh assignment after it's been set

- Since statement `3` is moved ahead, `while` loop in thread `2` will now be exited
  
  - Possible that Thread `2`’s instructions execute before remaining of Thread `1`
  
  - This would result in an older value of `assign` being submitted

- Even though the values are being retrieved directly from the `RAM`, it doesn't work
  
  - If the instructions are executed in the wrong order, as above
  
  - Thus, *happens-before* guarantee is introduced, concerning `volatile` variable

    

###### Happens-Before in Volatile (1 / 2)

*Happens-before* states about reordering as follows:

- When reordering a *write* that's before another *write* to a `volatile`, should remain:
  
  - Before that *write* after reordering

- When reordering a *read* of a `volatile`, gurantee it's before any subsequent *read*
  
  - No matter if the subsequent *read*s are `volatile` or not

    

###### Happens-Before in Volatile (2 / 2)

In the context of the example above, the 1st point is relevant

- Any write to a variable (Statements `1` & 2) that happened: 
  
  - Before a write to a `volatile` (Statement `3`), should remain before `3`

- Thus reordering of Statement `3` before `1` & `2` is prohibited, Guaranteeing: 
  
  - The `true` is only set after the `assign` binds to the new value

- This is called **happens-before visibility guarantee of volatile**
  
  - Statements `1` & `2` may be reordered among themselves 
  
  - As long as they are not being reordered after statement `3`

    

###### Synchronization Block

In the case of a synchronization block in java:

- When a thread enters a synchronization block, the thread will refresh the values
  
  - Of all variables that are visible to the thread, from the `RAM`

- When a thread exits a synchronization block, the values of all those variables
  
  - Will be written to the `RAM`

    

###### Happens-Before in Synchronization Block

In the case of a synchronization block, *happens-before* states:

- Any *write* to a variable that happens before the exit of a synchronization block 
  
  - Is guaranteed to remain before the exit of the block

- Entrance to a synchronization block that happens before a *read* of a variable
  
  - Will remain before any *read*s to the variables following that entrance

    

# 
