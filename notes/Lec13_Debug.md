###### Objectives

- Sometimes a bug is found only when you plug the whole system together 
  
  - Or reported by a user after the system is deployed

- In cases above, it might be hard to localize the bug to a particular module
  
  - We can suggest a systematic strategy for more effective debugging

    

###### Reproduce the Bug (1 / 2)

- Tricky case: bug in graphical user interface or multithreaded programs
  
  - Hard to reproduce the bug since it may depend on timing

- Example of a piece of code that's intuitively hard to bug
  
  ```java
  /** 
   * @param: words concatenated as a whole string
   * @return: the word that occurs most often in the string
  */
  public static String mostCommonWord(String text) { ... }
  ```

- Say a user passes the whole text of Shakespeare's plays into this method
  
  - Instead of returning `the` or `a`, it returns `e`, obviously unexpected

    

###### Reproduce the Bug (2 / 2)

- Shakespeare's plays have over 800,000 words, hard to debug by breakpoints

- It'd be easier if you first work on reducing size of the input to a manageable scale
  
  - But still exhibits the same (or very simialr) bug

- e.g., does the 1st half show the same bug (binary search, always a good technique)
  
  - Once you’ve found a small test case, find and fix the bug using that case
  
  - Then go back to the original buggy input & see if it's solved

    

###### Find the Bug Using the Scientific Method

