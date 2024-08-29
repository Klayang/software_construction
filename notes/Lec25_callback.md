###### Objectives

Talking about *callback*s, where an implementer calls a function provided by the client

- We'll discuss this idea in 2 concepts, graphical user interfaces & web servers
  
  - Where the *callback*s are used to respond to incoming input events

- *Callback*s are an example of *first-class function*s, that is to treat functions like data:
  
  - Passing them as parameters, returing as results, and storing in variables
  
  - We'll see more uses for *first-class* functions over the next few classes

    

###### Input Handling in A Graphical User Interface (1 / 9)

- Input is handled differently in graphical user interfaces (`GUI`s) than: 
  
  - Console user interfaces and servers

- In those other systems, a single *input loop* is used to read commands from user
  
  - Parse them and decide how to direct them to different modules

- If a `GUI` email client were written that way, it might look like this:
  
  ```c
  while (true) {
      read mouse click
      if (clicked on Check Mail button) doRefreshInbox();
      else if (clicked on Compose button) doStartNewEmailMessage();
      else if (clicked on an email in the inbox) doOpenEmail(...);
      ...
  }
  ```

    

###### Input Handling in A Graphical User Interface (2 / 9)

But in a `GUI`, we don't directly write this kind of method, since it's not modular

- `GUI`s are put together from lots of components - buttons, scrollbars, menus, ...
  
  - That need to be self-contained and handle their own input

- e.g., here is how you create a button (the button is on the right side underneath)
  
  ![](C:\Users\yangs\AppData\Roaming\marktext\images\2024-04-08-11-17-11-image.png)     <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-04-08-11-17-39-image.png" title="" alt="" width="50">

- The button handles its input from mouse & keyboard. Deep inside the `GUI` library
  
  - There is an input loop that's reading from the mouse & keyboard
  
  - And passing those input events to the appropriate components of `GUI`

    

###### Input Handling in A Graphical User Interface (3 / 9)

In order for your program to respond when the button is clicked, attach a *listener* to it

```java
playButton.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent event) {
        playSound();
    }
});
```

- This code creates an instance of an *annoymous class* implementing the:
  
  - `ActionListener` interface, which has 1 method: `actionPerformed`

- When this `ActionListener` instance is given to button by `addActionListener`
  
  - The `actionPerformed`'ll be called every time when button is clicked

    

###### Input Handling in A Graphical User Interface (4 / 9)

`GUI` event handling is an instance of the *Listener Pattern* (aka *Publish-Subscribe*), where:

- An event source generates (or *publish*es) a stream of discrete events, which:
  
  - Corresponds to state transitions in the source

- One or more *listener*s register interest (*subscribe*s) to the stream of events
  
  - Providing a function to be called when a new event occurs

    

###### Input Handling in A Graphical User Interface (5 / 9)

In this example:

- The `JButton` is the *event source*

- Its events are button presses

- The listener is the anonymous `ActionListener` instance

- The method `actionPerformed` is called when the evnt happens

    

###### Input Handling in A Graphical User Interface (6 / 9)

- An event often includes additional information which might be: 
  
  - Bundled into an event object (like the `ActionEvent` here)
  
  - Or passed as parameters to the listener function

- When an event occurs, the event source distributes it to all subscribed listeners
  
  - By calling their *listener method*s

    

###### Input Handling in A Graphical User Interface (7 / 9)

So the control flow through a `GUI` proceeds like this:

- A top-level *event loop* reads input from mouse & keyboard. In most `GUI` toolkits:
  
  - The loop is hidden from you, but buried inside the toolkit
  
  - Often running on a separate thread created by the toolkit
  
  - And listeners appear to be called magically

