###### Behavoral Equivalence (1 / 2)

- Say you are working on a method, which finds the index of an integer in an array
  
  ```java
  static int find(int[] arr, int val) {
      for (int i = 0; i < arr.length; i++)
          if (arr[i] == val) return i;
      return -1;
  }
  ```

- If that integer shows up at the end of the array, our search could be pretty slow
  
  ```java
  static int find(int[] arr, int val) {
      for (int i = 0, j = arr.length-1; i <= j; i++, j--) {
          if (arr[i] == val) return i;
          if (arr[j] == val) return j;
      }
      return -1;
  }
  ```
  
  - Thus we can do the search from both ends of the array

    

###### Behavoral Equivalence (2 / 2)

Q: Is it safe to replace `find` with this new solution? (any possibility to introduce bugs?)

- These 2 case can have different behavior. If `val` appears *more than once* 
  
  - The original `find` always returns the lowest index at which it occurs
  
  - The new `find` might return the lowest index or the highest index

- But when `val` occurs at exactly one index of the array:
  
  - The 2 implementations behave the same: they both return that index

- Thus, the notion of behavioral equivalence depends on the client
  
  - We need a specification that states exactly what the client exactly wants

- A possible specification that allow these two `find` to be *behaviorally equivalent*
  
  <img src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-14-17-05-30-image.png" title="" alt="" width="401">

    

###### Why Specifications?

- Though every coder has specifications in mind, not every one write them down 
  
  - As a result, different coders on a team have *different* specifications in mind

- Precise specifications let you apportion blame (not to people but code fragments!)
  
  - And spare you the agony of puzzling over where a bug fix should go

- Having a specification lets a client understand what the module does 
  
  - Without having to read the module’s code

- Specifications are good for the implementer since they give the implementer: 
  
  - Freedom to change the implementation without telling clients

- Specifications result in *decoupling*, that is would allow: 
  
  - The code of the module and the code of a client to be changed independently 
  
  - So long as the changes respect the specification — each obeying its obligation

    

###### Specification Structure (1 / 4)

Abstractly speaking, a *specification* of a method has several parts:

- Method *signature*, giving name, para types, return type, and exceptions thrown

- *Requires* clause, giving additional restrictions on the parameters

- *Effects* clause, giving return value, exceptions, & other effects of the method

    

###### Specification Structure: Precondition

Taken together, these parts form the *precondition* and the *postcondition* of the method

- Precondition's an obligation on client (caller), the state when the method's invoked

- One aspect of the precondition is # and types of the parameters in the signature 

- Additional conditions are written down in the *requires* clause, for example
  
  - Narrowing a parameter type (e.g. `x >= 0` on an `int x`)
  
  - Interactions between parameters (e.g., `val` occurs exactly once in `arr`)

    

###### Specification Structure (3 / 4)

The postcondition is an obligation on the implementer of the method

- Includes what Java can statically check: return type & declared checked exceptions

- Additional conditions are written down in the *effects* clause, including:
  
  - How the return value relates to the inputs
  
  - Which exceptions are thrown, and when
  
  - Whether and how objects are mutated

    

###### Specification Structure (4 / 4)

- If the precondition holds when the method is called, *then* the postcondition: 
  
  - Must hold when the method completes.

- If precondition doesn't hold, the implementation isn't bound by postcondition
  
  - It is free to do anything, including never returning, throwing an exception, etc 

    

###### Specifications in Java

Java has a convention for documentation comments called Javadoc, in which:

- Paras are described by `@param` clauses & res are described by `@return` clauses

- Should put preconditions into `@param` where possible, & post ones into `@return`
  
  ```java
  /**
   * Find a value in an array.
   * @param arr array to search, requires that val occurs exactly once
   * @param val value to search for
   * @return index i such that arr[i] = val
   */
  static int find(int[] arr, int val)
  ```

    

###### Do not Allow Null References

Null values are an unfortunate hole in Java's type system

- You can’t call any methods or use any fields with one of null references
  
  ```java
  String name = null;
  name.length();   // throws NullPointerException  
  ```

- General convention: **null values are disallowed in parameters & return values** 
  
  - Unless the spec explicitly says otherwise

- Extensions to Java that allow you to forbid `null` directly in the type declaration
  
  ```java
  static boolean addAll(@NonNull List<T> list1, @NonNull List<T> list2)
  ```

- Still sometimes a need for a para or return value to indicate a value is missing
  
  - Can use `Optional<T>`, which either contains one `T`, or is empty
  
  - Can clearly expresses that the para or return value could be null

    

###### Testing & Specification

- Test cases (both black & glass tests) must follow the specification

- Unit testing: write tests of each module of our program in isolation
  
  - A good unit test is focused on just a single specification

