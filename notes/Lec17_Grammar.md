###### Objectives

1. Understand grammar productions & regular expression operators

2. Read a grammar & determine if it matches a sequence of `char`s

3. Write a grammar to match a set of `char`s & parse them into a data structure

    

###### Terminology

Today's reading introduces several ideas:

- Grammars (productions, terminals, non-terminals, operators)

- Regular expressions

    

###### Introduction (1 / 2)

- A program can input or output in the form of a sequence of bytes or characters
  
  - Which is called a `string` when it's stored in memory
  
  - Or a `stream` when it flows into or out of a program

- We'll talk about how to write a spec for such a sequence. The sequence can be:
  
  - A string
  
  - A file on disk, in which case the spec is called the *file format*
  
  - Messages sent over a network, in which case the spec is a *wire protocol*
  
  - A command typed by the user on the console. Spec: *command line interface*

    

###### Introduction (2 / 2)

For these kinds of sequences, we introduce the notion of a *grammar*

- Which not only allows distinguishing legal / illegal sequences, but also:
  
  - Parsing a seq into a data structure that a program can work with
  
  - The data structure produced from a *grammar* is often a recursive data type

- We'll also talk about a special form of *grammar* called *regular expression*s
  
  - It's a widely-used tool for many string-processing tasks, that need to:
  
  - Disassemble a string, extract information from it, or transform it

    

###### Grammars (1 / 2)

- A *grammar* defines a set of strings. Say we write a grammar that represents `URL`s
  
  - Literal strings in a *grammar* are called *terminal*s, since they can't be expanded
  
  - We generally write *terminal*s in quotes, like `'http'`, or `':'`

- A *grammar* is described by a set of *production*s, where each defines a *non-terminal*
  
  - *Non-terminal*s are internal nodes of the tree representing a string

- A *production* in a *grammar* has the form:
  
  ```c
  nonterminal ::= expression of terminals, nonterminals, and operators
  ```

    

###### Grammars (2 / 2)

- One of the *non-terminal*s of the *grammar* is designated as the `root`

- Here's a grammar that represents a singleton set, with only 1 *production*
  
  ```html
  url ::= 'http://mit.edu/'
  ```

    

###### Grammar Operators (1 / 2)

- *Production*s use *operator*s to combine *terminal*s & *nonterminal*s on the right-hand 

- The 3 most important operators are repetition (`*`), concatenation (` `), union (`|`)
  
  ```c
  x ::= y* // x matches zero or more y
  x ::= y z // x matches y followed by z
  x ::= y | z // x matches either y or z 
  ```

- Precedence from high to low: reptition (`*`) > concatenation (` `) > union (`|`) 
  
  ```c
  x ::= c d | a b* // this'd be parsed as: either cd or a, ab, abb, ...
  ```

    

###### Grammar Operators (2 / 2)

- Let's expand our grammar to match `https://standford.edu/`
  
  ```c
  url ::= 'https://' hostname '/'
  hostname ::= 'mit.edu' | 'standford.edu' 
  ```

- Let’s take it one step further by allowing any lowercase word
  
  ```c
  url ::= 'http://' hostname '/'
  hostname ::= word '.' word
  word ::= letter letter*
  letter ::= ('a' | 'b' | 'c' | 'd' | 'e' | 'f' | 'g' | 'h' | 'i' 
                  | 'j' | 'k' | 'l' | 'm' | 'n' | 'o' | 'p' | 'q' 
                  | 'r' | 's' | 't' | 'u' | 'v' | 'w' | 'x' | 'y' | 'z')
  ```

    

###### More Grammar Operators (1 / 2)

You can use operational operators which are just syntactic sugar

- 0 or 1 occurrence: `?`
  
  ```c
  x ::= y?       // an x is a y or is the empty string
  ```

- 1 or more occurrences: `+`
  
  ```c
  x ::= y+       // an x is one or more y, equivalent to x ::= y y*
  ```

