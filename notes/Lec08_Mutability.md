###### Objectives

- Understand mutability and mutable objects

- Identify aliasing and understand the dangers of mutability

- Use immutability to improve correctness, clarity, & changeability

###### Mutability

- When *mutability* matters: when there is a 2nd variable pointing to the same object
  
  ```java
  String s = "ab", t = s;
  s += "c"; // t doesn't change
  
  StringBuilder sb = "ab", tb = sb;
  sb.append("c") // t changes 
  ```

- Why do we have `StringBuilder`: more efficient (no need to create new objects)
  
  - And more convenient (more variables share the same object)

    

###### Risks of Mutation

- Not safe from bugs: passing mutable objects to a function is a latent bug
  
  - For possible changes to the object by the function

- Not easy to understand: see risk example #1 on [course website](http://web.mit.edu/6.031/www/sp21/classes/08-immutability/#risks_of_mutation)
  
  ```java
  List<Integer> myData = Arrays.asList(-5, -3, -2);
  System.out.println(sumAbsolute(myData)); // 10
  System.out.println(sum(myData)); // 10
  ```

- Returning a mutable object can also cause bugs. See risk example #2 on [website](http://web.mit.edu/6.031/www/sp21/classes/08-immutability/#risks_of_mutation)
  
  - Since the object might be reused in the original object / method

- Never use `Date`. use classes from `java.time`: `LocalDateTime`, `Instant`
  
  - They are immutable

- A way to avoid bugs caused by returning mutable objects: *defensive copying*
  
  ```java
  // rather than "return groundhogAnswer;"
  return new Date(groundhogAnswer.getTime());
  ```
  
  - However, this way wastes lots of memory (copy happens every time it's called )

    

###### Aliasing is What Makes Mutable Types Tricky

- When mutables safe: used in the same method, only 1 ref to the object

- Multiple references, a.k.a, aliases, bring the program potential bugs

    

###### Specs for Mutating Methods

- Now it'd be clear that: when a method does mutation, must include that in specs

- Check the following signature & spec of a method that does not mutate the arg:
  
  ```java
  // requires: nothing
  // effects: returns a new list t where t[i] = list[i].toLowerCase()
  static List<String> toLowerCase(List<String> list);
  ```

- If *effects* don't explicitly say the input can be mutated, then mutation's disallowed

    

###### Iterating over Arrays & Lists

- Why iterators exist: different data structure with varied internal representations
  
  - Iterators allow a uniform way to access all, so that client code is simpler

- See bugs occurring in the process of iteration & mutation:
  
  ```java
  // remove all strings in the list that start with 6.
  public static void dropCourse6(ArrayList<String> subjects) {
      MyIterator iter = new MyIterator(subjects);
      while (iter.hasNext()) {
          String subject = iter.next();
          if (subject.startsWith("6.")) subjects.remove(subject);
      }
  }
  ```

- 2 reasons this bug occurs: 
  
  - As we modify `subjects`, we may not know `iter` is also modified (aliases)
  
  - As we modify `iter` (by modifying `subjects`), we may assume: 
    
    - Elements in `iter` don't move around (indeed they do)

    

###### Mutation & Contracts

- *Mutation makes contracts complex*: if aliases exist, mutables must be consistent
  
  - Contracts cannot be enforced in only one place, but all where aliases exist

- Sharing a mutable object complicates a contract, in that we had to specify:
  
  - Either the client would not modify the object it got back, or
  
  - The implementor would not hold on to the object it returned

- See the following example that enforces the strategy above:
  
  ```java
  /** requires: nothing
  effects: returns an array containing the 9-digit MIT identifier of 
           username, or throws NoSuchUser­Exception if nobody with 
           username is in MIT’s database. Caller may never modify the 
           returned array */
  
  public static char[] getMitId(String username) throws NoUsrException 
  ```
  
  - Bad idea: a lifetime contract. Other contracts terminate as the call is over

    

###### Mutation & Contracts

Ironically, no good way to solve *Mutation & Contracts* issue. Just use immutable

```java
public static String getMitId(String username) throws NoSuchUserException
```

    

###### Immutability & Performance

Still possible to keep an immutable object efficient when it's heavily edited

- For example, a clever `String` implementation, might internally: 
  
  - Share the unchanged regions of characters before and after the edit

- Would see more of this in a few classes, when we talk about [*abstract data types*](D:\Data\personalData\大三下学习\software_construction\notes\abstract_data_type.md)
  
  - This kind of internal sharing is only possible because of immutability