[4 steps]([Reading 13: Debugging](https://web.mit.edu/6.031/www/sp21/classes/13-debugging/#find_the_bug_using_the_scientific_method) in the scientific method: *study the data*, *hypothesize*, *experiment*, *repeat*

- If 10-minute causal debugging makes no progress, should turn to scientific way

- Write down each iteration of the process rather than keep it all in your mind

    

###### Study the Data

One important form of data is the stack trace from an exception

- Should practice reading the stack traces you get, since they'll offer information:
  
  - About where & what the bug might be

- In a Java stack trace, the latest call is on top, and the oldest call in on the bottom
  
  - But those top/bottom calls might be library code that's not written by you

- Don't let them dissuade you, scan through the stack trace until seeing sth familiar
  
  - And then find it in your code

    

###### Hypothesize (1 / 2)

- The point where program fail, by throwing exception or producing wrong answer
  
  - May not be where the bug is

- The buggy code may have propagated some bad values through good parts 
  
  - Before it eventually failed

- It'd help to think about your program as steps in an algorithm, and try to rule out:
  
  - Some whole sections of the program at once

    

###### Hypothesize (2 / 2)

Let's think about this in the context of the `mostCommonWord` example:

```java
public static String mostCommonWord(String text) {
    List<String> words = splitIntoWords(text);
    Map<String,Integer> frequencies = countOccurrences(words);
    String winner = findMostFrequent(frequencies);
    return winner;
}
```

    

###### Slicing (1 / 6)

What we try to do above is an example of *slicing*, which means:

- Finding the parts of a program that contributed to computing a certain value

- When you have a failure (i.e., a bad value at a particular point in the program) 
  
  - The *slice* consists of the lines of the program that computed that value
  
  - The bug lies somewhere in that slice, so that’s your search space

    

###### Slicing (2 / 6)

Suppose `x` should never be negative. A debugging print reports it's gone bad:

```java
int x = 0; // must be >= 0
...
System.out.println("x=" + x);  // prints a negative number
```

    

###### Slicing (3 / 6)

What is the slice for this bad value? Let’s dig into the `...` in the code above

- Lines that directly change the value of `x` are part of the slice:
  
  ```java
  int x = 0; // must be >= 0
  ...
      x += bonus;
  ...
  System.out.println("x=" + x);  // prints a negative number
  ```

- The value of `bonus` contributes to the value of `x`, so its slice is included:
  
  ```java
  int x = 0; // must be >= 0
  final int bonus = getBonus();
  ...
      x += bonus;
  ...
  System.out.println("x=" + x);  // prints a negative number
  ```

- The function `getBonus()` is included in the slice. Let's consider control statements
  
  ```java
  int x = 0;
  final int bonus = getBonus();
  ...
    if (isWorthABonus(s)) {
      x += bonus;
    }
  ...
  System.out.println("x=" + x);  // prints a negative number
  ```

    

###### Slicing (4 / 6)

- This `for` included since it’s involved with `s`, & affects # of times slice executed
  
  ```java
  int x = 0;
  final int bonus = getBonus();
  ...
  for (final Sale s : salesList) {
    ...
    if (isWorthABonus(s)) {
      x += bonus;
    }
    ...
  }
  ...
  System.out.println("x=" + x);  // prints a negative number
  ```

- Now `saleslist` should also be included in the slice. Have to find where it's from
  
  ```java
  int calculateTotalBonus(final List<Sale> salesList) {
    ...
    int x = 0;
    final int bonus = getBonus();
    ...
    for (final Sale s : salesList) {
      ...
      if (isWorthABonus(s)) {
        x += bonus;
      }
      ...
    }
    ...
    System.out.println("x=" + x);  // prints a negative number
    ...
  }
  ```

    

###### Slicing (5 / 6)

Can dig further but let's stop here and do some analysis:

- From `x+=bonus`: maybe `bonus` is negative, so `x+=bonus` goes negative, which:
  
  - Further imply that `getBonus()` returned a negative value for `bonus`

- From `if`: maybe `isWorthABonus()` returns true for too many sales, so 
  
  - The sum accumulated in `x` overflows the size of an `int` and goes negative

- From `for`: maybe the entire loop is executing over too many sales, again 
  
  - Making `x` overflow from all the bonuses

- From signature: maybe a bad value is being passed in for `salesList` 
  
  - With far too many sales on it, which later caused overflow

    

###### Slicing (6 / 6)

Note design choices can affect the efficiency of, reasoning by slicing (e.g., immutability)

- Considering `final int bonus = getBonus()`, no need to check further for `bonus`

- When we saw `final Sale s`, though, we didn’t have quite the same confidence
  
  - `s` is unassignable, but if `Sale` is a mutable type, its value can still change

- Another design choice that helps slicing is *scope minimization*. With local variables:
  
  - Only need to check code within the method. With instance variables:
  
  - Have to check lines within the whole class. For a global variable (gasp): 
    
    - The search expands to the entire program

    

###### Delta Debugging

Say 2 closely-related test cases *bracket* the bug, i.e., one succeeds & one fails

- e.g., `mostCommonWords("c c, b")` is broken & `mostCommonWords("c c b")` is fine

- Can examine the diff between the execution of the 2 to help form your hypothesis
  
  - Which code is executed for the passing but not for the failing one? Vice versa?

- One hypothesis is that the bug lies in those lines of code, the *delta*: 
  
  - Between the passing run and the failing run

- Another kind of delta debugging is useful when a regression test starts failing
  
  - Using version control, retrieve the most recent code that still passed the test
  
  - Then explore the code-changes between the older & the newer failing version
    
    - Until you find the change that introduced the bug

    

###### Prioritize Hypotheses

Remember that different parts have different likelihoods of failure

- e.g., old, well-tested code is probably more trustworthy than recently-added code

- Java library code is probably more trustworthy than yours

- The Java compiler & runtime, OS platform, and hardware are: 
  
  - Increasingly more trustworthy, because they are more tried and tested
  
  - You should trust these lower levels until you’ve found good reason not to

    

###### Experiment

After hypothesizing, we should experiment to test the prediction

    

###### Swap Components

- If you hypothesize that the bug is in a module, and you happen to have: 
  
  - A different implementation of it that satisfies the same interface
  
  - Then one experiment you can do is to try swapping in the alternative

- e.g., If you suspect your `binarySearch()` implementation, then substitute: 
  
  - A simpler `linearSearch()` instead

    

###### Advice

- *One bug at a time*: keep a bug list & don’t get distracted from the current bug

- *Don't fix yet*: don't try to fix the bug when you experiment it

    

###### 