- An exact # of occurrences: `{n}`. A range of occurences: `{n,m}`, `{n,}`, or `{,m}`:
  
  ```c
  x ::= y{3}     // an x is three y
                 // equivalent to x ::= y y y 
  
  x ::= y{1,3}   // an x is between one and three y
                 // equivalent to x ::= y | y y | y y y
  
  x ::= y{,4}    // an x is at most four y
                 // equivalent to x ::=   | y | y y | y y y | y y y y
                 // ^--- note can also match an empty string 
  
  x ::= y{2,}    // an x is two or more y
                 // equivalent to x ::= y y y*
  ```

    

###### More Grammar Operators (2 / 2)

- A character class `[...]`: single-char string matching any listed in the bracket
  
  ```c
  x ::= [aeiou]  // equivalent to  x ::= 'a' | 'e' | 'i' | 'o' | 'u'
  ```

- Ranges of characters can be described compactly using `-`:
  
  ```c
  x ::= [a-ckx-z] // equivalent to x ::= 'a'|'b'|'c'|'k'|'x'|'y'|'z'
  ```

- An inverted character class `[^...]`: single-char strings matching any *not* listed:
  
  ```c
  x ::= [^a-c]  // equivalent to  x ::= 'd'|'e'|'f'|...|'0'|
                //  | ... (all other possible characters)
  ```

    

###### Previous URL Example

These additional *operator*s allow the `word` production to be more compact:

```c
url ::= 'http://' hostname '/'
hostname ::= word '.' word
word ::= [a-z]+
```

    

###### Recursion in Grammars (1 / 2)

- Note a `hostname` can have more than 2 components & an optional port number:
  
  ```html
  http://didit.csail.mit.edu:4949/
  ```

- To handle this case, the grammar is now below:
  
  ```c
  url ::= 'http://' hostname (':' port)? '/' 
  hostname ::= word '.' hostname | word '.' word
  port ::= [0-9]+
  word ::= [a-z]+
  ```

- Note the current grammar is recursive. Can rewrite it without recursion:
  
  ```c
  hostname ::= (word '.')+ word
  ```
  
  - Recursion can be eliminated sometimes but not always

    

###### Recursion in Grammars (2 / 2)

- Note what this grammar allows for port numbers are not technically legal
  
  - Since port numbers can only range from 0 to 65535 (2<sup>16</sup>-1)
  
  - We could write a more complex `port` to match only these `int`s
  
  - That’s typically done in a program that uses the grammar, not grammar itself

- There are more things we should do to go farther:
  
  - Generalizing `http` to support other protocols that `URL`s can have
  
  - Generalizing the `/` at the end to a slash-separated path
  
  - Allowing hostnames with the full set of legal characters instead of just `a-z`

    

###### Parse Trees (1 / 2)

- Matching a *grammar* against a *string* can generate a *parse tree*, that shows:
  
  - How parts of the *string* correspond to parts of the *grammar*

- The leaves of the parse tree are labeled with *terminal*s, representing: 
  
  - The parts of the string that have been parsed

- If we concatenate the leaves together in order, we get back the original string
  
  <img title="" src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-03-03-image.png" alt="" width="334">     <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-03-20-image.png" title="" alt="" width="117">

    

###### Parse Trees (2 / 2)

- Internal nodes of the parse tree are labeled with *nonterminal*s

        <img title="" src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-05-37-image.png" alt="" width="329">          <img title="" src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-05-55-image.png" alt="" width="125">

- Another example that involves recursive *nonterminal*s:
  
  <img title="" src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-07-37-image.png" alt="" width="394">   <img title="" src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-07-49-image.png" alt="" width="156">

    

###### Example: Markdown and HTML (1 / 2)

- Let's look at 2 markup languages that represent typographic style in text:
  
  - Markdown: `This is _italic_.`
  
  - HTML: `Here is an <i>italic<i> word.`

- For simplicity:
  
  - Our example `HTML` & `Markdown` grammars will only specify *italic*s
  
  - The plain text cannot use formatting punctuation, like `_` or `<`

    

###### Example: Markdown and HTML (2 / 2)