- Integration testing: testing various modules to see if they function together
  
  - Making sure that different methods have compatible specifications

    

###### Specifications for Mutating Methods

Two specifications as follows (a mutable and an immutable)

```java
static void sort(List<String> list)
requires: nothing
effects: puts list in sorted order
```

```java
static List<String> toLowerCase(List<String> list)
requires: nothing
effects: returns a new list t, same length as list, 
         where t[i] = list[i].toLowerCase() for all valid indices i
```

- Just as `null` is implicitly disallowed unless stated otherwise, coders assume: 
  
  - **Mutation is disallowed unless stated otherwise**

- The spec of `to­Lower­Case` could state as an *effect* that *list is not modified*
  
  - But without describing mutation, we demand no mutation of the inputs

    

###### Exceptions

- Exceptions like `Index­OutOfBounds­Exception` generally indicate **bugs** in your code 
  
  - The info displayed when the exception's thrown can help you find & fix the bug

- Exceptions improve the structure of code that involves funcs with special results
  
  - As you expect an index, you get `-1`, indicating the value doesn't exist in array
  
  - Shortage: tedious to check, easy to forget, not always a good special value

- Example when it's hard to find a special value
  
  ```java
  class BirthdayBook {
      LocalDate lookup(String name) { ... }
  }
  ```

    

###### Using Exception in Place of Special Values

- The function above could be written like this:
  
  ```java
  LocalDate lookup(String name) throws NotFoundException {
      ...
      if ( ...not found... ) throw new NotFoundException();
      ...
  }
  ```

- And the caller of the function may write the following code:
  
  ```java
  BirthdayBook birthdays = ...
  try {
      LocalDate birthdate = birthdays.lookup("Alyssa");
      // we know Alyssa's birthday
  } catch (NotFoundException nfe) {
      // her birthday was not in the birthday book
  }
  ```

- No need for any special value, nor the checking associated with it

    

###### Checked & Unchecked Exceptions

- We’ve seen 2 purposes for exceptions: special results & bug detection
  
  - You’ll want to use checked exceptions to signal special results
  
  - And unchecked exceptions to signal bugs

- Rules concerning *checked exceptions*:
  
  - If a method may throw any, the possibility must be declared in its signature
    
    - `Not­Found­Exception`'d be a checked exception, so signature ends with that
  
  - If a method calls another method that may throw a checked exception:
    
    - It must either handle it, or declare the exception itself
    
    - Since if it isn’t caught locally it will be propagated up to callers

- Unchecked exceptions: no need to explicitly handle by `try-catch` clause
  
  - Or declaration of throwing it in signature

    

###### Exception Message

All exceptions may have a message associated with them

- If not provided in the constructor, the reference to the message string is `null`

- This can result in confusing stack traces that start, for example:
  
  ```shell
  birthday.NotFoundException: null
  at birthday.BirthdayBook.lookup(BirthdayBook.java:42)
  ```

- The `null` is misleading: it tells you the `NotFoundException` had no message 
  
  - Not that a `null` value was responsible for the exception

    

###### Exception Hierarchy (1 / 2)

<img src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-15-17-47-25-image.png" title="" alt="" width="387">

    

###### Exception Hierarchy (2 / 2)

- What do *checked* & *unchecked* mean? *Checked* means error can&should be handled
  
  - *Unchecked* means either the error is not serious enough to explicitly indicate 
  
  - Or it's too serious to indicate, i.e., unrecoverable, must be covered by java

- When you define your own exceptions, you should: 
  
  - Either subclass `Exception` (checked) or `RuntimeException` (unchecked)
  
  - Don’t subclass `Error`, as those are reserved by Java itself

- Catch the most specific exception class possible when you do `try-catch`
  
  - Doing that generally catches every possible subclass of these exceptions
  
  - Which may interfere with static checking and hide bugs

- What's confusing: `RuntimeException` is itself a subclass of `Exception` 
  
  - So the `Exception` family includes both checked & unchecked exceptions 
  
  - But `Error` is *not* a subclass of `Exception`, so all `Error`-like exceptions 
    
    - Are outside `Exception` family (can't be caught by clause with `Exception`)

    

###### How to declare exceptions in a specification

- The Java way to document an exception is a `@throws` clause 
  
  - Java may also use a `throws` declaration in the method signature

- **Exceptions that signal a special result** are always noted with a `@throws` clause
  
  - Specifying the conditions under which that special result occurs

- For unchecked exception, java allows but doesn't require the `throws` declaration
  
  - Better to omit that in the signature, to avoid misleading a programmer

- **Exceptions that signal unexpected failures** shouldn't appear: 
  
  - In either `@throws` or `throws`
  
  - They arebugs in either the client or the implementation, thus are not: 
    
    - Part of the postcondition of a method

    
