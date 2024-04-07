###### First Defense: Make Bugs Impossible

- *Immutability*: guaranteed by type. e.g., `String` is an immutable type

- *Unreassignable references*: guaranteed by `final` keyword

    

###### Second Defense: Localize Bugs

```java
public double sqrt(double x) { 
    if (! (x >= 0)) throw new IllegalArgumentException("required x >= 0,
                    but was: " + x);
    ...
}
```

    

###### Assertions

- An assertion may include a descriptive message, which can be string or other types
  
  ```java
  assert x >= 0 : "x is " + x;
  ```
  
  - If `x == -1`, then this assertion fails with the error message: `x is -1`

- Asserions are off by default. You can enable assertions by:
  
  - Going to `Run -> Edit Configurations -> Add VM options -> -ea` 

- Can ensure the assertions are enabled by the following test:
  
  ```java
  @Test
  public void testAssertionsEnabled() {
      assertThrows(AssertionError.class, () -> { assert false; });
  }
  ```

- Note that JUnit *assert* methods are different from java *assert* statements
  
  - The former one can run without `-ea` option

    

###### What not to Assert (1 / 2)

Never use assertions to test conditions that are external to your program

- e.g., existence of files, availability of the network, the correctness of user input

- Why: assertions are used to ensure your program doesn't have bugs, not others

- External failures should be handled using exceptions, like `FileNotFoundException` 

    

###### What not to Assert (2 / 2)

Assertions should never have side effects on your program

- if you want to assert an element to be removed from a list was in the list, don’t do:
  
  ```java
  assert list.remove(x);
  ```
  
  - If assertions are unabled, the element would not be removed

- Good to block illegal cases if conditional statements don't cover all possible cases
  
  ```java
  switch (vowel) {
      case 'a':
      case 'e':
      case 'i':
      case 'o':
      case 'u': return "A";
      default: throw new AssertionError("must be a vowel, but was: " 
                                         + vowel);
  }
  ```
  
  - But don’t use the assertions here, since it can be turned off. Use eceptions