- Each listener does its thing (may involve e.g., modifying objects in the [view tree](https://en.wikipedia.org/wiki/Tree_view))
  
  - And then return immediately to the *event loop*

    

###### Input Handling in A Graphical User Interface (8 / 9)

- The last part, listeners return to the *event loop* as fast as possible, really matters
  
  - Since it preserves the responsiveness of the user interface

- The *listener* pattern isn't just used for button presses. Every `GUI` object: 
  
  - Generates events, often as a result of combination of low-level input events
  
  - Here comes some examples

    

###### Input Handling in A Graphical User Interface (9 / 9)

Example of events being generated:

- `JButton` sends an action event when it is pressed

- `JList` sends a selction event when the selected elements changes

- `JTextField` sends change events when the text inside it changes

    

###### Callbacks (1 / 2)

The `actionPerformed` listener function is an example of a design pattern: *callback*

- A *callback* is a function that a client provides to a module and for the module to call

- This is contrast to normal control flow, in which the client is doing all the calling
  
  - Calling down into functions that the module provides

    

###### Analogy for Where Callbacks Came (1 / 3)

Here's on analogy for thinking about this idea:

- Normal function calling is like picking up the phone & calling a service
  
  - Calling your bank to find out the balance of your account

- You give the info that the bank operator needs to look up your account
  
  - They read back the account balance to you over the phone, and you hang up
  
  - You are the client, and the bank is the module that you’re calling into

    

###### Analogy for Where Callbacks Came (2 / 3)

- Sometimes the bank is slow to give an answer. You’re put on hold, and you wait:
  
  - Until they figure out the answer for you

- This is like a function call that *blocks* until it is ready to return, which we saw: 
  
  - When we talked about sockets and message passing

    

###### Analogy for Where Callbacks Came (3 / 3)

But the task may take so long that the bank doesn’t want to put you on hold

- Then the bank will ask you for a callback phone number

- And they will promise to call you back with the answer
  
  - This is analogous to providing a callback function

    

###### Callbacks (2 / 2)

- The kind of *callback* used in listener pattern is not an answer to a 1-time request
  
  - Like your account balance

- It’s more like a regular service that the bank is promising to provide
  
  - Using your callback number as needed to reach you

- A better analogy for the listener pattern is account fraud protection, where bank:
  
  - Calls you on the phone whenever a suspicious transaction occurs

    

###### Route Handling in A Web Server (1 / 5)

Here's another system where *callback*s are also used for input handling: *web server*

- When a new connection arrives, the server reads & parses the request 
  
  - (using `HTTP` *wire protocol*)

- The request is routed to handler registered for the route matching the request
  
  - The handler'd construct the response to the request

- Typically only a prefix of the request has to match the route, so the request: 
  
  - `http://web.mit.edu/community/topic/arts.html` will be routed to: 
  
  - The handler for `/community`, unless some more specific (longer prefix) routes

    

###### Route Handling in A Web Server (2 / 5)

Oracle's Java implementation has a web server: `HttpServer`. Can create one like this:

```java
// port on which the server will listen for incoming connections
final int port = 8081;

// incoming connections may be queued until the server is ready to handle
// 0 means use the default system value for the length of this queue
final int useDefaultBacklog = 0;

// make the server
HttpServer server = HttpServer.create(new InetSocket(port), 
    useDefaultBacklog);
```

And then add routes to it using `createContext`:

```java
server.createContext("/song/", new HttpHandler(){
    public void handle(HttpExchange exchange) throws IOException {
        ...
    }
});
```

    

###### Route Handling in A Web Server (3 / 5)

Here we made an annoymous instance that implements the `HttpHandler` interface

- Only one method, `handle`, is within the interface, which is a *callback*, being called:
  
  - Every time an incoming requests starting with the `/song/` prefix

- The `args` to the callback is an `HttpExchange` object, which provides: 
  
  - Observor method with information about the request, and
  
  - Mutator methods for generating a response that goes back to the browser

    

###### Route Handling in A Web Server (4 / 5)

The body of *callback* (i.e., `...`) should examine the request & generate a response

    

###### Examine Request

- Here's an example of how it might examine the request

- If the incoming request was `http://localhost:8081/song/cello.wav`, then:
  
  ```java
  // get path of the http request
  String path = exchange.getRequestURI().getPath(); // "/song/cello.wav"
  // remove /song/ from the start of the path to get the filename
  String soundFilename = path.substring(exchange.getHttpContext()
      .getPath().length()); // "cello.wav"
  ```

    

###### Generating Response

Here's an example of generating a response

- The handler'd first state the type of response it is returning (`HTML`, plain text, etc)
  
  ```java
  exchange.getResponseHeaders().add("Content-Type", "text/html; 
      charset=utf-8");
  final int statusOK = 200;
  final int responseLengthNotKnownYet = 0;
  exchange.sendResponseHeaders(statusOK, responseLengthNotKnownYet);
  ```

- And then write the response using the `exchange`'s output stream:
  
  ```java
  PrintWriter out = new PrintWriter(
      new OutputStreamWriter(exchange.getResponseBody(),
      StandardCharsets.UTF_8), true /* autoflush */);
  out.println("playing " + soundFilename);
  exchange.close();
  ```

    

###### Route Handling in A Web Server (5 / 5)

This `HttpHandler` (a scheduler & multiple handlers) is an example of a callback:

- A piece of code that we as clients (who sets up the server) pass to the module

- For the module to call when an event occurs, i.e., a matching request arrives

    

###### Client

Client can mean two things:

- A client of a web server is a browser, which sends request, gets result & renders it

- A client of a class is the code that creates an instance of that class, and thus:
  
  ```java
  HttpServer server = new HttpServer(); // this code itself is client
  server.createContext("/song/", handle(){...}); // client calls server
  // by calling its API, and provides a function for it to call
  // how the server calls handle() is within  createContext(){...} 
  ```
  
  - Can call any API that the class provides (what's within API belongs to server)

    

###### First Class Functions (1 / 3)

- Callbacks can only be used in languages where functions are *first-class*, that is:
  
  - They can be treated like any other value in the language
  
  - Passed as parameters, returned as return values, stored in data structures

- Lots of things in a programming is not *first-class*, e.g., *access control*:
  
  - Cannot pass `public`/`private` to any function, or stored in any variable
  
  - Similarly, a `while` loop or an `if` statement don't apply to *first class* either

    

###### First Class Functions (2 / 3)

In Java, the only *first-class* values are primitive values and object references

- But objects can carry functions with them, in the form of methods

- Thus in Java, a *first-class* function is supported indirectly, by using an object
  
  - With a method representing the function

- Having seen this for several times before. All of the following are *first-class*:
  
  - `Runnable` object passed to a `Thread`, which contains `void run()`
  
  - `Comparator<T>` object passed to a sorted collection (`int compare(T, T)`)
  
  - `ActionListener` object passed to a `JButton` (`void actionPerform()`)
  
  - `HttpHandler` object passed to an `HttpServer` (`void handle()`)

    

###### First Class Functions (3 / 3)

This design pattern is called a *function object*:

- An object whose purpose is to represent a function

- The spec for a *function object* in Java is given by an interface, called a:
  
  - *Single Abstract Method* (*SAM*) interface (contains only 1 method)

    

###### Lambda Expressions (1 / 2)

Java's *lambda expression* syntax provides a succinct way to create *functional object*s

```java
new Thread(new Runnable() {
    public void run() { System.out.println("Hello!");}
}).start();
```

```java
new Thread(
    () -> {System.out.println("Hello!");}
).start();
```

Instead of the first snippet of code, we can have the 2nd one

    

###### Lambda Expressions (2 / 2)

Java still doesn't use *first-class* functions. When a *lambda* is used, the compiler:

- Has to be able to determine the type of *function object* that the lambda creates
  
  - Here the compiler sees the `Thread` constructor takes a `Runnable`
  
  - So it'll infer the type must be `Runnable`

- This inferred type must be a *function interface* (i.e., an *SAM* interface)
  
  - Here indeed within `Runnable`, there's only 1 method (`void run`)
  
  - So the compiler knows the method is an inplementation of `run`

    

###### Concurreny in Event Processing System (1 / 2)

- Using *callback*s inevitably forces the programmer to think about concurrency
  
  - Since control flow is no longer under their control

- A *callback* might be called at any time, and might be from a different thread
  
  - Than the client originally came from

- Java's `GUI` library automatically creates a thread as a `GUI` object is created
  
  - The *event-handling thread* is different from the program's `main` thread
  
  - It runs the *event loop* that's reading from the mouse & keyboard
    
    - And calls listener callbacks

    

###### Concurreny in Event Processing System (2 / 2)

- Likewise, `HttpServer` creates a new thread to listen for incoming connections
  
  - And parse the Http request, then calls the route handler callback

- So both of these systems are concurrent, even though the additional threads:
  
  - Are invisible to the user

    

###### Implementing an Event Source

- All our examples of *callback*s so far have been from the clients' point of view
  
  - The client creates a *callback* function, passes it to the implementor
  
  - And when the desired event occurs, the implementor calls it

- What about the implementer’s side? How does an implementer: 
  
  - keep track of callback functions, and make the calls?  To illustrate this: 
  
  - We’ll implement a simple `Counter` that calls listeners when it increments

    

###### Counter Spec (1 / 3)

`Counter` has several operations, plus the listener interface

```java
public class Counter {
    /* Make a counter initially set to zero. */
    public Counter() { ... }

    /* @return the value of this counter. */
    public synchronized BigInteger number() { ... }

    /* Increment this counter. */
    public synchronized void increment() { ... }

    /**
     * Modifies this counter by adding a listener.
     * @param listener called by this counter when it changes.
     */
    public synchronized void addNumListener(NumberListener listener){...}

    /**
     * Modifies this counter by removing a listener.
     * @param listener will no longer be called by this counter.
     */
    public synchronized void removeNumListener(NumberListener listener){}
}
```

```java
/** A listener for this counter. */
public interface NumberListener {
    /**
     * Called when the counter changes.
     * @param number the new number
     */
    public void numberReached(BigInteger number); 
}
```

    

###### Counter Spec (2 / 3)

So from a client's point of view, we can make a counter and attach listeners to it:

```java
Counter counter = new Counter();
counter.addNumberListener(new NumberListener() {
    public void numberReached(BigInteger number) {
        System.out.println(number);
    }
});
```

Or using a *lambda expression* for the listener:

```java
Counter counter = new Counter();
counter.addNumberListener(number -> System.out.println(number));
```

    

###### Counter Spec (3 / 3)

Whenever `counter.increment` is called, `Counter` calls its listener (print the number)

```java
for (int i = 0; i < 10000; ++i) {
    counter.increment();
} // let's continuously call this method to let it happen
```

    

###### Counter Implementation (1 / 3)

Now let's turn to the implementor's view

- `Counter` needs to track both the number it is counting, and the set of listeners
  
  - Who want to be called back

- So we'll use this straightforward rep, and design for thread safety from the outset
  
  ```java
  private BigInteger number = BigInteger.ZERO;
  private Set<NumberListener> listeners = new HashSet<>();
  // Abstract function
  //  AF(number, listeners) = a counter currently at `number` 
  //   that sends message to `listeners` whenever it changes
  // Rep invariant
  //  true
  // Thread safety argument
  //  uses the monitor pattern -- the rep is guarded by object's lock
  //  acquired on entering every public method
  ```

    

###### Counter Implementation (2 / 3)

The counting itself is handled by `increment()`

```java
public synchronized void increment() {
    number = number.add(BigInteger.ONE);
    callListeners();        
}
```

    

###### Counter Implementation (3 / 3)

```java
public synchronized void addNumberListener(NumberListener listener) {
    listeners.add(listener);
}

public synchronized void removeNumberListener(NumberListener listener) {
    listeners.remove(listener);
}

private void callListeners() {
    for (NumberListener listener : listeners) {
        listener.numberReached(number);
    }
}
```

- `addNumberListener()`/`removeNumberListener()` maintain the set of listeners

- Whenever the counter increments, it calls `callListeners()` to iterate the set
  
  - Calling each listener’s `numberReached()` method

    

###### Summary (1 / 2)

- Callbacks is the 1st example of first-class functions, where we treat them as data

- i.e., passing them to and returning them from functions, storing them to call later

    

###### Summary (2 / 2)

- Callbacks make code more **ready for change** by allowing clients to: 
  
  - Provide their own code to run when an event occurs, so that the behavior:
  
  - Doesn’t have to be hard-coded into the implementation beforehand.

- Writing a single giant input loop to handle every possible event is: 
  
  - Neither **safe from bugs** nor **easy to understand**
  
  - Callbacks allow each module to be responsible for their own events  
