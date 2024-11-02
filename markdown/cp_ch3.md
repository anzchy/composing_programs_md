# Chapter 3: Interpreting Computer Programs

## 3.1 Introduction

Chapters 1 and 2 describe the close connection between two fundamental
elements of programming: functions and data. We saw how functions can be
manipulated as data using higher-order functions. We also saw how data can be
endowed with behavior using message passing and an object system. We have also
studied techniques for organizing large programs, such as functional
abstraction, data abstraction, class inheritance, and generic functions. These
core concepts constitute a strong foundation upon which to build modular,
maintainable, and extensible programs.

This chapter focuses on the third fundamental element of programming: programs
themselves. A Python program is just a collection of text. Only through the
process of interpretation do we perform any meaningful computation based on
that text. A programming language like Python is useful because we can define
an _interpreter_ , a program that carries out Python's evaluation and
execution procedures. It is no exaggeration to regard this as the most
fundamental idea in programming, that an interpreter, which determines the
meaning of expressions in a programming language, is just another program.

To appreciate this point is to change our images of ourselves as programmers.
We come to see ourselves as designers of languages, rather than only users of
languages designed by others.

### 3.1.1 Programming Languages

Programming languages vary widely in their syntactic structures, features, and
domain of application. Among general purpose programming languages, the
constructs of function definition and function application are pervasive. On
the other hand, powerful languages exist that do not include an object system,
higher-order functions, assignment, or even control constructs such as `while`
and `for` statements. As an example of a powerful language with a minimal set
of features, we will introduce the
[Scheme](https://en.wikipedia.org/wiki/Scheme_\(programming_language\))
programming language. The subset of Scheme introduced in this text does not
allow mutable values at all.

In this chapter, we study the design of interpreters and the computational
processes that they create when executing programs. The prospect of designing
an interpreter for a general programming language may seem daunting. After
all, interpreters are programs that can carry out any possible computation,
depending on their input. However, many interpreters have an elegant common
structure: two mutually recursive functions. The first evaluates expressions
in environments; the second applies functions to arguments.

These functions are recursive in that they are defined in terms of each other:
applying a function requires evaluating the expressions in its body, while
evaluating an expression may involve applying one or more functions.

_Continue_ : [ 3.2 Functional Programming ](../pages/32-functional-
programming.html)

## 3.2 Functional Programming

The software running on any modern computer is written in a variety of
programming languages. There are physical languages, such as the machine
languages for particular computers. These languages are concerned with the
representation of data and control in terms of individual bits of storage and
primitive machine instructions. The machine-language programmer is concerned
with using the given hardware to erect systems and utilities for the efficient
implementation of resource-limited computations. High-level languages, erected
on a machine-language substrate, hide concerns about the representation of
data as collections of bits and the representation of programs as sequences of
primitive instructions. These languages have means of combination and
abstraction, such as function definition, that are appropriate to the larger-
scale organization of software systems.

In this section, we introduce a high-level programming language that
encourages a functional style. Our object of study, a subset of the Scheme
language, employs a very similar model of computation to Python's, but uses
only expressions (no statements), specializes in symbolic computation, and
employs only immutable values.

Scheme is a dialect of
[Lisp](http://en.wikipedia.org/wiki/Lisp_\(programming_language\)), the
second-oldest programming language that is still widely used today (after
[Fortran](http://en.wikipedia.org/wiki/Fortran)). The community of Lisp
programmers has continued to thrive for decades, and new dialects of Lisp such
as [Clojure](http://en.wikipedia.org/wiki/Clojure) have some of the fastest
growing communities of developers of any modern programming language. To
follow along with the examples in this text, you can [download a Scheme
interpreter](http://inst.eecs.berkeley.edu/~scheme/).

### 3.2.1 Expressions

Scheme programs consist of expressions, which are either call expressions or
special forms. A call expression consists of an operator expression followed
by zero or more operand sub-expressions, as in Python. Both the operator and
operand are contained within parentheses:


​    

    (quotient 10 2)


Scheme exclusively uses prefix notation. Operators are often symbols, such as
`+` and `*`. Call expressions can be nested, and they may span more than one
line:


​    

    (+ (* 3 5) (- 10 6))


​    
​    

    (+ (* 3
          (+ (* 2 4)
             (+ 3 5)))
       (+ (- 10 7)
          6))


As in Python, Scheme expressions may be primitives or combinations. Number
literals are primitives, while call expressions are combined forms that
include arbitrary sub-expressions. The evaluation procedure of call
expressions matches that of Python: first the operator and operand expressions
are evaluated, and then the function that is the value of the operator is
applied to the arguments that are the values of the operands.

The `if` expression in Scheme is a _special form_ , meaning that while it
looks syntactically like a call expression, it has a different evaluation
procedure. The general form of an `if` expression is:


​    

    (if <predicate> <consequent> <alternative>)


To evaluate an `if` expression, the interpreter starts by evaluating the
`<predicate>` part of the expression. If the `<predicate>` evaluates to a true
value, the interpreter then evaluates the `<consequent>` and returns its
value. Otherwise it evaluates the `<alternative>` and returns its value.

Numerical values can be compared using familiar comparison operators, but
prefix notation is used in this case as well:


​    

    (>= 2 1)


The boolean values `#t` (or `true`) and `#f` (or `false`) in Scheme can be
combined with boolean special forms, which have evaluation procedures similar
to those in Python.

>   * `(and <e1> ... <en>)` The interpreter evaluates the expressions `<e>`
>     one at a time, in left-to-right order. If any `<e>` evaluates to `false`,
>     the value of the `and` expression is `false`, and the rest of the `<e>`'s
>     are not evaluated. If all `<e>`'s evaluate to true values, the value of the
>     `and` expression is the value of the last one.
>   * `(or <e1> ... <en>)` The interpreter evaluates the expressions `<e>` one
>     at a time, in left-to-right order. If any `<e>` evaluates to a true value,
>     that value is returned as the value of the `or` expression, and the rest of
>     the `<e>`'s are not evaluated. If all `<e>`'s evaluate to `false`, the value
>     of the `or` expression is `false`.
>   * `(not <e>)` The value of a not expression is `true` when the expression
>     `<e>` evaluates to `false`, and `false` otherwise.

### 3.2.2 Definitions

Values can be named using the `define` special form:


​    

    (define pi 3.14)
    (* pi 2)


New functions (called _procedures_ in Scheme) can be defined using a second
version of the `define` special form. For example, to define squaring, we
write:


​    

    (define (square x) (* x x))


The general form of a procedure definition is:


​    

    (define (<name> <formal parameters>) <body>)


The `<name>` is a symbol to be associated with the procedure definition in the
environment. The `<formal parameters>` are the names used within the body of
the procedure to refer to the corresponding arguments of the procedure. The
`<body>` is an expression that will yield the value of the procedure
application when the formal parameters are replaced by the actual arguments to
which the procedure is applied. The `<name>` and the `<formal parameters>` are
grouped within parentheses, just as they would be in an actual call to the
procedure being defined.

Having defined square, we can now use it in call expressions:


​    

    (square 21)


​    
​    

    (square (+ 2 5))


​    
​    

    (square (square 3))


User-defined functions can take multiple arguments and include special forms:


​    

    (define (average x y)
      (/ (+ x y) 2))


​    
​    

    (average 1 3)


​    
​    

    (define (abs x)
        (if (< x 0)
            (- x)
            x))


​    
​    

    (abs -3)


Scheme supports local definitions with the same lexical scoping rules as
Python. Below, we define an iterative procedure for computing square roots
using nested definitions and recursion:


​    

    (define (sqrt x)
      (define (good-enough? guess)
        (< (abs (- (square guess) x)) 0.001))
      (define (improve guess)
        (average guess (/ x guess)))
      (define (sqrt-iter guess)
        (if (good-enough? guess)
            guess
            (sqrt-iter (improve guess))))
      (sqrt-iter 1.0))
    (sqrt 9)


Anonymous functions are created using the `lambda` special form. `Lambda` is
used to create procedures in the same way as `define`, except that no name is
specified for the procedure:


​    

    (lambda (<formal-parameters>) <body>)


The resulting procedure is just as much a procedure as one that is created
using `define`. The only difference is that it has not been associated with
any name in the environment. In fact, the following expressions are
equivalent:


​    

    (define (plus4 x) (+ x 4))
    (define plus4 (lambda (x) (+ x 4)))


Like any expression that has a procedure as its value, a lambda expression can
be used as the operator in a call expression:


​    

    ((lambda (x y z) (+ x y (square z))) 1 2 3)


### 3.2.3 Compound values

Pairs are built into the Scheme language. For historical reasons, pairs are
created with the `cons` built-in function, and the elements of a pair are
accessed with `car` and `cdr`:


​    

    (define x (cons 1 2))


​    
​    

    x


​    
​    

    (car x)


​    
​    

    (cdr x)


Recursive lists are also built into the language, using pairs. A special value
denoted `nil` or `'()` represents the empty list. A recursive list value is
rendered by placing its elements within parentheses, separated by spaces:


​    

    (cons 1
          (cons 2
                (cons 3
                      (cons 4 nil))))


​    
​    

    (list 1 2 3 4)


​    
​    

    (define one-through-four (list 1 2 3 4))


​    
​    

    (car one-through-four)


​    
​    

    (cdr one-through-four)


​    
​    

    (car (cdr one-through-four))


​    
​    

    (cons 10 one-through-four)


​    
​    

    (cons 5 one-through-four)


Whether a list is empty can be determined using the primitive `null?`
predicate. Using it, we can define the standard sequence operations for
computing `length` and selecting elements:


​    

    (define (length items)
      (if (null? items)
          0
          (+ 1 (length (cdr items)))))
    (define (getitem items n)
      (if (= n 0)
          (car items)
          (getitem (cdr items) (- n 1))))
    (define squares (list 1 4 9 16 25))


​    
​    

    (length squares)


​    
​    

    (getitem squares 3)


### 3.2.4 Symbolic Data

All the compound data objects we have used so far were constructed ultimately
from numbers. One of Scheme's strengths is working with arbitrary symbols as
data.

In order to manipulate symbols we need a new element in our language: the
ability to _quote_ a data object. Suppose we want to construct the list `(a
b)`. We can't accomplish this with `(list a b)`, because this expression
constructs a list of the values of `a` and `b` rather than the symbols
themselves. In Scheme, we refer to the symbols `a` and `b` rather than their
values by preceding them with a single quotation mark:


​    

    (define a 1)
    (define b 2)


​    
​    

    (list a b)


​    
​    

    (list 'a 'b)


​    
​    

    (list 'a b)


In Scheme, any expression that is not evaluated is said to be _quoted_. This
notion of quotation is derived from a classic philosophical distinction
between a thing, such as a dog, which runs around and barks, and the word
"dog" that is a linguistic construct for designating such things. When we use
"dog" in quotation marks, we do not refer to some dog in particular but
instead to a word. In language, quotation allow us to talk about language
itself, and so it is in Scheme:


​    

    (list 'define 'list)


Quotation also allows us to type in compound objects, using the conventional
printed representation for lists:


​    

    (car '(a b c))


​    
​    

    (cdr '(a b c))


The full Scheme language contains additional features, such as mutation
operations, vectors, and maps. However, the subset we have introduced so far
provides a rich functional programming language capable of implementing many
of the ideas we have discussed so far in this text.

### 3.2.5 Turtle graphics

The implementation of Scheme that serves as a companion to this text includes
Turtle graphics, an illustrating environment developed as part of the Logo
language (another Lisp dialect). This turtle begins in the center of a canvas,
moves and turns based on procedures, and draws lines behind it as it moves.
While the turtle was invented to engage children in the act of programming, it
remains an engaging graphical tool for even advanced programmers.

At any moment during the course of executing a Scheme program, the turtle has
a position and heading on the canvas. Single-argument procedures such as
`forward` and `right` change the position and heading of the turtle. Common
procedures have abbreviations: `forward` can also be called as `fd`, etc. The
`begin` special form in Scheme allows a single expression to include multiple
sub-expressions. This form is useful for issuing multiple commands:


​    

    > (define (repeat k fn) (if (> k 0)
                                (begin (fn) (repeat (- k 1) fn))
                                nil))
    > (repeat 5
              (lambda () (fd 100)
                         (repeat 5
                                 (lambda () (fd 20) (rt 144)))
                         (rt 144)))
    nil


![](http://www.composingprograms.com/img/star.png)

The full repertoire of Turtle procedures is also built into Python as the
[turtle library module](http://docs.python.org/py3k/library/turtle.html).

As a final example, Scheme can express recursive drawings using its turtle
graphics in a remarkably compact form. Sierpinski's triangle is a fractal that
draws each triangle as three neighboring triangles that have vertexes at the
midpoints of the legs of the triangle that contains them. It can be drawn to a
finite recursive depth by this Scheme program:


​    

    > (define (repeat k fn)
        (if (> k 0)
            (begin (fn) (repeat (- k 1) fn))
            nil))
    
    > (define (tri fn)
        (repeat 3 (lambda () (fn) (lt 120))))
    
    > (define (sier d k)
        (tri (lambda ()
               (if (= k 1) (fd d) (leg d k)))))
    
    > (define (leg d k)
        (sier (/ d 2) (- k 1))
        (penup)
        (fd d)
        (pendown))


The `triangle` procedure is a general method for repeating a drawing procedure
three times with a left turn following each repetition. The `sier` procedure
takes a length `d` and a recursive depth `k`. It draws a plain triangle if the
depth is 1, and otherwise draws a triangle made up of calls to `leg`. The
`leg` procedure draws a single leg of a recursive Sierpinski triangle by a
recursive call to `sier` that fills the first half of the length of the leg,
then by moving the turtle to the next vertex. The procedures `penup` and
`pendown` stop the turtle from drawing as it moves by lifting its pen up and
the placing it down again. The mutual recursion between `sier` and `leg`
yields this result:


​    

    > (sier 400 6)


![](http://www.composingprograms.com/img/sier.png)

_Continue_ : [ 3.3 Exceptions ](../pages/33-exceptions.html)

## 3.3 Exceptions

Programmers must be always mindful of possible errors that may arise in their
programs. Examples abound: a function may not receive arguments that it is
designed to accept, a necessary resource may be missing, or a connection
across a network may be lost. When designing a program, one must anticipate
the exceptional circumstances that may arise and take appropriate measures to
handle them.

There is no single correct approach to handling errors in a program. Programs
designed to provide some persistent service like a web server should be robust
to errors, logging them for later consideration but continuing to service new
requests as long as possible. On the other hand, the Python interpreter
handles errors by terminating immediately and printing an error message, so
that programmers can address issues as soon as they arise. In any case,
programmers must make conscious choices about how their programs should react
to exceptional conditions.

_Exceptions_ , the topic of this section, provides a general mechanism for
adding error-handling logic to programs. _Raising an exception_ is a technique
for interrupting the normal flow of execution in a program, signaling that
some exceptional circumstance has arisen, and returning directly to an
enclosing part of the program that was designated to react to that
circumstance. The Python interpreter raises an exception each time it detects
an error in an expression or statement. Users can also raise exceptions with
`raise` and `assert` statements.

**Raising exceptions.** An exception is a object instance with a class that
inherits, either directly or indirectly, from the `BaseException` class. The
`assert` statement introduced in Chapter 1 raises an exception with the class
`AssertionError`. In general, any exception instance can be raised with the
`raise` statement. The general form of raise statements are described in the
[Python docs](http://docs.python.org/py3k/reference/simple_stmts.html#raise).
The most common use of `raise` constructs an exception instance and raises it.


​    

    >>> raise Exception('An error occurred')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    Exception: an error occurred


When an exception is raised, no further statements in the current block of
code are executed. Unless the exception is _handled_ (described below), the
interpreter will return directly to the interactive read-eval-print loop, or
terminate entirely if Python was started with a file argument. In addition,
the interpreter will print a _stack backtrace_ , which is a structured block
of text that describes the nested set of active function calls in the branch
of execution in which the exception was raised. In the example above, the file
name `<stdin>` indicates that the exception was raised by the user in an
interactive session, rather than from code in a file.

**Handling exceptions.** An exception can be handled by an enclosing `try`
statement. A `try` statement consists of multiple clauses; the first begins
with `try` and the rest begin with `except`:


​    

    try:
        <try suite>
    except <exception class> as <name>:
        <except suite>
    ...


The `<try suite>` is always executed immediately when the `try` statement is
executed. Suites of the `except` clauses are only executed when an exception
is raised during the course of executing the `<try suite>`. Each `except`
clause specifies the particular class of exception to handle. For instance, if
the `<exception class>` is `AssertionError`, then any instance of a class
inheriting from `AssertionError` that is raised during the course of executing
the `<try suite>` will be handled by the following `<except suite>`. Within
the `<except suite>`, the identifier `<name>` is bound to the exception object
that was raised, but this binding does not persist beyond the `<except
suite>`.

For example, we can handle a `ZeroDivisionError` exception using a `try`
statement that binds the name `x` to 0 when the exception is raised.


​    

    >>> try:
            x = 1/0
        except ZeroDivisionError as e:
            print('handling a', type(e))
            x = 0
    handling a <class 'ZeroDivisionError'>
    >>> x
    0


A `try` statement will handle exceptions that occur within the body of a
function that is applied (either directly or indirectly) within the `<try
suite>`. When an exception is raised, control jumps directly to the body of
the `<except suite>` of the most recent `try` statement that handles that type
of exception.


​    

    >>> def invert(x):
            result = 1/x  # Raises a ZeroDivisionError if x is 0
            print('Never printed if x is 0')
            return result


​    
​    

    >>> def invert_safe(x):
            try:
                return invert(x)
            except ZeroDivisionError as e:
                return str(e)


​    
​    

    >>> invert_safe(2)
    Never printed if x is 0
    0.5
    >>> invert_safe(0)
    'division by zero'


This example illustrates that the `print` expression in `invert` is never
evaluated, and instead control is transferred to the suite of the `except`
clause in `invert_safe`. Coercing the `ZeroDivisionError` `e` to a string
gives the human-interpretable string returned by `invert_safe`: `'division by
zero'`.

### 3.3.1 Exception Objects

Exception objects themselves can have attributes, such as the error message
stated in an `assert` statement and information about where in the course of
execution the exception was raised. User-defined exception classes can have
additional attributes.

In Chapter 1, we implemented Newton's method to find the zeros of arbitrary
functions. The following example defines an exception class that returns the
best guess discovered in the course of iterative improvement whenever a
`ValueError` occurs. A math domain error (a type of `ValueError`) is raised
when `sqrt` is applied to a negative number. This exception is handled by
raising an `IterImproveError` that stores the most recent guess from Newton's
method as an attribute.

First, we define a new class that inherits from `Exception`.


​    

    >>> class IterImproveError(Exception):
            def __init__(self, last_guess):
                self.last_guess = last_guess


Next, we define a version of `improve`, our generic iterative improvement
algorithm. This version handles any `ValueError` by raising an
`IterImproveError` that stores the most recent guess. As before, `improve`
takes as arguments two functions, each of which takes a single numerical
argument. The `update` function returns new guesses, while the `done` function
returns a boolean indicating that improvement has converged to a correct
value.


​    

    >>> def improve(update, done, guess=1, max_updates=1000):
            k = 0
            try:
                while not done(guess) and k < max_updates:
                    guess = update(guess)
                    k = k + 1
                return guess
            except ValueError:
                raise IterImproveError(guess)


Finally, we define `find_zero`, which returns the result of `improve` applied
to a Newton update function returned by `newton_update`, which is defined in
Chapter 1 and requires no changes for this example. This version of
`find_zero` handles an `IterImproveError` by returning its last guess.


​    

    >>> def find_zero(f, guess=1):
            def done(x):
                return f(x) == 0
            try:
                return improve(newton_update(f), done, guess)
            except IterImproveError as e:
                return e.last_guess


Consider applying `find_zero` to find the zero of the function $2x^2 +
\sqrt{x}$. This function has a zero at 0, but evaluating it on any negative
number will raise a `ValueError`. Our Chapter 1 implementation of Newton's
Method would raise that error and fail to return any guess of the zero. Our
revised implementation returns the last guess found before the error.


​    

    >>> from math import sqrt
    >>> find_zero(lambda x: 2*x*x + sqrt(x))
    -0.030211203830201594


Although this approximation is still far from the correct answer of 0, some
applications would prefer this coarse approximation to a `ValueError`.

Exceptions are another technique that help us as programs to separate the
concerns of our program into modular parts. In this example, Python's
exception mechanism allowed us to separate the logic for iterative
improvement, which appears unchanged in the suite of the `try` clause, from
the logic for handling errors, which appears in `except` clauses. We will also
find that exceptions are a useful feature when implementing interpreters in
Python.

_Continue_ : [ 3.4 Interpreters for Languages with Combination

](../pages/34-interpreters-for-languages-with-combination.html)

## 3.4 Interpreters for Languages with Combination

We now embark on a tour of the technology by which languages are established
in terms of other languages. _Metalinguistic abstraction_  establishing new
languages  plays an important role in all branches of engineering design. It
is particularly important to computer programming, because in programming not
only can we formulate new languages but we can also implement these languages
by constructing interpreters. An interpreter for a programming language is a
function that, when applied to an expression of the language, performs the
actions required to evaluate that expression.

We will first define an interpreter for a language that is a limited subset of
Scheme, called Calculator. Then, we will develop a sketch of an interpreter
for Scheme as a whole. The interpreter we create will be complete in the sense
that it will allow us to write fully general programs in Scheme. To do so, it
will implement the same environment model of evaluation that we introduced for
Python programs in Chapter 1.

Many of the examples in this section are contained in the companion [Scheme-
Syntax Calculator example](../examples/scalc/scalc.html), as they are too
complex to fit naturally in the format of this text.

### 3.4.1 A Scheme-Syntax Calculator

The Scheme-Syntax Calculator (or simply Calculator) is an expression language
for the arithmetic operations of addition, subtraction, multiplication, and
division. Calculator shares Scheme's call expression syntax and operator
behavior. Addition (`+`) and multiplication (`*`) operations each take an
arbitrary number of arguments:


​    

    > (+ 1 2 3 4)
    10
    > (+)
    0
    > (* 1 2 3 4)
    24
    > (*)
    1


Subtraction (`-`) has two behaviors. With one argument, it negates the
argument. With at least two arguments, it subtracts all but the first from the
first. Division (`/`) has a similar pair of two behaviors: compute the
multiplicative inverse of a single argument or divide all but the first into
the first:


​    

    > (- 10 1 2 3)
    4
    > (- 3)
    -3
    > (/ 15 12)
    1.25
    > (/ 30 5 2)
    3
    > (/ 10)
    0.1


A call expression is evaluated by evaluating its operand sub-expressions, then
applying the operator to the resulting arguments:


​    

    > (- 100 (* 7 (+ 8 (/ -12 -3))))
    16.0


We will implement an interpreter for the Calculator language in Python. That
is, we will write a Python program that takes string lines as input and
returns the result of evaluating those lines as a Calculator expression. Our
interpreter will raise an appropriate exception if the calculator expression
is not well formed.

### 3.4.2 Expression Trees

Until this point in the course, expression trees have been conceptual entities
to which we have referred in describing the process of evaluation; we have
never before explicitly represented expression trees as data in our programs.
In order to write an interpreter, we must operate on expressions as data.

A primitive expression is just a number or a string in Calculator: either an
`int` or `float` or an operator symbol. All combined expressions are call
expressions. A call expression is a Scheme list with a first element (the
operator) followed by zero or more operand expressions.

**Scheme Pairs.** In Scheme, lists are nested pairs, but not all pairs are
lists. To represent Scheme pairs and lists in Python, we will define a class
`Pair` that is similar to the `Rlist` class earlier in the chapter. The
implementation appears in
[scheme_reader](../examples/scalc/scheme_reader.py.html).

The empty list is represented by an object called `nil`, which is an instance
of the class `nil`. We assume that only one `nil` instance will ever be
created.

The `Pair` class and `nil` object are Scheme values represented in Python.
They have `repr` strings that are Python expressions and `str` strings that
are Scheme expressions.


​    

    >>> s = Pair(1, Pair(2, nil))
    >>> s
    Pair(1, Pair(2, nil))
    >>> print(s)
    (1 2)


They implement the basic Python sequence interface of length and element
selection, as well as a `map` method that returns a Scheme list.


​    

    >>> len(s)
    2
    >>> s[1]
    2
    >>> print(s.map(lambda x: x+4))
    (5 6)


**Nested Lists.** Nested pairs can represent lists, but the elements of a list
can also be lists themselves. Pairs are therefore sufficient to represent
Scheme expressions, which are in fact nested lists.


​    

    >>> expr = Pair('+', Pair(Pair('*', Pair(3, Pair(4, nil))), Pair(5, nil)))
    >>> print(expr)
    (+ (* 3 4) 5)
    >>> print(expr.second.first)
    (* 3 4)
    >>> expr.second.first.second.first
    3


This example demonstrates that all Calculator expressions are nested Scheme
lists. Our Calculator interpreter will read in nested Scheme lists, convert
them into expression trees represented as nested `Pair` instances (_Parsing
expressions_ below), and then evaluate the expression trees to produce values
(_Calculator evaluation_ below).

### 3.4.3 Parsing Expressions

Parsing is the process of generating expression trees from raw text input. A
parser is a composition of two components: a lexical analyzer and a syntactic
analyzer. First, the _lexical analyzer_ partitions the input string into
_tokens_ , which are the minimal syntactic units of the language such as names
and symbols. Second, the _syntactic analyzer_ constructs an expression tree
from this sequence of tokens. The sequence of tokens produced by the lexical
analyzer is consumed by the syntactic analyzer.

**Lexical analysis.** The component that interprets a string as a token
sequence is called a _tokenizer_ or _lexical analyzer_. In our implementation,
the tokenizer is a function called `tokenize_line` in
[scheme_tokens](../examples/scalc/scheme_tokens.py.html). Scheme tokens are
delimited by white space, parentheses, dots, or single quotation marks.
Delimiters are tokens, as are symbols and numerals. The tokenizer analyzes a
line character by character, validating the format of symbols and numerals.

Tokenizing a well-formed Calculator expression separates all symbols and
delimiters, but identifies multi-character numbers (e.g., 2.3) and converts
them into numeric types.


​    

    >>> tokenize_line('(+ 1 (* 2.3 45))')
    ['(', '+', 1, '(', '*', 2.3, 45, ')', ')']


Lexical analysis is an iterative process, and it can be applied to each line
of an input program in isolation.

**Syntactic analysis.** The component that interprets a token sequence as an
expression tree is called a _syntactic analyzer_. Syntactic analysis is a
tree-recursive process, and it must consider an entire expression that may
span multiple lines.

Syntactic analysis is implemented by the `scheme_read` function in
[scheme_reader](../examples/scalc/scheme_reader.py.html). It is tree-recursive
because analyzing a sequence of tokens often involves analyzing a subsequence
of those tokens into a subexpression, which itself serves as a branch (e.g.,
operand) of a larger expression tree. Recursion generates the hierarchical
structures consumed by the evaluator.

The `scheme_read` function expects its input `src` to be a `Buffer` instance
that gives access to a sequence of tokens. A `Buffer`, defined in the
[buffer](../examples/scalc/buffer.py.html) module, collects tokens that span
multiple lines into a single object that can be analyzed syntactically.


​    

    >>> lines = ['(+ 1', '   (* 2.3 45))']
    >>> expression = scheme_read(Buffer(tokenize_lines(lines)))
    >>> expression
    Pair('+', Pair(1, Pair(Pair('*', Pair(2.3, Pair(45, nil))), nil)))
    >>> print(expression)
    (+ 1 (* 2.3 45))


The `scheme_read` function first checks for various base cases, including
empty input (which raises an end-of-file exception, called `EOFError` in
Python) and primitive expressions. A recursive call to `read_tail` is invoked
whenever a `(` token indicates the beginning of a list.

The `read_tail` function continues to read from the same input `src`, but
expects to be called after a list has begun. Its base cases are an empty input
(an error) or a closing parenthesis that terminates the list. Its recursive
call reads the first element of the list with `scheme_read`, reads the rest of
the list with `read_tail`, and then returns a list represented as a `Pair`.

This implementation of `scheme_read` can read well-formed Scheme lists, which
are all we need for the Calculator language. Parsing dotted lists and quoted
forms is left as an exercise.

Informative syntax errors improve the usability of an interpreter
substantially. The `SyntaxError` exceptions that are raised include a
description of the problem encountered.

### 3.4.4 Calculator Evaluation

The [scalc](../examples/scalc/scalc.py.html) module implements an evaluator
for the Calculator language. The `calc_eval` function takes an expression as
an argument and returns its value. Definitions of the helper functions
`simplify`, `reduce`, and `as_scheme_list` appear in the model and are used
below.

For Calculator, the only two legal syntactic forms of expressions are numbers
and call expressions, which are `Pair` instances representing well-formed
Scheme lists. Numbers are _self-evaluating_ ; they can be returned directly
from `calc_eval`. Call expressions require function application.


​    

    >>> def calc_eval(exp):
            """Evaluate a Calculator expression."""
            if type(exp) in (int, float):
                return simplify(exp)
            elif isinstance(exp, Pair):
                arguments = exp.second.map(calc_eval)
                return simplify(calc_apply(exp.first, arguments))
            else:
                raise TypeError(exp + ' is not a number or call expression')


Call expressions are evaluated by first recursively mapping the `calc_eval`
function to the list of operands, which computes a list of `arguments`. Then,
the operator is applied to those arguments in a second function, `calc_apply`.

The Calculator language is simple enough that we can easily express the logic
of applying each operator in the body of a single function. In `calc_apply`,
each conditional clause corresponds to applying one operator.


​    

    >>> def calc_apply(operator, args):
            """Apply the named operator to a list of args."""
            if not isinstance(operator, str):
                raise TypeError(str(operator) + ' is not a symbol')
            if operator == '+':
                return reduce(add, args, 0)
            elif operator == '-':
                if len(args) == 0:
                    raise TypeError(operator + ' requires at least 1 argument')
                elif len(args) == 1:
                    return -args.first
                else:
                    return reduce(sub, args.second, args.first)
            elif operator == '*':
                return reduce(mul, args, 1)
            elif operator == '/':
                if len(args) == 0:
                    raise TypeError(operator + ' requires at least 1 argument')
                elif len(args) == 1:
                    return 1/args.first
                else:
                    return reduce(truediv, args.second, args.first)
            else:
                raise TypeError(operator + ' is an unknown operator')


Above, each suite computes the result of a different operator or raises an
appropriate `TypeError` when the wrong number of arguments is given. The
`calc_apply` function can be applied directly, but it must be passed a list of
_values_ as arguments rather than a list of operand expressions.


​    

    >>> calc_apply('+', as_scheme_list(1, 2, 3))
    6
    >>> calc_apply('-', as_scheme_list(10, 1, 2, 3))
    4
    >>> calc_apply('*', nil)
    1
    >>> calc_apply('*', as_scheme_list(1, 2, 3, 4, 5))
    120
    >>> calc_apply('/', as_scheme_list(40, 5))
    8.0


The role of `calc_eval` is to make proper calls to `calc_apply` by first
computing the value of operand sub-expressions before passing them as
arguments to `calc_apply`. Thus, `calc_eval` can accept a nested expression.


​    

    >>> print(exp)
    (+ (* 3 4) 5)
    >>> calc_eval(exp)
    17


The structure of `calc_eval` is an example of dispatching on type: the form of
the expression. The first form of expression is a number, which requires no
additional evaluation step. In general, primitive expressions that do not
require an additional evaluation step are called _self-evaluating_. The only
self-evaluating expressions in our Calculator language are numbers, but a
general programming language might also include strings, boolean values, etc.

**Read-eval-print loops.** A typical approach to interacting with an
interpreter is through a read-eval-print loop, or REPL, which is a mode of
interaction that reads an expression, evaluates it, and prints the result for
the user. The Python interactive session is an example of such a loop.

An implementation of a REPL can be largely independent of the interpreter it
uses. The function `read_eval_print_loop` below buffers input from the user,
constructs an expression using the language-specific `scheme_read` function,
then prints the result of applying `calc_eval` to that expression.


​    

    >>> def read_eval_print_loop():
            """Run a read-eval-print loop for calculator."""
            while True:
                src = buffer_input()
                while src.more_on_line:
                    expression = scheme_read(src)
                    print(calc_eval(expression))


This version of `read_eval_print_loop` contains all of the essential
components of an interactive interface. An example session would look like:


​    

    > (* 1 2 3)
    6
    > (+)
    0
    > (+ 2 (/ 4 8))
    2.5
    > (+ 2 2) (* 3 3)
    4
    9
    > (+ 1
         (- 23)
         (* 4 2.5))
    -12


This loop implementation has no mechanism for termination or error handling.
We can improve the interface by reporting errors to the user. We can also
allow the user to exit the loop by signalling a keyboard interrupt
(`Control-C` on UNIX) or end-of-file exception (`Control-D` on UNIX). To
enable these improvements, we place the original suite of the `while`
statement within a `try` statement. The first `except` clause handles
`SyntaxError` and `ValueError` exceptions raised by `scheme_read` as well as
`TypeError` and `ZeroDivisionError` exceptions raised by `calc_eval`.


​    

    >>> def read_eval_print_loop():
            """Run a read-eval-print loop for calculator."""
            while True:
                try:
                    src = buffer_input()
                    while src.more_on_line:
                        expression = scheme_read(src)
                        print(calc_eval(expression))
                except (SyntaxError, TypeError, ValueError, ZeroDivisionError) as err:
                    print(type(err).__name__ + ':', err)
                except (KeyboardInterrupt, EOFError):  # <Control>-D, etc.
                    print('Calculation completed.')
                    return


This loop implementation reports errors without exiting the loop. Rather than
exiting the program on an error, restarting the loop after an error message
lets users revise their expressions. Upon importing the `readline` module,
users can even recall their previous inputs using the up arrow or `Control-P`.
The final result provides an informative error reporting interface:


​    

    > )
    SyntaxError: unexpected token: )
    > 2.3.4
    ValueError: invalid numeral: 2.3.4
    > +
    TypeError: + is not a number or call expression
    > (/ 5)
    TypeError: / requires exactly 2 arguments
    > (/ 1 0)
    ZeroDivisionError: division by zero


As we generalize our interpreter to new languages other than Calculator, we
will see that the `read_eval_print_loop` is parameterized by a parsing
function, an evaluation function, and the exception types handled by the `try`
statement. Beyond these changes, all REPLs can be implemented using the same
structure.

_Continue_ : [ 3.5 Interpreters for Languages with Abstraction

](../pages/35-interpreters-for-languages-with-abstraction.html)

## 3.5 Interpreters for Languages with Abstraction

The Calculator language provides a means of combination through nested call
expressions. However, there is no way to define new operators, give names to
values, or express general methods of computation. Calculator does not support
abstraction in any way. As a result, it is not a particularly powerful or
general programming language. We now turn to the task of defining a general
programming language that supports abstraction by binding names to values and
defining new operations.

Unlike the previous section, which presented a complete interpreter as Python
source code, this section takes a descriptive approach. The companion project
asks you to implement the ideas presented here by building a fully functional
Scheme interpreter.

### 3.5.1 Structure

This section describes the general structure of a Scheme interpreter.
Completing that project will produce a working implementation of the
interpreter described here.

An interpreter for Scheme can share much of the same structure as the
Calculator interpreter. A parser produces an expression that is interpreted by
an evaluator. The evaluation function inspects the form of an expression, and
for call expressions it calls a function to apply a procedure to some
arguments. Much of the difference in evaluators is associated with special
forms, user-defined functions, and implementing the environment model of
computation.

**Parsing.** The [scheme_reader](../examples/scalc/scheme_reader.py.html) and
[scheme_tokens](../examples/scalc/scheme_tokens.py.html) modules from the
Calculator interpreter are nearly sufficient to parse any valid Scheme
expression. However, it does not yet support quotation or dotted lists. A full
Scheme interpreter should be able to parse the following input expression.


​    

    >>> read_line("(car '(1 . 2))")
    Pair('car', Pair(Pair('quote', Pair(Pair(1, 2), nil)), nil))


Your first task in implementing the Scheme interpreter will be to extend
[scheme_reader](../examples/scalc/scheme_reader.py.html) to correctly parse
dotted lists and quotation.

**Evaluation.** Scheme is evaluated one expression at a time. A skeleton
implementation of the evaluator is defined in `scheme.py` of the companion
project. Each expression returned from `scheme_read` is passed to the
`scheme_eval` function, which evaluates an expression `expr` in the current
environment `env`.

The `scheme_eval` function evaluates the different forms of expressions in
Scheme: primitives, special forms, and call expressions. The form of a
combination in Scheme can be determined by inspecting its first element. Each
special form has its own evaluation rule. A simplified implementation of
`scheme_eval` appears below. Some error checking and special form handling has
been removed in order to focus our discussion. A complete implementation
appears in the companion project.


​    

    >>> def scheme_eval(expr, env):
            """Evaluate Scheme expression expr in environment env."""
            if scheme_symbolp(expr):
                return env[expr]
            elif scheme_atomp(expr):
                return expr
            first, rest = expr.first, expr.second
            if first == "lambda":
                return do_lambda_form(rest, env)
            elif first == "define":
                do_define_form(rest, env)
                return None
            else:
                procedure = scheme_eval(first, env)
                args = rest.map(lambda operand: scheme_eval(operand, env))
                return scheme_apply(procedure, args, env)


**Procedure application.** The final case above invokes a second process,
procedure application, that is implemented by the function `scheme_apply`. The
procedure application process in Scheme is considerably more general than the
`calc_apply` function in Calculator. It applies two kinds of arguments: a
`PrimtiveProcedure` or a `LambdaProcedure`. A `PrimitiveProcedure` is
implemented in Python; it has an instance attribute `fn` that is bound to a
Python function. In addition, it may or may not require access to the current
environment. This Python function is called whenever the procedure is applied.

A `LambdaProcedure` is implemented in Scheme. It has a `body` attribute that
is a Scheme expression, evaluated whenever the procedure is applied. To apply
the procedure to a list of arguments, the body expression is evaluated in a
new environment. To construct this environment, a new frame is added to the
environment, in which the formal parameters of the procedure are bound to the
arguments. The body is evaluated using `scheme_eval`.

**Eval/apply recursion.** The functions that implement the evaluation process,
`scheme_eval` and `scheme_apply`, are mutually recursive. Evaluation requires
application whenever a call expression is encountered. Application uses
evaluation to evaluate operand expressions into arguments, as well as to
evaluate the body of user-defined procedures. The general structure of this
mutually recursive process appears in interpreters quite generally: evaluation
is defined in terms of application and application is defined in terms of
evaluation.

This recursive cycle ends with language primitives. Evaluation has a base case
that is evaluating a primitive expression. Some special forms also constitute
base cases without recursive calls. Function application has a base case that
is applying a primitive procedure. This mutually recursive structure, between
an eval function that processes expression forms and an apply function that
processes functions and their arguments, constitutes the essence of the
evaluation process.

### 3.5.2 Environments

Now that we have described the structure of our Scheme interpreter, we turn to
implementing the `Frame` class that forms environments. Each `Frame` instance
represents an environment in which symbols are bound to values. A frame has a
dictionary of `bindings`, as well as a `parent` frame that is `None` for the
global frame.

Bindings are not accessed directly, but instead through two `Frame` methods:
`lookup` and `define`. The first implements the look-up procedure of the
environment model of computation described in Chapter 1. A symbol is matched
against the `bindings` of the current frame. If it is found, the value to
which it is bound is returned. If it is not found, look-up proceeds to the
`parent` frame. On the other hand, the `define` method always binds a symbol
to a value in the current frame.

The implementation of `lookup` and the use of `define` are left as exercises.
As an illustration of their use, consider the following example Scheme
program:


​    

    (define (factorial n)
      (if (= n 0) 1 (* n (factorial (- n 1)))))


​    
​    

    (factorial 5)


The first input expression is a `define` special form, evaluated by the
`do_define_form` Python function. Defining a function has several steps:

    1. Check the format of the expression to ensure that it is a well-formed Scheme list with at least two elements following the keyword `define`.
    2. Analyze the first element, in this case a `Pair`, to find the function name `factorial` and formal parameter list `(n)`.
    3. Create a `LambdaProcedure` with the supplied formal parameters, body, and parent environment.
    4. Bind the symbol `factorial` to this function, in the first frame of the current environment. In this case, the environment consists only of the global frame.

The second input is a call expression. The `procedure` passed to
`scheme_apply` is the `LambdaProcedure` just created and bound to the symbol
`factorial`. The `args` passed is a one-element Scheme list `(5)`. To apply
the procedure, a new frame is created that extends the global frame (the
parent environment of the `factorial` procedure). In this frame, the symbol
`n` is bound to the value 5. Then, the body of `factorial` is evaluated in
that environment, and its value is returned.

### 3.5.3 Data as Programs

In thinking about a program that evaluates Scheme expressions, an analogy
might be helpful. One operational view of the meaning of a program is that a
program is a description of an abstract machine. For example, consider again
this procedure to compute factorials:


​    

    (define (factorial n)
      (if (= n 0) 1 (* n (factorial (- n 1)))))


We could express an equivalent program in Python as well, using a conditional
expression.


​    

    >>> def factorial(n):
            return 1 if n == 1 else n * factorial(n - 1)

We may regard this program as the description of a machine containing parts
that decrement, multiply, and test for equality, together with a two-position
switch and another factorial machine. (The factorial machine is infinite
because it contains another factorial machine within it.) The figure below is
a flow diagram for the factorial machine, showing how the parts are wired
together.

![](http://www.composingprograms.com/img/factorial_machine.png)

In a similar way, we can regard the Scheme interpreter as a very special
machine that takes as input a description of a machine. Given this input, the
interpreter configures itself to emulate the machine described. For example,
if we feed our evaluator the definition of factorial the evaluator will be
able to compute factorials.

From this perspective, our Scheme interpreter is seen to be a universal
machine. It mimics other machines when these are described as Scheme programs.
It acts as a bridge between the data objects that are manipulated by our
programming language and the programming language itself. Image that a user
types a Scheme expression into our running Scheme interpreter. From the
perspective of the user, an input expression such as `(+ 2 2)` is an
expression in the programming language, which the interpreter should evaluate.
From the perspective of the Scheme interpreter, however, the expression is
simply a sentence of words that is to be manipulated according to a well-
defined set of rules.

That the user's programs are the interpreter's data need not be a source of
confusion. In fact, it is sometimes convenient to ignore this distinction, and
to give the user the ability to explicitly evaluate a data object as an
expression. In Scheme, we use this facility whenever employing the `run`
procedure. Similar functions exist in Python: the `eval` function will
evaluate a Python expression and the `exec` function will execute a Python
statement. Thus,


​    

    >>> eval('2+2')
    4


and


​    

    >>> 2+2
    4

both return the same result. Evaluating expressions that are constructed as a
part of execution is a common and powerful feature in dynamic programming
languages. In few languages is this practice as common as in Scheme, but the
ability to construct and evaluate expressions during the course of execution
of a program can prove to be a valuable tool for any programmer.