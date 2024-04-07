###### Introduction

- Why *ADT*: separate how we use a data structure, from particular form of it

- *ADT* solves a dangerous issue: clients making assumption about type's internal
  
  - We'll see why it's dangerous, and how it can be avoided

    

###### What Abstraction Means (1 / 2)

*ADT* is an instance of a general principle in software engineering, which could be:

- ***Abstraction***: hide low-level details with a simpler, higher-level idea

- ***Modularity***: divide a system into parts, each of which can be designed & reused

- ***Encapsulation***: build a wall around a module => bugs elsewhere don't damage it

- ***Info hiding***: hide detail of a module from the rest of the system => can update later

- ***Separation of concerns***: make a feature the responsibility of 1 module, not more

    

###### What Abstraction Means (2 / 2)

Previously we focus on methods. See how these ideas work on methods:

- ***Abstraction***: by spec clients'd only know how to use a method, not how it's written

- ***Modularity***: unit testing and specs help make methods into modules

- ***Encapsulation***: the local variables of a method are encapsulated. Globals are not

- ***Information hiding***: a spec leaves some freedom in how the method is written

- ***Separation of concerns***: a good method spec is [coherent](http://web.mit.edu/6.031/www/sp21/classes/07-designing-specs/#the_specification_should_be_coherent), responsible for 1 concern

    

###### User-defined Types

- In the early days of computing, languages have built-in types & functions
  
  - Users can define functions but cannot do data types

- A major advance is that later users can define their own types, which then:
  
  - Brings up the concept of *ADT*

- But what made abstract types new & different was the focus on operations:
  
  - The user of the type'd not need to worry how its values were actually stored
  
  - Just care about what you can do on it, and then what it can give back to you

    

###### Classify Types & Operations (1 / 3)

The operations of an *ADT* are classified as follows (focus is on types, not objects):

- **Creators** create new objects of the type. May use other types as args but not itself
  
  ```java
  List<Integer> lst = new ArrayList<>(); // creator
  ```

- **Producers** create new objects of the type too, but need objects of the type as args
  
  ```java
  String greeting = "Hello".concat("World");
  ```

- **Observers** take objects of the type and return objects of a different type
  
  ```java
  int length = "Hello".length();
  ```

- **Mutators** change those objects which are mutable
  
  ```java
  new ArrayList<Integer>().add(3);
  ```

        

###### Classify Types & Operations (2 / 3)

can summarize these distinctions schematically like this (explanation to follow):

```c
Creator: t* → T
producer: T+, t* → T
observer: T+, t* → t
mutator: T+, t* → void | t | T
```

    

###### Classify Types & Operations (3 / 3)

- A creator operation is often implemented as a *constructor*

- But a creator can simply be a static method instead, like `List.of()`

- A creator implemented as a static method is often called a *factory method*

    

###### Principles in Designing an ADT

An *ADT* is defined by its operations. Here are a few rules of thumb to define one:

- A few, simple operations that can be combined in powerful ways
  
  - Rather than lots of complex operations

- Basic information should not be extremely difficult to obtain
  
  - Should have `size` for `List`, though we can apply `get` until exceptions

- Should not mix generic and domain-specific features. e.g., do not have:
  
  - `sum` for `List`, since `List` can process multiple types not just `int`s

    

###### ADT for String

We can use a `char[]` to contain the content, 2 `int`s to label start & end of the string

- In this way, whenever `substring` method is called, we just create a new object:

- With the `int` value, i.e., start & end changed, instead of creating a new `char[]`
