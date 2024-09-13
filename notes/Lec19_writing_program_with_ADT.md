###### Objectives

- Learn how to apply spec, test, iteration at different levels of software development:

- From single methods, to abstract datatypes, to entire programs

    

###### Introduction

- Bringing together what we've learnt of abstarct datatypes & recursive datatypes:

- This reading demonstrates the development of a recursive ADT to solve a problem

    

###### Recipes for Programming (1 / 4)

Recall the [test-first programming approach](https://web.mit.edu/6.031/www/sp21/classes/03-testing/#iterative_test-first_programming) for writing a method:

1. Spec: write the spec, including the signature (name, arg & return type, exception)
   
   - And the precondition & postcondition as a javadoc comment

2. Test: write systematic test cases & put them in a JUnit class to be run automatically
   
   - You may have to go back & change your spec when you start to run tests
   
   - When you write test cases, you'll think about how a client'd call the method
   
   - Therefore, step 1 & 2 iterate until you've got a better spec & test cases
   
   - Make sure some of your tests are failing at first, good for finding bugs later

3. Implement: write the body of the method. It's done when all tests are green
   
   - Implementing the method puts pressure on both step 2 & 3
   
   - So potentially need to chane the implementation, tests, and the spec
     
     - And bouncing back and forth among them

    

###### Recipes for Programming (2 / 4)

Let's broaden this to a recipe for writing an *abstract data type*:

1. Spec: write spec for operation of the spec, including signature, pre & post condition

2. Test: write tests for ADT's operations, which may require iteration between 1 & 2

3. Implement: it expands to: choose rep, assert rep invariant, implement operation
   
   - Choose rep: write down private field, variants of a recursive datatype, `RI`, `AF`
   
   - Assert rep invariant: implement a `CheckRep`method that enforces `RI`
   
   - Implement operation: write the method bodies of the operations
     
     - Making sure to call `CheckRep` in them

    

###### Recipes for Programming (3 / 4)

Let's broaden it further to a recipe for a program (consisting of static methods & ADTs)

1. Choose datatypes: decide which ones'll be mutable, and which one's immutable

2. Choose static methods: write your top-level `main` & break it down to small steps

3. Spec: specs out the ADTs & methods. Keep ADT operation simple & few as possible
   
   - Only add complex operations as you need them

    

###### Recipes for Programming (4 / 4)

4. Test: write test case for each unit

5. Implement simply first: choose simple, brute-force reps. 
   
   - Why? Put pressure on the specs & tests, pulling your whole program together
   
   - Skip advanced feature, performance optimization, corner cases for now
   
   - Make the program work correctly first, and keep a to-do list for revisits

6. Iterate: reimplement, optimize, redesign if necessary
   
   - Now that it’s all working, make it work better

---

###### Problem: Matrix Multiplication

Suppose we want a way to represent matrix multiplication to optimize the computation

- e.g., if `a, b` are scalar constants, `X` a big matrix, then `(aX)b` is slow to compute
  
  - Because it loops over `X` twice. Equivalent & cheaper to do `(ab)X`

- Remember however that matrix multiplication is associative but not commutative
  
  - Only scalars commute

---

###### Spec

    

###### Choose Datatypes

Let's call this a `MatrixExpression` (in short, `MatExpr`). Let's define some operations

- `make`: `double` -> `MatExpr`
  
  - effects: return an expression consisting of just the given scalar

- `make`: `double[][] array` -> `MatExpr`
  
  - requires: `array.length` > 0, `array[i].length` > 0
  
  - effects: returns an expression consisting of just the given matrix

- `times`: `MatExpr m1, MatExpr m2` -> `MatExpr`
  
  - requires: `m1` & `m2` are compatible for multiplication
  
  - effects: returns `m1` * `m2`

- `isIdentity`: `MatExpr` -> `boolean` 
  
  - effects: returns `true` if & only if the expression is the multiplicative identity

- `optimize`: `MatExpr` -> `MatExpr`
  
  - effects: returns expression with the same value, but may be faster to computer

---

###### Test

Let's test `optimize`. Partitions:

- Partition on # scalars in expression: 0, 1, 2, >2

- Partition on position of scalar in expression tree: left / right child
  
  - Left-child-of-left-child, left-child-of-right-child
  
  - Right-child-of-left-child, right-child-of-right-child
  
  - And more than 2 levels deep

    

###### Test Case Example

Here are somes test cases and what paitition they're being covered

<img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-09-13-11-56-09-image.png" title="" alt="" width="529">

---

###### Implement

    

###### Choose A Rep

This problem is a natural one for a [recursive datatype](C:\Users\yangs\Desktop\software_construction\Lec16_recursive_data_type.md)

```html
MatExpr = Identity + Scalar(value:double) + Matrix(array:double[][])
    + Product(m1:MatExpr, m2:MatExpr)
```

```java
/** Represents an immutable expression of matrix and scalar products. */
public interface MatrixExpression {
    // Datatype definition:
    //   MatExpr = Identity + Scalar(value:double)
    //    + Matrix(array:double[][]) + Product(m1:MatExpr, m2:MatExpr)
    // ...
}
```

```java
class Identity implements MatrixExpression {
    public Identity() {
    }
}
```

```java
class Scalar implements MatrixExpression {
    private final double value;
    // RI: true
    // AF(value) = the real scalar represented by value

    public Scalar(double value) {
        this.value = value;
    }
}
```

```java
class Matrix implements MatrixExpression {
    private final double[][] array;
    // RI: array.length > 0, and all array[i] are equal nonzero length
    // AF(array) = the matrix with array.length rows 
    // and array[0].length columns, whose (row,column) entry 
    // is array[row][column]

    public Matrix(double[][] array) {
        this.array = array; // note: danger! (rep exposure)
    }
}
```

```java
class Product implements MatrixExpression {
    private final MatrixExpression m1;
    private final MatrixExpression m2;
    // RI: m1's column count == m2's row count, or m1 or m2 is scalar
    // AF(m1, m2) = the matrix product m1*m2

    public Product(MatrixExpression m1, MatrixExpression m2) {
        this.m1 = m1;
        this.m2 = m2;
    }
}
```

    

###### Choose Identity

- Always good to have a value in a datatype that represents nothing, to avoid `null`

- For a matrix product, the natural choice is the identity matrix. So let's define that:
  
  ```java
  /** Identity for all matrix computations. */
  public static final MatrixExpression I = new Identity();
  ```
  
  - One concern is that other expressions can also be an identity

    

###### Implementing `make`: Use Factory Methods

Let's start implementing, starting with the `make` creators

- We don't want to expose our concrete rep classes (`Scalar`, `Matrix`, `Product`)
  
  - So that clients won't depend on them and thus they are ready for change

- Therefore, we'll create static factory methods in `MatrixExpression` to do this:
  
  ```java
  /** @return a matrix expression consisting of just the scalar value */
  public static MatrixExpression make(double value) {
      return new Scalar(value);
  }
  ```
  
  ```java
  /** @return a matrix expression consisting of just the matrix given */
  public static MatrixExpression make(double[][] array) {
      return new Matrix(array);
  }
  ```

    

###### Implementing `isIdentity`: don't use `instanceof` (1 / 2)

Now let's implement the `isIdentity` observer. Here's a **bad** way to do this:

```java
// don't do this
public static boolean isIdentity(MatrixExpression m) {
    if (m instanceof Scalar) {
        return ((Scalar)m).value == 1;
    } else if (m instanceof Matrix) {
        // ... check for 1s on the diagonal and 0s everywhere else
    } else ... // do the right thing for other variant classes
}
```

- In general, using `instanceof` in object-oriented language is a bad smell

- Break the operations down into pieces that are appropriate for each class
  
  - And write instance method instead

    

###### Implementing `isIdentity`: don't use `instanceof` (2 / 2)

- Implementing `isIdentity` exposes an issue that we'd have found by writing tests:
  
  - Cannot return `true`, for all `Product`s whose value is the identity (`A` * `A`<sup>-1</sup>)

- We probably want to resolve this by weakening the spec of `isIdentity`
  
  ```java
  isIdentity: MatExpr → boolean
  effects:
   returns true only if the expression is the multiplicative identity
  ```

    

###### Implementing `optimize` Without `instanceof` (1 / 6)

Let's implement `optimize`. Again here's a bad way which'll get you mired in weeds

```java
// don't do this
class Product implements MatrixExpression {
    public MatrixExpression optimize() {
        if (m1 instanceof Scalar) {
            ...
        } else if (m2 instanceof Scalar) {
            ...
        }
        ...
    }
}
```

    

###### Implementing `optimize` Without `instanceof` (2 / 6)

- If you find your code having `instanceof` all over the place, rethink the problem

- To optimize scalars, we need 2 recursive helpers, which'll be added to the type
  
  ```java
  scalars: MatExpr → MatExpr
  effects: returns a MatExpr with no matrices in it, 
      only the scalars in the input expression
  
  matrices: MatExpr → MatExpr
  effects: returns a MatExpr with no scalars in it, only the matrices 
      in the same order they appear in the input expression
  ```

    

###### Implementing `optimize` Without `instanceof` (3 / 6)

These expressions allow pulling scalars out of expressions & moving them together

```java
/** Represents an immutable expression of matrix and scalar products. */
public interface MatrixExpression {

    // ...

    /** @return the product of all the scalars in this expression */
    public MatrixExpression scalars();

    /** @return product of all the matrices in this expression in order
     * times(scalars(), matrices()) is equivalent to this expression. */
    public MatrixExpression matrices();
}
```

    

###### Implementing `optimize` Without `instanceof` (4 / 6)

Now we'll implement them as expected:

```java
class Identity implements MatrixExpression {
    // no fields
    ...
    public MatrixExpression scalars() { return this; }
    public MatrixExpression matrices() { return this; }
}
```

```java
class Scalar implements MatrixExpression {
    private final double value;
    ...
    public MatrixExpression scalars() { return this; }
    public MatrixExpression matrices() { return I; }
}
```

```java
class Matrix implements MatrixExpression {
    private final double[][] array;
    ...
    public MatrixExpression scalars() { return I; }
    public MatrixExpression matrices() { return this; }
}
```

```java
class Product implements MatrixExpression {
    private final MatrixExpression m1, m2;
    ...
    public MatrixExpression scalars() {
        return times(m1.scalars(), m2.scalars());
    }
    public MatrixExpression matrices() {
        return times(m1.matrices(), m2.matrices());
    }
}
```

    

###### Implementing `optimize` Without `instanceof` (5 / 6)

- `MatrixExpression.I` is the constant we defined to represent the [identity matrix](https://web.mit.edu/6.031/www/sp21/classes/19-programming-with-adts/#choose_an_identity)

- The implementations of `Scalar.matrices()` and `Matrix.scalars()` return `I`: 
  
  - To represent an empty product

    

###### Implementing `optimize` Without `instanceof` (6 / 6)

With these helpers, `optimize` can just separate the scalars & matrices

```java
class Identity implements MatrixExpression {
    ...
    public MatrixExpression optimize() { return this; }
}
```

```java
class Scalar implements MatrixExpression {
    ...
    public MatrixExpression optimize() { return this; }
}
```

```java
class Matrix implements MatrixExpression {
    ...
    public MatrixExpression optimize() { return this; }
}
```

```java
class Product implements MatrixExpression {
    ...
    public MatrixExpression optimize() {
        return times(scalars(), matrices());
    }
}
```

    

###### Summary

This reading's a case of developing a recursive datatype, so it applies ideas we've seen

- Static typing, testing, specs, immutability, interfaces

- In support of the 3 properties of good software

    
