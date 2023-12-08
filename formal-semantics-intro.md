\## What's News

Last night in Washington, DC this year's winners of the [International Obfuscated C Code Contest](https://www.ioccc.org/)were hosted by the current administration and treated to a formal dinner in the White House. During the dinner, the winners reached a historic pact with the Recognizing Testing Functions Matters (RTFM) Union by agreeing to stop torturing their compilers in exchange for a break on having to write so many unit tests.

\## Formal Program Semantics

Formally specifying the syntax of a programming language can only take us so far along the path of understanding the meaning of a program. Regular expressions, tokenizers, lexemes, context-free grammars, and parsers are the tools at our disposal for learning about a program's syntax. Unfortunately, when we limit ourselves to analyzing a program in terms of a context-free grammar there are certain aspects of a program that we \_should\_ be able to analyze statically that we cannot (because, well, they require context to analyze). So, we extended our discussion of syntax formalisms by discussing \_static semantics\_. That may seem like an oxymoron -- determining the meaning of a statement by looking only at the \_syntax\_ (the \_very\_ thing that \_only\_ specifies the \_form\_ of a valid program)? Nevertheless, it's possible (to an extent) using \_attribute grammars\_.

The formal tools that we used to analyze the syntax and form of a program are relatively standardized. On the other hand, there is less of a consensus about how a program language designer formally describes the \_dynamic semantics\_, the meaning of a program when it is executed, of programs written in their language. The codification of the semantics of a program written in a particular is known as \_formal program semantics\_. In other words, formal program semantics are a  precise mathematical description of the semantics of an executing program.​ Sebesta uses the term \_dynamic semantics\_ which he defines as the "meaning[] of the expressions, statements and program units of a programming language." 

The goal of defining formal program semantics is to understand and reason about the behavior of programs. There are many, many reasons why PL designers want a formal semantics of their language. However, there are two really important reasons: With formal semantics it is possible to prove that

1. two programs calculate the same result (in other words, that two programs are equivalent), and
1. a program calculates the correct result.

The alternative to formal program semantics are \_standards\_ promulgated by committees that use natural language to define the meaning of program elements. Here is an example of a page from the standard for the C programming language:

![](./graphics/CSpecificationSemantics.png)

