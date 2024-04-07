###### Objectives

- Understand test-first programming
- Able to judge a test suite for correctness, thoroughness, and size
- Able to design a test suite by partitioning its input space & choosing good cases
- Able to judge a test suite by measuring its code coverage
- Understand black box vs. glass box testing, unit tests vs. integration tests
- Understand automated regression testing

    

###### Systematic Testing

Systematic testing means designing a test suite with 3 desirable properties:

- Correct: accepting all legal implementations of the spec without complaint

- Thorough: could find actual bugs as many as possible

- Small: faster to write & run, easier to update 

    

###### Choosing Test Cases by Partitioning

- We want to pick a small set of test cases that can be easy to write & maintain 
  
  - Yet thorough enough to find bugs in the program

- To do it, we divide input space into *subdomains*, each consisting of a set of inputs
  
  - Taken together, the subdomains form a *partition*: 
  
  - A collection of disjoint sets that completely covers the input space
  
  - We choose one test case from each subdomain, and that’s our test suite

    

###### Include Boundaries in the Partition

- We incorporate boundaries as single-element subdomains in the partition

- Here's an example of how to partition subdomains with boundary cases:
  
  ```java
  /**
   * @param wsAndLs  a string of length at most 5 consisting of 'W'/'L'
   * @return wsAndLs contains 'W'
   */
  double winLossRatio(String wsAndLs);
  // Q: What are appropriate boundary values for testing this function?
  ```
  
  - Answer: `""`, `"WWWWW"`, `LLLLL`
  
  - Partition: 0-len, 0-5 with `W`, 0-5 without `W`, 5 with `W`, 5 without `W`

    

###### Example: Multiplying Big Integers (1 / 3)

[`BigInteger`](http://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/math/BigInteger.html) is a class that can represent integers of any size, with `multiply` method

```java
/**
 * @param val another BigInteger
 * @return a BigInteger whose value is (this * val).
 */
public BigInteger multiply(BigInteger val)
```

- We divide `BigInteger` into 6 subdomains: 0, 1, small `+` , small `-`, big `+`, big `-`

- Given the fact we have 2 args (including `this`), we'd have 6 x 6 = 36 subdomains
  
  <img src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-17-17-51-17-image.png" title="" alt="" width="221">

    

###### Example: Multiplying Big Integers (2 / 3)

The above cartesian product produces a massive input space. Here's an alternative:

- Ignore combination of partitions. Guarantee cases'd cover each single partition

- With this approach, we only need 6 cases
  
  <img src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-17-17-57-26-image.png" title="" alt="" width="241">

    

###### Example: Multiplying Big Integers (3 / 3)

With this approach, we're actually viewing each partition separately

- In this case, 12 subdomains in total. But 6 can cover all of them

- However, this raises the risk you no longer testing interaction between partitions

- Can add an additional partition that captures this interaction, which is the sign:
  
  - `a > 0, b > 0;` `a < 0 & b < 0;` `a > 0, b < 0;` `a < 0, b > 0;` `0` 

- Now we have 3 partitions, with 6, 6, 5 subdomains each, instead of 6 x 6 x 5 cases:
  
  - A test suite with 6 carefully-chosen test cases can cover all of them

    

###### Documenting Your Testing Strategy

- Good if you write down the testing strategy of the test suite: 
  
  - The partitions, their subdomains & which subdomains each test case covers 
  
  - This makes the thoroughness of your test suite more visible to the reader

- Could document the partitions & subdomains at the top of the testing class
  
  ```java
  public class MaxTest {
    /*
     * Testing strategy
     *
     * partition:
     *    a < b
     *    a > b
     *    a = b
     */
  ```

- Each case should have a comment above saying which subdomain it covers:
  
  ```java
  // covers a < b
  @Test
  public void testALessThanB() {
      assertEquals(2, Math.max(1, 2));
  }
  ```

    

###### Black-box & Glass-box Testing

- Black-box testing means choosing test cases only from the specification
  
  - Not the implementation of the function

- Glass-box testing means choosing test cases with knowledge of: 
  
  - How the function is actually implemented
  
  - e.g., if the implementation keeps an internal cache that remembers: 
    
    - The answers to previous inputs, you should test repeated inputs

    

###### Coverage

- Coverage judges how thorough a test suite is. Here are 3 types of coverage:
  
  - Statement coverage: is every statement is executed?
  
  - Branch coverage: is every direction of `if` or `while` is taken?
  
  - Path coverage: is every combination of branches (i.e., *path*) taken?

- Statement coverage is a common goal, but 100% of it sometimes is hard
  
  - e.g., some *you should never be here* assertions
  
  - Branch coverage is a desirable goal, but path coverage is infeasible

    

###### Unit & Integration Testing

We've so far been talking about unit tests, which tests a single module

- In contrast, an integration test tests a combination of modules

- It's important since a program can fail at the connections between modules

    

###### Automated Regression Testing (1 / 2)

Automated testing means running the tests & checking their results automatically

- The code that runs tests on a module is a *test driver* (a.k.a, *test harness*, *test runner*)

- It should not prompt you for inputs & print out results for you to manually check
  
  - Instead, it'd invoke the module itself on fixed test cases & automatically check
  
  - The result should be either *all tests OK* or *these tests failed: …*

    

###### Automated Regression Testing (2 / 2)

Regressing: bugs introduced when you fix new bugs or add new features

- Running the tests frequently as you change the code prevents *regressing*
  
  - Running all your tests after every change is called *regression testing*

- The idea can also lead to *test-first debugging*. When a bug arises: 
  
  - Immediately write a test case and add it to your suite
  
  - Once you find & fix that bug, you solve the problem & expand the test suite

    

###### Iterative Test-first Programming (1 / 4)

Here we'll revisit & refine the test-first programming idea 

- In reality, effective software engineering does not follow a linear process of:
  
  - Spec => test => implement

- Rather, Practice *iterative test-first programming*, in which you are prepared: 
  
  - To go back and revise your work in previous steps

    

###### Iterative Test-first Programming (2 / 4)

Steps of *iterative test-first programming*:

- Write a spec for the module

- Write tests that exercies the spec. Iterate on spec & tests if you find issues

- Write an implementation. Iterate on spec, test & implementation if any issues

    

###### Iterative Test-first Programming (3 / 4)

Why this iterative way: each step helps to validate the previous steps

- e.g., writing tests is good for understanding the specification

- The specification can be incorrect, incomplete, ambiguous, or missing corner cases

- Trying to write tests can uncover problems early, before you’ve wasted time: 
  
  - Working on an implementation of a buggy spec

    

###### Iterative Test-first Programming (4 / 4)

Further, no need to make one step perfect before moving on to the next step. Instead:

- For a large spec, start with 1 part of, proceed to test & implement that part 
  
  - Then iterate with a more complete spec

- For a complex test suite, start with a few important partitions & a small test suite 
  
  - Proceed with implementation that pass, then iterate with more partitions

- For a tricky implementation, first write a simple brute-force implementation 
  
  - Then move on to the harder one with confidence of your spec & tests

      

  
