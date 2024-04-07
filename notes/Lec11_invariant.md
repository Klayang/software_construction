###### Invariants (1 / 2)

- An invariant is a property of a program that's always true, for every runtime state
  
  - An important property of a good ADT, is that it preserves its own invariants

- Immutability is a crucial invariant we've already encountered, i.e., once created:
  
  - An immutable object'd always represent the same value, for its entire lifetime

- There are other invariants, including types of variables, relationship between them
  
  - e.g. when `i` is used to loop over a list, then `0 <= i < list.size()`

    

###### Invariants (2 / 2)

- If an ADT *preserves its own invariants*, it's responsible for ensuring its invariants hold

- Done by hiding / protecting variables involved in the invariants (e.g., use `private`)
  
  - And allowing access only through operations with well-defined contracts

    

###### Immutability (1 / )

```java
/**
 * This immutable data type represents a tweet from Twitter.
 */
public class Tweet {

    public String author;
    public String text;
    public Date timestamp;

    /**
     * Make a Tweet.
     * @param author    Twitter user who wrote the tweet
     * @param text      text of the tweet
     * @param timestamp date/time when the tweet was sent
     */
    public Tweet(String author, String text, Date timestamp) { 
        this.author = author;
        this.text = text;
        this.timestamp = timestamp;
    }
}
```

    

###### Immutability (2 / )

- How to guarantee `Tweet` objects are immutable, i.e., once a tweet is created: 
  
  - Its author, message, and date can never be changed?

- The 1st threat to immutability comes from that clients can directly access its fields
  
  ```java
  Tweet t = new Tweet("justinbieber", "Baby~", new Date());
  t.author = "Yang";
  ```
  
  - Called representation exposure, i.e., code outside class can modify rep directly
  
  - It also threatens rep independence. Implementor can't update implementation
    
    - Without affecting client who directly access those fields

    

###### Immutability (3 / )

Fortunately, Java gives us language mechanisms to deal with this kind of rep exposure:

```java
public class Tweet {
    private final String author;
    private final String text;
    private final Date timestamp;

    public Tweet(String author, String text, Date timestamp) {
        this.author = author;
        this.text = text;
        this.timestamp = timestamp;
    }

    public String getAuthor() { return author; }
    public String getText() { return text; }
    public Date getTimestamp() { return timestamp; }
}
```

    

###### Immutability (4 / )

Not the end of the story. The rep is still exposed! Consider the following client code:

```java
/**
 * @return a tweet that retweets t, one hour later
 */
public static Tweet retweetLater(Tweet t) {
    Date d = t.getTimestamp();
    d.setHours(d.getHours() + 1);
    return new Tweet("yang", t.getText(), d);
}
```

    

###### Immutability (5 / )

- Problem: `Date` is mutable, and `Tweet` leaked out a reference to a mutable object 
  
  - That `Tweet`'s immutability depended on

- Thus, the old `Tweet`'s `timestamp` gets changed. Use *defensive copy* to make it up
  
  ```java
  public Date getTimestamp() {
      return new Date(timestamp.getTime());
  }
  ```

    

###### Copy Constructor

- Mutable types often have a *copy constructor* that allows you to:
  
  - Make a new instance that duplicates the value of an existing instance

- In the case above, `Date`’s *copy constructor* uses the timestamp value
  
  - Measured in milliseconds since January 1, 1970

- Another way to copy a mutable object is `clone()`, supported by some types
  
  - But not all, don't use it

    

###### Immutability (6 / )

Not done yet, still representation exposure!

```java
/**
 * @return a list of 24 inspiring tweets, one per hour today
 */
public static List<Tweet> tweetEveryHourToday () {
    List<Tweet> list = new ArrayList<Tweet>(); 
    Date date = new Date();
    for (int i = 0; i < 24; i++) {
        date.setHours(i);
        list.add(new Tweet("rbmllr", "keep it up! you can do it", date));
    } 
    return list;
}
```

`date` is reused, setting a later `Tweet` modifies that of previous `Tweet`s

    

###### Immutability (7 / )

Can fix this by using *defensive copy* in the constructor:

```java
public Tweet(String author, String text, Date timestamp) {
    this.author = author;
    this.text = text;
    this.timestamp = new Date(timestamp.getTime());
}
```

    

###### Immutability (8 / )

In general, should carefully inspect arg & return types of all the ADT operations

- If any types are mutable, make sure the args are not directly stored in the rep
  
  - Or return direct references to the rep. Use *defensive copy* instead

- You may object that this seems wasteful. Why make all these copies of dates? 
  
  - Why can’t we just solve this problem by a carefully written specification?

    

###### Immutability (9 / )

```java
/**
 * Make a Tweet.
 * @param author    Twitter user who wrote the tweet
 * @param text      text of the tweet
 * @param timestamp date/time when the tweet was sent. Caller must never 
 *                   mutate this Date object again!
 */
public Tweet(String author, String text, Date timestamp) {...}
```

- This approach is taken if the mutable object is too large to copy efficiently

- Almost always worth it for an ADT to guarantee its own invariants 
  
  - And preventing rep exposure is essential to that

- An even better solution is to prefer immutable types, e.g., instead of `Date`
  
  - We could use  `java.time.ZonedDateTime`

    

###### Immutable Wrappers around Mutable Data Types

The Java collections classes offer a compromise: immutable wrappers