If you are interested, you can find the [C++ language standard](https://isocpp.org/std/the-standard), the [Java language standard](https://docs.oracle.com/javase/specs/), the [C language standard](http://www.open-std.org/jtc1/sc22/wg14/), the [Go language standard](https://golang.org/ref/spec) and the [Python language standard](https://docs.python.org/3/reference/) all online.

\## Testing vs Proving

There is absolutely a benefit to testing software. No doubt about it. However, testing that a piece of software behaves a certain way does not prove that it operates a certain way.

\_"Program testing can be used to show the presence of bugs, but never to show their absence!" - [Edsger Dijkstra​](https://en.wikipedia.org/wiki/Edsger\_W.\_Dijkstra)\_

There is an entire field of computer science known as formal methods whose goal is to understand how to write software that is provably correct. There are systems available for writing programs about which things can be proven. There is [PVS](https://pvs.csl.sri.com/), [Coq](https://coq.inria.fr/), [Isabelle](https://isabelle.in.tum.de/doc/tutorial.pdf), and [TLA+](https://lamport.azurewebsites.net/tla/tla.html), to name a few. PVS is used by NASA to write its mission-critical software and it even makes an appearance in the movie [The Martian](https://shemesh.larc.nasa.gov/fm/pvs/TheMartian/).

\### Three Types of Formal Semantics

There are three common types of formal semantics. It is important that you know the names of these systems, but we will only focus on one in this course!

1. Operational Semantics: The meaning of a program is defined by how the program executes on an idealized virtual machine.
1. Denotational Semantics: Program units "denote" mathematical functions and those functions transform the mathematically defined state of the program.
1. Axiomatic Semantics: The meaning of the program is based on proof rules for each programming unit with an emphasis on proving the correctness of a program.

Again, we will only focus on one of the styles, operational semantics.

\## Operational Semantics

\### Program State

We have referred to the state of the program throughout this course. We have talked about how statements in imperative languages can have side effects that affect the value of the state and we have talked about how the assignment statement's raison d'etre is to change a program's state. Because of the central role that the state plays in the meaning of an imperative programming language, we have to very precisely define a program's state.

At all times, a program has a state. A state is just a function whose domain is the set of \_defined\_ program variables and whose range is $V \times T$ where $V$ is the set of all valid variable values (e.g., `5`, `6.0`, `True`, `"Hello"`, etc) and $T$ is the set of all valid variable types (e.g., $Integer$, $Floating Point$, $Boolean$, $String$, etc). ​In other words, you can ask a state about a particular variable and, if it is defined, the function will return the variable's current value and its type.

The state of a program is abbreviated with the $\sigma$. Again, $\sigma$ is just a function, so we can \_call\_ it:

$$

\sigma(x) = (v, \tau)

$$

you can read the definition above as "Variable \_x\_ has value $v$ and type $\tau$ in state $\sigma$. $\tau$ always represents some arbitrary variable type. Generally, $v$ represents a value.

\## Program Configuration

Between execution \_steps\_ (a term that we will define shortly), a program is always in a particular \_configuration\_:

$$

<e,\sigma>

$$

This means that the program in state $\sigma$ is \_about to\_ evaluate expression $e$. Note well that the configuration describes the expression that is next to be evaluated -- it has not yet been evaluated. You can think of it like the debugger stopped on a particular statement/expression -- that will be the \_next\_ thing to execute.

\## Program Steps

A program step is an atomic (indivisible) change from one program configuration to another. Operational semantics defines steps using rules. The general form of a rule is

$$

\frac{premises}{conclusion}

$$

The conclusion is generally written like

$$

\langle e, \sigma  \rangle \longrightarrow \langle v, \tau, \sigma'  \rangle

$$

This statement is the \_evaluation relation\_ of a rule. The rule defines that, when the premises hold, the program configuration $\langle e,\sigma  \rangle$ evaluates to a value ($v$), type ($\tau$) and (possibly modified) state ($\sigma'$)  after a single step of evaluation. Note that rules \_do not\_ yield configurations. All this will make sense when we see an example.

\*\*Example 1: \_Defining the semantics of variable access.\_\*\*

In STIMPL, the expression to access a variable, say \_i\_, is written like `Variable("i")`. Our operational semantics rule for \_evaluating\_ such an access should "read" something like:

\rangle When the program is about to execute an expression to access variable $i$ in a state $\tau$, the value of the expression will be the triple of $i$'s value and type (in the current state) and that state $\sigma$, unchanged.

In other words, without disturbing the program's current state, the evaluation of the next step of a program that is about to access a value is the value and type of the variable being accessed. To reiterate, accessing a variable \_does not\_ change the value of the program's state.

Let's write that formally!

$$

\frac{\sigma(x)\longrightarrow(v,\tau)}{\langle Variable(x),\sigma \rangle\longrightarrow(v,\tau,\sigma)}

$$

What, exactly, does this mean? Let's look at the premises. The premises are (partly) responsible for determining whether a rule defines the semantics of a program's step.

The premises here say that one of the conditions that determines whether this rule is applicable is that variable $x$ has value $v$ and type $t$ in state $\sigma$. Because our premise includes variables ($v$ and $\tau$), the premises are irrefutable (remember that term from the Python Pattern discussion?). In other words, this premise is really just a means for us to give a symbol to the variable's type and value in a way that we can use in the conclusion of our rule.

Well, what other piece of the rule defines whether it is the applicable rule defining the next step? The left side of the $\longrightarrow$! The other part of the requirement for "choosing" this rule's semantics as the description of the next step is that the program is in state $\sigma$ and the next expression to execute is $Variable(x)$. When \*both\* of those hold, this is the rule that defines the next step!

Now, let's look on the right side of the $\longrightarrow$ to decipher the evaluation result specified by this rule. We see that the result of looking up variable $x$ in state $\sigma$ is just that variable's value, it's type and an unchanged state!

Why are we able to carry the existing state across the program step? Because simply accessing a variable will not have an impact of the state! How cool is that?

\#### State Update

How do we write down a change to the state? Why would we want to change the state? Let's answer the second question first: we want to change the state when, for example, there is an assignment statement. If $\sigma(i)= (4, Integer)$ and then the program evaluated an expression like `Assign(Variable("i"), IntLiteral(2))`, we don't want the $\sigma$ to return $(4, Integer)$ any more! We want it to return $(2, Integer)$. We can define that mathematically like:


$$

\sigma \lbrack (v,\tau) / x \rbrack(y) = \begin{cases}

\sigma (y) &  y \ne x \\

(v, \tau) & y = x

\end{cases}

$$

This function definition means that if you are querying the updated state for the variable that was just reassigned ($x$), then return its new value and type ($m$ and $\tau$). Otherwise, just return the value that you would get from accessing the existing state of the program \_before\_ the update!

\rangle Note: Can you see now why we are implementing the state function in the "odd" way that we described in class? The code that we wrote in Python is almost \_exactly\_ what the mathematical function (above) specifies! Prove it to yourself!

\*\*Example 2: \_Defining the semantics of variable assignment (for a variable that already exists).\_\*\*

In STIMPL, the expression to overwrite the value of an existing variable, say \_i\_, with, say, an integer literal 5 is written like `Assign(Variable("i"), IntLiteral(5))`. Our operational semantic rule for \_evaluating\_ such an assignment should "read" something like:

\rangle When the program is about to execute an expression to assign variable \_i\_ to the integer literal 5 in a state $\sigma$ and the type of the variable \_i\_ in state $\sigma$ is $Integer$, the value of the expression will be the triple of $5, Integer$ and the changed state $\sigma'$ which is exactly the same as state $s\sigma$  except where $(5, Integer)$ replaced \_i\_'s earlier contents."

That's such a mouthful! But, I think we got it. Let's replace some of those literals with symbols for abstraction purposes and then write it down!

$$

\frac{\langle e,\sigma \rangle\longrightarrow\left(v,\tau,\sigma'\right),\sigma'\left(x\right)\longrightarrow\left(\*,\tau\right)}{\langle Assign\left(Variable\left(x\right),e\right),\sigma \rangle\longrightarrow\left(v,\tau,\sigma'\left[\left(v,\tau\right)/x\right]\right)}

$$

Let's look at it step-by-step:

$$

\langle Assign(Variable(x), e), \sigma \rangle

$$

is the configuration and means that we are about to execute an expression that will assign the value of expression \_e\_ to variable \_x\_. But what is the value of expression \_e?\_ The premise

$$

\langle e, \sigma \rangle \longrightarrow (v, \tau,\sigma')

$$

tells us that this rule only applies when the value and type of \_e\_ when evaluated in state $\sigma$ is $v$, and $\tau$. Moreover, the premise tells us that the rule only applies when the state may have changed during evaluation of expression \_e\_ and that subsequent evaluation should use a new state, $\sigma'$. Our mouthful above had another important caveat: the type of the value to be assigned to variable \_x\_ must match the type of the value already stored in variable \_x\_. The second premise

$$

\sigma'(x) \longrightarrow (\*, \tau)

$$

tells us that the type of the expression's value and the type of variable \_i\_ match -- see how the $\tau$s are the same in the two premises? (We use the $\*$ to indicate that the applicability of this rule is not predicated on the variable's value, just its type!).

Now we can just put together everything we have and say that the expression assigning the value of expression \_e\_ to variable \_x\_ evaluates to

$$

(v,\tau,\sigma'[v,\tau)/x])

$$

You did it!!

\## That's Great, But Show Me Code!

Well, Will, that's fine and good and all that stuff. But, how do I \_use\_ this when I am implementing STIMPL? I'll show you! Remember the operational semantics for variable access:

$$

\frac{\sigma(x)\longrightarrow(v,\tau)}{\langle Variable(x),\sigma \rangle\longrightarrow(v,\tau,\sigma)}

$$

Compare that with the code for it's implementation in the STIMPL skeleton that you are provided for Assignment 2:

\```Python

def evaluate(expression, state):

...

case Variable(variable\_name=variable\_name):

value = state.get\_value(variable\_name)

if value == None:

raise InterpSyntaxError(f"Cannot read from {variable\_name} before assignment.")

return (\*value, state)

\```

At this point in the code we are in a function named `evaluate` whose first parameter is the next expression to evaluate and whose second parameter is a state. Does that sound familiar? That's because it's the same as a \_configuration\_! We use \_pattern matching\_ to select the code to execute. The pattern is based on the structure of the `expression` variable and we match in the code above when `expression` is a variable access. Refer to [Pattern Matching in Python](TODO) for the exact form of the syntax. The `state` variable is an instance of the `State` object that provides a method called `get\_value` (see [Assignment 2: Implementing STIMPL](TODO) for more information about that function) that returns a tuple of $(v, \tau)$. In other words, `get\_value` works the same as $\sigma$. So,

\```Python

value = state.get\_value(variable\_name)

\```

is a means of carrying out the premise of the operational semantics.

\```Python

return (\*value, state)

\```

yields the final result! Pretty cool, right?

Let's do the same analysis for assignment:

$$

\frac{\langle e,\sigma \rangle\longrightarrow\left(v,\tau,\sigma'\right),\sigma'\left(x\right)\longrightarrow\left(\*,\tau\right)}{\langle Assign\left(Variable\left(x\right),e\right),\sigma \rangle\longrightarrow\left(v,\tau,\sigma'\left[\left(v,\tau\right)/x\right]\right)}

$$

And here's the implementation:

\```Python

def evaluate(expression, state):

...

case Assign(variable=variable, value=value):

value\_result, value\_type, new\_state = evaluate(value, state)

variable\_from\_state = new\_state.get\_value(variable.variable\_name)

\_, variable\_type = variable\_from\_state if variable\_from\_state else (None, None)

if value\_type != variable\_type and variable\_type != None:

raise InterpTypeError(f"""Mismatched types for Assignment:

Cannot assign {value\_type} to {variable\_type}""")

new\_new\_state = new\_state.set\_value(variable.variable\_name, value\_result, value\_type)

return (value\_result, value\_type, new\_new\_state)

\```

First, look at

\```Python

value\_result, value\_type, new\_state = evaluate(value, state)

\```

which is how we are able to find the values needed to satisfy the left-hand premise. `value\_result` is $v$, `value\_type` is $\tau$ and `new\_state` is $\sigma'$'.

\```Python

variable\_from\_state = new\_state.get\_value(variable.variable\_name)

\```

is how we are able to find the values needed to satisfy the right-hand premise. Notice that we are using `new\_state` ($\sigma'$) to get `variable.variable\_name` ($x$). There is some trickiness in

\```Python

\_, variable\_type = variable\_from\_state if variable\_from\_state else (None, None)

\```

to set things up in case we are doing the first assignment to the variable (which sets its type), so ignore that for now! Remember that in our premises we guaranteed that the type of the variable in state $\sigma$' matches the type of the expression:

\```Python

if value\_type != variable\_type and variable\_type != None:

raise InterpTypeError(f"""Mismatched types for Assignment:

Cannot assign {value\_type} to {variable\_type}""")

\```

performs that check!

\```Python

new\_new\_state = new\_state.set\_value(variable.variable\_name, value\_result, value\_type)

\```

generates a new, new state ($\sigma'[(v,\tau/x])$) and

\```Python

return (value\_result, value\_type, new\_new\_state)

\```

yields the final result!

\## From Theory to Code and Back Again

Now that we have explored how we write down operational semantics for certain constructs and then compared the implementation of an interpreter that evaluates the actual constructs in a living, breathing language, I hope that it is obvious how one leads easily to another. The operational semantics are, in a sense, a mathematically rigorous form of pseudocode. In fact, that pseudocode is \_so\_ similar to the code that we write in the implementation of the interpreter, it's almost a mechanical translation process between the two.

We turn the page from imperative programming and begin our work on object-oriented programming!

\### The Definitions of Object-Oriented Programming

We can define object-oriented programming using four different definitions:

1. A language with support for abstraction of abstract data types (ADTs) (yes, I do mean to say that twice!).​ (from Sebesta)
1. A language with support for objects which are containers of data (attributes, properties, fields, etc.) and code (methods).​ (from [Wikipedia](https://en.wikipedia.org/wiki/Object-oriented\_programming))
1. Pertaining to a technique or a programming language that supports objects, classes, and inheritance; Some authorities list the following requirements for object-oriented programming: information hiding or encapsulation, data abstraction, message passing, polymorphism, dynamic binding, and inheritance. (from ISO in standard 2382:2015)
1. Messaging (i.e., message passing), local retention and protection and  hiding of state-process, and extreme late-binding of all things. (from [personal correspondence with Alan Kay](http://userpage.fu-berlin.de/~ram/pub/pub\_jf47ht81Ht/doc\_kay\_oop\_en))

As graduates of CS1021C and CS1080C, the second definition is probably not surprising. The first definition, however, leaves something to be desired. Using Definition (1) means that we have to a) know the definition of \_abstraction\_ and \_abstract data types\_ and b) know what it means to apply abstraction to ADTs.

\### Abstraction (Reprise)

There are two fundamental types of abstractions in programming: process and data. We have talked about the former but the latter is new. When we talked previously about process abstractions, we did attempt to define the term abstraction and even had some success. Let's review that discussion and our conclusions.

\_Sebesta\_ formally defines \_abstraction\_ as the view or representation of an entity that includes only the most significant attributes. This definition seems to align with our notion of abstraction especially the way we use the term in phrases like "abstract away the details." It didn't feel like a good definition to me until I thought of it this way:

Consider that you and I are both humans. As humans, we are both carbon based and have to breath to survive. But, we may not have the same color hair. I can say that I have red hair and you have blue hair to point out the significant attributes that distinguish us. I need not say that we are both carbon based and have to breath to survive because we are both human and we have "abstracted away" those facts into our common humanity.

Again, this definition of abstraction feels intuitive especially when we put it together with the notion of inheritance (read on!).

There's another interesting definition of \_abstraction\_ that I think works slightly better. Let's try it on for size:

\_Abstraction\_: Removing the detail to simplify and focus attention on the essence.

Oh, I like that! Or, if that doesn't suit your fancy, how about this:

\_Abstraction\_: Remembering what is important in a given context and forgetting what's not.

That's pithy!

(Kramer, Jeff. Is Abstraction the Key to Computing? \_Communications of the ACM.\_ April 2007. Vol. 50, No. 4)

\### Abstract Data Types (ADTs)

We are all very familiar with process abstraction but we need to spend time talking about it's powerful sibling: \_data abstraction\_. As functions, procedures and methods are the syntactic and semantic means of abstracting processes in programming languages, ADTs are the syntactic and semantic means of abstracting \_data\_ in programming languages. ADTs combine (\_encapsulate\_) data (usually called the ADT's \_attributes\_, \_properties\_, etc) and operations that operate on that data (usually called the ADT's \_methods\_) into a single entity.

If that sounds almost exactly like the definition of a type, that's because ... well, it is! Remember how we said that a type defines

1. The range of valid values; and
1. The range of valid operations on those values?

Well, (1) for ADTs is the data that we encapsulated (those properties or attributes) and (2) are the methods.

\_Hiding\_ is a significant advantage of ADTs. ADTs hide the data being represented and allow its manipulation only through pre-defined methods, the ADT's \_interface\_. The interface typically gives the ADT's user the ability to manipulate/access the data internal to the type and perform other semantically meaningful operations (e.g., sorting a list).

Although the term may be new, I am sure that you have used ADTs before:

1. Stack
1. Queue
1. List
1. Array
1. Dictionary
1. Graph
1. Tree
1. Heap

These are are so-called \_user-defined ADTs\_ because they are defined by the user (deep, huh?) of a programming language. In other words, they are not built in to the language by the language designers. A user-defined ADT can be composed of primitive data types \_and\_ other user-defined ADTs.

Here's something to consider: Are primitives a type of ADT? A primitive type, like floating point numbers, would seem to meet the definition of an abstract data type:

1. It's underlying representation is hidden from the user (the programmer does not care whether FPs are represented according to IEEE754 or some other specification)
1. There are operations that manipulate the data (addition, subtraction, multiplication, division).

\#### Parameterized ADTs

One of the most \_fundamental\_ ADTs in the computer scientist's toolchest is the stack. I am sure that you have all worked with a stack before. The stack \_contains\_ a list of items chronologically ordered according to the time that they were added to the list. The stack is a last-in-first-out (LIFO) data structure. The stack supports the following operations:

1. top: Retrieve a copy of the item most recently inserted into the list.
1. pop: Retrieve the item most recently inserted into the list and remove it from the list.
1. push: Add a new element to the list.

As developers we obviously want to know, just \_what type\_ of elements can we store in this "list"? Well, anything really, we just need to write the code appropriately. Let's say that we are writing a simple integer calculator so we will need a stack of numbers. In Java, we would have something that looked like this:

\```Java

package edu.uc.cs.cs3003;

import java.util.ArrayList;

import java.util.EmptyStackException;

import java.util.List;

public class Stack {

Stack() {

m\_data = new ArrayList<Integer>();

}

public void push(Integer newelement) {

m\_data.add(newelement);

}

public Integer top() throws EmptyStackException {

if (m\_data.size() == 0) {

throw new EmptyStackException();

}

return m\_data.get(m\_data.size() - 1);

}

public Integer pop() throws EmptyStackException {

if (m\_data.size() == 0) {

throw new EmptyStackException();

}

Integer top = top();

m\_data.remove(m\_data.size() - 1);

return top;

}

private List<Integer> m\_data;

}

\```

Great. We have all the operations that we need. We can easily use that data structure in a Java program like this:

\```Java

package edu.uc.cs.cs3003;

import java.util.EmptyStackException;

public final class App {

private static void TestStack() {

Stack stack = new Stack();

stack.push(5);

stack.push(10);

stack.push(15);

stack.push(20);

System.out.println(stack.pop());

System.out.println(stack.pop());

System.out.println(stack.pop());

System.out.println(stack.pop());

try {

System.out.println(stack.pop());

} catch (EmptyStackException ese) {

System.out.println("Oops: " + ese.toString());

}

}

public static void main(String[] args) {

TestStack();

}

}

\```

When we run it, we see the output:

\```console

20

15

10

5

Oops: java.util.EmptyStackException

\```

Perfect! Exactly what we wanted. Even the logical error that we (I) committed is caught.

There's just one. small. problem. What do we do if there comes a time when I want to use my Stack ADT in an application that is not a calculator? In other words, I will need the Stack ADT to contain elements that are, say, characters when I am using a Stack ADT to determine if two strings are palindromic.

I guess I could just write the entire ADT again. I could name it something like `CharacterStack`. That just leads us to another question: What if, now, I need a Stack ADT that will let me contain elements that are Pancakes (what better thing in the world is there than a stack of pancakes?). I rewrite the Stack ADT and name it something like `PancakeStack` ... where does the craziness stop??

It stops right here, with \_parameterized ADTs\_: An ADT that can store elements of any type. (alternate definition: An ADT parameterized with respect to the component type [[courtesy](http://www.cs.kent.edu/~durand/CS43101Fall2004/DT-UserDefinedADT.html)]).

In Java, these parameterized ADTs are known as \_generics\_ and they look like templates in C++. Here is how we would write the same Stack in a generic manner:

\```Java

package edu.uc.cs.cs3003;

import java.util.ArrayList;

import java.util.EmptyStackException;

import java.util.List;

public class GStack<ElementType> {

GStack() {

m\_data = new ArrayList<ElementType>();

}

public void push(ElementType newelement) {

m\_data.add(newelement);

}

public ElementType top() throws EmptyStackException {

if (m\_data.size() == 0) {

throw new EmptyStackException();

}

return m\_data.get(m\_data.size() - 1);

}

public ElementType pop() throws EmptyStackException {

if (m\_data.size() == 0) {

throw new EmptyStackException();

}

ElementType top = top();

m\_data.remove(m\_data.size() - 1);

return top;

}

private List<ElementType> m\_data;

}

\```

`ElementType` here is a parameter to the class and will help Java create versions of the `Stack` class that are customized to the type of data that programmers want to store in their stacks.

`ElementType` acts like a variable, albeit a special variable. All variables in Java have types. `ElementType` has a type, too, and, as we know, that types define the range of valid values that a variable of that type can hold. So, just what are those valid values for a variable that represents a generic?

Drumroll ...

\_types\_

The \_type variable\_ named `ElementType` can hold values like `String` or `int` or `float` or ... you get the idea. The class above, `Stack` is said to be \_parameterized\_ by a type. As you know, with functions, parameters usually need arguments. The same is true for type parameters. In order to instantiate an instance of the parametric `Stack` type, we need to give it a \_type argument\_. As arguments give a value to a parameter in a function, a type argument gives a value to the type parameter. As such, it should be one of the type variable's valid values.

In order to instantiate an element of the parametric `Stack` ADT that will hold, say, `char`acters, we would write something like

\```Java

GStack<Character> stack = new GStack<Character>();

\```

Let's use that to write something that is more than trivial to see how we can harness the power of generic types:

\```Java

private static void TestGStack() {

GStack<Character> stack = new GStack<Character>();

stack.push('a');

stack.push('b');

stack.push('c');

stack.push('d');

System.out.println(stack.pop());

System.out.println(stack.pop());

System.out.println(stack.pop());

System.out.println(stack.pop());

try {

System.out.println(stack.pop());

} catch (EmptyStackException ese) {

System.out.println("Oops: " + ese.toString());

}

}

\```

And look at this amazing output:

\```console

d

c

b

a

Oops: java.util.EmptyStackException

\```

If you want to see all the code for this demo so that you can explore it yourself, it's online [here](https://github.com/hawkinsw/cs3003/tree/main/adts/stack).

\### What's News

Mathematicians have completed a longitudinal study on the fortunes present in the cookies of many restaurants throughout the United States. While they were not able to consistently determine the veracity of most of the predictions, they were able to prove that one came true more often than not: the ominous message, "You are about to come in to great wealth" foretold a person receiving a massive inheritance.

\### Overriding in OOP

Inheritance is the formalism that describes the way \_a language supports abstraction of ADTs\_ (and, therefore, makes it a nominally OO language) and provides a practical benefit in software engineering: it allows developers to build hierarchies of types which allow developers to build pieces of code that share data and functionality with one another.

Hierarchies are composed in pairs of classes -- one is the superclass and the other is the subclass. A superclass could conceivably be itself a subclass (to some other class). A subclass could itself be a superclass (to some other class). In terms of a family tree, we could say that the subclass is a descendant of the superclass (It is important to note that the terms \_superclass\_ and \_subclass\_ are generic terms used by people who study object-oriented programming languages and may not necessarily be the words that a particular programming language chooses to refer to those adjectives; in particular, C++ refers to superclasses as \_base\_ classes and subclasses as \_derived\_ classes).

A subclass inherits both the data and methods from its superclass(es). However, as Sebesta says, sometimes "... the features and capabilities of the [superclass] are not quite right for the new use." \_Overriding\_ methods allows the programmer to keep most of the functionality of the superclass and customize the parts that are "not quite right."

An \_overridden\_ method is defined in a subclass and replaces the method with the same name (and usually protocol) in the parent.

The official documentation and tutorials for Java describe overriding in the language this way: ["An instance method in a subclass with the same signature (name, plus the number and the type of its parameters) and return type as an instance method in the superclass \_overrides\_ the superclass's method."](https://docs.oracle.com/javase/tutorial/java/IandI/override.html) The exact rules for overriding methods in Java are [online at the language specification](https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.4.8.1).

Let's make it concrete with an example:

\```Java

class Car {

protected boolean electric = false;

protected int wheels = 4;

Car() {

}

boolean ignite() {

System.out.println("Igniting a generic car's engine!");

return true;

}

}

class Tesla extends Car {

Tesla() {

super();

electric = true;

}

@Override

boolean ignite() {

System.out.println("Igniting a Tesla's engine!");

return true;

}

}

class Chevrolet extends Car {

Chevrolet() {

super();

}

@Override

boolean ignite() {

System.out.println("Igniting a Chevrolet's engine!");

return false;

}

}

\```

In this example, `Car` is the superclass of `Tesla` and `Chevrolet`. The `Car` class defines a method named `ignite`. That method will \_ignite\_ the engine of the car -- an action whose mechanics differ based on the car's type. In other words, this is a perfect candidate for overriding. Both `Tesla` and `Chevrolet` implement a method with the same name, return value and parameters, thereby meeting Java's requirements for overriding. In Java, the `@Override` is known as an annotation. Annotations are ["a form of metadata [that] provide data about a program that is not part of the program itself."](https://docs.oracle.com/javase/tutorial/java/annotations/) Annotations in Java are attached to particular syntactic units. In this case, the `@Override` annotation is attached to a method and it tells the compiler that the annotated method is overriding a method from its superclass. If the compiler does not find a method in the superclass(es) that is capable of being overridden by the method, an error is generated. This is a good check for the programmer. (Note: C++ offers similar functionality through the [override specifier](https://en.cppreference.com/w/cpp/language/override).)

Let's say that the programmer actually implemented the `Tesla` class like this:

\```Java

class Tesla extends Car {

Tesla() {

super();

electric = true;

}

@Override

boolean ignite(int testing) {

super.ignite();

System.out.println("Igniting a Tesla's engine!");

return true;

}

}

\```

The `ignite` method implemented in `Tesla` \_does not\_ override the `ignite` method from `Car` because it has a different set of parameters. The `@Override` annotation tells the compiler that the programmer thought they were overriding something. An error is generated and the programmer can make the appropriate fix. Without the `@Override` annotation, the code will compile but produce incorrect output when executed.

Assume that the following program exists:

\```Java

public class CarDemo {

public static void main(String args\[\]) {

Car c = new Car();

Car t = new Tesla();

Car v = new Chevrolet();

c.ignite();

t.ignite();

v.ignite();

}

}

\```

This code instantiates three different cars -- the first is a generic `Car`, the second is a `Tesla` and the third is a `Chevrolet`. Look carefully and note that the type of each of the three is actually stored in a variable whose type is `Car` and not a more-specific type (i.e., not a `Tesla` or a `Chevy`). This is part of the beauty and power of object-oriented programming: I can treat an instance of the `Tesla` class as an instance of the `Car` class because a `Tesla` "\_is a\_" `Car`. See [\*\*Inheritance\*\*](https://learning.oreilly.com/library/view/the-object-oriented-thought/9780135182130/ch01.xhtml#:-:text=Inheritance) in Chapter 1 of \_Object-oriented Thought Process\_ (Weisfeld, M. (2019). Object-oriented thought process, the (5th ed.). Pearson Education.) for more information on the is-a relationship.

What's more, thanks to \_dynamic dispatch\_ no matter that the programmer treats the object as a `Car`, the methods associated with the actual type of the object will be invoked at runtime. With dynamic dispatch, at runtime, the JVM will find the proper `ignite` function and invoke it according to the \_actual\_ type of the variable on which the member function is invoked and not its static type, the type known to the compiler when code is generated. Because `ignite` is overridden by `Chevy` and `Tesla`, the output of the program above is:

\```console

Igniting a generic car's engine!

Igniting a Tesla's engine!

Igniting a Chevrolet's engine!

\```

Most OOP languages provide the programmer the option to invoke the method they are overriding from the superclass. Java is no different. If an overriding method implementation wants to invoke the functionality of the method that it is overriding, it can do so using the `super` keyword.

\```Java

class Tesla extends Car {

Tesla() {

super();

electric = true;

}

@Override

boolean ignite() {

super.ignite();

System.out.println("Igniting a Tesla's engine!");

return true;

}

}

class Chevrolet extends Car {

Chevrolet() {

super();

}

@Override

boolean ignite() {

super.ignite();

System.out.println("Igniting a Chevrolet's engine!");

return false;

}

}

\```

With these changes, the program now outputs:

\```console

Igniting a generic car's engine!

Igniting a generic car's engine!

Igniting a Tesla's engine!

Igniting a generic car's engine!

Igniting a Chevrolet's engine!

\```

What if the programmer does not want a subclass to be able to customize the behavior of a certain method? For example, no matter how a programmer subclasses `Dog`, it's noise method is always going to bark -- no inheriting class should be allowed to change that. Java provides the `final` keyword to guarantee that the implementation of a method cannot be overridden by a subclass. Let's change the code for the classes from above to look like this:

\```Java

class Car {

protected boolean electric = false;

protected int wheels = 4;

Car() {

}

void start() {

System.out.println("Starting a car ...");

if (this.ignite()) {

System.out.println("Ignited the engine!");

} else {

System.out.println("Did NOT ignite the engine!");

}

}

final boolean ignite() {

System.out.println("Igniting a generic car's engine!");

return true;

}

}

class Tesla extends Car {

Tesla() {

super();

electric = true;

}

@Override

boolean ignite() {

super.ignite();

System.out.println("Igniting a Tesla's engine!");

return true;

}

}

class Chevrolet extends Car {

Chevrolet() {

super();

}

@Override

boolean ignite() {

super.ignite();

System.out.println("Igniting a Chevrolet's engine!");

return false;

}

}

\```

Notice that `ignite` in the `Car` class has a `final` before the return type. This makes `ignite` a [final method](https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.4.3.3): "A method can be declared `final` to prevent subclasses from overriding or hiding it". (C++ has something similar -- [the `final` specifier](https://en.cppreference.com/w/cpp/language/final).) Attempting to compile the code above produces this output:

\```console

CarDemo.java:30: error: ignite() in Tesla cannot override ignite() in Car

boolean ignite() {

^

overridden method is final

CarDemo.java:43: error: ignite() in Chevrolet cannot override ignite() in Car

boolean ignite() {

^

overridden method is final

2 errors

\```

\### Subclass vs Subtype

In OOP there is fascinating distinction between subclasses and subtypes. All those classes that inherit from other classes are considered subclasses. However, they are not all subtypes. For a type/class $S$ to be a subtype of type/class $T$, the following must hold

\> Assume that $\phi(t)$ is some provable property that is true of \_t\_, an object of type \_T\_. Then $\phi(s)$ must be true as well forÂ \_s,\_ an object of type \_S\_.

This formal definition can be phrased simply in terms of behaviors: If it is possible to pass objects of type T as arguments to a function that expects objects of type S without any change in the behavior, then S is a subtype of T. In other words, a true subtype behaves exactly like the "supertype".

Barbara Liskov who pioneered the definition and study of subtypes [put it this way](https://doi-org.uc.idm.oclc.org/10.1145/62139.62141): "If for each object $o\_1$ of type $S$ there is an object $o\_2$ of type $T$ such that for all programs $P$ defined in terms of $T$, the behavior of $P$ is unchanged when $o\_1$ is substituted for $o\_2$, then $S$ is a subtype of $T$."

In software engineering, it is likely that when you write a subclass you are really aiming to write a subtype, as much as possible. However, there are cases where you want the behavior of a subtype to differ from the behavior of its associated supertype.

The power of inheritance, virtual methods and dynamic is limitless. However, it's important to recognize that there is a difference between subclasses and subtypes and that you should be careful what you expect from a person providing an argument to a function (or method!).

\### Open Recursion

Open recursion in an OO PL is a fancy term for the combination of a) functionality that gives the programmer the ability to refer to the current object from within a method (usually through a variable named `this` or `self`) and b) dynamic dispatch. Thanks to open recursion, some method `A` of class `C` can call some method `B` of the same class. [But wait, there's more!](https://en.wikipedia.org/wiki/Ron\_Popeil) Continuing our example, in open recursion, if method `B` is overriden in class `D` (which is a subclass of `C`), then the overriden version of the method is invoked when called from method `A` on an object of type `D` \_even though method `A` is implemented entirely within class `C`\_. That's a mouthful, but it certainly sounds wild! It is far easier to see this work in real life than talk about it abstractly. So, consider our cars again:

\```Java

class Car {

protected boolean electric = false;

protected int wheels = 4;

Car() {

}

void start() {

System.out.println("Starting a car ...");

if (this.ignite()) {

System.out.println("Ignited the engine!");

} else {

System.out.println("Did NOT ignite the engine!");

}

}

boolean ignite() {

System.out.println("Igniting a generic car's engine!");

return true;

}

}

class Tesla extends Car {

Tesla() {

super();

electric = true;

}

@Override

boolean ignite() {

System.out.println("Igniting a Tesla's engine!");

return true;

}

}

class Chevrolet extends Car {

Chevrolet() {

super();

}

@Override

boolean ignite() {

System.out.println("Igniting a Chevrolet's engine!");

return false;

}

}

\```

The `start` method is only implemented in the `Car` class. At the time that it is compiled, the `Car` class has no awareness of any subclasses (i.e,, `Tesla` and `Chevrolet`). Let's run this code and see what happens:

\```Java

public class CarDemo {

public static void main(String args\[\]) {

Car c = new Car();

Car t = new Tesla();

Car v = new Chevrolet();

c.start();

t.start();

v.start();

}

}

\```

Here's the output:

\```console

Starting a car ...

Igniting a generic car's engine!

Ignited the engine!

Starting a car ...

Igniting a Tesla's engine!

Ignited the engine!

Starting a car ...

Igniting a Chevrolet's engine!

Did NOT ignite the engine!

\```

Wow! Even though the implementation of `start` is entirely within the `Car` class and the `Car` class knows nothing about the `Tesla` or `Chevrolet` subclasses, when the `start` method is invoked on object's of those types, the call to `this`'s `ignite` method triggers the execution of code specific to the type of car!

How cool is that?

\## What's News

Last night the Westminster (OS) Kernel Club crowned its 2023 Best In Show. The winner? A short-haired pointer named [Tony](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/).

\## Pointers

Although we programmers tend to freak out when we hear the word "pointer", it is comforting to recall that pointers are like any other type -- they have a range of valid values and a set of valid operations that you can perform on those values. What are the range of valid values for a pointer? All valid memory addresses (but see below). And what are the valid operations? Creation, addition, subtraction, dereference, and assignment (Note: the specific operations that are available for pointers may differ between languages).

![](./graphics/Pointer.png)

In the diagram, the gray area is the memory of the computer. The blue box is a pointer. It points to the gold area of memory.

\> To keep things from getting repetitive, we may also say that a pointer "targets" a piece of memory or that a pointer has a "target".

It is important to remember that pointers and their targets \_both\_ exist in memory! In fact, in true [Inception](https://www.imdb.com/title/tt1375666/) style, a pointer can point to a pointer! If the programmer \_dereferences\_ a blue pointer, they are accessing the memory targeted by that pointer.

In some languages, pointers are used primarily to point to untyped \_memory cells\_, areas of a computer's memory. In these languages, the programmer gets no support in determining the meaning of what is in memory at a pointer's target.

In other languages, it is possible to think of pointers "having" types. In languages like these, the programmer is guided into thinking that pointers always target places in memory that actually hold variables. It is for this reason that the type of pointers in languages such as these need longer names. For example, if the gold box held an object of type `T`, then the pointer stored in the blue box would have type "pointer to type `T`." In these languages, it would be ideal if the type of the pointer and the type of the variable stored at the target of the pointer are the same.Â  However, that's not always the case.

When we were discussing the nature of the \_type\_ of pointers, we specified that the range of valid values for a pointer are all memory addresses. In some languages this may be true. However, some other languages specify that the range of valid values for a pointer are all memory addresses \_and\_ a special \_null\_ value that can be used to explicitly signal that a pointer does not point to a target.

Like any other type in a programming language, a certain set of operations can be performed on a pointers. We discussed already the \_dereference\_ operation. In some languages it is possible to \_add\_ and \_subtract\_ pointer variables (with different results in different languages). Certainly it is the case that you can \_(re)assign\_ pointers to target different pieces of memory.

The only operation that we have failed to discuss so far is the operation that helps us find a region of memory to target in the first place. After all, we don't want to just guess where in the computer's vast memory is the region we want to target. That is the \_creation\_ operation. In C/C++, the `&` operator can be used to get the address of a variable in memory which can be assigned to a pointer. In other words, the \_address of\_ operator in C/C++ gives us the power to precisely specify the region in the computer's memory where we want a pointer to target.

\## The Pros of Pointers

Though a very [famous and influential computer scientist](https://en.wikipedia.org/wiki/Tony\_Hoare) once called his invention of \_null references\_ a "billion dollar mistake" (he low balled it, I think!), the presence and power of pointers in a language is important for at least two reasons:

1. Without pointers, the programmer could not utilize the power of indirection.
1. Pointers give the programmer the power to address and manage heap-dynamic memory.

\_Indirection\_ gives the programmer the power to link between different objects in memory -- something that makes writing certain data structures (like trees, graphs, linked lists, etc) easier. Management of heap-dynamic memory gives the programmer the ability to allocate, manipulate and deallocate memory at runtime. Without this power, the programmer would have to know \_before execution\_ the amount of memory their program will require. Although memory allocation at runtime seems initially like just a neat parlor trick, there are certain applications that cannot be written without it. Remember the criticism of Pascal from Brian Kernighan that we discussed? Brian lamented the fact that there is no dynamic allocation of strings. As a result, he said, he was forced to make some nasty choices and build sloppy workarounds.

\## Heap-Dynamic Memory: A Digression

We have talked on more than one occasion about the \_stack\_ and its utility in providing space for a local variable during its lifetime. We have talked less (but have mentioned it, at least!) about the memory used to store static variables during the lifetime of those variables. But we have not talked at all about the memory used to store heap-dynamic variables during their lifetime.

Given the term, it seems obvious that the storage for those variables will be in the heap. But just what is the heap? In a modern multiprocessing operating system, each program believes that they have complete access to the \_entire\_ memory of the computer. In a sense they are right and in a sense they are woefully wrong!

The operating system stage manages the illusion to make programmers' lives easier. So, we'll suspend disbelief, too, and go with the illusion. You learned that that stack (the memory region and not the data structure) is both the place with storage for stack-local variables and the stack frames themselves and it grows down. Even though our programs don't have to coexist (knowingly) with other user applications, every process must make space for the kernel. Therefore, the kernel must exclusively own some space in memory so that it can continue to always be executing. That explains most of the figure below.

![](./graphics/A%20Map%20of%20Memory%20(Linux%20Operating%20System,%20x86).png)

The only remaining item to locate is the Heap. The Heap is memory (and an area of memory) that exists on the opposite end of the address space from the stack. It grows in the opposite direction (as the operating system allocates pieces of it to programs/programmers for their use). Memory that is \_dynamically allocated\_ while the program is executing comes from the heap (almost exclusively!). It is memory in this area that pointers are most often used to address. Memory in the heap is dynamically allocated to user programs by the kernel on demand. The programmer uses an API provided by the language (or the system) in order to ask the operating system for the exclusive rights to a certain subset of the computer's memory. Pointers target the allocated resources and the programmer can use that pointer to update its value or release it back to the system for other programs to use.

\## The Cons of Pointers

Their use as a means of indirection and heap-dynamic memory management are powerful, but misusing either can cause serious problems.

\### Possible Problems when Using Pointers for Indirection

As long as a pointer targets memory that contains the expected type of object, everything is a-okay. Problems arise, however, when the target of the pointer is an area in memory that does not contain an object of the expected type (maybe even garbage) and/or the pointer targets an area of memory that is inaccessible to the program (or any reason).

The former problem can arise when code in a program writes to areas of memory beyond their control (this behavior is usually an error, but is very common). It can also arise because of a \_use after free\_ error. As the name implies, a use-after-free error occurs when a programmer uses memory after it has been freed (brilliant, I know). There are two common scenarios that give rise to a use-after-free error:

1. Scenario 1:
1. One part of a program (part A) frees an area of memory that held a variable of type `T` that it no longer needs
1. Another part of the program (part B) also has a pointer to that very memory
1. A third part of the program (part C) overwrites that "freed" area of memory with a variable of type `S`
1. Part B accesses the memory assuming that it still holds a variable of Type `T`
1. Scenario 2:
1. One part of a program (part A) frees an area of memory that held a variable of type `T` that it no longer needs
1. Part A never nullifies the pointer it used to point to that area of memory though the pointer is now invalid because the program has released the space
1. A second part of the program (part C) overwrites that "freed" area of memory with a variable of type `S`
1. Part A incorrectly accesses the memory using the invalid pointer assuming that it still holds a variable of Type `T`

Scenario 2 is depicted visually in the following scenario and intimates why use-after-free errors are considered security vulnerabilities:

![](./graphics/Use%20After%20Free.png)

In the example above, the program's use of the invalid pointer means that the user of the invalid pointer can now access an object that is at a higher privilege level (Restricted vs Regular) than the programmer intended. When the programmer calls a function through the invalid pointer they expect that a method on the Regular object will be called. Unfortunately, a method on the Restricted object will be called instead. Trouble!

The problem of a programmer's use of a pointer to memory beyond their control is equally as troublesome. This situation most often occurs when the program sets a variable's address to the special value that indicates the \_absence\_ of value (e.g., `NULL`, `null`, `nil`) to indicate that it is invalid but later uses that pointer without checking its validity. For compiled languages this often results in the dreaded \_segmentation fault\_ and for interpreted languages it often results in other anomalous behavior (like Java's \_Null Pointer Exception (NPE)\_). Neither are good!

\### Possible Solutions

Wouldn't it be nice if we had a way to make sure that a pointer being dereferenced is valid so we don't fall victim to some of the aforementioned problems? What would be the requirements of such a solution?

1. Pointers to areas of memory that have been deallocated cannot be dereferenced.
1. The type of the object at the target of a pointer always matches the programmer's expectation.

Sebesta describes two potential solutions that meet these requirements. First, are tombstones.

Tombstones are essentially an intermediary between a pointer and its target. When the programming language implements pointers with tombstones for protection, it allocates a new tombstone for each pointer the programmer generates. The programmer's pointer targets the tombstone and the tombstone targets the pointer's actual target. The tombstone also contains an extra bit of information: whether it is valid. When the programmer first instantiates a pointer to some target \_a\_, the compiler/interpreter

1. generates a tombstone whose target is \_a\_
1. sets the valid bit of the tombstone to \_valid\_
1. points the programmer's pointer to the tombstone.

When the programmer dereferences their pointer (in an attempt to access the memory at its target), the compiler/runtime will check to make sure that the target tombstone's valid flag is set before doing anything else.

The compiler/interpreter guarantees that when the programmer "destroys" a pointer (by releasing the memory at its target or by some other means), the target tombstone's valid flag is set to invalid. As a result, if the programmer later attempts to dereference the pointer after it was destroyed, the compiler/runtime will see that the tombstone's valid flag is invalid and generate an appropriate error.

This process is depicted visually in the following diagram.

![](./graphics/Tombstones.png)

This seems like a great solution! Unfortunately, there are downsides. In order for the tombstone to provide protection for the duration of the program's execution, once a tombstone has been allocated to protect a particular pointer, it cannot be reclaimed. It must remain in place forever because it is always possible that a programmer can incorrectly reuse an invalid pointer \_sometime real soon now\_. As soon as the tombstone is deallocated, the protection that it provides is gone. The other problem is that the use of tombstones adds an additional layer of indirection to dereference a pointer and every indirection causes memory accesses. Though memory access times are small, they are not zero -- the cost of these additional memory accesses add up and ultimately significantly affects program performance (especially programs that perform large amounts of I/O).

What about a solution that does not require an additional level of indirection? There is a so-called lock-and-key technique. This protection method requires that the pointer hold an additional piece of information beyond the address of the target: the key. The memory at the target of the pointer is also required to hold a key. When the system allocates memory it sets the keys of the pointer and the target to be the same value. When the programmer dereferences a pointer, the two keys are compared and the operation is only allowed to continue if the keys are the same. The process is depicted visually below.

![](./graphics/Lock%20and%20Key.png)

With this technique, there is no additional memory access -- that's good! However, there are still downsides. First, there is a speed cost. For every dereference there must be a check of the equality of the keys. Depending on the length of the key, that can take a significant amount of time. Second, there is a space cost. Every pointer and block of allocated memory now must have enough space to store the key in addition to storing the value. For systems where memory allocations are done in big chunks, the relative size overhead of storing, say, an 8 byte key is not significant. However, if the system allocates many small areas of memory, the relative size overhead is tremendous. Moreover, the more heavily the system relies on pointers, the more space will be used to store keys rather than meaningful data.

Well, let's just make the keys smaller? Great idea. There's only one problem: The smaller the space reversed to hold the keys, the fewer unique key values. Fewer unique key values mean that it is more likely an invalid pointer randomly points to a chunk of memory with a matching key ([pigeonhole principle](https://en.wikipedia.org/wiki/Pigeonhole\_principle)). In this scenario, the protection afforded by the scheme is vitiated. (I just wanted to type that word -- I'm not even sure I am using it correctly!)

\## Introduction to Functional Programming

As we said at the beginning of the semester when we were learning about programming paradigms, FP is very different than imperative programming. In imperative programming, developers tell the computer how to do the operation. While functional programming is \_not\_ logic programming (where developers just tell the computerÂ \_what\_ to compute and leave the \_how\_ entirely to the language implementation), the writer of a program in a functional PL is much more concerned with specifying what to compute than how to compute it.

![](./graphics/Programming-Language-What-How-Continuum.png)

\### Four Characteristics of Pure, Strongly-Typed Functional Programming

In this class, we are going to study a subset of functional programming languages. We are going to study pure, strongly-typed functional programming languages. There are four characteristics that epitomize these languages:

1. There is no state
1. Functions are central
1. Functions can be parameters to other functions
1. Functions can be return values from other others
1. Program execution \_is\_ function evaluation
1. Control flow is performed by recursion and conditional expressions
1. Lists are a fundamental data type

In a pure, strongly-typed functional programming language, there are no variables, per se. And because there are no variables, there is no state. That does \_not\_ mean there are no names. Names are still important. It simply means that names refer to expressions themselves and not their values. The distinction will become more obvious as we continue to learn more about writing programs in functional languages.

Because there is no state, a pure, strongly-typed functional programming language is not \_history sensitive.\_ A language that is \_history sensitive\_ means that results of operations in that language can be affected by operations that have come before it. For example, in an imperative programming language, a function may be history sensitive if it relies on the value of a global variable to calculate its return value. Why does that count as history sensitive? Because the value in the global variable could be affected by prior operations.

A language that is not history sensitive has \_referential transparency.\_ We learned the definition of referential transparency before, but now it might make a little more sense. In a language that has referential transparency, the same function called with the same arguments generates the same result no matter what operations have preceded it.

In all functional programming languages, there are no loops (unless they are added as syntactic sugar) -- recursion is \_the\_ way to accomplish repetition. Selective execution (as opposed to sequential execution) is accomplished using the \_conditional expression\_. A conditional expression is, well, an \_expression\_ that evaluates to one of two values depending on the value of a condition. We have seen conditional expressions in STIMPL. That a conditional statement can have a value (thus making it a conditional expression) is relatively surprising for people who only have experience in imperative programming languages. Nevertheless, the conditional expression is a very, very sharp sword in the sheath of the functional programmer.

A functional program is a series of functions and the execution of a functional program is simply an evaluation of those functions. That sounds abstract at this point, but will become more clear when we see some real functional programs.

Lists are a fundamental data type in functional programming languages. Powerful syntactic tools for manipulating lists are built in to most functional PLs. Understanding how to wield these tools effectively is vital for writing code in functional PLs.

\## The Historical Setting of the Development of Functional PLs

The first functional programming language was developed in the mid-1950s by [John McCarthy](https://en.wikipedia.org/wiki/John\_McCarthy\_(computer\_scientist)). At the time, computing was most associated with mathematical calculations. McCarthy was instead focused on artificial intelligence which involved symbolic computing. Computer scientists thought that it was possible to represent cognitive processes as lists of symbols. A language that made it possible to process those lists would allow developers to build systems that work like our brains, or so they thought.

![](./graphics/johnmccarthy.jpg)

McCarthy started with the goal of writing a system of meta notation that programmers could attach to Fortran. These meta notations would be reduced to actual Fortran programs, he thought. As they did their work, they found their way to a program representation built entirely of lists (and lists of lists, and lists of lists of lists, etc.). Their thinking resulted in the development of Lisp, a \*lis\*t \*p\*rocessing language. In Lisp, both data and programs are lists. McCarthy and his team showed that list processing, the basis of the semantics of Lisp, is capable of universal computing. In other words, Lisp, and other list processing languages, is/are Turing complete.

The inability to execute a Lisp program efficiently on a physical computer based on the von Neumann model has given Lisp (and other functional programming languages) a reputation as slow and wasteful.

\> Note:\_ This is \_not true\_ today!

Until the late 1980s hardware vendors thought that it would be worthwhile to build physical machines with non-von Neumann architectures. In fact, they built one such machine that made executing Lisp programs faster. Here is an image of a so-called Lisp Machine.

![](./graphics/LISP\_machine.jpg)

\## LISP

Though we will not study Lisp in this course, there are a few aspects of Lisp that you should know because they pervade the general field of computer science.

First, you should know `CAR`, `CDR` and `CONS` -- pronounced \*car\*, \*could-er\*, and \*cahns\*, respectively. `CAR` is a function that takes a list as a parameter and returns the first element of the list. `CDR` is a function that takes a list as a parameter and returns the tail, everything but the head, of the list. `CONS` takes two parameters -- a single element and a list -- and \*cons\*tructs a new list with the first argument appended to the front of the second argument.

For instance,

\```lisp

(car (1 2 3))

\```

is `1`.

\```lisp

(cdr (1 2 3))

\```

is `(2 3)`.

Second, you should know that, in Lisp, all data are lists \_and\_ programs are lists.

\```lisp

(a b c)

\```

is a list in Lisp. In Lisp, `(a b c)` could be interpreted as a list of \_atoms\_ `a`, `b` and `c` \_or\_ an invocation of function \_a\_ with parameters \_b\_ and \_c\_. â€‹

\## Lambda Calculus

Lambda Calculus is the theoretical basis of functional programming languages in the same way that the Turing Machine is the theoretical basis of the imperative programming languages. The Lambda Calculus is nothing like "calculus" -- the word calculus is used here in its strict sense: [a method or system of calculation](https://en.wikipedia.org/wiki/Calculus\_(disambiguation)). It is better to think of Lambda Calculus as a (albeit primitive) programming language rather than a branch of mathematics.

Lambda Calculus is a model of computation defined entirely by function application. The Lambda Calculus is as powerful as a Turning Machine which means that anything computable can be computed in the Lambda Calculus. For a language as simple as the Lambda Calculus, that's remarkable!

The entirety of the Lambda Calculus is made up of three entities:

1. Expression: a \_name\_,Â a \_function\_ or an \_application\_
1. Function:Â  $\lambda$ \*\<name\>\* `.` \*\<expression\>\*
1. Application: \*\<expression\>\* \*\<expression\>\*

Notice how the elements of the Lambda Calculus are defined in terms of themselves. In most cases it is possible to restrict \_name\_s in the Lambda Calculus to be any single letter of the alphabet -- `a` is a name, `z` is a name, etc. Strictly speaking, functions in the Lambda Calculus are anonymous -- in other words they have no name. The \_name\_ after theÂ $\lambda$ in a function in the Lambda Calculus can be thought of as the parameter of the function. Here's an example of a function in the Lambda Calculus:

$$

\lambda x . x

$$

Lambda Calculiticians (yes, I just made up that term) refer to this as the \_identity\_ function. This function simply returns the value of its argument! But didn't I say that functions in the Lambda Calculus don't have names? Yes, I did. \_Within\_ the language there is no way to name a function. That does not mean that we cannot assign semantic values to those functions. In fact, associating meaning with functions of a certain format is \_exactly\_ how high-level computing is done with the Lambda Calculus.

\## What's News

Although the economy is cooling off going in to the holiday shopping season and consumers are looking out for bargains wherever they can find them, job seekers continue to apply for jobs! It looks like a state of tightness in the labor market is something that will be part of the \*new normal\*.

\## Function Invocation in Functional Programming Languages

In imperative programming languages, it may matter to the correctness of a program the order in which parameters to a function are evaluated.

\> Note: For the purposes of this discussion we will assume that all operators [`+`, `-`, `/`, etc] are implemented as functions that take the operands as arguments. That will simplify the discussion! In other words, when describing the order of evaluation of a function's arguments we are also talking about the order of evaluation of the operands to a binary (or unary) operator.

While the choice of the order in which we evaluate the operands is the language designer's prerogative, the choice has consequences. Why? Because of side effects! For example:

\```C++

#include <stdio.h>

int operation(int parameter) {

static int divisor = 1;

return parameter / (divisor++);

}

int main() {

int result = operation(5) + operation(2);

printf("result: %d\n", result);

return 0;

}

\```

prints

\```

result: 6

\```

whereas

\```C++

#include <stdio.h>

int operation(int parameter) {

static int divisor = 1;

return parameter / (divisor++);

}

int main() {

int result = operation(2) + operation(5);

printf("result: %d\\n", result);

return 0;

}

\```

prints

\```console

result: 4

\```

In the difference between the two programs we see vividly the role that the `static` variable plays in the state of the program and its ultimate output.

Because of the referential transparency in pure functional programming languages, the designer of such a language does not need to worry about the consequences of the decision about the order of evaluation of arguments to functions. However, that does not mean that the language designer of a pure functional programming language does not have choices to make in this area.

A very important choice the designer has to make is the \_time\_ when function arguments are evaluated. There are two options available:

1. All function arguments are evaluated \_before\_ the function is invoked.
1. Function arguments are evaluated \_only when their results are needed by the function\_.

Let's look at an example: Assume that there are two functions: `dbl`, a function that doubles its input, and `average`, a function that averages its three parameters:

\```haskell

dbl x \= (+) x x

average a b c \= (/) ((+) a ((+) b c)) 3

\```

Both functions are written using prefix notation (i.e., (<\_operator\_> <\_operand1\_> ... <\_operandN\_>). We will call these functions like this:

\```haskell

dbl (average 3 4 5)

\```

If the language designer chooses to evaluate function arguments \_only when their results are needed\_, the execution of this function call proceeds as follows:

\```console

dbl (average 3 4 5)

\+ (average 3 4 5) (average 3 4 5)

\+ ((/) ((+) 3 ((+) 4 5)) 3) (average 3 4 5)

\+ (4) (average 3 4 5)

\+ (4) ((/) ((+) 3 ((+) 4 5)) 3)

\+ (4) (4)

8

\```

The outermost function is always \_reduced\_ (expanded) before the inner functions. Note: Primitive functions (`+`, and `/` in this example) cannot be expanded further so we move inward in evaluation if we encounter such a function for reduction.

If, however, the language designer chooses to evaluate function arguments before the function is evaluated, the execution of the function call proceeds as follows:

\```console

dbl (average 3 4 5)

dbl ((/) ((+) 3 ((+) 4 5)) 3)

dbl ((/) ((+) 3 9) 3)

dbl ((/) 12 3)

dbl 4

\+ 4 4

8

\```

No matter the designer's choice, the outcome of the evaluation is the same. However, there is something strikingly different about the two. Notice that in the first derivation, the calculation of the average of the three numbers happens twice. In the second derivation, it happens only once! That efficiency is not a fluke! Generally speaking, the method of function invocation where arguments are evaluated \_before\_ the function runs faster.

These two techniques have technical names:

1. \_applicative order\_:Â  "all the arguments to ... procedures are evaluated when the procedure is applied."â€‹
1. \_normal order\_: "delay evaluation of procedure arguments until the actual argument values are needed."â€‹

These definitions come from

[Abelson, H., Sussman, G. J.,, with Julie Sussman (1996). Structure and Interpretation of Computer Programs. Cambridge: MIT Press/McGraw-Hill. ISBN: 0-262-01153-0](http://uclid.uc.edu/record=b2528617~S39)

It is obvious, then, that any serious language designer would choose applicative order for their language.

There's no obvious redeeming value for the inefficiency of normal order.

\## The Implications of Applicative Order

Scheme is a [Lisp \_dialect\_](https://en.wikipedia.org/wiki/Lisp\_(programming\_language)#Major\_dialects). I told you that we weren't going to work much with Lisp, but I lied. Sort of. Scheme is an applicative-order language with the same list-is-everything syntax as all other Lisps. In Scheme, you would define an \_if\_ function named `myif` like this:

\```scheme

(define (myif c t f) (cond (c t) (else f)))

\```

`c` is a boolean and `myif` returns `t` when `c` is true and `f` when `c` is false. No surprises.

We can define a name `a` and set its value to 5:

\```scheme

(define a 5)

\```

Now, let's call `myif`:

\```scheme

(myif (\= a 0) 1 (/ 1 a))

\```

If `a` is equal to 0, then the call returns 1. Perfect. If `a` is not zero, the call returns the reciprocal of `a`. Given the value of `a`, the result is `1/7`.

Let's define the name `b` and set its value to 0:

\```scheme

(define b 0)

\```

Now, let's call `myif`:

\```scheme

(myif (\= b 0) 1 (/ 1 b))

\```

If `b` is equal to $0$, then the call returns $1$. If `b` is not zero, the call returns the reciprocal of `b`. Given the value of `b`, the result is $1$:

\```console

/: division by zero

context...:

"/home/hawkinsw/code/uc/cs3003/scheme/applicative/applicative.rkt": \[running body\]

temp37\\_0

for-loop

run-module-instance!125

perform-require!78

\```

That looks \_exactly\_ like $1$. Uhm, no! What happened?

Remember we said that Scheme is an applicative order language. No matter what the value of `b`, both of the arguments are going to be evaluated \_before\_ `myif` starts. Therefore, Scheme attempts to evaluate `1 / b` which is `1 / 0` which is `division by zero`.

Thanks to situations like this, the Scheme programming language is forced into defining special semantics for certain functions, like the built-in \_if\_ expression. As a result, function invocation is not \_orthogonal\_ in Scheme -- the general rules of function evaluation in Scheme must make an exception for applying functions like the built-in if expression. Remember that the orthogonality decreases as exceptions in a language's specification increase.

\### Sidebar: Solving the problem in Scheme

Feel free to skip this section if you are not interested in details of the Scheme programming language. That said, the concepts in this section are applicable to other languages.

In Scheme, you can specify that the evaluation of an expression be `delay`ed until it is `force`d.

\```scheme

(define d (delay (/ 1 7)))

\```

defines `d` to be the \_eventual\_ result of the evaluation of the division of $1$ by $7$. If we ask Scheme to print out `d`, we see

\```console

#<promise:d>

\```

To bring the future tense into the present tense, we `force` a `delay`ed evaluation:

\```scheme

(force d)

\```

If we ask Scheme to print the result of that expression, we see:

\```console

1/7

\```

Exactly what we expect! With this newfound knowledge, we can rewrite the `myif` function:


\```scheme

(define (myif c t f) (cond (c (force t)) (else (force f))))

\```

Now `myif` can accept `t`s and `f`s that are `delay`ed and we can use `myif` safely:

\```scheme

(define b 0)

(myif (\*\*\=\*\* b 0) (delay 1) (delay (\*\*/\*\* 1 b)))

\```

and we see the reasonable result:

\```console

1

\```

Kotlin, a modern language, has a concept similar to `delay` called [lazy](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html). OCaml, an object-oriented functional programming language, [contains the same concept](https://ocaml.org/manual/coreexamples.html#s:lazy-expr). Swift has some sense of [laziness](https://docs.swift.org/swift-book/LanguageGuide/Properties.html), too!

\## Well, We Are Back to Normal Order

I guess that we are stuck with the inefficiency inherent in the normal order function application. Going back to the `dbl`/`average` example, we will just have to live with invoking `average` twice.

Or will we?

Real-world functional programming languages that are normal order use an interesting optimization to avoid these recalculations! When an expression is passed around and it is unevaluated, Haskell and languages like it represent it as a [\_thunk\_](https://wiki.haskell.org/Thunk). The thunk data structure is generated in such a way that it can calculate the value of its associated expression some time in the future when the value is needed. Additionally, the thunk then caches (or \_memoizes)\_ the value so that future evaluations of the associated expression do not need to be repeated.

As a result, in the dbl/average example,

1. a thunk is created for `(average 3 4 5)`,
1. that thunk is passed to `dbl`, where it is duplicated during the reduction of `dbl`,
1. `(average 3 4 5)` is (eventually) calculated when the value of `(average 3 4 5)` is needed,
1. 4 is stored (cached, memoized) in the thunk, and
1. the cached/memoized value is retrieved from the thunk instead of doing a fresh evaluation the next time that the value of `(average 3 4 5)`is needed.

A thunk, then, is the mechanism that allows the programmer to have the efficiency of applicative order function invocation with the semantics of the normal order function invocation! The best of both worlds!
