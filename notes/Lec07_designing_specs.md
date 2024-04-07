###### Objectives

- Understand what is an underdetermined spec, and able to identify it

- Understand declarative vs. operational specs, and write declarative specs

- Understand strength of preconditions, postconditions, and specs

- Be able to write coherent, useful specifications of appropriate strength

    

###### How to Compare Specs

The intent of this doc is to show how to compare specs, on 3 dimensions:

- How *deterministic* it is: given an input, does it specify only 1 output, or a set?

- How *declarative* it is: does it just characterize the output, or say how to compute it?

- How *strong* it is: does it have a small set of legal implementations, or a large one?

    

###### Deterministic vs. Underdetermined Specs

Given the `static int find(int[] arr, int val)` method, 2 specs are shown below:

```java
requires: val occurs exactly once in arr
effects: returns index i such that arr[i] = val
```

```java
requires: val occurs in arr
effects: returns index i such that arr[i] = val
```

- The 1st spec is deterministic while the 2nd is under-determined

- Since different results could be returned

    

###### Declarative vs. Operational Specs

- 2 types of specs: operational & declarative ones
  
  - *Operational spec*s  give a series of steps to perform (e.g., pseudocode)
  
  - *Declarative spec*s give properties of final outcome & how it’s related to the initial

- Almost always the declarative ones are preferable, since that doesn't expose biase
  
  - If you want to explain your behavior, do that in method body, not on the top

- Here comes a declarative spec for method `startsWith`:
  
  ![](C:\Users\86188\AppData\Roaming\marktext\images\2023-07-20-20-05-42-image.png)

    

###### Stronger vs. Weaker Specs (1 / 2)

- Say you want to change spec of a method, but clients've used this method
  
  - How can you tell if the new spec is good so we can keep the client code?

- Answer: spec `S2` is *stronger* than `S1` if the implementations that satisfy `S2` 
  
  - Can always satisfy `S1`

- To see which is *stronger* (i.e., more constrained), can compare pre & post conditions 
  
  - A precondition is stronger means shrinking the set of legal inputs
  
  - A stronger postcondition shrinks the set of allowed outputs and effects

- Therefore we generate the rule: `S2` is stronger than `S1` if and only if:
  
  - `S2`'s precondition is weaker or its postcondition is stronger

    

###### Stronger vs. Weaker Specs (2 / 2)

- Can always weaken precondition, as demanding less on a client never upsets them 
  
  - Similarly, can always strengthen postcondition, i.e., promising more to client

- The following specs are stronger from left to right, which means could be replaced
  
  <img title="" src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-20-21-26-28-image.png" alt="" width="244"> => <img title="" src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-20-21-26-48-image.png" alt="" width="284"> =>
  
  <img src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-20-21-27-30-image.png" title="" alt="" width="299">

    

###### Diagramming Specifications

Image the space of all java methods, where each point represents exactly one

- A *spec* defines a region in which all methods within are legal (implementation)

- A *stronger* *spec* defines a smaller region, as the figure bellow illustrates:
  
  <img src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-20-21-37-31-image.png" title="" alt="" width="205">

- *Spec*s act as a firewall for clients:
  
  - Implementor can move around inside the *spec* (they may wanna improve algo)
  
  - Clients don’t know which solution used, but know what they get'd meet needs

    

###### Designing Good Specs: Coherent

- The specs should be coherent, they'd behave as a single, complete unit

- Instead of having lots of different cases. See an example:
  
  <img src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-20-22-28-18-image.png" title="" alt="" width="447">
  
  - Incoherent, since it does 2 things (counting words & finding the longest word)
  
  - Separating them would be better to understand and more ready for change

    

###### Designing Good Specs: Informative

Consider the spec of a method that puts a value in a map:

![](C:\Users\86188\AppData\Roaming\marktext\images\2023-07-20-22-33-42-image.png)

- Since we can store `null`s as values in the map, when this method return a `null`:

- We don't know whether `null` is the value of the key, or the key isn't in the map

    

###### Designing Good Specs: Strong

- Obviously specs should give clients a strong enough guarantee that: 
  
  - It'll satisfy their basic requirements

- We must use extra care when specifying the special cases, to make sure: 
  
  - They don’t undermine what would originally be a useful method

- Consider the following spec:
  
  ![](C:\Users\86188\AppData\Roaming\marktext\images\2023-07-20-22-47-35-image.png)
  
  - The spec isn't strong enough. If exception occurs, what'd happen to `list1`?

- Can improve the spec by strengthening it further as the following:
  
  ![](C:\Users\86188\AppData\Roaming\marktext\images\2023-07-20-22-50-42-image.png)

    

###### Designing Good Specs: Weak

Consider a spec for a method that opens a file:

<img src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-20-22-52-21-image.png" title="" alt="" width="297">

- This spec is too strong, since impossible to guarantee can open a file in every case

- Should add more options to the postcondition (i.e., make it weaker)
  
  - e.g., print error info if fail to open the file

    

###### Designing Good Specs: Use Abstract Types

<img src="file:///C:/Users/86188/AppData/Roaming/marktext/images/2023-07-20-22-57-29-image.png" title="" alt="" width="572">

- Better to use `List`, as its behavior depends on nothing specific about `ArrayList` 

- Use interface more like `Map` or `Reader`, which gives more freedom to: 
  
  - Both the client and the implementor

    

###### Precondition vs. Postcondition

- Precondition inconveniences clients. So Java classes, specify as a postcondition that 
  
  - They throw unchecked exceptions when arguments are inappropriate

- Precondition is necessary, if checking that slows down the method
  
  - e.g., if we want to write `find` with binary search, the array has to be sorted

- Aside from cost of check, the method scope can also be a measurement
  
  - If only called locally in a class, can check each usage so that precondition is met 
  
  - But if the method is public & used by other coders, should use postcondition

    