- `Collections.unmodifiableList()` takes a (mutable) `List` & wraps with an object
  
  - That looks like a `List`, but mutators like `set`, `add`, `remove` disabled

- The downside here is that you get immutability at runtime, not at complie time
  
  - No warning at complie time if you try to sort this unmodifiable list

- And no errors if the original one gets modified, in turn getting wrapper modified
  
  - So need to give up other reference to the original if you use the wrapper

    

###### Rep Invariant & Abstract Function (1 / 7)

Two spaces of value:

- Abstract values, consisting of the values that the type is designed to support
  
  - Client view. e.g., Java's `BigInteger` for unbounded integers

- Representation values, consists of Java objects that implement the abstract values
  
  - Implementor view. e.g., `int[]` that's used to implement `BigInteger`

    

###### Rep Invariant & Abstract Function (2 / 7)

Now we choose to use a string to represent a set of characters

```java
public class CharSet {
    private String s; ...
}
```

- The *rep space* `R` contains `String`s, and the *abstract space* `A` is math sets of chars

- Can show the two value spaces graphically, with an arrow from `R` to `A`
  
  <img src="file:///C:/Users/yangs/screenshots/aToR.PNG" title="" alt="" width="305">

    

###### Rep Invariant & Abstract Function (3 / 7)

For the picture above, there are several notes here:

- Every *abstract value* is mapped to by at least one *rep value*

- Not all *rep value*s are mapped. e.g., `abbc` is not mapped since: 
  
  - No support for duplicates

    

###### Rep Invariant & Abstract Function (4 / 7)

Since the graph between `R` & `A` is infinite, we describe it by giving 2 things:

- An *abstraction function* (`AF`: `R -> A` ) that maps *rep value*s to the *abstract value*s
  
  - The function is *surjective*, but neither *injective* nor *bijective*

- A *rep invariant* (`RI: R -> I`) that maps *rep value*s to `boolean`s
  
  - For a *rep value* `r`, `RI(r)` is true if and only if `r` is mapped by `AF`
  
  - i.e., `RI` tells us whether a given *rep value* is well-formed (it describes a set)
    
    - Where all valid rep values are all in that set

    

###### Rep Invariant & Abstract Function (5 / 7)

The diagram below shows a rep for CharSet that forbids repeated characters

<img src="file:///C:/Users/yangs/screenshots/RI.PNG" title="" alt="" width="311">

- Rep values that obey the rep invariant are shown in the green part of the `R` space
  
  - And they must be mapped to an abstract value in the `A` space

- Rep values that violate the rep invariant are shown in the red zone
  
  - And have no equivalent abstract value in the `A` space

    

###### Rep Invariant & Abstract Function (6 / 7)

- The *abstract space* alone doesn't determine `AF` / `RI` (can be several reps for 1 `A`)

- The choice of both spaces doesn't determine `AF` & `RI` either. They are just bases
  
  - With them, you can define your `AF` & `RI` on top of these 2 spaces

    

###### Rep Invariant & Abstract Function (7 / 7)

Summary: implementing an *ADT* means not only choosing the 2 spaces

- But also deciding which reps are legal & how to interpret them as abstract values

- Important to write down these assumptions (`AF` & `RI`) in your code, so that:
  
  - Future programmers (and yourself) are aware of what the rep actually means

    

###### Beneficent Mutation

Recall that a type is immutable iff a value of the type never changes after being created

- With understanding of `A` & `R`, can refine this to: *abstract value*'d never change

- The implementation is free to mutate a *rep value*, as long as it maps to the original
  
  - This kind of change is called *beneficent mutation*

    

###### Documenting AF, RI & Safety from Rep Exposure

Besides `AF` & `RI`, you should also write a *rep exposure safety argument*

- It's a comment that looks at rep code & presents why it doesn't expose the rep

- Here's an example of `Tweet` with its `AF`, `RI` and safefy from exposure:
  
  ```java
  public class Tweet {
  
      private final String author;
      private final String text;
      private final Date timestamp;
  
      // Rep invariant:
      //   author is a Twitter username (a nonempty string of letters, 
      //       digits, underscores)
      //   text.length <= 280
  
      // Abstraction function:
      //   AF(author, text, timestamp) = a tweet posted by author, 
                                   with content text, at time timestamp 
      // Safety from rep exposure:
      //   All fields are private;
      //   author and text are Strings, so are guaranteed immutable;
      //   timestamp is a mutable Date, so Tweet() methods 
      //        make defensive copies to avoid sharing the rep's Date 
      //        object with clients.
  }
  ```

    

###### How to Establish Invariants

An *invariant* is a property that's true for the entire program

- To make an invariant hold, need to make it true initially and ensure all the changes:
  
  - Also keep the invariant true

- This means *creator*s & *producer*s establish the invariant for ner object instances
  
  - And *mutator*s, *observer*s must preserve the invariant

- If invariant of *ADT* is built by *creator*s, preserved by *mutator*s, no rep exposure
  
  - The invariant is true of all instances of the *ADT*

    

###### Summary

- An invariant is a property that is always true of an ADT object instance

- A good ADT preserves its own invariants. Invariants must be:
  
  - Established by creators & producers, preserved by observers & mutators

- The rep invariant specifies legal values of the representation and should be: 
  
  - Checked at runtime with `checkRep()`

- The abstraction function maps a concrete rep to the abstract value it represents

- Rep exposure threatens both rep independence & invariant preservation

- An invariant should be documented, by comments or even better by assertions 
  
  - Like `checkRep()`