- Here's the *grammar* for our simplified version of Markdown

        <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-23-17-image.png" title="" alt="" width="311">     <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-23-47-image.png" title="" alt="" width="191">

- Here's the *grammar* for our simplified version of HTML:
  
  <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-24-58-image.png" title="" alt="" width="309">    <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-10-25-28-image.png" title="" alt="" width="223">

    

###### Regular Expressions (1 / 3)

- *Regular expression*'s property: by substituting each *nonterminal* with right-hand side
  
  - You can reduce it down to a single production for the `root` 
  
  - With only *terminal*s and *operator*s on the right-hand side

- Our `URL` grammar is regular. It can be reduced to a single expression:
  
  ```c
  url ::= 'http://' ([a-z]+ '.')+ [a-z]+ (':' [0-9]+)? '/' 
  ```

- But our `HTML` grammar can’t be reduced completely:
  
  ```c
  html ::= ( [^<>]* | '<i>' html '</i>' )*
  ```

    

###### Regular Expressions (2 / 3)

The reduced expression of terminals & operators can be written: 

- In an even more compact form, called a *regular expression*

- It consists just of terminals, parentheses for grouping, and operators

- A regular expression is called *regex* for short. Here are some useful syntax:
  
  ```java
  .   // matches any single character (sometimes excluding newline)
  \d  // matches any digit, same as [0-9]
  \s  // matches any whitespace character, including space, tab, newline
  \w  // matches any word character including underscore, 
      // same as [a-zA-Z_0-9]
  ```

    

###### Regular Expressions (3 / 3)

- Backslash (`\`) is used to *escape* an *operator* so that it could be matched literally
  
  ```html
  \.  \(  \)  \*  \+  \|  \[  \]  \\
  ```

- Using backslashes is important whenever there are terminal characters: 
  
  - That would be confused with special characters

- Because our `url` has `.` as a terminal, need to use a backslash to escape it:
  
  ```html
  http://([a-z]+\.)+[a-z]+(:[0-9]+)?/
  ```

    

###### Using Regular Expressions in Practice (1 / 2)

- Replace all runs of spaces in a string `s` with a single space:
  
  ```java
  String singleSpacedString = s.replaceAll(" +", " ");
  ```

- Match a `URL`:
  
  ```java
  if (s.matches("http://([a-z]+\\.)+[a-z]+(:[0-9]+)?/")) {
      // then s is a url
  }
  ```

    

###### Why do We Have Double Backslashes?

In `s.matches(...)`, we have this snippet of `\\.`, why double slashes?

- Because this string will not be considered as a literal string, but a *regex*

- If we use only `\.`, it'd be parsed as `.`, which is a sign to match any single char

- However, we want it to be a literal dot, so first in java string, `\\`'ll be parsed as `\`

- Then in the *regex*, `\.` will be parsed as a literal dot, rather than a special sign



###### Using Regular Expressions in Practice (2 / 2)

Now let's extract parts of a data like `2023-03-18`

<img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-11-06-18-image.png" title="" alt="" width="567">

- This example uses *named capturing group*s (`?<year>`) to extract:
  
  - Parts of the matched string, and assign name to the part

- Note `?` here doesn't mean 0/1 repetition. Right after an `(`:
  
  - The `?` signals the `()` have special meaning, not just grouping

- *Named capturing group*s can be retrieved by the `group()` method after a match
  
  - `m.group("year")` would return `"2025"`
  
  - `m.group("month")` would return `"03"`
  
  - `m.group("day")` would return `"18"`

    

###### Context-free Grammars

- A language that can be expressed with our system of *grammar*s is *context-free*
  
  - But not all *context-free* grammars are *regular* (e.g., our `HTML`)

- The grammars for most programming languages are also *context-free*. In general:
  
  - Any language with nested structure (like nesting `{}` or `()`) Is context-free

- That description applies to the `Java` grammar, shown here in part:
  
  <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2024-03-25-11-30-09-image.png" title="" alt="" width="510">
