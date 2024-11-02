# Chapter 2: Building Abstractions with Data

## 2.1 Introduction

We concentrated in Chapter 1 on computational processes and on the role of
functions in program design. We saw how to use primitive data (numbers) and
primitive operations (arithmetic), how to form compound functions through
composition and control, and how to create functional abstractions by giving
names to processes. We also saw that higher-order functions enhance the power
of our language by enabling us to manipulate, and thereby to reason, in terms
of general methods of computation. This is much of the essence of programming.

This chapter focuses on data. The techniques we investigate here will allow us
to represent and manipulate information about many different domains. Due to
the explosive growth of the Internet, a vast amount of structured information
is freely available to all of us online, and computation can be applied to a
vast range of different problems. Effective use of built-in and user-defined
data types are fundamental to data processing applications.

### 2.1.1 Native Data Types

Every value in Python has a _class_ that determines what type of value it is.
Values that share a class also share behavior. For example, the integers `1`
and `2` are both instances of the `int` class. These two values can be treated
similarly. For example, they can both be negated or added to another integer.
The built-in `type` function allows us to inspect the class of any value.


​    

    >>> type(2)
    <class 'int'>


The values we have used so far are instances of a small number of _native_
data types that are built into the Python language. Native data types have the
following properties:

    1. There are expressions that evaluate to values of native types, called _literals_.
    2. There are built-in functions and operators to manipulate values of native types.

The `int` class is the native data type used to represent integers. Integer
literals (sequences of adjacent numerals) evaluate to `int` values, and
mathematical operators manipulate these values.


​    

    >>> 12 + 3000000000000000000000000
    3000000000000000000000012


Python includes three native numeric types: integers (`int`), real numbers
(`float`), and complex numbers (`complex`).


​    

    >>> type(1.5)
    <class 'float'>
    >>> type(1+1j)
    <class 'complex'>


**Floats.** The name `float` comes from the way in which real numbers are
represented in Python and many other programming languages: a "floating point"
representation. While the details of how numbers are represented is not a
topic for this text, some high-level differences between `int` and `float`
objects are important to know. In particular, `int` objects represent integers
exactly, without any approximation or limits on their size. On the other hand,
`float` objects can represent a wide range of fractional numbers, but not all
numbers can be represented exactly, and there are minimum and maximum values.
Therefore, `float` values should be treated as approximations to real values.
These approximations have only a finite amount of precision. Combining `float`
values can lead to approximation errors; both of the following expressions
would evaluate to `7` if not for approximation.


​    

    >>> 7 / 3 * 3
    7.0


​    
​    

    >>> 1 / 3 * 7 * 3
    6.999999999999999


Although `int` values are combined above, dividing one `int` by another yields
a `float` value: a truncated finite approximation to the actual ratio of the
two integers divided.


​    

    >>> type(1/3)
    <class 'float'>
    >>> 1/3
    0.3333333333333333


Problems with this approximation appear when we conduct equality tests.


​    

    >>> 1/3 == 0.333333333333333312345  # Beware of float approximation
    True


These subtle differences between the `int` and `float` class have wide-ranging
consequences for writing programs, and so they are details that must be
memorized by programmers. Fortunately, there are only a handful of native data
types, limiting the amount of memorization required to become proficient in a
programming language. Moreover, these same details are consistent across many
programming languages, enforced by community guidelines such as the [IEEE 754
floating point standard](http://en.wikipedia.org/wiki/IEEE_floating_point).

**Non-numeric types.** Values can represent many other types of data, such as
sounds, images, locations, web addresses, network connections, and more. A few
are represented by native data types, such as the `bool` class for values
`True` and `False`. The type for most values must be defined by programmers
using the means of combination and abstraction that we will develop in this
chapter.

The following sections introduce more of Python's native data types, focusing
on the role they play in creating useful data abstractions. For those
interested in further details, a chapter on [native data
types](http://getpython3.com/diveintopython3/native-datatypes.html) in the
online book Dive Into Python 3 gives a pragmatic overview of all Python's
native data types and how to manipulate them, including numerous usage
examples and practical tips.

_Continue_ : [ 2.2 Data Abstraction ](../pages/22-data-abstraction.html)

## 2.2 Data Abstraction

As we consider the wide set of things in the world that we would like to
represent in our programs, we find that most of them have compound structure.
For example, a geographic position has latitude and longitude coordinates. To
represent positions, we would like our programming language to have the
capacity to couple together a latitude and longitude to form a pair, a
_compound data_ value that our programs can manipulate as a single conceptual
unit, but which also has two parts that can be considered individually.

The use of compound data enables us to increase the modularity of our
programs. If we can manipulate geographic positions as whole values, then we
can shield parts of our program that compute using positions from the details
of how those positions are represented. The general technique of isolating the
parts of a program that deal with how data are represented from the parts that
deal with how data are manipulated is a powerful design methodology called
_data abstraction_. Data abstraction makes programs much easier to design,
maintain, and modify.

Data abstraction is similar in character to functional abstraction. When we
create a functional abstraction, the details of how a function is implemented
can be suppressed, and the particular function itself can be replaced by any
other function with the same overall behavior. In other words, we can make an
abstraction that separates the way the function is used from the details of
how the function is implemented. Analogously, data abstraction isolates how a
compound data value is used from the details of how it is constructed.

The basic idea of data abstraction is to structure programs so that they
operate on abstract data. That is, our programs should use data in such a way
as to make as few assumptions about the data as possible. At the same time, a
concrete data representation is defined as an independent part of the program.

These two parts of a program, the part that operates on abstract data and the
part that defines a concrete representation, are connected by a small set of
functions that implement abstract data in terms of the concrete
representation. To illustrate this technique, we will consider how to design a
set of functions for manipulating rational numbers.

### 2.2.1 Example: Rational Numbers

A rational number is a ratio of integers, and rational numbers constitute an
important sub-class of real numbers. A rational number such as `1/3` or
`17/29` is typically written as:


​    

    <numerator>/<denominator>


where both the `<numerator>` and `<denominator>` are placeholders for integer
values. Both parts are needed to exactly characterize the value of the
rational number. Actually dividing integers produces a `float` approximation,
losing the exact precision of integers.


​    

    >>> 1/3
    0.3333333333333333
    >>> 1/3 == 0.333333333333333300000  # Dividing integers yields an approximation
    True


However, we can create an exact representation for rational numbers by
combining together the numerator and denominator.

We know from using functional abstractions that we can start programming
productively before we have an implementation of some parts of our program.
Let us begin by assuming that we already have a way of constructing a rational
number from a numerator and a denominator. We also assume that, given a
rational number, we have a way of selecting its numerator and its denominator
component. Let us further assume that the constructor and selectors are
available as the following three functions:

  * `rational(n, d)` returns the rational number with numerator `n` and denominator `d`.
  * `numer(x)` returns the numerator of the rational number `x`.
  * `denom(x)` returns the denominator of the rational number `x`.

We are using here a powerful strategy for designing programs: _wishful
thinking_. We haven't yet said how a rational number is represented, or how
the functions `numer`, `denom`, and `rational` should be implemented. Even so,
if we did define these three functions, we could then add, multiply, print,
and test equality of rational numbers:


​    

    >>> def add_rationals(x, y):
            nx, dx = numer(x), denom(x)
            ny, dy = numer(y), denom(y)
            return rational(nx * dy + ny * dx, dx * dy)


​    
​    

    >>> def mul_rationals(x, y):
            return rational(numer(x) * numer(y), denom(x) * denom(y))


​    
​    

    >>> def print_rational(x):
            print(numer(x), '/', denom(x))


​    
​    

    >>> def rationals_are_equal(x, y):
            return numer(x) * denom(y) == numer(y) * denom(x)


Now we have the operations on rational numbers defined in terms of the
selector functions `numer` and `denom`, and the constructor function
`rational`, but we haven't yet defined these functions. What we need is some
way to glue together a numerator and a denominator into a compound value.

### 2.2.2 Pairs

To enable us to implement the concrete level of our data abstraction, Python
provides a compound structure called a `list`, which can be constructed by
placing expressions within square brackets separated by commas. Such an
expression is called a list literal.


​    

    >>> [10, 20]
    [10, 20]


The elements of a list can be accessed in two ways. The first way is via our
familiar method of multiple assignment, which unpacks a list into its elements
and binds each element to a different name.


​    

    >>> pair = [10, 20]
    >>> pair
    [10, 20]
    >>> x, y = pair
    >>> x
    10
    >>> y
    20


A second method for accessing the elements in a list is by the element
selection operator, also expressed using square brackets. Unlike a list
literal, a square-brackets expression directly following another expression
does not evaluate to a `list` value, but instead selects an element from the
value of the preceding expression.


​    

    >>> pair[0]
    10
    >>> pair[1]
    20


Lists in Python (and sequences in most other programming languages) are
0-indexed, meaning that the index 0 selects the first element, index 1 selects
the second, and so on. One intuition that supports this indexing convention is
that the index represents how far an element is offset from the beginning of
the list.

The equivalent function for the element selection operator is called
`getitem`, and it also uses 0-indexed positions to select elements from a
list.


​    

    >>> from operator import getitem
    >>> getitem(pair, 0)
    10
    >>> getitem(pair, 1)
    20


Two-element lists are not the only method of representing pairs in Python. Any
way of bundling two values together into one can be considered a pair. Lists
are a common method to do so. Lists can also contain more than two elements,
as we will explore later in the chapter.

**Representing Rational Numbers.** We can now represent a rational number as a
pair of two integers: a numerator and a denominator.


​    

    >>> def rational(n, d):
            return [n, d]


​    
​    

    >>> def numer(x):
            return x[0]


​    
​    

    >>> def denom(x):
            return x[1]


Together with the arithmetic operations we defined earlier, we can manipulate
rational numbers with the functions we have defined.


​    

    >>> half = rational(1, 2)
    >>> print_rational(half)
    1 / 2
    >>> third = rational(1, 3)
    >>> print_rational(mul_rationals(half, third))
    1 / 6
    >>> print_rational(add_rationals(third, third))
    6 / 9


As the example above shows, our rational number implementation does not reduce
rational numbers to lowest terms. We can remedy this flaw by changing the
implementation of `rational`. If we have a function for computing the greatest
common denominator of two integers, we can use it to reduce the numerator and
the denominator to lowest terms before constructing the pair. As with many
useful tools, such a function already exists in the Python Library.


​    

    >>> from fractions import gcd
    >>> def rational(n, d):
            g = gcd(n, d)
            return (n//g, d//g)


The floor division operator, `//`, expresses integer division, which rounds
down the fractional part of the result of division. Since we know that `g`
divides both `n` and `d` evenly, integer division is exact in this case. This
revised `rational` implementation ensures that rationals are expressed in
lowest terms.


​    

    >>> print_rational(add_rationals(third, third))
    2 / 3


This improvement was accomplished by changing the constructor without changing
any of the functions that implement the actual arithmetic operations.

### 2.2.3 Abstraction Barriers

Before continuing with more examples of compound data and data abstraction,
let us consider some of the issues raised by the rational number example. We
defined operations in terms of a constructor `rational` and selectors `numer`
and `denom`. In general, the underlying idea of data abstraction is to
identify a basic set of operations in terms of which all manipulations of
values of some kind will be expressed, and then to use only those operations
in manipulating the data. By restricting the use of operations in this way, it
is much easier to change the representation of abstract data without changing
the behavior of a program.

For rational numbers, different parts of the program manipulate rational
numbers using different operations, as described in this table.

| **Parts of the program that...**                  | **Treat rationals as...**   | **Using only...**                                            |
| ------------------------------------------------- | --------------------------- | ------------------------------------------------------------ |
| Use rational numbers to perform computation       | whole data values           | `add_rational, mul_rational, rationals_are_equal, print_rational` |
| Create rationals or implement rational operations | numerators and denominators | `rational, numer, denom`                                     |
| Implement selectors and constructor for rationals | two-element lists           | list literals and element selection                          |

In each layer above, the functions in the final column enforce an abstraction
barrier. These functions are called by a higher level and implemented using a
lower level of abstraction.

An abstraction barrier violation occurs whenever a part of the program that
can use a higher level function instead uses a function in a lower level. For
example, a function that computes the square of a rational number is best
implemented in terms of `mul_rational`, which does not assume anything about
the implementation of a rational number.


​    

    >>> def square_rational(x):
            return mul_rational(x, x)


Referring directly to numerators and denominators would violate one
abstraction barrier.


​    

    >>> def square_rational_violating_once(x):
            return rational(numer(x) * numer(x), denom(x) * denom(x))


Assuming that rationals are represented as two-element lists would violate two
abstraction barriers.


​    

    >>> def square_rational_violating_twice(x):
            return [x[0] * x[0], x[1] * x[1]]


Abstraction barriers make programs easier to maintain and to modify. The fewer
functions that depend on a particular representation, the fewer changes are
required when one wants to change that representation. All of these
implementations of `square_rational` have the correct behavior, but only the
first is robust to future changes. The `square_rational` function would not
require updating even if we altered the representation of rational numbers. By
contrast, `square_rational_violating_once` would need to be changed whenever
the selector or constructor signatures changed, and
`square_rational_violating_twice` would require updating whenever the
implementation of rational numbers changed.

### 2.2.4 The Properties of Data

Abstraction barriers shape the way in which we think about data. A valid
representation of a rational number is not restricted to any particular
implementation (such as a two-element list); it is a value returned by
`rational` that can be passed to `numer`, and `denom`. In addition, the
appropriate relationship must hold among the constructor and selectors. That
is, if we construct a rational number `x` from integers `n` and `d`, then it
should be the case that `numer(x)/denom(x)` is equal to `n/d`.

In general, we can express abstract data using a collection of selectors and
constructors, together with some behavior conditions. As long as the behavior
conditions are met (such as the division property above), the selectors and
constructors constitute a valid representation of a kind of data. The
implementation details below an abstraction barrier may change, but if the
behavior does not, then the data abstraction remains valid, and any program
written using this data abstraction will remain correct.

This point of view can be applied broadly, including to the pair values that
we used to implement rational numbers. We never actually said much about what
a pair was, only that the language supplied the means to create and manipulate
lists with two elements. The behavior we require to implement a pair is that
it glues two values together. Stated as a behavior condition,

  * If a pair `p` was constructed from values `x` and `y`, then `select(p, 0)` returns `x`, and `select(p, 1)` returns `y`.

We don't actually need the `list` type to create pairs. Instead, we can
implement two functions `pair` and `select` that fulfill this description just
as well as a two-element list.


​    

    >>> def pair(x, y):
            """Return a function that represents a pair."""
            def get(index):
                if index == 0:
                    return x
                elif index == 1:
                    return y
            return get


​    
​    

    >>> def select(p, i):
            """Return the element at index i of pair p."""
            return p(i)


With this implementation, we can create and manipulate pairs.


​    

    >>> p = pair(20, 14)
    >>> select(p, 0)
    20
    >>> select(p, 1)
    14


This use of higher-order functions corresponds to nothing like our intuitive
notion of what data should be. Nevertheless, these functions suffice to
represent pairs in our programs. Functions are sufficient to represent
compound data.

The point of exhibiting the functional representation of a pair is not that
Python actually works this way (lists are implemented more directly, for
efficiency reasons) but that it could work this way. The functional
representation, although obscure, is a perfectly adequate way to represent
pairs, since it fulfills the only conditions that pairs need to fulfill. The
practice of data abstraction allows us to switch among representations easily.

_Continue_ : [ 2.3 Sequences ](../pages/23-sequences.html)

## 2.3 Sequences

A sequence is an ordered collection of values. The sequence is a powerful,
fundamental abstraction in computer science. Sequences are not instances of a
particular built-in type or abstract data representation, but instead a
collection of behaviors that are shared among several different types of data.
That is, there are many kinds of sequences, but they all share common
behavior. In particular,

**Length.** A sequence has a finite length. An empty sequence has length 0.

**Element selection.** A sequence has an element corresponding to any non-
negative integer index less than its length, starting at 0 for the first
element.

Python includes several native data types that are sequences, the most
important of which is the `list`.

### 2.3.1 Lists

A `list` value is a sequence that can have arbitrary length. Lists have a
large set of built-in behaviors, along with specific syntax to express those
behaviors. We have already seen the list literal, which evaluates to a `list`
instance, as well as an element selection expression that evaluates to a value
in the list. The built-in `len` function returns the length of a sequence.
Below, `digits` is a list with four elements. The element at index 3 is 8.


​    

    >>> digits = [1, 8, 2, 8]
    >>> len(digits)
    4
    >>> digits[3]
    8


Additionally, lists can be added together and multiplied by integers. For
sequences, addition and multiplication do not add or multiply elements, but
instead combine and replicate the sequences themselves. That is, the `add`
function in the `operator` module (and the `+` operator) yields a list that is
the concatenation of the added arguments. The `mul` function in `operator`
(and the `*` operator) can take a list and an integer `k` to return the list
that consists of `k` repetitions of the original list.


​    

    >>> [2, 7] + digits * 2
    [2, 7, 1, 8, 2, 8, 1, 8, 2, 8]


Any values can be included in a list, including another list. Element
selection can be applied multiple times in order to select a deeply nested
element in a list containing lists.


​    

    >>> pairs = [[10, 20], [30, 40]]
    >>> pairs[1]
    [30, 40]
    >>> pairs[1][0]
    30


### 2.3.2 Sequence Iteration

In many cases, we would like to iterate over the elements of a sequence and
perform some computation for each element in turn. This pattern is so common
that Python has an additional control statement to process sequential data:
the `for` statement.

Consider the problem of counting how many times a value appears in a sequence.
We can implement a function to compute this count using a `while` loop.


​    

    >>> def count(s, value):
            """Count the number of occurrences of value in sequence s."""
            total, index = 0, 0
            while index < len(s):
                if s[index] == value:
                    total = total + 1
                index = index + 1
            return total


​    
​    

    >>> count(digits, 8)
    2


The Python `for` statement can simplify this function body by iterating over
the element values directly without introducing the name `index` at all.


​    

    >>> def count(s, value):
            """Count the number of occurrences of value in sequence s."""
            total = 0
            for elem in s:
                if elem == value:
                    total = total + 1
            return total


​    
​    

    >>> count(digits, 8)
    2


A `for` statement consists of a single clause with the form:


​    

    for <name> in <expression>:
        <suite>


A `for` statement is executed by the following procedure:

    1. Evaluate the header `<expression>`, which must yield an iterable value.
    2. For each element value in that iterable value, in order:
           1. Bind `<name>` to that value in the current frame.
                  2. Execute the `<suite>`.

This execution procedure refers to _iterable values_. Lists are a type of
sequence, and sequences are iterable values. Their elements are considered in
their sequential order. Python includes other iterable types, but we will
focus on sequences for now; the general definition of the term "iterable"
appears in the section on iterators in Chapter 4.

An important consequence of this evaluation procedure is that `<name>` will be
bound to the last element of the sequence after the `for` statement is
executed. The `for` loop introduces yet another way in which the environment
can be updated by a statement.

**Sequence unpacking.** A common pattern in programs is to have a sequence of
elements that are themselves sequences, but all of a fixed length. A `for`
statement may include multiple names in its header to "unpack" each element
sequence into its respective elements. For example, we may have a list of two-
element lists.


​    

    >>> pairs = [[1, 2], [2, 2], [2, 3], [4, 4]]


and wish to find the number of these pairs that have the same first and second
element.


​    

    >>> same_count = 0


The following `for` statement with two names in its header will bind each name
`x` and `y` to the first and second elements in each pair, respectively.


​    

    >>> for x, y in pairs:
            if x == y:
                same_count = same_count + 1


​    
​    

    >>> same_count
    2


This pattern of binding multiple names to multiple values in a fixed-length
sequence is called _sequence unpacking_ ; it is the same pattern that we see
in assignment statements that bind multiple names to multiple values.

**Ranges.** A `range` is another built-in type of sequence in Python, which
represents a range of integers. Ranges are created with `range`, which takes
two integer arguments: the first number and one beyond the last number in the
desired range.


​    

    >>> range(1, 10)  # Includes 1, but not 10
    range(1, 10)


Calling the `list` constructor on a range evaluates to a list with the same
elements as the range, so that the elements can be easily inspected.


​    

    >>> list(range(5, 8))
    [5, 6, 7]


If only one argument is given, it is interpreted as one beyond the last value
for a range that starts at 0.


​    

    >>> list(range(4))
    [0, 1, 2, 3]


Ranges commonly appear as the expression in a `for` header to specify the
number of times that the suite should be executed: A common convention is to
use a single underscore character for the name in the `for` header if the name
is unused in the suite:


​    

    >>> for _ in range(3):
            print('Go Bears!')
    
    Go Bears!
    Go Bears!
    Go Bears!


This underscore is just another name in the environment as far as the
interpreter is concerned, but has a conventional meaning among programmers
that indicates the name will not appear in any future expressions.

### 2.3.3 Sequence Processing

Sequences are such a common form of compound data that whole programs are
often organized around this single abstraction. Modular components that have
sequences as both inputs and outputs can be mixed and matched to perform data
processing. Complex components can be defined by chaining together a pipeline
of sequence processing operations, each of which is simple and focused.

**List Comprehensions.** Many sequence processing operations can be expressed
by evaluating a fixed expression for each element in a sequence and collecting
the resulting values in a result sequence. In Python, a list comprehension is
an expression that performs such a computation.


​    

    >>> odds = [1, 3, 5, 7, 9]
    >>> [x+1 for x in odds]
    [2, 4, 6, 8, 10]


The `for` keyword above is not part of a `for` statement, but instead part of
a list comprehension because it is contained within square brackets. The sub-
expression `x+1` is evaluated with `x` bound to each element of `odds` in
turn, and the resulting values are collected into a list.

Another common sequence processing operation is to select a subset of values
that satisfy some condition. List comprehensions can also express this
pattern, for instance selecting all elements of `odds` that evenly divide
`25`.


​    

    >>> [x for x in odds if 25 % x == 0]
    [1, 5]


The general form of a list comprehension is:


​    

    [<map expression> for <name> in <sequence expression> if <filter expression>]


To evaluate a list comprehension, Python evaluates the `<sequence
expression>`, which must return an iterable value. Then, for each element in
order, the element value is bound to `<name>`, the filter expression is
evaluated, and if it yields a true value, the map expression is evaluated. The
values of the map expression are collected into a list.

**Aggregation.** A third common pattern in sequence processing is to aggregate
all values in a sequence into a single value. The built-in functions `sum`,
`min`, and `max` are all examples of aggregation functions.

By combining the patterns of evaluating an expression for each element,
selecting a subset of elements, and aggregating elements, we can solve
problems using a sequence processing approach.

A perfect number is a positive integer that is equal to the sum of its
divisors. The divisors of `n` are positive integers less than `n` that divide
evenly into `n`. Listing the divisors of `n` can be expressed with a list
comprehension.


​    

    >>> def divisors(n):
            return [1] + [x for x in range(2, n) if n % x == 0]


​    
​    

    >>> divisors(4)
    [1, 2]
    >>> divisors(12)
    [1, 2, 3, 4, 6]


Using `divisors`, we can compute all perfect numbers from 1 to 1000 with
another list comprehension. (1 is typically considered to be a perfect number
as well, but it does not qualify under our definition of `divisors`.)


​    

    >>> [n for n in range(1, 1000) if sum(divisors(n)) == n]
    [6, 28, 496]


We can reuse our definition of `divisors` to solve another problem, finding
the minimum perimeter of a rectangle with integer side lengths, given its
area. The area of a rectangle is its height times its width. Therefore, given
the area and height, we can compute the width. We can assert that both the
width and height evenly divide the area to ensure that the side lengths are
integers.


​    

    >>> def width(area, height):
            assert area % height == 0
            return area // height


The perimeter of a rectangle is the sum of its side lengths.


​    

    >>> def perimeter(width, height):
            return 2 * width + 2 * height


The height of a rectangle with integer side lengths must be a divisor of its
area. We can compute the minimum perimeter by considering all heights.


​    

    >>> def minimum_perimeter(area):
            heights = divisors(area)
            perimeters = [perimeter(width(area, h), h) for h in heights]
            return min(perimeters)


​    
​    

    >>> area = 80
    >>> width(area, 5)
    16
    >>> perimeter(16, 5)
    42
    >>> perimeter(10, 8)
    36
    >>> minimum_perimeter(area)
    36
    >>> [minimum_perimeter(n) for n in range(1, 10)]
    [4, 6, 8, 8, 12, 10, 16, 12, 12]


**Higher-Order Functions.** The common patterns we have observed in sequence
processing can be expressed using higher-order functions. First, evaluating an
expression for each element in a sequence can be expressed by applying a
function to each element.


​    

    >>> def apply_to_all(map_fn, s):
            return [map_fn(x) for x in s]


Selecting only elements for which some expression is true can be expressed by
applying a function to each element.


​    

    >>> def keep_if(filter_fn, s):
            return [x for x in s if filter_fn(x)]


Finally, many forms of aggregation can be expressed as repeatedly applying a
two-argument function to the `reduced` value so far and each element in turn.


​    

    >>> def reduce(reduce_fn, s, initial):
            reduced = initial
            for x in s:
                reduced = reduce_fn(reduced, x)
            return reduced


For example, `reduce` can be used to multiply together all elements of a
sequence. Using `mul` as the `reduce_fn` and 1 as the `initial` value,
`reduce` can be used to multiply together a sequence of numbers.


​    

    >>> reduce(mul, [2, 4, 8], 1)
    64


We can find perfect numbers using these higher-order functions as well.


​    

    >>> def divisors_of(n):
            divides_n = lambda x: n % x == 0
            return [1] + keep_if(divides_n, range(2, n))


​    
​    

    >>> divisors_of(12)
    [1, 2, 3, 4, 6]
    >>> from operator import add
    >>> def sum_of_divisors(n):
            return reduce(add, divisors_of(n), 0)


​    
​    

    >>> def perfect(n):
            return sum_of_divisors(n) == n


​    
​    

    >>> keep_if(perfect, range(1, 1000))
    [1, 6, 28, 496]


**Conventional Names.** In the computer science community, the more common
name for `apply_to_all` is `map` and the more common name for `keep_if` is
`filter`. In Python, the built-in `map` and `filter` are generalizations of
these functions that do not return lists. These functions are discussed in
Chapter 4. The definitions above are equivalent to applying the `list`
constructor to the result of built-in `map` and `filter` calls.


​    

    >>> apply_to_all = lambda map_fn, s: list(map(map_fn, s))
    >>> keep_if = lambda filter_fn, s: list(filter(filter_fn, s))


The `reduce` function is built into the `functools` module of the Python
standard library. In this version, the `initial` argument is optional.


​    

    >>> from functools import reduce
    >>> from operator import mul
    >>> def product(s):
            return reduce(mul, s)


​    
​    

    >>> product([1, 2, 3, 4, 5])
    120


In Python programs, it is more common to use list comprehensions directly
rather than higher-order functions, but both approaches to sequence processing
are widely used.

### 2.3.4 Sequence Abstraction

We have introduced two native data types that satisfy the sequence
abstraction: lists and ranges. Both satisfy the conditions with which we began
this section: length and element selection. Python includes two more behaviors
of sequence types that extend the sequence abstraction.

**Membership.** A value can be tested for membership in a sequence. Python has
two operators `in` and `not in` that evaluate to `True` or `False` depending
on whether an element appears in a sequence.


​    

    >>> digits
    [1, 8, 2, 8]
    >>> 2 in digits
    True
    >>> 1828 not in digits
    True


**Slicing.** Sequences contain smaller sequences within them. A _slice_ of a
sequence is any contiguous span of the original sequence, designated by a pair
of integers. As with the `range` constructor, the first integer indicates the
starting index of the slice and the second indicates one beyond the ending
index.

In Python, sequence slicing is expressed similarly to element selection, using
square brackets. A colon separates the starting and ending indices. Any bound
that is omitted is assumed to be an extreme value: 0 for the starting index,
and the length of the sequence for the ending index.


​    

    >>> digits[0:2]
    [1, 8]
    >>> digits[1:]
    [8, 2, 8]


Enumerating these additional behaviors of the Python sequence abstraction
gives us an opportunity to reflect upon what constitutes a useful data
abstraction in general. The richness of an abstraction (that is, how many
behaviors it includes) has consequences. For users of an abstraction,
additional behaviors can be helpful. On the other hand, satisfying the
requirements of a rich abstraction with a new data type can be challenging.
Another negative consequence of rich abstractions is that they take longer for
users to learn.

Sequences have a rich abstraction because they are so ubiquitous in computing
that learning a few complex behaviors is justified. In general, most user-
defined abstractions should be kept as simple as possible.

**Further reading.** Slice notation admits a variety of special cases, such as
negative starting values, ending values, and step sizes. A complete
description appears in the subsection called [slicing a
list](http://getpython3.com/diveintopython3/native-
datatypes.html#slicinglists) in Dive Into Python 3. In this chapter, we will
only use the basic features described above.

### 2.3.5 Strings

Text values are perhaps more fundamental to computer science than even
numbers. As a case in point, Python programs are written and stored as text.
The native data type for text in Python is called a string, and corresponds to
the constructor `str`.

There are many details of how strings are represented, expressed, and
manipulated in Python. Strings are another example of a rich abstraction, one
that requires a substantial commitment on the part of the programmer to
master. This section serves as a condensed introduction to essential string
behaviors.

String literals can express arbitrary text, surrounded by either single or
double quotation marks.


​    

    >>> 'I am string!'
    'I am string!'
    >>> "I've got an apostrophe"
    "I've got an apostrophe"
    >>> 'æ¨å¥½'
    'æ¨å¥½'


We have seen strings already in our code, as docstrings, in calls to `print`,
and as error messages in `assert` statements.

Strings satisfy the two basic conditions of a sequence that we introduced at
the beginning of this section: they have a length and they support element
selection.


​    

    >>> city = 'Berkeley'
    >>> len(city)
    8
    >>> city[3]
    'k'


The elements of a string are themselves strings that have only a single
character. A character is any single letter of the alphabet, punctuation mark,
or other symbol. Unlike many other programming languages, Python does not have
a separate character type; any text is a string, and strings that represent
single characters have a length of 1.

Like lists, strings can also be combined via addition and multiplication.


​    

    >>> 'Berkeley' + ', CA'
    'Berkeley, CA'
    >>> 'Shabu ' * 2
    'Shabu Shabu '


**Membership.** The behavior of strings diverges from other sequence types in
Python. The string abstraction does not conform to the full sequence
abstraction that we described for lists and ranges. In particular, the
membership operator `in` applies to strings, but has an entirely different
behavior than when it is applied to sequences. It matches substrings rather
than elements.


​    

    >>> 'here' in "Where's Waldo?"
    True


**Multiline Literals.** Strings aren't limited to a single line. Triple quotes
delimit string literals that span multiple lines. We have used this triple
quoting extensively already for docstrings.


​    

    >>> """The Zen of Python
    claims, Readability counts.
    Read more: import this."""
    'The Zen of Python\nclaims, "Readability counts."\nRead more: import this.'


In the printed result above, the `\n` (pronounced "_backslash en_ ") is a
single element that represents a new line. Although it appears as two
characters (backslash and "n"), it is considered a single character for the
purposes of length and element selection.

**String Coercion.** A string can be created from any object in Python by
calling the `str` constructor function with an object value as its argument.
This feature of strings is useful for constructing descriptive strings from
objects of various types.


​    

    >>> str(2) + ' is an element of ' + str(digits)
    '2 is an element of [1, 8, 2, 8]'


**Further reading.** Encoding text in computers is a complex topic. In this
chapter, we will abstract away the details of how strings are represented.
However, for many applications, the particular details of how strings are
encoded by computers is essential knowledge. [The strings chapter of Dive Into
Python 3](http://getpython3.com/diveintopython3/strings.html) provides a
description of character encodings and Unicode.

### 2.3.6 Trees

Our ability to use lists as the elements of other lists provides a new means
of combination in our programming language. This ability is called a _closure
property_ of a data type. In general, a method for combining data values has a
closure property if the result of combination can itself be combined using the
same method. Closure is the key to power in any means of combination because
it permits us to create hierarchical structures  structures made up of
parts, which themselves are made up of parts, and so on.

We can visualize lists in environment diagrams using _box-and-pointer_
notation. A list is depicted as adjacent boxes that contain the elements of
the list. Primitive values such as numbers, strings, boolean values, and
`None` appear within an element box. Composite values, such as function values
and other lists, are indicated by an arrow.

one_two = [1, 2] nested = [[1, 2], [], [[3, False, None], [4, lambda: 5]]]

Nesting lists within lists can introduce complexity. The _tree_ is a
fundamental data abstraction that imposes regularity on how hierarchical
values are structured and manipulated.

A tree has a root label and a sequence of branches. Each branch of a tree is a
tree. A tree with no branches is called a leaf. Any tree contained within a
tree is called a sub-tree of that tree (such as a branch of a branch). The
root of each sub-tree of a tree is called a node in that tree.

The data abstraction for a tree consists of the constructor `tree` and the
selectors `label` and `branches`. We begin with a simplified version.


​    

    >>> def tree(root_label, branches=[]):
            for branch in branches:
                assert is_tree(branch), 'branches must be trees'
            return [root_label] + list(branches)


​    
​    

    >>> def label(tree):
            return tree[0]


​    
​    

    >>> def branches(tree):
            return tree[1:]


A tree is well-formed only if it has a root label and all branches are also
trees. The `is_tree` function is applied in the `tree` constructor to verify
that all branches are well-formed.


​    

    >>> def is_tree(tree):
            if type(tree) != list or len(tree) < 1:
                return False
            for branch in branches(tree):
                if not is_tree(branch):
                    return False
            return True


The `is_leaf` function checks whether or not a tree has branches.


​    

    >>> def is_leaf(tree):
            return not branches(tree)


Trees can be constructed by nested expressions. The following tree `t` has
root label 3 and two branches.


​    

    >>> t = tree(3, [tree(1), tree(2, [tree(1), tree(1)])])
    >>> t
    [3, [1], [2, [1], [1]]]
    >>> label(t)
    3
    >>> branches(t)
    [[1], [2, [1], [1]]]
    >>> label(branches(t)[1])
    2
    >>> is_leaf(t)
    False
    >>> is_leaf(branches(t)[0])
    True


Tree-recursive functions can be used to construct trees. For example, the nth
Fibonacci tree has a root label of the nth Fibonacci number and, for `n > 1`,
two branches that are also Fibonacci trees. A Fibonacci tree illustrates the
tree-recursive computation of a Fibonacci number.


​    

    >>> def fib_tree(n):
            if n == 0 or n == 1:
                return tree(n)
            else:
                left, right = fib_tree(n-2), fib_tree(n-1)
                fib_n = label(left) + label(right)
                return tree(fib_n, [left, right])
    >>> fib_tree(5)
    [5, [2, [1], [1, [0], [1]]], [3, [1, [0], [1]], [2, [1], [1, [0], [1]]]]]


Tree-recursive functions are also used to process trees. For example, the
`count_leaves` function counts the leaves of a tree.


​    

    >>> def count_leaves(tree):
          if is_leaf(tree):
              return 1
          else:
              branch_counts = [count_leaves(b) for b in branches(tree)]
              return sum(branch_counts)
    >>> count_leaves(fib_tree(5))
    8


**Partition trees.** Trees can also be used to represent the partitions of an
integer. A partition tree for `n` using parts up to size `m` is a binary (two
branch) tree that represents the choices taken during computation. In a non-
leaf partition tree:

  * the left (index 0) branch contains all ways of partitioning `n` using at least one `m`,
  * the right (index 1) branch contains partitions using parts up to `m-1`, and
  * the root label is `m`.

The labels at the leaves of a partition tree express whether the path from the
root of the tree to the leaf represents a successful partition of `n`.


​    

    >>> def partition_tree(n, m):
            """Return a partition tree of n using parts of up to m."""
            if n == 0:
                return tree(True)
            elif n < 0 or m == 0:
                return tree(False)
            else:
                left = partition_tree(n-m, m)
                right = partition_tree(n, m-1)
                return tree(m, [left, right])


​    
​    

    >>> partition_tree(2, 2)
    [2, [True], [1, [1, [True], [False]], [False]]]


Printing the partitions from a partition tree is another tree-recursive
process that traverses the tree, constructing each partition as a list.
Whenever a `True` leaf is reached, the partition is printed.


​    

    >>> def print_parts(tree, partition=[]):
            if is_leaf(tree):
                if label(tree):
                    print(' + '.join(partition))
            else:
                left, right = branches(tree)
                m = str(label(tree))
                print_parts(left, partition + [m])
                print_parts(right, partition)


​    
​    

    >>> print_parts(partition_tree(6, 4))
    4 + 2
    4 + 1 + 1
    3 + 3
    3 + 2 + 1
    3 + 1 + 1 + 1
    2 + 2 + 2
    2 + 2 + 1 + 1
    2 + 1 + 1 + 1 + 1
    1 + 1 + 1 + 1 + 1 + 1


Slicing can be used on the branches of a tree as well. For example, we may
want to place a restriction on the number of branches in a tree. A binary tree
is either a leaf or a sequence of at most two binary trees. A common tree
transformation called _binarization_ computes a binary tree from an original
tree by grouping together adjacent branches.


​    

    >>> def right_binarize(tree):
            """Construct a right-branching binary tree."""
            if is_leaf(tree):
                return tree
            if len(tree) > 2:
                tree = [tree[0], tree[1:]]
            return [right_binarize(b) for b in tree]


​    
​    

    >>> right_binarize([1, 2, 3, 4, 5, 6, 7])
    [1, [2, [3, [4, [5, [6, 7]]]]]]


### 2.3.7 Linked Lists

So far, we have used only native types to represent sequences. However, we can
also develop sequence representations that are not built into Python. A common
representation of a sequence constructed from nested pairs is called a _linked
list_. The environment diagram below illustrates the linked list
representation of a four-element sequence containing 1, 2, 3, and 4.

four = [1, [2, [3, [4, 'empty']]]]

A linked list is a pair containing the first element of the sequence (in this
case 1) and the rest of the sequence (in this case a representation of 2, 3,
4). The second element is also a linked list. The rest of the inner-most
linked list containing only 4 is `'empty'`, a value that represents an empty
linked list.

Linked lists have recursive structure: the rest of a linked list is a linked
list or `'empty'`. We can define an abstract data representation to validate,
construct, and select the components of linked lists.


​    

    >>> empty = 'empty'
    >>> def is_link(s):
            """s is a linked list if it is empty or a (first, rest) pair."""
            return s == empty or (len(s) == 2 and is_link(s[1]))


​    
​    

    >>> def link(first, rest):
            """Construct a linked list from its first element and the rest."""
            assert is_link(rest), "rest must be a linked list."
            return [first, rest]


​    
​    

    >>> def first(s):
            """Return the first element of a linked list s."""
            assert is_link(s), "first only applies to linked lists."
            assert s != empty, "empty linked list has no first element."
            return s[0]


​    
​    

    >>> def rest(s):
            """Return the rest of the elements of a linked list s."""
            assert is_link(s), "rest only applies to linked lists."
            assert s != empty, "empty linked list has no rest."
            return s[1]


Above, `link` is a constructor and `first` and `rest` are selectors for an
abstract data representation of linked lists. The behavior condition for a
linked list is that, like a pair, its constructor and selectors are inverse
functions.

  * If a linked list `s` was constructed from first element `f` and linked list `r`, then `first(s)` returns `f`, and `rest(s)` returns `r`.

We can use the constructor and selectors to manipulate linked lists.


​    

    >>> four = link(1, link(2, link(3, link(4, empty))))
    >>> first(four)
    1
    >>> rest(four)
    [2, [3, [4, 'empty']]]


Our implementation of this kind of abstract data uses pairs that are two-
element `list` values. It is worth noting that we were also able to implement
pairs using functions, and we can implement linked lists using any pairs,
therefore we could implement linked lists using functions alone.

The linked list can store a sequence of values in order, but we have not yet
shown that it satisfies the sequence abstraction. Using the abstract data
representation we have defined, we can implement the two behaviors that
characterize a sequence: length and element selection.


​    

    >>> def len_link(s):
            """Return the length of linked list s."""
            length = 0
            while s != empty:
                s, length = rest(s), length + 1
            return length


​    
​    

    >>> def getitem_link(s, i):
            """Return the element at index i of linked list s."""
            while i > 0:
                s, i = rest(s), i - 1
            return first(s)


Now, we can manipulate a linked list as a sequence using these functions. (We
cannot yet use the built-in `len` function, element selection syntax, or `for`
statement, but we will soon.)


​    

    >>> len_link(four)
    4
    >>> getitem_link(four, 1)
    2


The series of environment diagrams below illustrate the iterative process by
which `getitem_link` finds the element 2 at index 1 in a linked list. Below,
we have defined the linked list `four` using Python primitives to simplify the
diagrams. This implementation choice violates an abstraction barrier, but
allows us to inspect the computational process more easily for this example.

def first(s): return s[0] def rest(s): return s[1] def getitem_link(s, i):
while i > 0: s, i = rest(s), i - 1 return first(s) four = [1, [2, [3, [4,
'empty']]]] getitem_link(four, 1)

First, the function `getitem_link` is called, creating a local frame.

def first(s): return s[0] def rest(s): return s[1] def getitem_link(s, i):
while i > 0: s, i = rest(s), i - 1 return first(s) four = [1, [2, [3, [4,
'empty']]]] getitem_link(four, 1)

The expression in the `while` header evaluates to true, which causes the
assignment statement in the `while` suite to be executed. The function `rest`
returns the sublist starting with 2.

def first(s): return s[0] def rest(s): return s[1] def getitem_link(s, i):
while i > 0: s, i = rest(s), i - 1 return first(s) four = [1, [2, [3, [4,
'empty']]]] getitem_link(four, 1)

Next, the local name `s` will be updated to refer to the sub-list that begins
with the second element of the original list. Evaluating the `while` header
expression now yields a false value, and so Python evaluates the expression in
the return statement on the final line of `getitem_link`.

def first(s): return s[0] def rest(s): return s[1] def getitem_link(s, i):
while i > 0: s, i = rest(s), i - 1 return first(s) four = [1, [2, [3, [4,
'empty']]]] getitem_link(four, 1)

This final environment diagram shows the local frame for the call to `first`,
which contains the name `s` bound to that same sub-list. The `first` function
selects the value 2 and returns it, which will also be returned from
`getitem_link`.

This example demonstrates a common pattern of computation with linked lists,
where each step in an iteration operates on an increasingly shorter suffix of
the original list. This incremental processing to find the length and elements
of a linked list does take some time to compute. Python's built-in sequence
types are implemented in a different way that does not have a large cost for
computing the length of a sequence or retrieving its elements. The details of
that representation are beyond the scope of this text.

**Recursive manipulation.** Both `len_link` and `getitem_link` are iterative.
They peel away each layer of nested pair until the end of the list (in
`len_link`) or the desired element (in `getitem_link`) is reached. We can also
implement length and element selection using recursion.


​    

    >>> def len_link_recursive(s):
            """Return the length of a linked list s."""
            if s == empty:
                return 0
            return 1 + len_link_recursive(rest(s))


​    
​    

    >>> def getitem_link_recursive(s, i):
            """Return the element at index i of linked list s."""
            if i == 0:
                return first(s)
            return getitem_link_recursive(rest(s), i - 1)


​    
​    

    >>> len_link_recursive(four)
    4
    >>> getitem_link_recursive(four, 1)
    2


These recursive implementations follow the chain of pairs until the end of the
list (in `len_link_recursive`) or the desired element (in
`getitem_link_recursive`) is reached.

Recursion is also useful for transforming and combining linked lists.


​    

    >>> def extend_link(s, t):
            """Return a list with the elements of s followed by those of t."""
            assert is_link(s) and is_link(t)
            if s == empty:
                return t
            else:
                return link(first(s), extend_link(rest(s), t))


​    
​    

    >>> extend_link(four, four)
    [1, [2, [3, [4, [1, [2, [3, [4, 'empty']]]]]]]]


​    
​    

    >>> def apply_to_all_link(f, s):
            """Apply f to each element of s."""
            assert is_link(s)
            if s == empty:
                return s
            else:
                return link(f(first(s)), apply_to_all_link(f, rest(s)))


​    
​    

    >>> apply_to_all_link(lambda x: x*x, four)
    [1, [4, [9, [16, 'empty']]]]


​    
​    

    >>> def keep_if_link(f, s):
            """Return a list with elements of s for which f(e) is true."""
            assert is_link(s)
            if s == empty:
                return s
            else:
                kept = keep_if_link(f, rest(s))
                if f(first(s)):
                    return link(first(s), kept)
                else:
                    return kept


​    
​    

    >>> keep_if_link(lambda x: x%2 == 0, four)
    [2, [4, 'empty']]


​    
​    

    >>> def join_link(s, separator):
            """Return a string of all elements in s separated by separator."""
            if s == empty:
                return ""
            elif rest(s) == empty:
                return str(first(s))
            else:
                return str(first(s)) + separator + join_link(rest(s), separator)


​    
​    

    >>> join_link(four, ", ")
    '1, 2, 3, 4'


**Recursive Construction.** Linked lists are particularly useful when
constructing sequences incrementally, a situation that arises often in
recursive computations.

The `count_partitions` function from Chapter 1 counted the number of ways to
partition an integer `n` using parts up to size `m` via a tree-recursive
process. With sequences, we can also enumerate these partitions explicitly
using a similar process.

We follow the same recursive analysis of the problem as we did while counting:
partitioning `n` using integers up to `m` involves either

    1. partitioning `n-m` using integers up to `m`, or
    2. partitioning `n` using integers up to `m-1`.

For base cases, we find that 0 has an empty partition, while partitioning a
negative integer or using parts smaller than 1 is impossible.


​    

    >>> def partitions(n, m):
            """Return a linked list of partitions of n using parts of up to m.
            Each partition is represented as a linked list.
            """
            if n == 0:
                return link(empty, empty) # A list containing the empty partition
            elif n < 0 or m == 0:
                return empty
            else:
                using_m = partitions(n-m, m)
                with_m = apply_to_all_link(lambda s: link(m, s), using_m)
                without_m = partitions(n, m-1)
                return extend_link(with_m, without_m)


In the recursive case, we construct two sublists of partitions. The first uses
`m`, and so we prepend `m` to each element of the result `using_m` to form
`with_m`.

The result of `partitions` is highly nested: a linked list of linked lists,
and each linked list is represented as nested pairs that are `list` values.
Using `join_link` with appropriate separators, we can display the partitions
in a human-readable manner.


​    

    >>> def print_partitions(n, m):
            lists = partitions(n, m)
            strings = apply_to_all_link(lambda s: join_link(s, " + "), lists)
            print(join_link(strings, "\n"))


​    
​    

    >>> print_partitions(6, 4)
    4 + 2
    4 + 1 + 1
    3 + 3
    3 + 2 + 1
    3 + 1 + 1 + 1
    2 + 2 + 2
    2 + 2 + 1 + 1
    2 + 1 + 1 + 1 + 1
    1 + 1 + 1 + 1 + 1 + 1


_Continue_ : [ 2.4 Mutable Data ](../pages/24-mutable-data.html)

## 2.4 Mutable Data

We have seen how abstraction is vital in helping us to cope with the
complexity of large systems. Effective programming also requires
organizational principles that can guide us in formulating the overall design
of a program. In particular, we need strategies to help us structure large
systems to be modular, meaning that they divide naturally into coherent parts
that can be separately developed and maintained.

One powerful technique for creating modular programs is to incorporate data
that may change state over time. In this way, a single data object can
represent something that evolves independently of the rest of the program. The
behavior of a changing object may be influenced by its history, just like an
entity in the world. Adding state to data is a central ingredient of a
paradigm called object-oriented programming.

### 2.4.1 The Object Metaphor

In the beginning of this text, we distinguished between functions and data:
functions performed operations and data were operated upon. When we included
function values among our data, we acknowledged that data too can have
behavior. Functions could be manipulated as data, but could also be called to
perform computation.

_Objects_ combine data values with behavior. Objects represent information,
but also _behave_ like the things that they represent. The logic of how an
object interacts with other objects is bundled along with the information that
encodes the object's value. When an object is printed, it knows how to spell
itself out in text. If an object is composed of parts, it knows how to reveal
those parts on demand. Objects are both information and processes, bundled
together to represent the properties, interactions, and behaviors of complex
things.

Object behavior is implemented in Python through specialized object syntax and
associated terminology, which we can introduce by example. A date is a kind of
object.


​    

    >>> from datetime import date


The name `date` is bound to a _class_. As we have seen, a class represents a
kind of value. Individual dates are called _instances_ of that class.
Instances can be _constructed_ by calling the class on arguments that
characterize the instance.


​    

    >>> tues = date(2014, 5, 13)


While `tues` was constructed from primitive numbers, it behaves like a date.
For instance, subtracting it from another date will give a time difference,
which we can print.


​    

    >>> print(date(2014, 5, 19) - tues)
    6 days, 0:00:00


Objects have _attributes_ , which are named values that are part of the
object. In Python, like many other programming languages, we use dot notation
to designated an attribute of an object.

> <expression> . <name>

Above, the `<expression>` evaluates to an object, and `<name>` is the name of
an attribute for that object.

Unlike the names that we have considered so far, these attribute names are not
available in the general environment. Instead, attribute names are particular
to the object instance preceding the dot.


​    

    >>> tues.year
    2014


Objects also have _methods_ , which are function-valued attributes.
Metaphorically, we say that the object "knows" how to carry out those methods.
By implementation, methods are functions that compute their results from both
their arguments and their object. For example, The `strftime` method (a
classic function name meant to evoke "string format of time") of `tues` takes
a single argument that specifies how to display a date (e.g., `%A` means that
the day of the week should be spelled out in full).


​    

    >>> tues.strftime('%A, %B %d')
    'Tuesday, May 13'


Computing the return value of `strftime` requires two inputs: the string that
describes the format of the output and the date information bundled into
`tues`. Date-specific logic is applied within this method to yield this
result. We never stated that the 13th of May, 2014, was a Tuesday, but knowing
the corresponding weekday is part of what it means to be a date. By bundling
behavior and information together, this Python object offers us a convincing,
self-contained abstraction of a date.

Dates are objects, but numbers, strings, lists, and ranges are all objects as
well. They represent values, but also behave in a manner that befits the
values they represent. They also have attributes and methods. For instance,
strings have an array of methods that facilitate text processing.


​    

    >>> '1234'.isnumeric()
    True
    >>> 'rOBERT dE nIRO'.swapcase()
    'Robert De Niro'
    >>> 'eyes'.upper().endswith('YES')
    True


In fact, all values in Python are objects. That is, all values have behavior
and attributes. They act like the values they represent.

### 2.4.2 Sequence Objects

Instances of primitive built-in values such as numbers are _immutable_. The
values themselves cannot change over the course of program execution. Lists on
the other hand are _mutable_.

Mutable objects are used to represent values that change over time. A person
is the same person from one day to the next, despite having aged, received a
haircut, or otherwise changed in some way. Similarly, an object may have
changing properties due to _mutating_ operations. For example, it is possible
to change the contents of a list. Most changes are performed by invoking
methods on list objects.

We can introduce many list modification operations through an example that
illustrates the history of playing cards (drastically simplified). Comments in
the examples describe the effect of each method invocation.

Playing cards were invented in China, perhaps around the 9th century. An early
deck had three suits, which corresponded to denominations of money.


​    

    >>> chinese = ['coin', 'string', 'myriad']  # A list literal
    >>> suits = chinese                         # Two names refer to the same list


As cards migrated to Europe (perhaps through Egypt), only the suit of coins
remained in Spanish decks (_oro_).


​    

    >>> suits.pop()             # Remove and return the final element
    'myriad'
    >>> suits.remove('string')  # Remove the first element that equals the argument


Three more suits were added (they evolved in name and design over time),


​    

    >>> suits.append('cup')              # Add an element to the end
    >>> suits.extend(['sword', 'club'])  # Add all elements of a sequence to the end


and Italians called swords _spades_.


​    

    >>> suits[2] = 'spade'  # Replace an element


giving the suits of a traditional Italian deck of cards.


​    

    >>> suits
    ['coin', 'cup', 'spade', 'club']


The French variant used today in the U.S. changes the first two suits:


​    

    >>> suits[0:2] = ['heart', 'diamond']  # Replace a slice
    >>> suits
    ['heart', 'diamond', 'spade', 'club']


Methods also exist for inserting, sorting, and reversing lists. All of these
mutation operations change the value of the list; they do not create new list
objects.

**Sharing and Identity.** Because we have been changing a single list rather
than creating new lists, the object bound to the name `chinese` has also
changed, because it is the same list object that was bound to `suits`!


​    

    >>> chinese  # This name co-refers with "suits" to the same changing list
    ['heart', 'diamond', 'spade', 'club']


This behavior is new. Previously, if a name did not appear in a statement,
then its value would not be affected by that statement. With mutable data,
methods called on one name can affect another name at the same time.

The environment diagram for this example shows how the value bound to
`chinese` is changed by statements involving only `suits`. Step through each
line of the following example to observe these changes.

chinese = ['coin', 'string', 'myriad'] suits = chinese suits.pop()
suits.remove('string') suits.append('cup') suits.extend(['sword', 'club'])
suits[2] = 'spade' suits[0:2] = ['heart', 'diamond']

Lists can be copied using the `list` constructor function. Changes to one list
do not affect another, unless they share structure.


​    

    >>> nest = list(suits)  # Bind "nest" to a second list with the same elements
    >>> nest[0] = suits     # Create a nested list


According to this environment, changing the list referenced by `suits` will
affect the nested list that is the first element of `nest`, but not the other
elements.


​    

    >>> suits.insert(2, 'Joker')  # Insert an element at index 2, shifting the rest
    >>> nest
    [['heart', 'diamond', 'Joker', 'spade', 'club'], 'diamond', 'spade', 'club']


And likewise, undoing this change in the first element of `nest` will change
`suit` as well.


​    

    >>> nest[0].pop(2)
    'Joker'
    >>> suits
    ['heart', 'diamond', 'spade', 'club']


Stepping through this example line by line will show the representation of a
nested list.

suits = ['heart', 'diamond', 'spade', 'club'] nest = list(suits) nest[0] =
suits suits.insert(2, 'Joker') joke = nest[0].pop(2)

Because two lists may have the same contents but in fact be different lists,
we require a means to test whether two objects are the same. Python includes
two comparison operators, called `is` and `is not`, that test whether two
expressions in fact evaluate to the identical object. Two objects are
identical if they are equal in their current value, and any change to one will
always be reflected in the other. Identity is a stronger condition than
equality.


​    

    >>> suits is nest[0]
    True
    >>> suits is ['heart', 'diamond', 'spade', 'club']
    False
    >>> suits == ['heart', 'diamond', 'spade', 'club']
    True


The final two comparisons illustrate the difference between `is` and `==`. The
former checks for identity, while the latter checks for the equality of
contents.

**List comprehensions.** A list comprehension always creates a new list. For
example, the `unicodedata` module tracks the official names of every character
in the Unicode alphabet. We can look up the characters corresponding to names,
including those for card suits.


​    

    >>> from unicodedata import lookup
    >>> [lookup('WHITE ' + s.upper() + ' SUIT') for s in suits]
    ['¡', '¢', '¤', '§']


This resulting list does not share any of its contents with `suits`, and
evaluating the list comprehension does not modify the `suits` list.

You can read more about the Unicode standard for representing text in the
[Unicode section](http://getpython3.com/diveintopython3/strings.html#one-ring-
to-rule-them-all) of Dive into Python 3.

**Tuples.** A tuple, an instance of the built-in `tuple` type, is an immutable
sequence. Tuples are created using a tuple literal that separates element
expressions by commas. Parentheses are optional but used commonly in practice.
Any objects can be placed within tuples.


​    

    >>> 1, 2 + 3
    (1, 5)
    >>> ("the", 1, ("and", "only"))
    ('the', 1, ('and', 'only'))
    >>> type( (10, 20) )
    <class 'tuple'>


Empty and one-element tuples have special literal syntax.


​    

    >>> ()    # 0 elements
    ()
    >>> (10,) # 1 element
    (10,)


Like lists, tuples have a finite length and support element selection. They
also have a few methods that are also available for lists, such as `count` and
`index`.


​    

    >>> code = ("up", "up", "down", "down") + ("left", "right") * 2
    >>> len(code)
    8
    >>> code[3]
    'down'
    >>> code.count("down")
    2
    >>> code.index("left")
    4


However, the methods for manipulating the contents of a list are not available
for tuples because tuples are immutable.

While it is not possible to change which elements are in a tuple, it is
possible to change the value of a mutable element contained within a tuple.

nest = (10, 20, [30, 40]) nest[2].pop()

Tuples are used implicitly in multiple assignment. An assignment of two values
to two names creates a two-element tuple and then unpacks it.

### 2.4.3 Dictionaries

Dictionaries are Python's built-in data type for storing and manipulating
correspondence relationships. A dictionary contains key-value pairs, where
both the keys and values are objects. The purpose of a dictionary is to
provide an abstraction for storing and retrieving values that are indexed not
by consecutive integers, but by descriptive keys.

Strings commonly serve as keys, because strings are our conventional
representation for names of things. This dictionary literal gives the values
of various Roman numerals.


​    

    >>> numerals = {'I': 1.0, 'V': 5, 'X': 10}


Looking up values by their keys uses the element selection operator that we
previously applied to sequences.


​    

    >>> numerals['X']
    10


A dictionary can have at most one value for each key. Adding new key-value
pairs and changing the existing value for a key can both be achieved with
assignment statements.


​    

    >>> numerals['I'] = 1
    >>> numerals['L'] = 50
    >>> numerals
    {'I': 1, 'X': 10, 'L': 50, 'V': 5}


Notice that `'L'` was not added to the end of the output above. Dictionaries
are unordered collections of key-value pairs. When we print a dictionary, the
keys and values are rendered in some order, but as users of the language we
cannot predict what that order will be. The order may change when running a
program multiple times.

Dictionaries can appear in environment diagrams as well.

numerals = {'I': 1, 'V': 5, 'X': 10} numerals['L'] = 50

The dictionary type also supports various methods of iterating over the
contents of the dictionary as a whole. The methods `keys`, `values`, and
`items` all return iterable values.


​    

    >>> sum(numerals.values())
    66


A list of key-value pairs can be converted into a dictionary by calling the
`dict` constructor function.


​    

    >>> dict([(3, 9), (4, 16), (5, 25)])
    {3: 9, 4: 16, 5: 25}


Dictionaries do have some restrictions:

  * A key of a dictionary cannot be or contain a mutable value.
  * There can be at most one value for a given key.

This first restriction is tied to the underlying implementation of
dictionaries in Python. The details of this implementation are not a topic of
this text. Intuitively, consider that the key tells Python where to find that
key-value pair in memory; if the key changes, the location of the pair may be
lost. Tuples are commonly used for keys in dictionaries because lists cannot
be used.

The second restriction is a consequence of the dictionary abstraction, which
is designed to store and retrieve values for keys. We can only retrieve _the_
value for a key if at most one such value exists in the dictionary.

A useful method implemented by dictionaries is `get`, which returns either the
value for a key, if the key is present, or a default value. The arguments to
`get` are the key and the default value.


​    

    >>> numerals.get('A', 0)
    0
    >>> numerals.get('V', 0)
    5


Dictionaries also have a comprehension syntax analogous to those of lists. A
key expression and a value expression are separated by a colon. Evaluating a
dictionary comprehension creates a new dictionary object.


​    

    >>> {x: x*x for x in range(3,6)}
    {3: 9, 4: 16, 5: 25}


### 2.4.4 Local State

Lists and dictionaries have _local state_ : they are changing values that have
some particular contents at any point in the execution of a program. The word
"state" implies an evolving process in which that state may change.

Functions can also have local state. For instance, let us define a function
that models the process of withdrawing money from a bank account. We will
create a function called `withdraw`, which takes as its argument an amount to
be withdrawn. If there is enough money in the account to accommodate the
withdrawal, then `withdraw` will return the balance remaining after the
withdrawal. Otherwise, `withdraw` will return the message `'Insufficient
funds'`. For example, if we begin with $100 in the account, we would like to
obtain the following sequence of return values by calling withdraw:


​    

    >>> withdraw(25)
    75
    >>> withdraw(25)
    50
    >>> withdraw(60)
    'Insufficient funds'
    >>> withdraw(15)
    35


Above, the expression `withdraw(25)`, evaluated twice, yields different
values. Thus, this user-defined function is non-pure. Calling the function not
only returns a value, but also has the side effect of changing the function in
some way, so that the next call with the same argument will return a different
result. This side effect is a result of `withdraw` making a change to a name-
value binding outside of the current frame.

For `withdraw` to make sense, it must be created with an initial account
balance. The function `make_withdraw` is a higher-order function that takes a
starting balance as an argument. The function `withdraw` is its return value.


​    

    >>> withdraw = make_withdraw(100)


An implementation of `make_withdraw` requires a new kind of statement: a
`nonlocal` statement. When we call `make_withdraw`, we bind the name `balance`
to the initial amount. We then define and return a local function, `withdraw`,
which updates and returns the value of `balance` when called.


​    

    >>> def make_withdraw(balance):
            """Return a withdraw function that draws down balance with each call."""
            def withdraw(amount):
                nonlocal balance                 # Declare the name "balance" nonlocal
                if amount > balance:
                    return 'Insufficient funds'
                balance = balance - amount       # Re-bind the existing balance name
                return balance
            return withdraw


The `nonlocal` statement declares that whenever we change the binding of the
name `balance`, the binding is changed in the first frame in which `balance`
is already bound. Recall that without the `nonlocal` statement, an assignment
statement would always bind a name in the first frame of the current
environment. The `nonlocal` statement indicates that the name appears
somewhere in the environment other than the first (local) frame or the last
(global) frame.

The following environment diagrams illustrate the effects of multiple calls to
a function created by `make_withdraw`.

def make_withdraw(balance): def withdraw(amount): nonlocal balance if amount >
balance: return 'Insufficient funds' balance = balance - amount return balance
return withdraw wd = make_withdraw(20) wd(5) wd(3)

The first def statement has the usual effect: it creates a new user-defined
function and binds the name `make_withdraw` to that function in the global
frame. The subsequent call to `make_withdraw` creates and returns a locally
defined function `withdraw`. The name `balance` is bound in the parent frame
of this function. Crucially, there will only be this single binding for the
name `balance` throughout the rest of this example.

Next, we evaluate an expression that calls this function, bound to the name
`wd`, on an amount 5. The body of `withdraw` is executed in a new environment
that extends the environment in which `withdraw` was defined. Tracing the
effect of evaluating `withdraw` illustrates the effect of a `nonlocal`
statement in Python: a name outside of the first local frame can be changed by
an assignment statement.

def make_withdraw(balance): def withdraw(amount): nonlocal balance if amount >
balance: return 'Insufficient funds' balance = balance - amount return balance
return withdraw wd = make_withdraw(20) wd(5) wd(3)

The `nonlocal` statement changes all of the remaining assignment statements in
the definition of `withdraw`. After executing `nonlocal balance`, any
assignment statement with `balance` on the left-hand side of `=` will not bind
`balance` in the first frame of the current environment. Instead, it will find
the first frame in which `balance` was already defined and re-bind the name in
that frame. If `balance` has not previously been bound to a value, then the
`nonlocal` statement will give an error.

By virtue of changing the binding for `balance`, we have changed the
`withdraw` function as well. The next time it is called, the name `balance`
will evaluate to 15 instead of 20. Hence, when we call `withdraw` a second
time, we see that its return value is 12 and not 17\. The change to `balance`
from the first call affects the result of the second call.

def make_withdraw(balance): def withdraw(amount): nonlocal balance if amount >
balance: return 'Insufficient funds' balance = balance - amount return balance
return withdraw wd = make_withdraw(20) wd(5) wd(3)

The second call to `withdraw` does create a second local frame, as usual.
However, both `withdraw` frames have the same parent. That is, they both
extend the environment for `make_withdraw`, which contains the binding for
`balance`. Hence, they share that particular name binding. Calling `withdraw`
has the side effect of altering the environment that will be extended by
future calls to `withdraw`. The `nonlocal` statement allows `withdraw` to
change a name binding in the `make_withdraw` frame.

Ever since we first encountered nested `def` statements, we have observed that
a locally defined function can look up names outside of its local frames. No
`nonlocal` statement is required to _access_ a non-local name. By contrast,
only after a `nonlocal` statement can a function _change_ the binding of names
in these frames.

By introducing `nonlocal` statements, we have created a dual role for
assignment statements. Either they change local bindings, or they change
nonlocal bindings. In fact, assignment statements already had a dual role:
they either created new bindings or re-bound existing names. Assignment can
also change the contents of lists and dictionaries. The many roles of Python
assignment can obscure the effects of executing an assignment statement. It is
up to you as a programmer to document your code clearly so that the effects of
assignment can be understood by others.

**Python Particulars.** This pattern of non-local assignment is a general
feature of programming languages with higher-order functions and lexical
scope. Most other languages do not require a `nonlocal` statement at all.
Instead, non-local assignment is often the default behavior of assignment
statements.

Python also has an unusual restriction regarding the lookup of names: within
the body of a function, all instances of a name must refer to the same frame.
As a result, Python cannot look up the value of a name in a non-local frame,
then bind that same name in the local frame, because the same name would be
accessed in two different frames in the same function. This restriction allows
Python to pre-compute which frame contains each name before executing the body
of a function. When this restriction is violated, a confusing error message
results. To demonstrate, the `make_withdraw` example is repeated below with
the `nonlocal` statement removed.

def make_withdraw(balance): def withdraw(amount): if amount > balance: return
'Insufficient funds' balance = balance - amount return balance return withdraw
wd = make_withdraw(20) wd(5)

This `UnboundLocalError` appears because `balance` is assigned locally in line
5, and so Python assumes that all references to `balance` must appear in the
local frame as well. This error occurs _before_ line 5 is ever executed,
implying that Python has considered line 5 in some way before executing line

3. As we study interpreter design, we will see that pre-computing facts about
   a function body before executing it is quite common. In this case, Python's
   pre-processing restricted the frame in which `balance` could appear, and thus
   prevented the name from being found. Adding a `nonlocal` statement corrects
   this error. The `nonlocal` statement did not exist in Python 2.

### 2.4.5 The Benefits of Non-Local Assignment

Non-local assignment is an important step on our path to viewing a program as
a collection of independent and autonomous _objects_ , which interact with
each other but each manage their own internal state.

In particular, non-local assignment has given us the ability to maintain some
state that is local to a function, but evolves over successive calls to that
function. The `balance` associated with a particular withdraw function is
shared among all calls to that function. However, the binding for balance
associated with an instance of withdraw is inaccessible to the rest of the
program. Only `wd` is associated with the frame for `make_withdraw` in which
it was defined. If `make_withdraw` is called again, then it will create a
separate frame with a separate binding for `balance`.

We can extend our example to illustrate this point. A second call to
`make_withdraw` returns a second `withdraw` function that has a different
parent. We bind this second function to the name `wd2` in the global frame.

def make_withdraw(balance): def withdraw(amount): nonlocal balance if amount >
balance: return 'Insufficient funds' balance = balance - amount return balance
return withdraw wd = make_withdraw(20) wd2 = make_withdraw(7) wd2(6) wd(8)

Now, we see that there are in fact two bindings for the name `balance` in two
different frames, and each `withdraw` function has a different parent. The
name `wd` is bound to a function with a balance of 20, while `wd2` is bound to
a different function with a balance of 7.

Calling `wd2` changes the binding of its non-local `balance` name, but does
not affect the function bound to the name `withdraw`. A future call to `wd` is
unaffected by the changing balance of `wd2`; its balance is still 20.

def make_withdraw(balance): def withdraw(amount): nonlocal balance if amount >
balance: return 'Insufficient funds' balance = balance - amount return balance
return withdraw wd = make_withdraw(20) wd2 = make_withdraw(7) wd2(6) wd(8)

In this way, each instance of `withdraw` maintains its own balance state, but
that state is inaccessible to any other function in the program. Viewing this
situation at a higher level, we have created an abstraction of a bank account
that manages its own internals but behaves in a way that models accounts in
the world: it changes over time based on its own history of withdrawal
requests.

### 2.4.6 The Cost of Non-Local Assignment

Our environment model of computation cleanly extends to explain the effects of
non-local assignment. However, non-local assignment introduces some important
nuances in the way we think about names and values.

Previously, our values did not change; only our names and bindings changed.
When two names `a` and `b` were both bound to the value 4, it did not matter
whether they were bound to the same 4 or different 4's. As far as we could
tell, there was only one 4 object that never changed.

However, functions with state do not behave this way. When two names `wd` and
`wd2` are both bound to a `withdraw` function, it _does_ matter whether they
are bound to the same function or different instances of that function.
Consider the following example, which contrasts the one we just analyzed. In
this case, calling the function named by `wd2` did change the value of the
function named by `wd`, because both names refer to the same function.

def make_withdraw(balance): def withdraw(amount): nonlocal balance if amount >
balance: return 'Insufficient funds' balance = balance - amount return balance
return withdraw wd = make_withdraw(12) wd2 = wd wd2(1) wd(1)

It is not unusual for two names to co-refer to the same value in the world,
and so it is in our programs. But, as values change over time, we must be very
careful to understand the effect of a change on other names that might refer
to those values.

The key to correctly analyzing code with non-local assignment is to remember
that only function calls can introduce new frames. Assignment statements
always change bindings in existing frames. In this case, unless
`make_withdraw` is called twice, there can be only one binding for `balance`.

**Sameness and change.** These subtleties arise because, by introducing non-
pure functions that change the non-local environment, we have changed the
nature of expressions. An expression that contains only pure function calls is
_referentially transparent_ ; its value does not change if we substitute one
of its subexpression with the value of that subexpression.

Re-binding operations violate the conditions of referential transparency
because they do more than return a value; they change the environment. When we
introduce arbitrary re-binding, we encounter a thorny epistemological issue:
what it means for two values to be the same. In our environment model of
computation, two separately defined functions are not the same, because
changes to one may not be reflected in the other.

In general, so long as we never modify data objects, we can regard a compound
data object to be precisely the totality of its pieces. For example, a
rational number is determined by giving its numerator and its denominator. But
this view is no longer valid in the presence of change, where a compound data
object has an "identity" that is something different from the pieces of which
it is composed. A bank account is still "the same" bank account even if we
change the balance by making a withdrawal; conversely, we could have two bank
accounts that happen to have the same balance, but are different objects.

Despite the complications it introduces, non-local assignment is a powerful
tool for creating modular programs. Different parts of a program, which
correspond to different environment frames, can evolve separately throughout
program execution. Moreover, using functions with local state, we are able to
implement mutable data types. In fact, we can implement abstract data types
that are equivalent to the built-in `list` and `dict` types introduced above.

### 2.4.7 Implementing Lists and Dictionaries

The Python language does not give us access to the implementation of lists,
only to the sequence abstraction and mutation methods built into the language.
To understand how a mutable list could be represented using functions with
local state, we will now develop an implementation of a mutable linked list.

We will represent a mutable linked list by a function that has a linked list
as its local state. Lists need to have an identity, like any mutable value. In
particular, we cannot use `None` to represent an empty mutable list, because
two empty lists are not identical values (e.g., appending to one does not
append to the other), but `None is None`. On the other hand, two different
functions that each have `empty` as their local state will suffice to
distinguish two empty lists.

If a mutable linked list is a function, what arguments does it take? The
answer exhibits a general pattern in programming: the function is a dispatch
function and its arguments are first a message, followed by additional
arguments to parameterize that method. This message is a string naming what
the function should do. Dispatch functions are effectively many functions in
one: the message determines the behavior of the function, and the additional
arguments are used in that behavior.

Our mutable list will respond to five different messages: `len`, `getitem`,
`push_first`, `pop_first`, and `str`. The first two implement the behaviors of
the sequence abstraction. The next two add or remove the first element of the
list. The final message returns a string representation of the whole linked
list.


​    

    >>> def mutable_link():
            """Return a functional implementation of a mutable linked list."""
            contents = empty
            def dispatch(message, value=None):
                nonlocal contents
                if message == 'len':
                    return len_link(contents)
                elif message == 'getitem':
                    return getitem_link(contents, value)
                elif message == 'push_first':
                    contents = link(value, contents)
                elif message == 'pop_first':
                    f = first(contents)
                    contents = rest(contents)
                    return f
                elif message == 'str':
                    return join_link(contents, ", ")
            return dispatch


We can also add a convenience function to construct a functionally implemented
linked list from any built-in sequence, simply by adding each element in
reverse order.


​    

    >>> def to_mutable_link(source):
            """Return a functional list with the same contents as source."""
            s = mutable_link()
            for element in reversed(source):
                s('push_first', element)
            return s


In the definition above, the function `reversed` takes and returns an iterable
value; it is another example of a function that processes sequences.

At this point, we can construct a functionally implemented mutable linked
lists. Note that the linked list itself is a function.


​    

    >>> s = to_mutable_link(suits)
    >>> type(s)
    <class 'function'>
    >>> print(s('str'))
    heart, diamond, spade, club


In addition, we can pass messages to the list `s` that change its contents,
for instance removing the first element.


​    

    >>> s('pop_first')
    'heart'
    >>> print(s('str'))
    diamond, spade, club


In principle, the operations `push_first` and `pop_first` suffice to make
arbitrary changes to a list. We can always empty out the list entirely and
then replace its old contents with the desired result.

**Message passing.** Given some time, we could implement the many useful
mutation operations of Python lists, such as `extend` and `insert`. We would
have a choice: we could implement them all as functions, which use the
existing messages `pop_first` and `push_first` to make all changes.
Alternatively, we could add additional `elif` clauses to the body of
`dispatch`, each checking for a message (e.g., `'extend'`) and applying the
appropriate change to `contents` directly.

This second approach, which encapsulates the logic for all operations on a
data value within one function that responds to different messages, is a
discipline called message passing. A program that uses message passing defines
dispatch functions, each of which may have local state, and organizes
computation by passing "messages" as the first argument to those functions.
The messages are strings that correspond to particular behaviors.

**Implementing Dictionaries.** We can also implement a value with similar
behavior to a dictionary. In this case, we use a list of key-value pairs to
store the contents of the dictionary. Each pair is a two-element list.


​    

    >>> def dictionary():
            """Return a functional implementation of a dictionary."""
            records = []
            def getitem(key):
                matches = [r for r in records if r[0] == key]
                if len(matches) == 1:
                    key, value = matches[0]
                    return value
            def setitem(key, value):
                nonlocal records
                non_matches = [r for r in records if r[0] != key]
                records = non_matches + [[key, value]]
            def dispatch(message, key=None, value=None):
                if message == 'getitem':
                    return getitem(key)
                elif message == 'setitem':
                    setitem(key, value)
            return dispatch


Again, we use the message passing method to organize our implementation. We
have supported two messages: `getitem` and `setitem`. To insert a value for a
key, we filter out any existing records with the given key, then add one. In
this way, we are assured that each key appears only once in records. To look
up a value for a key, we filter for the record that matches the given key. We
can now use our implementation to store and retrieve values.


​    

    >>> d = dictionary()
    >>> d('setitem', 3, 9)
    >>> d('setitem', 4, 16)
    >>> d('getitem', 3)
    9
    >>> d('getitem', 4)
    16


This implementation of a dictionary is _not_ optimized for fast record lookup,
because each call must filter through all records. The built-in dictionary
type is considerably more efficient. The way in which it is implemented is
beyond the scope of this text.

### 2.4.8 Dispatch Dictionaries

The dispatch function is a general method for implementing a message passing
interface for abstract data. To implement message dispatch, we have thus far
used conditional statements to compare the message string to a fixed set of
known messages.

The built-in dictionary data type provides a general method for looking up a
value for a key. Instead of using conditionals to implement dispatching, we
can use dictionaries with string keys.

The mutable `account` data type below is implemented as a dictionary. It has a
constructor `account` and selector `check_balance`, as well as functions to
`deposit` or `withdraw` funds. Moreover, the local state of the account is
stored in the dictionary alongside the functions that implement its behavior.

def account(initial_balance): def deposit(amount): dispatch['balance'] +=
amount return dispatch['balance'] def withdraw(amount): if amount >
dispatch['balance']: return 'Insufficient funds' dispatch['balance'] -= amount
return dispatch['balance'] dispatch = {'deposit': deposit, 'withdraw':
withdraw, 'balance': initial_balance} return dispatch def withdraw(account,
amount): return account['withdraw'](amount) def deposit(account, amount):
return account['deposit'](amount) def check_balance(account): return
account['balance'] a = account(20) deposit(a, 5) withdraw(a, 17)
check_balance(a)

The name `dispatch` within the body of the `account` constructor is bound to a
dictionary that contains the messages accepted by an account as keys. The
_balance_ is a number, while the messages _deposit_ and _withdraw_ are bound
to functions. These functions have access to the `dispatch` dictionary, and so
they can read and change the balance. By storing the balance in the dispatch
dictionary rather than in the `account` frame directly, we avoid the need for
`nonlocal` statements in `deposit` and `withdraw`.

The operators `+=` and `-=` are shorthand in Python (and many other languages)
for combined lookup and re-assignment. The last two lines below are
equivalent.


​    

    >>> a = 2
    >>> a = a + 1
    >>> a += 1


### 2.4.9 Propagating Constraints

Mutable data allows us to simulate systems with change, but also allows us to
build new kinds of abstractions. In this extended example, we combine nonlocal
assignment, lists, and dictionaries to build a _constraint-based system_ that
supports computation in multiple directions. Expressing programs as
constraints is a type of _declarative programming_ , in which a programmer
declares the structure of a problem to be solved, but abstracts away the
details of exactly how the solution to the problem is computed.

Computer programs are traditionally organized as one-directional computations,
which perform operations on pre-specified arguments to produce desired
outputs. On the other hand, we often want to model systems in terms of
relations among quantities. For example, we previously considered the ideal
gas law, which relates the pressure (`p`), volume (`v`), quantity (`n`), and
temperature (`t`) of an ideal gas via Boltzmann's constant (`k`):


​    

    p * v = n * k * t


Such an equation is not one-directional. Given any four of the quantities, we
can use this equation to compute the fifth. Yet translating the equation into
a traditional computer language would force us to choose one of the quantities
to be computed in terms of the other four. Thus, a function for computing the
pressure could not be used to compute the temperature, even though the
computations of both quantities arise from the same equation.

In this section, we sketch the design of a general model of linear
relationships. We define primitive constraints that hold between quantities,
such as an `adder(a, b, c)` constraint that enforces the mathematical
relationship `a + b = c`.

We also define a means of combination, so that primitive constraints can be
combined to express more complex relations. In this way, our program resembles
a programming language. We combine constraints by constructing a network in
which constraints are joined by connectors. A connector is an object that
"holds" a value and may participate in one or more constraints.

For example, we know that the relationship between Fahrenheit and Celsius
temperatures is:


​    

    9 * c = 5 * (f - 32)


This equation is a complex constraint between `c` and `f`. Such a constraint
can be thought of as a network consisting of primitive `adder`, `multiplier`,
and `constant` constraints.

![](http://www.composingprograms.com/img/constraints.png)

In this figure, we see on the left a multiplier box with three terminals,
labeled `a`, `b`, and `c`. These connect the multiplier to the rest of the
network as follows: The `a` terminal is linked to a connector `celsius`, which
will hold the Celsius temperature. The `b` terminal is linked to a connector
`w`, which is also linked to a constant box that holds 9. The `c` terminal,
which the multiplier box constrains to be the product of `a` and `b`, is
linked to the `c` terminal of another multiplier box, whose `b` is connected
to a constant 5 and whose `a` is connected to one of the terms in the sum
constraint.

Computation by such a network proceeds as follows: When a connector is given a
value (by the user or by a constraint box to which it is linked), it awakens
all of its associated constraints (except for the constraint that just
awakened it) to inform them that it has a value. Each awakened constraint box
then polls its connectors to see if there is enough information to determine a
value for a connector. If so, the box sets that connector, which then awakens
all of its associated constraints, and so on. For instance, in conversion
between Celsius and Fahrenheit, `w`, `x`, and `y` are immediately set by the
constant boxes to 9, 5, and 32, respectively. The connectors awaken the
multipliers and the adder, which determine that there is not enough
information to proceed. If the user (or some other part of the network) sets
the `celsius` connector to a value (say 25), the leftmost multiplier will be
awakened, and it will set `u` to `25 * 9 = 225`. Then `u` awakens the second
multiplier, which sets `v` to 45, and `v` awakens the adder, which sets the
`fahrenheit` connector to 77.

**Using the Constraint System.** To use the constraint system to carry out the
temperature computation outlined above, we first create two named connectors,
`celsius` and `fahrenheit`, by calling the `connector` constructor.


​    

    >>> celsius = connector('Celsius')
    >>> fahrenheit = connector('Fahrenheit')


Then, we link these connectors into a network that mirrors the figure above.
The function `converter` assembles the various connectors and constraints in
the network.


​    

    >>> def converter(c, f):
            """Connect c to f with constraints to convert from Celsius to Fahrenheit."""
            u, v, w, x, y = [connector() for _ in range(5)]
            multiplier(c, w, u)
            multiplier(v, x, u)
            adder(v, y, f)
            constant(w, 9)
            constant(x, 5)
            constant(y, 32)


​    
​    

    >>> converter(celsius, fahrenheit)


We will use a message passing system to coordinate constraints and connectors.
Constraints are dictionaries that do not hold local states themselves. Their
responses to messages are non-pure functions that change the connectors that
they constrain.

Connectors are dictionaries that hold a current value and respond to messages
that manipulate that value. Constraints will not change the value of
connectors directly, but instead will do so by sending messages, so that the
connector can notify other constraints in response to the change. In this way,
a connector represents a number, but also encapsulates connector behavior.

One message we can send to a connector is to set its value. Here, we (the
`'user'`) set the value of `celsius` to 25.


​    

    >>> celsius['set_val']('user', 25)
    Celsius = 25
    Fahrenheit = 77.0


Not only does the value of `celsius` change to 25, but its value propagates
through the network, and so the value of `fahrenheit` is changed as well.
These changes are printed because we named these two connectors when we
constructed them.

Now we can try to set `fahrenheit` to a new value, say 212.


​    

    >>> fahrenheit['set_val']('user', 212)
    Contradiction detected: 77.0 vs 212


The connector complains that it has sensed a contradiction: Its value is 77.0,
and someone is trying to set it to 212. If we really want to reuse the network
with new values, we can tell `celsius` to forget its old value:


​    

    >>> celsius['forget']('user')
    Celsius is forgotten
    Fahrenheit is forgotten


The connector `celsius` finds that the `user`, who set its value originally,
is now retracting that value, so `celsius` agrees to lose its value, and it
informs the rest of the network of this fact. This information eventually
propagates to `fahrenheit`, which now finds that it has no reason for
continuing to believe that its own value is 77. Thus, it also gives up its
value.

Now that `fahrenheit` has no value, we are free to set it to 212:


​    

    >>> fahrenheit['set_val']('user', 212)
    Fahrenheit = 212
    Celsius = 100.0


This new value, when propagated through the network, forces `celsius` to have
a value of 100. We have used the very same network to compute `celsius` given
`fahrenheit` and to compute `fahrenheit` given `celsius`. This non-
directionality of computation is the distinguishing feature of constraint-
based systems.

**Implementing the Constraint System.** As we have seen, connectors are
dictionaries that map message names to function and data values. We will
implement connectors that respond to the following messages:

  * `connector['set_val'](source, value)` indicates that the `source` is requesting the connector to set its current value to `value`.
  * `connector['has_val']()` returns whether the connector already has a value.
  * `connector['val']` is the current value of the connector.
  * `connector['forget'](source)` tells the connector that the `source` is requesting it to forget its value.
  * `connector['connect'](source)` tells the connector to participate in a new constraint, the `source`.

Constraints are also dictionaries, which receive information from connectors
by means of two messages:

  * `constraint['new_val']()` indicates that some connector that is connected to the constraint has a new value.
  * `constraint['forget']()` indicates that some connector that is connected to the constraint has forgotten its value.

When constraints receive these messages, they propagate them appropriately to
other connectors.

The `adder` function constructs an adder constraint over three connectors,
where the first two must add to the third: `a + b = c`. To support
multidirectional constraint propagation, the adder must also specify that it
subtracts `a` from `c` to get `b` and likewise subtracts `b` from `c` to get
`a`.


​    

    >>> from operator import add, sub
    >>> def adder(a, b, c):
            """The constraint that a + b = c."""
            return make_ternary_constraint(a, b, c, add, sub, sub)


We would like to implement a generic ternary (three-way) constraint, which
uses the three connectors and three functions from `adder` to create a
constraint that accepts `new_val` and `forget` messages. The response to
messages are local functions, which are placed in a dictionary called
`constraint`.


​    

    >>> def make_ternary_constraint(a, b, c, ab, ca, cb):
            """The constraint that ab(a,b)=c and ca(c,a)=b and cb(c,b) = a."""
            def new_value():
                av, bv, cv = [connector['has_val']() for connector in (a, b, c)]
                if av and bv:
                    c['set_val'](constraint, ab(a['val'], b['val']))
                elif av and cv:
                    b['set_val'](constraint, ca(c['val'], a['val']))
                elif bv and cv:
                    a['set_val'](constraint, cb(c['val'], b['val']))
            def forget_value():
                for connector in (a, b, c):
                    connector['forget'](constraint)
            constraint = {'new_val': new_value, 'forget': forget_value}
            for connector in (a, b, c):
                connector['connect'](constraint)
            return constraint


The dictionary called `constraint` is a dispatch dictionary, but also the
constraint object itself. It responds to the two messages that constraints
receive, but is also passed as the `source` argument in calls to its
connectors.

The constraint's local function `new_value` is called whenever the constraint
is informed that one of its connectors has a value. This function first checks
to see if both `a` and `b` have values. If so, it tells `c` to set its value
to the return value of function `ab`, which is `add` in the case of an
`adder`. The constraint passes _itself_ (`constraint`) as the `source`
argument of the connector, which is the adder object. If `a` and `b` do not
both have values, then the constraint checks `a` and `c`, and so on.

If the constraint is informed that one of its connectors has forgotten its
value, it requests that all of its connectors now forget their values. (Only
those values that were set by this constraint are actually lost.)

A `multiplier` is very similar to an `adder`.


​    

    >>> from operator import mul, truediv
    >>> def multiplier(a, b, c):
            """The constraint that a * b = c."""
            return make_ternary_constraint(a, b, c, mul, truediv, truediv)


A constant is a constraint as well, but one that is never sent any messages,
because it involves only a single connector that it sets on construction.


​    

    >>> def constant(connector, value):
            """The constraint that connector = value."""
            constraint = {}
            connector['set_val'](constraint, value)
            return constraint


These three constraints are sufficient to implement our temperature conversion
network.

**Representing connectors.** A connector is represented as a dictionary that
contains a value, but also has response functions with local state. The
connector must track the `informant` that gave it its current value, and a
list of `constraints` in which it participates.

The constructor `connector` has local functions for setting and forgetting
values, which are the responses to messages from constraints.


​    

    >>> def connector(name=None):
            """A connector between constraints."""
            informant = None
            constraints = []
            def set_value(source, value):
                nonlocal informant
                val = connector['val']
                if val is None:
                    informant, connector['val'] = source, value
                    if name is not None:
                        print(name, '=', value)
                    inform_all_except(source, 'new_val', constraints)
                else:
                    if val != value:
                        print('Contradiction detected:', val, 'vs', value)
            def forget_value(source):
                nonlocal informant
                if informant == source:
                    informant, connector['val'] = None, None
                    if name is not None:
                        print(name, 'is forgotten')
                    inform_all_except(source, 'forget', constraints)
            connector = {'val': None,
                         'set_val': set_value,
                         'forget': forget_value,
                         'has_val': lambda: connector['val'] is not None,
                         'connect': lambda source: constraints.append(source)}
            return connector


A connector is again a dispatch dictionary for the five messages used by
constraints to communicate with connectors. Four responses are functions, and
the final response is the value itself.

The local function `set_value` is called when there is a request to set the
connector's value. If the connector does not currently have a value, it will
set its value and remember as `informant` the source constraint that requested
the value to be set. Then the connector will notify all of its participating
constraints except the constraint that requested the value to be set. This is
accomplished using the following iterative function.


​    

    >>> def inform_all_except(source, message, constraints):
            """Inform all constraints of the message, except source."""
            for c in constraints:
                if c != source:
                    c[message]()


If a connector is asked to forget its value, it calls the local function
`forget-value`, which first checks to make sure that the request is coming
from the same constraint that set the value originally. If so, the connector
informs its associated constraints about the loss of the value.

The response to the message `has_val` indicates whether the connector has a
value. The response to the message `connect` adds the source constraint to the
list of constraints.

The constraint program we have designed introduces many ideas that will appear
again in object-oriented programming. Constraints and connectors are both
abstractions that are manipulated through messages. When the value of a
connector is changed, it is changed via a message that not only changes the
value, but validates it (checking the source) and propagates its effects
(informing other constraints). In fact, we will use a similar architecture of
dictionaries with string-valued keys and functional values to implement an
object-oriented system later in this chapter.

_Continue_ : [ 2.5 Object-Oriented Programming ](../pages/25-object-oriented-
programming.html)

## 2.5 Object-Oriented Programming

Object-oriented programming (OOP) is a method for organizing programs that
brings together many of the ideas introduced in this chapter. Like the
functions in data abstraction, classes create abstraction barriers between the
use and implementation of data. Like dispatch dictionaries, objects respond to
behavioral requests. Like mutable data structures, objects have local state
that is not directly accessible from the global environment. The Python object
system provides convenient syntax to promote the use of these techniques for
organizing programs. Much of this syntax is shared among other object-oriented
programming languages.

The object system offers more than just convenience. It enables a new metaphor
for designing programs in which several independent agents interact within the
computer. Each object bundles together local state and behavior in a way that
abstracts the complexity of both. Objects communicate with each other, and
useful results are computed as a consequence of their interaction. Not only do
objects pass messages, they also share behavior among other objects of the
same type and inherit characteristics from related types.

The paradigm of object-oriented programming has its own vocabulary that
supports the object metaphor. We have seen that an object is a data value that
has methods and attributes, accessible via dot notation. Every object also has
a type, called its _class_. To create new types of data, we implement new
classes.

### 2.5.1 Objects and Classes

A class serves as a template for all objects whose type is that class. Every
object is an instance of some particular class. The objects we have used so
far all have built-in classes, but new user-defined classes can be created as
well. A class definition specifies the attributes and methods shared among
objects of that class. We will introduce the class statement by revisiting the
example of a bank account.

When introducing local state, we saw that bank accounts are naturally modeled
as mutable values that have a `balance`. A bank account object should have a
`withdraw` method that updates the account balance and returns the requested
amount, if it is available. To complete the abstraction: a bank account should
be able to return its current `balance`, return the name of the account
`holder`, and an amount for `deposit`.

An `Account` class allows us to create multiple instances of bank accounts.
The act of creating a new object instance is known as _instantiating_ the
class. The syntax in Python for instantiating a class is identical to the
syntax of calling a function. In this case, we call `Account` with the
argument `'Kirk'`, the account holder's name.


​    

    >>> a = Account('Kirk')


An _attribute_ of an object is a name-value pair associated with the object,
which is accessible via dot notation. The attributes specific to a particular
object, as opposed to all objects of a class, are called _instance
attributes_. Each `Account` has its own balance and account holder name, which
are examples of instance attributes. In the broader programming community,
instance attributes may also be called _fields_ , _properties_ , or _instance
variables_.


​    

    >>> a.holder
    'Kirk'
    >>> a.balance
    0


Functions that operate on the object or perform object-specific computations
are called methods. The return values and side effects of a method can depend
upon and change other attributes of the object. For example, `deposit` is a
method of our `Account` object `a`. It takes one argument, the amount to
deposit, changes the `balance` attribute of the object, and returns the
resulting balance.


​    

    >>> a.deposit(15)
    15


We say that methods are _invoked_ on a particular object. As a result of
invoking the `withdraw` method, either the withdrawal is approved and the
amount is deducted, or the request is declined and the method returns an error
message.


​    

    >>> a.withdraw(10)  # The withdraw method returns the balance after withdrawal
    5
    >>> a.balance       # The balance attribute has changed
    5
    >>> a.withdraw(10)
    'Insufficient funds'


As illustrated above, the behavior of a method can depend upon the changing
attributes of the object. Two calls to `withdraw` with the same argument
return different results.

### 2.5.2 Defining Classes

User-defined classes are created by `class` statements, which consist of a
single clause. A class statement defines the class name, then includes a suite
of statements to define the attributes of the class:


​    

    class <name>:
        <suite>


When a class statement is executed, a new class is created and bound to
`<name>` in the first frame of the current environment. The suite is then
executed. Any names bound within the `<suite>` of a `class` statement, through
`def` or assignment statements, create or modify attributes of the class.

Classes are typically organized around manipulating instance attributes, which
are the name-value pairs associated with each instance of that class. The
class specifies the instance attributes of its objects by defining a method
for initializing new objects. For example, part of initializing an object of
the `Account` class is to assign it a starting balance of 0.

The `<suite>` of a `class` statement contains `def` statements that define new
methods for objects of that class. The method that initializes objects has a
special name in Python, `__init__` (two underscores on each side of the word
"init"), and is called the _constructor_ for the class.


​    

    >>> class Account:
            def __init__(self, account_holder):
                self.balance = 0
                self.holder = account_holder


The `__init__` method for `Account` has two formal parameters. The first one,
`self`, is bound to the newly created `Account` object. The second parameter,
`account_holder`, is bound to the argument passed to the class when it is
called to be instantiated.

The constructor binds the instance attribute name `balance` to 0. It also
binds the attribute name `holder` to the value of the name `account_holder`.
The formal parameter `account_holder` is a local name in the `__init__`
method. On the other hand, the name `holder` that is bound via the final
assignment statement persists, because it is stored as an attribute of `self`
using dot notation.

Having defined the `Account` class, we can instantiate it.


​    

    >>> a = Account('Kirk')


This "call" to the `Account` class creates a new object that is an instance of
`Account`, then calls the constructor function `__init__` with two arguments:
the newly created object and the string `'Kirk'`. By convention, we use the
parameter name `self` for the first argument of a constructor, because it is
bound to the object being instantiated. This convention is adopted in
virtually all Python code.

Now, we can access the object's `balance` and `holder` using dot notation.


​    

    >>> a.balance
    0
    >>> a.holder
    'Kirk'


**Identity.** Each new account instance has its own balance attribute, the
value of which is independent of other objects of the same class.


​    

    >>> b = Account('Spock')
    >>> b.balance = 200
    >>> [acc.balance for acc in (a, b)]
    [0, 200]


To enforce this separation, every object that is an instance of a user-defined
class has a unique identity. Object identity is compared using the `is` and
`is not` operators.


​    

    >>> a is a
    True
    >>> a is not b
    True


Despite being constructed from identical calls, the objects bound to `a` and
`b` are not the same. As usual, binding an object to a new name using
assignment does not create a new object.


​    

    >>> c = a
    >>> c is a
    True


New objects that have user-defined classes are only created when a class (such
as `Account`) is instantiated with call expression syntax.

**Methods.** Object methods are also defined by a `def` statement in the suite
of a `class` statement. Below, `deposit` and `withdraw` are both defined as
methods on objects of the `Account` class.


​    

    >>> class Account:
            def __init__(self, account_holder):
                self.balance = 0
                self.holder = account_holder
            def deposit(self, amount):
                self.balance = self.balance + amount
                return self.balance
            def withdraw(self, amount):
                if amount > self.balance:
                    return 'Insufficient funds'
                self.balance = self.balance - amount
                return self.balance


While method definitions do not differ from function definitions in how they
are declared, method definitions do have a different effect when executed. The
function value that is created by a `def` statement within a `class` statement
is bound to the declared name, but bound locally within the class as an
attribute. That value is invoked as a method using dot notation from an
instance of the class.

Each method definition again includes a special first parameter `self`, which
is bound to the object on which the method is invoked. For example, let us say
that `deposit` is invoked on a particular `Account` object and passed a single
argument value: the amount deposited. The object itself is bound to `self`,
while the argument is bound to `amount`. All invoked methods have access to
the object via the `self` parameter, and so they can all access and manipulate
the object's state.

To invoke these methods, we again use dot notation, as illustrated below.


​    

    >>> spock_account = Account('Spock')
    >>> spock_account.deposit(100)
    100
    >>> spock_account.withdraw(90)
    10
    >>> spock_account.withdraw(90)
    'Insufficient funds'
    >>> spock_account.holder
    'Spock'


When a method is invoked via dot notation, the object itself (bound to
`spock_account`, in this case) plays a dual role. First, it determines what
the name `withdraw` means; `withdraw` is not a name in the environment, but
instead a name that is local to the `Account` class. Second, it is bound to
the first parameter `self` when the `withdraw` method is invoked.

### 2.5.3 Message Passing and Dot Expressions

Methods, which are defined in classes, and instance attributes, which are
typically assigned in constructors, are the fundamental elements of object-
oriented programming. These two concepts replicate much of the behavior of a
dispatch dictionary in a message passing implementation of a data value.
Objects take messages using dot notation, but instead of those messages being
arbitrary string-valued keys, they are names local to a class. Objects also
have named local state values (the instance attributes), but that state can be
accessed and manipulated using dot notation, without having to employ
`nonlocal` statements in the implementation.

The central idea in message passing was that data values should have behavior
by responding to messages that are relevant to the abstract type they
represent. Dot notation is a syntactic feature of Python that formalizes the
message passing metaphor. The advantage of using a language with a built-in
object system is that message passing can interact seamlessly with other
language features, such as assignment statements. We do not require different
messages to "get" or "set" the value associated with a local attribute name;
the language syntax allows us to use the message name directly.

**Dot expressions.** The code fragment `spock_account.deposit` is called a
_dot expression_. A dot expression consists of an expression, a dot, and a
name:


​    

    <expression> . <name>


The `<expression>` can be any valid Python expression, but the `<name>` must
be a simple name (not an expression that evaluates to a name). A dot
expression evaluates to the value of the attribute with the given `<name>`,
for the object that is the value of the `<expression>`.

The built-in function `getattr` also returns an attribute for an object by
name. It is the function equivalent of dot notation. Using `getattr`, we can
look up an attribute using a string, just as we did with a dispatch
dictionary.


​    

    >>> getattr(spock_account, 'balance')
    10


We can also test whether an object has a named attribute with `hasattr`.


​    

    >>> hasattr(spock_account, 'deposit')
    True


The attributes of an object include all of its instance attributes, along with
all of the attributes (including methods) defined in its class. Methods are
attributes of the class that require special handling.

**Methods and functions.** When a method is invoked on an object, that object
is implicitly passed as the first argument to the method. That is, the object
that is the value of the `<expression>` to the left of the dot is passed
automatically as the first argument to the method named on the right side of
the dot expression. As a result, the object is bound to the parameter `self`.

To achieve automatic `self` binding, Python distinguishes between _functions_
, which we have been creating since the beginning of the text, and _bound
methods_ , which couple together a function and the object on which that
method will be invoked. A bound method value is already associated with its
first argument, the instance on which it was invoked, which will be named
`self` when the method is called.

We can see the difference in the interactive interpreter by calling `type` on
the returned values of dot expressions. As an attribute of a class, a method
is just a function, but as an attribute of an instance, it is a bound method:


​    

    >>> type(Account.deposit)
    <class 'function'>
    >>> type(spock_account.deposit)
    <class 'method'>


These two results differ only in the fact that the first is a standard two-
argument function with parameters `self` and `amount`. The second is a one-
argument method, where the name `self` will be bound to the object named
`spock_account` automatically when the method is called, while the parameter
`amount` will be bound to the argument passed to the method. Both of these
values, whether function values or bound method values, are associated with
the same `deposit` function body.

We can call `deposit` in two ways: as a function and as a bound method. In the
former case, we must supply an argument for the `self` parameter explicitly.
In the latter case, the `self` parameter is bound automatically.


​    

    >>> Account.deposit(spock_account, 1001)  # The deposit function takes 2 arguments
    1011
    >>> spock_account.deposit(1000)           # The deposit method takes 1 argument
    2011


The function `getattr` behaves exactly like dot notation: if its first
argument is an object but the name is a method defined in the class, then
`getattr` returns a bound method value. On the other hand, if the first
argument is a class, then `getattr` returns the attribute value directly,
which is a plain function.

**Naming Conventions.** Class names are conventionally written using the
CapWords convention (also called CamelCase because the capital letters in the
middle of a name look like humps). Method names follow the standard convention
of naming functions using lowercased words separated by underscores.

In some cases, there are instance variables and methods that are related to
the maintenance and consistency of an object that we don't want users of the
object to see or use. They are not part of the abstraction defined by a class,
but instead part of the implementation. Python's convention dictates that if
an attribute name starts with an underscore, it should only be accessed within
methods of the class itself, rather than by users of the class.

### 2.5.4 Class Attributes

Some attribute values are shared across all objects of a given class. Such
attributes are associated with the class itself, rather than any individual
instance of the class. For instance, let us say that a bank pays interest on
the balance of accounts at a fixed interest rate. That interest rate may
change, but it is a single value shared across all accounts.

Class attributes are created by assignment statements in the suite of a
`class` statement, outside of any method definition. In the broader developer
community, class attributes may also be called class variables or static
variables. The following class statement creates a class attribute for
`Account` with the name `interest`.


​    

    >>> class Account:
            interest = 0.02            # A class attribute
            def __init__(self, account_holder):
                self.balance = 0
                self.holder = account_holder
            # Additional methods would be defined here


This attribute can still be accessed from any instance of the class.


​    

    >>> spock_account = Account('Spock')
    >>> kirk_account = Account('Kirk')
    >>> spock_account.interest
    0.02
    >>> kirk_account.interest
    0.02


However, a single assignment statement to a class attribute changes the value
of the attribute for all instances of the class.


​    

    >>> Account.interest = 0.04
    >>> spock_account.interest
    0.04
    >>> kirk_account.interest
    0.04


**Attribute names.** We have introduced enough complexity into our object
system that we have to specify how names are resolved to particular
attributes. After all, we could easily have a class attribute and an instance
attribute with the same name.

As we have seen, a dot expression consists of an expression, a dot, and a
name:


​    

    <expression> . <name>


To evaluate a dot expression:

    1. Evaluate the `<expression>` to the left of the dot, which yields the _object_ of the dot expression.
    2. `<name>` is matched against the instance attributes of that object; if an attribute with that name exists, its value is returned.
    3. If `<name>` does not appear among instance attributes, then `<name>` is looked up in the class, which yields a class attribute value.
    4. That value is returned unless it is a function, in which case a bound method is returned instead.

In this evaluation procedure, instance attributes are found before class
attributes, just as local names have priority over global in an environment.
Methods defined within the class are combined with the object of the dot
expression to form a bound method during the fourth step of this evaluation
procedure. The procedure for looking up a name in a class has additional
nuances that will arise shortly, once we introduce class inheritance.

**Attribute assignment.** All assignment statements that contain a dot
expression on their left-hand side affect attributes for the object of that
dot expression. If the object is an instance, then assignment sets an instance
attribute. If the object is a class, then assignment sets a class attribute.
As a consequence of this rule, assignment to an attribute of an object cannot
affect the attributes of its class. The examples below illustrate this
distinction.

If we assign to the named attribute `interest` of an account instance, we
create a new instance attribute that has the same name as the existing class
attribute.


​    

    >>> kirk_account.interest = 0.08


and that attribute value will be returned from a dot expression.


​    

    >>> kirk_account.interest
    0.08


However, the class attribute `interest` still retains its original value,
which is returned for all other accounts.


​    

    >>> spock_account.interest
    0.04


Changes to the class attribute `interest` will affect `spock_account`, but the
instance attribute for `kirk_account` will be unaffected.


​    

    >>> Account.interest = 0.05  # changing the class attribute
    >>> spock_account.interest     # changes instances without like-named instance attributes
    0.05
    >>> kirk_account.interest     # but the existing instance attribute is unaffected
    0.08


### 2.5.5 Inheritance

When working in the object-oriented programming paradigm, we often find that
different types are related. In particular, we find that similar classes
differ in their amount of specialization. Two classes may have similar
attributes, but one represents a special case of the other.

For example, we may want to implement a checking account, which is different
from a standard account. A checking account charges an extra $1 for each
withdrawal and has a lower interest rate. Here, we demonstrate the desired
behavior.


​    

    >>> ch = CheckingAccount('Spock')
    >>> ch.interest     # Lower interest rate for checking accounts
    0.01
    >>> ch.deposit(20)  # Deposits are the same
    20
    >>> ch.withdraw(5)  # withdrawals decrease balance by an extra charge
    14


A `CheckingAccount` is a specialization of an `Account`. In OOP terminology,
the generic account will serve as the base class of `CheckingAccount`, while
`CheckingAccount` will be a subclass of `Account`. (The terms _parent class_
and _superclass_ are also used for the base class, while _child class_ is also
used for the subclass.)

A subclass _inherits_ the attributes of its base class, but may _override_
certain attributes, including certain methods. With inheritance, we only
specify what is different between the subclass and the base class. Anything
that we leave unspecified in the subclass is automatically assumed to behave
just as it would for the base class.

Inheritance also has a role in our object metaphor, in addition to being a
useful organizational feature. Inheritance is meant to represent _is-a_
relationships between classes, which contrast with _has-a_ relationships. A
checking account _is-a_ specific type of account, so having a
`CheckingAccount` inherit from `Account` is an appropriate use of inheritance.
On the other hand, a bank _has-a_ list of bank accounts that it manages, so
neither should inherit from the other. Instead, a list of account objects
would be naturally expressed as an instance attribute of a bank object.

### 2.5.6 Using Inheritance

First, we give a full implementation of the `Account` class, which includes
docstrings for the class and its methods.


​    

    >>> class Account:
            """A bank account that has a non-negative balance."""
            interest = 0.02
            def __init__(self, account_holder):
                self.balance = 0
                self.holder = account_holder
            def deposit(self, amount):
                """Increase the account balance by amount and return the new balance."""
                self.balance = self.balance + amount
                return self.balance
            def withdraw(self, amount):
                """Decrease the account balance by amount and return the new balance."""
                if amount > self.balance:
                    return 'Insufficient funds'
                self.balance = self.balance - amount
                return self.balance


A full implementation of `CheckingAccount` appears below. We specify
inheritance by placing an expression that evaluates to the base class in
parentheses after the class name.


​    

    >>> class CheckingAccount(Account):
            """A bank account that charges for withdrawals."""
            withdraw_charge = 1
            interest = 0.01
            def withdraw(self, amount):
                return Account.withdraw(self, amount + self.withdraw_charge)


Here, we introduce a class attribute `withdraw_charge` that is specific to the
`CheckingAccount` class. We assign a lower value to the `interest` attribute.
We also define a new `withdraw` method to override the behavior defined in the
`Account` class. With no further statements in the class suite, all other
behavior is inherited from the base class `Account`.


​    

    >>> checking = CheckingAccount('Sam')
    >>> checking.deposit(10)
    10
    >>> checking.withdraw(5)
    4
    >>> checking.interest
    0.01


The expression `checking.deposit` evaluates to a bound method for making
deposits, which was defined in the `Account` class. When Python resolves a
name in a dot expression that is not an attribute of the instance, it looks up
the name in the class. In fact, the act of "looking up" a name in a class
tries to find that name in every base class in the inheritance chain for the
original object's class. We can define this procedure recursively. To look up
a name in a class.

    1. If it names an attribute in the class, return the attribute value.
    2. Otherwise, look up the name in the base class, if there is one.

In the case of `deposit`, Python would have looked for the name first on the
instance, and then in the `CheckingAccount` class. Finally, it would look in
the `Account` class, where `deposit` is defined. According to our evaluation
rule for dot expressions, since `deposit` is a function looked up in the class
for the `checking` instance, the dot expression evaluates to a bound method
value. That method is invoked with the argument 10, which calls the deposit
method with `self` bound to the `checking` object and `amount` bound to 10.

The class of an object stays constant throughout. Even though the `deposit`
method was found in the `Account` class, `deposit` is called with `self` bound
to an instance of `CheckingAccount`, not of `Account`.

**Calling ancestors.** Attributes that have been overridden are still
accessible via class objects. For instance, we implemented the `withdraw`
method of `CheckingAccount` by calling the `withdraw` method of `Account` with
an argument that included the `withdraw_charge`.

Notice that we called `self.withdraw_charge` rather than the equivalent
`CheckingAccount.withdraw_charge`. The benefit of the former over the latter
is that a class that inherits from `CheckingAccount` might override the
withdrawal charge. If that is the case, we would like our implementation of
`withdraw` to find that new value instead of the old one.

**Interfaces.** It is extremely common in object-oriented programs that
different types of objects will share the same attribute names. An _object
interface_ is a collection of attributes and conditions on those attributes.
For example, all accounts must have `deposit` and `withdraw` methods that take
numerical arguments, as well as a `balance` attribute. The classes `Account`
and `CheckingAccount` both implement this interface. Inheritance specifically
promotes name sharing in this way. In some programming languages such as Java,
interface implementations must be explicitly declared. In others such as
Python, Ruby, and Go, any object with the appropriate names implements an
interface.

The parts of your program that use objects (rather than implementing them) are
most robust to future changes if they do not make assumptions about object
types, but instead only about their attribute names. That is, they use the
object abstraction, rather than assuming anything about its implementation.

For example, let us say that we run a lottery, and we wish to deposit $5 into
each of a list of accounts. The following implementation does not assume
anything about the types of those accounts, and therefore works equally well
with any type of object that has a `deposit` method:


​    

    >>> def deposit_all(winners, amount=5):
            for account in winners:
                account.deposit(amount)


The function `deposit_all` above assumes only that each `account` satisfies
the account object abstraction, and so it will work with any other account
classes that also implement this interface. Assuming a particular class of
account would violate the abstraction barrier of the account object
abstraction. For example, the following implementation will not necessarily
work with new kinds of accounts:


​    

    >>> def deposit_all(winners, amount=5):
            for account in winners:
                Account.deposit(account, amount)


We will address this topic in more detail later in the chapter.

### 2.5.7 Multiple Inheritance

Python supports the concept of a subclass inheriting attributes from multiple
base classes, a language feature called _multiple inheritance_.

Suppose that we have a `SavingsAccount` that inherits from `Account`, but
charges customers a small fee every time they make a deposit.


​    

    >>> class SavingsAccount(Account):
            deposit_charge = 2
            def deposit(self, amount):
                return Account.deposit(self, amount - self.deposit_charge)


Then, a clever executive conceives of an `AsSeenOnTVAccount` account with the
best features of both `CheckingAccount` and `SavingsAccount`: withdrawal fees,
deposit fees, and a low interest rate. It's both a checking and a savings
account in one! "If we build it," the executive reasons, "someone will sign up
and pay all those fees. We'll even give them a dollar."


​    

    >>> class AsSeenOnTVAccount(CheckingAccount, SavingsAccount):
            def __init__(self, account_holder):
                self.holder = account_holder
                self.balance = 1           # A free dollar!


In fact, this implementation is complete. Both withdrawal and deposits will
generate fees, using the function definitions in `CheckingAccount` and
`SavingsAccount` respectively.


​    

    >>> such_a_deal = AsSeenOnTVAccount("John")
    >>> such_a_deal.balance
    1
    >>> such_a_deal.deposit(20)            # $2 fee from SavingsAccount.deposit
    19
    >>> such_a_deal.withdraw(5)            # $1 fee from CheckingAccount.withdraw
    13


Non-ambiguous references are resolved correctly as expected:


​    

    >>> such_a_deal.deposit_charge
    2
    >>> such_a_deal.withdraw_charge
    1


But what about when the reference is ambiguous, such as the reference to the
`withdraw` method that is defined in both `Account` and `CheckingAccount`? The
figure below depicts an _inheritance graph_ for the `AsSeenOnTVAccount` class.
Each arrow points from a subclass to a base class.

![](http://www.composingprograms.com/img/multiple_inheritance.png)

For a simple "diamond" shape like this, Python resolves names from left to
right, then upwards. In this example, Python checks for an attribute name in
the following classes, in order, until an attribute with that name is found:


​    

    AsSeenOnTVAccount, CheckingAccount, SavingsAccount, Account, object


There is no correct solution to the inheritance ordering problem, as there are
cases in which we might prefer to give precedence to certain inherited classes
over others. However, any programming language that supports multiple
inheritance must select some ordering in a consistent way, so that users of
the language can predict the behavior of their programs.

**Further reading.** Python resolves this name using a recursive algorithm
called the C3 Method Resolution Ordering. The method resolution order of any
class can be queried using the `mro` method on all classes.


​    

    >>> [c.__name__ for c in AsSeenOnTVAccount.mro()]
    ['AsSeenOnTVAccount', 'CheckingAccount', 'SavingsAccount', 'Account', 'object']


The precise algorithm for finding method resolution orderings is not a topic
for this text, but is [described by Python's primary author](http://python-
history.blogspot.com/2010/06/method-resolution-order.html) with a reference to
the original paper.

### 2.5.8 The Role of Objects

The Python object system is designed to make data abstraction and message
passing both convenient and flexible. The specialized syntax of classes,
methods, inheritance, and dot expressions all enable us to formalize the
object metaphor in our programs, which improves our ability to organize large
programs.

In particular, we would like our object system to promote a _separation of
concerns_ among the different aspects of the program. Each object in a program
encapsulates and manages some part of the program's state, and each class
statement defines the functions that implement some part of the program's
overall logic. Abstraction barriers enforce the boundaries between different
aspects of a large program.

Object-oriented programming is particularly well-suited to programs that model
systems that have separate but interacting parts. For instance, different
users interact in a social network, different characters interact in a game,
and different shapes interact in a physical simulation. When representing such
systems, the objects in a program often map naturally onto objects in the
system being modeled, and classes represent their types and relationships.

On the other hand, classes may not provide the best mechanism for implementing
certain abstractions. Functional abstractions provide a more natural metaphor
for representing relationships between inputs and outputs. One should not feel
compelled to fit every bit of logic in a program within a class, especially
when defining independent functions for manipulating data is more natural.
Functions can also enforce a separation of concerns.

Multi-paradigm languages such as Python allow programmers to match
organizational paradigms to appropriate problems. Learning to identify when to
introduce a new class, as opposed to a new function, in order to simplify or
modularize a program, is an important design skill in software engineering
that deserves careful attention.

_Continue_ : [ 2.6 Implementing Classes and Objects

](../pages/26-implementing-classes-and-objects.html)

## 2.6 Implementing Classes and Objects

When working in the object-oriented programming paradigm, we use the object
metaphor to guide the organization of our programs. Most logic about how to
represent and manipulate data is expressed within class declarations. In this
section, we see that classes and objects can themselves be represented using
just functions and dictionaries. The purpose of implementing an object system
in this way is to illustrate that using the object metaphor does not require a
special programming language. Programs can be object-oriented, even in
programming languages that do not have a built-in object system.

In order to implement objects, we will abandon dot notation (which does
require built-in language support), but create dispatch dictionaries that
behave in much the same way as the elements of the built-in object system. We
have already seen how to implement message-passing behavior through dispatch
dictionaries. To implement an object system in full, we send messages between
instances, classes, and base classes, all of which are dictionaries that
contain attributes.

We will not implement the entire Python object system, which includes features
that we have not covered in this text (e.g., meta-classes and static methods).
We will focus instead on user-defined classes without multiple inheritance and
without introspective behavior (such as returning the class of an instance).
Our implementation is not meant to follow the precise specification of the
Python type system. Instead, it is designed to implement the core
functionality that enables the object metaphor.

### 2.6.1 Instances

We begin with instances. An instance has named attributes, such as the balance
of an account, which can be set and retrieved. We implement an instance using
a dispatch dictionary that responds to messages that "get" and "set" attribute
values. Attributes themselves are stored in a local dictionary called
`attributes`.

As we have seen previously in this chapter, dictionaries themselves are
abstract data types. We implemented dictionaries with lists, we implemented
lists with pairs, and we implemented pairs with functions. As we implement an
object system in terms of dictionaries, keep in mind that we could just as
well be implementing objects using functions alone.

To begin our implementation, we assume that we have a class implementation
that can look up any names that are not part of the instance. We pass in a
class to `make_instance` as the parameter `cls`.


​    

    >>> def make_instance(cls):
            """Return a new object instance, which is a dispatch dictionary."""
            def get_value(name):
                if name in attributes:
                    return attributes[name]
                else:
                    value = cls['get'](name)
                    return bind_method(value, instance)
            def set_value(name, value):
                attributes[name] = value
            attributes = {}
            instance = {'get': get_value, 'set': set_value}
            return instance


The `instance` is a dispatch dictionary that responds to the messages `get`
and `set`. The `set` message corresponds to attribute assignment in Python's
object system: all assigned attributes are stored directly within the object's
local attribute dictionary. In `get`, if `name` does not appear in the local
`attributes` dictionary, then it is looked up in the class. If the `value`
returned by `cls` is a function, it must be bound to the instance.

**Bound method values.** The `get_value` function in `make_instance` finds a
named attribute in its class with `get`, then calls `bind_method`. Binding a
method only applies to function values, and it creates a bound method value
from a function value by inserting the instance as the first argument:


​    

    >>> def bind_method(value, instance):
            """Return a bound method if value is callable, or value otherwise."""
            if callable(value):
                def method(*args):
                    return value(instance, *args)
                return method
            else:
                return value


When a method is called, the first parameter `self` will be bound to the value
of `instance` by this definition.

### 2.6.2 Classes

A class is also an object, both in Python's object system and the system we
are implementing here. For simplicity, we say that classes do not themselves
have a class. (In Python, classes do have classes; almost all classes share
the same class, called `type`.) A class can respond to `get` and `set`
messages, as well as the `new` message:


​    

    >>> def make_class(attributes, base_class=None):
            """Return a new class, which is a dispatch dictionary."""
            def get_value(name):
                if name in attributes:
                    return attributes[name]
                elif base_class is not None:
                    return base_class['get'](name)
            def set_value(name, value):
                attributes[name] = value
            def new(*args):
                return init_instance(cls, *args)
            cls = {'get': get_value, 'set': set_value, 'new': new}
            return cls


Unlike an instance, the `get` function for classes does not query its class
when an attribute is not found, but instead queries its `base_class`. No
method binding is required for classes.

**Initialization.** The `new` function in `make_class` calls `init_instance`,
which first makes a new instance, then invokes a method called `__init__`.


​    

    >>> def init_instance(cls, *args):
            """Return a new object with type cls, initialized with args."""
            instance = make_instance(cls)
            init = cls['get']('__init__')
            if init:
                init(instance, *args)
            return instance


This final function completes our object system. We now have instances, which
`set` locally but fall back to their classes on `get`. After an instance looks
up a name in its class, it binds itself to function values to create methods.
Finally, classes can create `new` instances, and they apply their `__init__`
constructor function immediately after instance creation.

In this object system, the only function that should be called by the user is
`make_class`. All other functionality is enabled through message passing.
Similarly, Python's object system is invoked via the `class` statement, and
all of its other functionality is enabled through dot expressions and calls to
classes.

### 2.6.3 Using Implemented Objects

We now return to use the bank account example from the previous section. Using
our implemented object system, we will create an `Account` class, a
`CheckingAccount` subclass, and an instance of each.

The `Account` class is created through a `make_account_class` function, which
has structure similar to a `class` statement in Python, but concludes with a
call to `make_class`.


​    

    >>> def make_account_class():
            """Return the Account class, which has deposit and withdraw methods."""
            interest = 0.02
            def __init__(self, account_holder):
                self['set']('holder', account_holder)
                self['set']('balance', 0)
            def deposit(self, amount):
                """Increase the account balance by amount and return the new balance."""
                new_balance = self['get']('balance') + amount
                self['set']('balance', new_balance)
                return self['get']('balance')
            def withdraw(self, amount):
                """Decrease the account balance by amount and return the new balance."""
                balance = self['get']('balance')
                if amount > balance:
                    return 'Insufficient funds'
                self['set']('balance', balance - amount)
                return self['get']('balance')
            return make_class(locals())


The final call to `locals` returns a dictionary with string keys that contains
the name-value bindings in the current local frame.

The `Account` class is finally instantiated via assignment.


​    

    >>> Account = make_account_class()


Then, an account instance is created via the `new` message, which requires a
name to go with the newly created account.


​    

    >>> kirk_account = Account['new']('Kirk')


Then, `get` messages passed to `kirk_account` retrieve properties and methods.
Methods can be called to update the balance of the account.


​    

    >>> kirk_account['get']('holder')
    'Kirk'
    >>> kirk_account['get']('interest')
    0.02
    >>> kirk_account['get']('deposit')(20)
    20
    >>> kirk_account['get']('withdraw')(5)
    15


As with the Python object system, setting an attribute of an instance does not
change the corresponding attribute of its class.


​    

    >>> kirk_account['set']('interest', 0.04)
    >>> Account['get']('interest')
    0.02


**Inheritance.** We can create a subclass `CheckingAccount` by overloading a
subset of the class attributes. In this case, we change the `withdraw` method
to impose a fee, and we reduce the interest rate.


​    

    >>> def make_checking_account_class():
            """Return the CheckingAccount class, which imposes a $1 withdrawal fee."""
            interest = 0.01
            withdraw_fee = 1
            def withdraw(self, amount):
                fee = self['get']('withdraw_fee')
                return Account['get']('withdraw')(self, amount + fee)
            return make_class(locals(), Account)


In this implementation, we call the `withdraw` function of the base class
`Account` from the `withdraw` function of the subclass, as we would in
Python's built-in object system. We can create the subclass itself and an
instance, as before.


​    

    >>> CheckingAccount = make_checking_account_class()
    >>> jack_acct = CheckingAccount['new']('Spock')


Deposits behave identically, as does the constructor function. withdrawals
impose the $1 fee from the specialized `withdraw` method, and `interest` has
the new lower value from `CheckingAccount`.


​    

    >>> jack_acct['get']('interest')
    0.01
    >>> jack_acct['get']('deposit')(20)
    20
    >>> jack_acct['get']('withdraw')(5)
    14


Our object system built upon dictionaries is quite similar in implementation
to the built-in object system in Python. In Python, an instance of any user-
defined class has a special attribute `__dict__` that stores the local
instance attributes for that object in a dictionary, much like our
`attributes` dictionary. Python differs because it distinguishes certain
special methods that interact with built-in functions to ensure that those
functions behave correctly for arguments of many different types. Functions
that operate on different types are the subject of the next section.

_Continue_ : [ 2.7 Object Abstraction ](../pages/27-object-abstraction.html)

## 2.7 Object Abstraction

The object system allows programmers to build and use abstract data
representations efficiently. It is also designed to allow multiple
representations of abstract data to coexist in the same program.

A central concept in object abstraction is a _generic function_ , which is a
function that can accept values of multiple different types. We will consider
three different techniques for implementing generic functions: shared
interfaces, type dispatching, and type coercion. In the process of building up
these concepts, we will also discover features of the Python object system
that support the creation of generic functions.

### 2.7.1 String Conversion

To represent data effectively, an object value should behave like the kind of
data it is meant to represent, including producing a string representation of
itself. String representations of data values are especially important in an
interactive language such as Python that automatically displays the string
representation of the values of expressions in an interactive session.

String values provide a fundamental medium for communicating information among
humans. Sequences of characters can be rendered on a screen, printed to paper,
read aloud, converted to braille, or broadcast as Morse code. Strings are also
fundamental to programming because they can represent Python expressions.

Python stipulates that all objects should produce two different string
representations: one that is human-interpretable text and one that is a
Python-interpretable expression. The constructor function for strings, `str`,
returns a human-readable string. Where possible, the `repr` function returns a
Python expression that evaluates to an equal object. The docstring for _repr_
explains this property:


​    

    repr(object) -> string
    
    Return the canonical string representation of the object.
    For most object types, eval(repr(object)) == object.


The result of calling `repr` on the value of an expression is what Python
prints in an interactive session.


​    

    >>> 12e12
    12000000000000.0
    >>> print(repr(12e12))
    12000000000000.0


In cases where no representation exists that evaluates to the original value,
Python typically produces a description surrounded by angled brackets.


​    

    >>> repr(min)
    '<built-in function min>'


The `str` constructor often coincides with `repr`, but provides a more
interpretable text representation in some cases. For instance, we see a
difference between `str` and `repr` with dates.


​    

    >>> from datetime import date
    >>> tues = date(2011, 9, 12)
    >>> repr(tues)
    'datetime.date(2011, 9, 12)'
    >>> str(tues)
    '2011-09-12'


Defining the `repr` function presents a new challenge: we would like it to
apply correctly to all data types, even those that did not exist when `repr`
was implemented. We would like it to be a generic or _polymorphic function_ ,
one that can be applied to many (_poly_) different forms (_morph_) of data.

The object system provides an elegant solution in this case: the `repr`
function always invokes a method called `__repr__` on its argument.


​    

    >>> tues.__repr__()
    'datetime.date(2011, 9, 12)'


By implementing this same method in user-defined classes, we can extend the
applicability of `repr` to any class we create in the future. This example
highlights another benefit of dot expressions in general, that they provide a
mechanism for extending the domain of existing functions to new object types.

The `str` constructor is implemented in a similar manner: it invokes a method
called `__str__` on its argument.


​    

    >>> tues.__str__()
    '2011-09-12'


These polymorphic functions are examples of a more general principle: certain
functions should apply to multiple data types. Moreover, one way to create
such a function is to use a shared attribute name with a different definition
in each class.

### 2.7.2 Special Methods

In Python, certain _special names_ are invoked by the Python interpreter in
special circumstances. For instance, the `__init__` method of a class is
automatically invoked whenever an object is constructed. The `__str__` method
is invoked automatically when printing, and `__repr__` is invoked in an
interactive session to display values.

There are special names for many other behaviors in Python. Some of those used
most commonly are described below.

**True and false values.** We saw previously that numbers in Python have a
truth value; more specifically, 0 is a false value and all other numbers are
true values. In fact, all objects in Python have a truth value. By default,
objects of user-defined classes are considered to be true, but the special
`__bool__` method can be used to override this behavior. If an object defines
the `__bool__` method, then Python calls that method to determine its truth
value.

As an example, suppose we want a bank account with 0 balance to be false. We
can add a `__bool__` method to the `Account` class to create this behavior.


​    

    >>> Account.__bool__ = lambda self: self.balance != 0


We can call the `bool` constructor to see the truth value of an object, and we
can use any object in a boolean context.


​    

    >>> bool(Account('Jack'))
    False
    >>> if not Account('Jack'):
            print('Jack has nothing')
    Jack has nothing


**Sequence operations.** We have seen that we can call the `len` function to
determine the length of a sequence.


​    

    >>> len('Go Bears!')
    9


The `len` function invokes the `__len__` method of its argument to determine
its length. All built-in sequence types implement this method.


​    

    >>> 'Go Bears!'.__len__()
    9


Python uses a sequence's length to determine its truth value, if it does not
provide a `__bool__` method. Empty sequences are false, while non-empty
sequences are true.


​    

    >>> bool('')
    False
    >>> bool([])
    False
    >>> bool('Go Bears!')
    True


The `__getitem__` method is invoked by the element selection operator, but it
can also be invoked directly.


​    

    >>> 'Go Bears!'[3]
    'B'
    >>> 'Go Bears!'.__getitem__(3)
    'B'


**Callable objects.** In Python, functions are first-class objects, so they
can be passed around as data and have attributes like any other object. Python
also allows us to define objects that can be "called" like functions by
including a `__call__` method. With this method, we can define a class that
behaves like a higher-order function.

As an example, consider the following higher-order function, which returns a
function that adds a constant value to its argument.


​    

    >>> def make_adder(n):
            def adder(k):
                return n + k
            return adder


​    
​    

    >>> add_three = make_adder(3)
    >>> add_three(4)
    7


We can create an `Adder` class that defines a `__call__` method to provide the
same functionality.


​    

    >>> class Adder(object):
            def __init__(self, n):
                self.n = n
            def __call__(self, k):
                return self.n + k


​    
​    

    >>> add_three_obj = Adder(3)
    >>> add_three_obj(4)
    7


Here, the `Adder` class behaves like the `make_adder` higher-order function,
and the `add_three_obj` object behaves like the `add_three` function. We have
further blurred the line between data and functions.

**Arithmetic.** Special methods can also define the behavior of built-in
operators applied to user-defined objects. In order to provide this
generality, Python follows specific protocols to apply each operator. For
example, to evaluate expressions that contain the `+` operator, Python checks
for special methods on both the left and right operands of the expression.
First, Python checks for an `__add__` method on the value of the left operand,
then checks for an `__radd__` method on the value of the right operand. If
either is found, that method is invoked with the value of the other operand as
its argument. Some examples are given in the following sections. For readers
interested in further details, the Python documentation describes the
exhaustive set of [method names for
operators](http://docs.python.org/py3k/reference/datamodel.html#special-
method-names). Dive into Python 3 has a chapter on [special method
names](http://getpython3.com/diveintopython3/special-method-names.html) that
describes how many of these special method names are used.

### 2.7.3 Multiple Representations

Abstraction barriers allow us to separate the use and representation of data.
However, in large programs, it may not always make sense to speak of "the
underlying representation" for a data type in a program. For one thing, there
might be more than one useful representation for a data object, and we might
like to design systems that can deal with multiple representations.

To take a simple example, complex numbers may be represented in two almost
equivalent ways: in rectangular form (real and imaginary parts) and in polar
form (magnitude and angle). Sometimes the rectangular form is more appropriate
and sometimes the polar form is more appropriate. Indeed, it is perfectly
plausible to imagine a system in which complex numbers are represented in both
ways, and in which the functions for manipulating complex numbers work with
either representation. We implement such a system below. As a side note, we
are developing a system that performs arithmetic operations on complex numbers
as a simple but unrealistic example of a program that uses generic operations.
A [complex number
type](http://docs.python.org/py3k/library/stdtypes.html#typesnumeric) is
actually built into Python, but for this example we will implement our own.

The idea of allowing for multiple representations of data arises regularly.
Large software systems are often designed by many people working over extended
periods of time, subject to requirements that change over time. In such an
environment, it is simply not possible for everyone to agree in advance on
choices of data representation. In addition to the data-abstraction barriers
that isolate representation from use, we need abstraction barriers that
isolate different design choices from each other and permit different choices
to coexist in a single program.

We will begin our implementation at the highest level of abstraction and work
towards concrete representations. A `Complex` number is a `Number`, and
numbers can be added or multiplied together. How numbers can be added or
multiplied is abstracted by the method names `add` and `mul`.


​    

    >>> class Number:
            def __add__(self, other):
                return self.add(other)
            def __mul__(self, other):
                return self.mul(other)


This class requires that Number objects have `add` and `mul` methods, but does
not define them. Moreover, it does not have an `__init__` method. The purpose
of `Number` is not to be instantiated directly, but instead to serve as a
superclass of various specific number classes. Our next task is to define
`add` and `mul` appropriately for complex numbers.

A complex number can be thought of as a point in two-dimensional space with
two orthogonal axes, the real axis and the imaginary axis. From this
perspective, the complex number `c = real + imag * i` (where `i * i = -1`) can
be thought of as the point in the plane whose horizontal coordinate is `real`
and whose vertical coordinate is `imag`. Adding complex numbers involves
adding their respective `real` and `imag` coordinates.

When multiplying complex numbers, it is more natural to think in terms of
representing a complex number in polar form, as a `magnitude` and an `angle`.
The product of two complex numbers is the vector obtained by stretching one
complex number by a factor of the length of the other, and then rotating it
through the angle of the other.

The `Complex` class inherits from `Number` and describes arithmetic for
complex numbers.


​    

    >>> class Complex(Number):
            def add(self, other):
                return ComplexRI(self.real + other.real, self.imag + other.imag)
            def mul(self, other):
                magnitude = self.magnitude * other.magnitude
                return ComplexMA(magnitude, self.angle + other.angle)


This implementation assumes that two classes exist for complex numbers,
corresponding to their two natural representations:

  * `ComplexRI` constructs a complex number from real and imaginary parts.
  * `ComplexMA` constructs a complex number from a magnitude and angle.

**Interfaces.** Object attributes, which are a form of message passing, allows
different data types to respond to the same message in different ways. A
shared set of messages that elicit similar behavior from different classes is
a powerful method of abstraction. An _interface_ is a set of shared attribute
names, along with a specification of their behavior. In the case of complex
numbers, the interface needed to implement arithmetic consists of four
attributes: `real`, `imag`, `magnitude`, and `angle`.

For complex arithmetic to be correct, these attributes must be consistent.
That is, the rectangular coordinates `(real, imag)` and the polar coordinates
`(magnitude, angle)` must describe the same point on the complex plane. The
`Complex` class implicitly defines this interface by determining how these
attributes are used to `add` and `mul` complex numbers.

**Properties.** The requirement that two or more attribute values maintain a
fixed relationship with each other is a new problem. One solution is to store
attribute values for only one representation and compute the other
representation whenever it is needed.

Python has a simple feature for computing attributes on the fly from zero-
argument functions. The `@property` decorator allows functions to be called
without call expression syntax (parentheses following an expression). The
`ComplexRI` class stores `real` and `imag` attributes and computes `magnitude`
and `angle` on demand.


​    

    >>> from math import atan2
    >>> class ComplexRI(Complex):
            def __init__(self, real, imag):
                self.real = real
                self.imag = imag
            @property
            def magnitude(self):
                return (self.real ** 2 + self.imag ** 2) ** 0.5
            @property
            def angle(self):
                return atan2(self.imag, self.real)
            def __repr__(self):
                return 'ComplexRI({0:g}, {1:g})'.format(self.real, self.imag)


As a result of this implementation, all four attributes needed for complex
arithmetic can be accessed without any call expressions, and changes to `real`
or `imag` are reflected in the `magnitude` and `angle`.


​    

    >>> ri = ComplexRI(5, 12)
    >>> ri.real
    5
    >>> ri.magnitude
    13.0
    >>> ri.real = 9
    >>> ri.real
    9
    >>> ri.magnitude
    15.0


Similarly, the `ComplexMA` class stores `magnitude` and `angle`, but computes
`real` and `imag` whenever those attributes are looked up.


​    

    >>> from math import sin, cos, pi
    >>> class ComplexMA(Complex):
            def __init__(self, magnitude, angle):
                self.magnitude = magnitude
                self.angle = angle
            @property
            def real(self):
                return self.magnitude * cos(self.angle)
            @property
            def imag(self):
                return self.magnitude * sin(self.angle)
            def __repr__(self):
                return 'ComplexMA({0:g}, {1:g} * pi)'.format(self.magnitude, self.angle/pi)


Changes to the magnitude or angle are reflected immediately in the `real` and
`imag` attributes.


​    

    >>> ma = ComplexMA(2, pi/2)
    >>> ma.imag
    2.0
    >>> ma.angle = pi
    >>> ma.real
    -2.0


Our implementation of complex numbers is now complete. Either class
implementing complex numbers can be used for either argument in either
arithmetic function in `Complex`.


​    

    >>> from math import pi
    >>> ComplexRI(1, 2) + ComplexMA(2, pi/2)
    ComplexRI(1, 4)
    >>> ComplexRI(0, 1) * ComplexRI(0, 1)
    ComplexMA(1, 1 * pi)


The interface approach to encoding multiple representations has appealing
properties. The class for each representation can be developed separately;
they must only agree on the names of the attributes they share, as well as any
behavior conditions for those attributes. The interface is also _additive_. If
another programmer wanted to add a third representation of complex numbers to
the same program, they would only have to create another class with the same
attributes.

Multiple representations of data are closely related to the idea of data
abstraction with which we began this chapter. Using data abstraction, we were
able to change the implementation of a data type without changing the meaning
of the program. With interfaces and message passing, we can have multiple
different representations within the same program. In both cases, a set of
names and corresponding behavior conditions define the abstraction that
enables this flexibility.

### 2.7.4 Generic Functions

Generic functions are methods or functions that apply to arguments of
different types. We have seen many examples already. The `Complex.add` method
is generic, because it can take either a `ComplexRI` or `ComplexMA` as the
value for `other`. This flexibility was gained by ensuring that both
`ComplexRI` and `ComplexMA` share an interface. Using interfaces and message
passing is only one of several methods used to implement generic functions. We
will consider two others in this section: type dispatching and type coercion.

Suppose that, in addition to our complex number classes, we implement a
`Rational` class to represent fractions exactly. The `add` and `mul` methods
express the same computations as the `add_rational` and `mul_rational`
functions from earlier in the chapter.


​    

    >>> from fractions import gcd
    >>> class Rational(Number):
            def __init__(self, numer, denom):
                g = gcd(numer, denom)
                self.numer = numer // g
                self.denom = denom // g
            def __repr__(self):
                return 'Rational({0}, {1})'.format(self.numer, self.denom)
            def add(self, other):
                nx, dx = self.numer, self.denom
                ny, dy = other.numer, other.denom
                return Rational(nx * dy + ny * dx, dx * dy)
            def mul(self, other):
                numer = self.numer * other.numer
                denom = self.denom * other.denom
                return Rational(numer, denom)


We have implemented the interface of the `Number` superclass by including
`add` and `mul` methods. As a result, we can add and multiply rational numbers
using familiar operators.


​    

    >>> Rational(2, 5) + Rational(1, 10)
    Rational(1, 2)
    >>> Rational(1, 4) * Rational(2, 3)
    Rational(1, 6)


However, we cannot yet add a rational number to a complex number, although in
mathematics such a combination is well-defined. We would like to introduce
this cross-type operation in some carefully controlled way, so that we can
support it without seriously violating our abstraction barriers. There is a
tension between the outcomes we desire: we would like to be able to add a
complex number to a rational number, and we would like to do so using a
generic `__add__` method that does the right thing with all numeric types. At
the same time, we would like to separate the concerns of complex numbers and
rational numbers whenever possible, in order to maintain a modular program.

**Type dispatching.** One way to implement cross-type operations is to select
behavior based on the types of the arguments to a function or method. The idea
of type dispatching is to write functions that inspect the type of arguments
they receive, then execute code that is appropriate for those types.

The built-in function `isinstance` takes an object and a class. It returns
true if the object has a class that either is or inherits from the given
class.


​    

    >>> c = ComplexRI(1, 1)
    >>> isinstance(c, ComplexRI)
    True
    >>> isinstance(c, Complex)
    True
    >>> isinstance(c, ComplexMA)
    False


A simple example of type dispatching is an `is_real` function that uses a
different implementation for each type of complex number.


​    

    >>> def is_real(c):
            """Return whether c is a real number with no imaginary part."""
            if isinstance(c, ComplexRI):
                return c.imag == 0
            elif isinstance(c, ComplexMA):
                return c.angle % pi == 0


​    
​    

    >>> is_real(ComplexRI(1, 1))
    False
    >>> is_real(ComplexMA(2, pi))
    True


Type dispatching is not always performed using `isinstance`. For arithmetic,
we will give a `type_tag` attribute to `Rational` and `Complex` instances that
has a string value. When two values `x` and `y` have the same `type_tag`, then
we can combine them directly with `x.add(y)`. If not, we need a cross-type
operation.


​    

    >>> Rational.type_tag = 'rat'
    >>> Complex.type_tag = 'com'
    >>> Rational(2, 5).type_tag == Rational(1, 2).type_tag
    True
    >>> ComplexRI(1, 1).type_tag == ComplexMA(2, pi/2).type_tag
    True
    >>> Rational(2, 5).type_tag == ComplexRI(1, 1).type_tag
    False


To combine complex and rational numbers, we write functions that rely on both
of their representations simultaneously. Below, we rely on the fact that a
`Rational` can be converted approximately to a `float` value that is a real
number. The result can be combined with a complex number.


​    

    >>> def add_complex_and_rational(c, r):
            return ComplexRI(c.real + r.numer/r.denom, c.imag)


Multiplication involves a similar conversion. In polar form, a real number in
the complex plane always has a positive magnitude. The angle 0 indicates a
positive number. The angle `pi` indicates a negative number.


​    

    >>> def mul_complex_and_rational(c, r):
            r_magnitude, r_angle = r.numer/r.denom, 0
            if r_magnitude < 0:
                r_magnitude, r_angle = -r_magnitude, pi
            return ComplexMA(c.magnitude * r_magnitude, c.angle + r_angle)


Both addition and multiplication are commutative, so swapping the argument
order can use the same implementations of these cross-type operations.


​    

    >>> def add_rational_and_complex(r, c):
            return add_complex_and_rational(c, r)


​    
​    

    >>> def mul_rational_and_complex(r, c):
            return mul_complex_and_rational(c, r)


The role of type dispatching is to ensure that these cross-type operations are
used at appropriate times. Below, we rewrite the `Number` superclass to use
type dispatching for its `__add__` and `__mul__` methods.

We use the `type_tag` attribute to distinguish types of arguments. One could
directly use the built-in `isinstance` method as well, but tags simplify the
implementation. Using type tags also illustrates that type dispatching is not
necessarily linked to the Python object system, but instead a general
technique for creating generic functions over heterogeneous domains.

The `__add__` method considers two cases. First, if two arguments have the
same type tag, then it assumes that `add` method of the first can take the
second as an argument. Otherwise, it checks whether a dictionary of cross-type
implementations, called `adders`, contains a function that can add arguments
of those type tags. If there is such a function, the `cross_apply` method
finds and applies it. The `__mul__` method has a similar structure.


​    

    >>> class Number:
            def __add__(self, other):
                if self.type_tag == other.type_tag:
                    return self.add(other)
                elif (self.type_tag, other.type_tag) in self.adders:
                    return self.cross_apply(other, self.adders)
            def __mul__(self, other):
                if self.type_tag == other.type_tag:
                    return self.mul(other)
                elif (self.type_tag, other.type_tag) in self.multipliers:
                    return self.cross_apply(other, self.multipliers)
            def cross_apply(self, other, cross_fns):
                cross_fn = cross_fns[(self.type_tag, other.type_tag)]
                return cross_fn(self, other)
            adders = {("com", "rat"): add_complex_and_rational,
                      ("rat", "com"): add_rational_and_complex}
            multipliers = {("com", "rat"): mul_complex_and_rational,
                           ("rat", "com"): mul_rational_and_complex}


In this new definition of the `Number` class, all cross-type implementations
are indexed by pairs of type tags in the `adders` and `multipliers`
dictionaries.

This dictionary-based approach to type dispatching is extensible. New
subclasses of `Number` could install themselves into the system by declaring a
type tag and adding cross-type operations to `Number.adders` and
`Number.multipliers`. They could also define their own `adders` and
`multipliers` in a subclass.

While we have introduced some complexity to the system, we can now mix types
in addition and multiplication expressions.


​    

    >>> ComplexRI(1.5, 0) + Rational(3, 2)
    ComplexRI(3, 0)
    >>> Rational(-1, 2) * ComplexMA(4, pi/2)
    ComplexMA(2, 1.5 * pi)


**Coercion.** In the general situation of completely unrelated operations
acting on completely unrelated types, implementing explicit cross-type
operations, cumbersome though it may be, is the best that one can hope for.
Fortunately, we can sometimes do better by taking advantage of additional
structure that may be latent in our type system. Often the different data
types are not completely independent, and there may be ways by which objects
of one type may be viewed as being of another type. This process is called
_coercion_. For example, if we are asked to arithmetically combine a rational
number with a complex number, we can view the rational number as a complex
number whose imaginary part is zero. After doing so, we can use `Complex.add`
and `Complex.mul` to combine them.

In general, we can implement this idea by designing coercion functions that
transform an object of one type into an equivalent object of another type.
Here is a typical coercion function, which transforms a rational number to a
complex number with zero imaginary part:


​    

    >>> def rational_to_complex(r):
            return ComplexRI(r.numer/r.denom, 0)


The alternative definition of the `Number` class performs cross-type
operations by attempting to coerce both arguments to the same type. The
`coercions` dictionary indexes all possible coercions by a pair of type tags,
indicating that the corresponding value coerces a value of the first type to a
value of the second type.

It is not generally possible to coerce an arbitrary data object of each type
into all other types. For example, there is no way to coerce an arbitrary
complex number to a rational number, so there will be no such conversion
implementation in the `coercions` dictionary.

The `coerce` method returns two values with the same type tag. It inspects the
type tags of its arguments, compares them to entries in the `coercions`
dictionary, and converts one argument to the type of the other using
`coerce_to`. Only one entry in `coercions` is necessary to complete our cross-
type arithmetic system, replacing the four cross-type functions in the type-
dispatching version of `Number`.


​    

    >>> class Number:
            def __add__(self, other):
                x, y = self.coerce(other)
                return x.add(y)
            def __mul__(self, other):
                x, y = self.coerce(other)
                return x.mul(y)
            def coerce(self, other):
                if self.type_tag == other.type_tag:
                    return self, other
                elif (self.type_tag, other.type_tag) in self.coercions:
                    return (self.coerce_to(other.type_tag), other)
                elif (other.type_tag, self.type_tag) in self.coercions:
                    return (self, other.coerce_to(self.type_tag))
            def coerce_to(self, other_tag):
                coercion_fn = self.coercions[(self.type_tag, other_tag)]
                return coercion_fn(self)
            coercions = {('rat', 'com'): rational_to_complex}


This coercion scheme has some advantages over the method of defining explicit
cross-type operations. Although we still need to write coercion functions to
relate the types, we need to write only one function for each pair of types
rather than a different function for each set of types and each generic
operation. What we are counting on here is the fact that the appropriate
transformation between types depends only on the types themselves, not on the
particular operation to be applied.

Further advantages come from extending coercion. Some more sophisticated
coercion schemes do not just try to coerce one type into another, but instead
may try to coerce two different types each into a third common type. Consider
a rhombus and a rectangle: neither is a special case of the other, but both
can be viewed as quadrilaterals. Another extension to coercion is iterative
coercion, in which one data type is coerced into another via intermediate
types. Consider that an integer can be converted into a real number by first
converting it into a rational number, then converting that rational number
into a real number. Chaining coercion in this way can reduce the total number
of coercion functions that are required by a program.

Despite its advantages, coercion does have potential drawbacks. For one,
coercion functions can lose information when they are applied. In our example,
rational numbers are exact representations, but become approximations when
they are converted to complex numbers.

Some programming languages have automatic coercion systems built in. In fact,
early versions of Python had a `__coerce__` special method on objects. In the
end, the complexity of the built-in coercion system did not justify its use,
and so it was removed. Instead, particular operators apply coercion to their
arguments as needed.

_Continue_ : [ 2.8 Efficiency ](../pages/28-efficiency.html)

## 2.8 Efficiency

Decisions of how to represent and process data are often influenced by the
efficiency of alternatives. Efficiency refers to the computational resources
used by a representation or process, such as how much time and memory are
required to compute the result of a function or represent an object. These
amounts can vary widely depending on the details of an implementation.

### 2.8.1 Measuring Efficiency

Measuring exactly how long a program requires to run or how much memory it
consumes is challenging, because the results depend upon many details of how a
computer is configured. A more reliable way to characterize the efficiency of
a program is to measure how many times some event occurs, such as a function
call.

Let's return to our first tree-recursive function, the `fib` function for
computing numbers in the Fibonacci sequence.


​    

    >>> def fib(n):
            if n == 0:
                return 0
            if n == 1:
                return 1
            return fib(n-2) + fib(n-1)


​    
​    

    >>> fib(5)
    5


Consider the pattern of computation that results from evaluating `fib(6)`,
depicted below. To compute `fib(5)`, we compute `fib(3)` and `fib(4)`. To
compute `fib(3)`, we compute `fib(1)` and `fib(2)`. In general, the evolved
process looks like a tree. Each blue dot indicates a completed computation of
a Fibonacci number in the traversal of this tree.

![](http://www.composingprograms.com/img/fib.png)

This function is instructive as a prototypical tree recursion, but it is a
terribly inefficient way to compute Fibonacci numbers because it does so much
redundant computation. The entire computation of `fib(3)` is duplicated.

We can measure this inefficiency. The higher-order `count` function returns an
equivalent function to its argument that also maintains a `call_count`
attribute. In this way, we can inspect just how many times `fib` is called.


​    

    >>> def count(f):
            def counted(*args):
                counted.call_count += 1
                return f(*args)
            counted.call_count = 0
            return counted


By counting the number of calls to `fib`, we see that the calls required grows
faster than the Fibonacci numbers themselves. This rapid expansion of calls is
characteristic of tree-recursive functions.


​    

    >>> fib = count(fib)
    >>> fib(19)
    4181
    >>> fib.call_count
    13529


**Space.** To understand the space requirements of a function, we must specify
generally how memory is used, preserved, and reclaimed in our environment
model of computation. In evaluating an expression, the interpreter preserves
all _active_ environments and all values and frames referenced by those
environments. An environment is active if it provides the evaluation context
for some expression being evaluated. An environment becomes inactive whenever
the function call for which its first frame was created finally returns.

For example, when evaluating `fib`, the interpreter proceeds to compute each
value in the order shown previously, traversing the structure of the tree. To
do so, it only needs to keep track of those nodes that are above the current
node in the tree at any point in the computation. The memory used to evaluate
the rest of the branches can be reclaimed because it cannot affect future
computation. In general, the space required for tree-recursive functions will
be proportional to the maximum depth of the tree.

The diagram below depicts the environment created by evaluating `fib(3)`. In
the process of evaluating the return expression for the initial application of
`fib`, the expression `fib(n-2)` is evaluated, yielding a value of 0. Once
this value is computed, the corresponding environment frame (grayed out) is no
longer needed: it is not part of an active environment. Thus, a well-designed
interpreter can reclaim the memory that was used to store this frame. On the
other hand, if the interpreter is currently evaluating `fib(n-1)`, then the
environment created by this application of `fib` (in which `n` is 2) is
active. In turn, the environment originally created to apply `fib` to 3 is
active because its return value has not yet been computed.

def fib(n): if n == 0: return 0 if n == 1: return 1 return fib(n-2) + fib(n-1)
result = fib(2)

The higher-order `count_frames` function tracks `open_count`, the number of
calls to the function `f` that have not yet returned. The `max_count`
attribute is the maximum value ever attained by `open_count`, and it
corresponds to the maximum number of frames that are ever simultaneously
active during the course of computation.


​    

    >>> def count_frames(f):
            def counted(*args):
                counted.open_count += 1
                counted.max_count = max(counted.max_count, counted.open_count)
                result = f(*args)
                counted.open_count -= 1
                return result
            counted.open_count = 0
            counted.max_count = 0
            return counted


​    
​    

    >>> fib = count_frames(fib)
    >>> fib(19)
    4181
    >>> fib.open_count
    0
    >>> fib.max_count
    19
    >>> fib(24)
    46368
    >>> fib.max_count
    24


To summarize, the space requirement of the `fib` function, measured in active
frames, is one less than the input, which tends to be small. The time
requirement measured in total recursive calls is larger than the output, which
tends to be huge.

### 2.8.2 Memoization

Tree-recursive computational processes can often be made more efficient
through _memoization_ , a powerful technique for increasing the efficiency of
recursive functions that repeat computation. A memoized function will store
the return value for any arguments it has previously received. A second call
to `fib(25)` would not re-compute the return value recursively, but instead
return the existing one that has already been constructed.

Memoization can be expressed naturally as a higher-order function, which can
also be used as a decorator. The definition below creates a _cache_ of
previously computed results, indexed by the arguments from which they were
computed. The use of a dictionary requires that the argument to the memoized
function be immutable.


​    

    >>> def memo(f):
            cache = {}
            def memoized(n):
                if n not in cache:
                    cache[n] = f(n)
                return cache[n]
            return memoized


If we apply `memo` to the recursive computation of Fibonacci numbers, a new
pattern of computation evolves, depicted below.

![](http://www.composingprograms.com/img/fib_memo.png)

In this computation of `fib(5)`, the results for `fib(2)` and `fib(3)` are
reused when computing `fib(4)` on the right branch of the tree. As a result,
much of the tree-recursive computation is not required at all.

Using `count`, we can see that the `fib` function is actually only called once
for each unique input to `fib`.


​    

    >>> counted_fib = count(fib)
    >>> fib  = memo(counted_fib)
    >>> fib(19)
    4181
    >>> counted_fib.call_count
    20
    >>> fib(34)
    5702887
    >>> counted_fib.call_count
    35


### 2.8.3 Orders of Growth

Processes can differ massively in the rates at which they consume the
computational resources of space and time, as the previous examples
illustrate. However, exactly determining just how much space or time will be
used when calling a function is a very difficult task that depends upon many
factors. A useful way to analyze a process is to categorize it along with a
group of processes that all have similar requirements. A useful categorization
is the _order of growth_ of a process, which expresses in simple terms how the
resource requirements of a process grow as a function of the input.

As an introduction to orders of growth, we will analyze the function
`count_factors` below, which counts the number of integers that evenly divide
an input `n`. The function attempts to divide `n` by every integer less than
or equal to its square root. The implementation takes advantage of the fact
that if $k$ divides $n$ and $k < \sqrt{n}$ , then there is another factor $j =
n / k$ such that $j > \sqrt{n}$.

from math import sqrt def count_factors(n): sqrt_n = sqrt(n) k, factors = 1, 0
while k < sqrt_n: if n % k == 0: factors += 2 k += 1 if k * k == n: factors +=
1 return factors result = count_factors(576)

How much time is required to evaluate `count_factors`? The exact answer will
vary on different machines, but we can make some useful general observations
about the amount of computation involved. The total number of times this
process executes the body of the `while` statement is the greatest integer
less than $\sqrt{n}$. The statements before and after this `while` statement
are executed exactly once. So, the total number of statements executed is $w
\cdot \sqrt{n} + v$, where $w$ is the number of statements in the `while` body
and $v$ is the number of statements outside of the `while` statement. Although
it isn't exact, this formula generally characterizes how much time will be
required to evaluate `count_factors` as a function of the input `n`.

A more exact description is difficult to obtain. The constants $w$ and $v$ are
not constant at all, because the assignment statements to `factors` are
sometimes executed but sometimes not. An order of growth analysis allows us to
gloss over such details and instead focus on the general shape of growth. In
particular, the order of growth for `count_factors` expresses in precise terms
that the amount of time required to compute `count_factors(n)` scales at the
rate $\sqrt{n}$, within a margin of some constant factors.

**Theta Notation.** Let $n$ be a parameter that measures the size of the input
to some process, and let $R(n)$ be the amount of some resource that the
process requires for an input of size $n$. In our previous examples we took
$n$ to be the number for which a given function is to be computed, but there
are other possibilities. For instance, if our goal is to compute an
approximation to the square root of a number, we might take $n$ to be the
number of digits of accuracy required.

$R(n)$ might measure the amount of memory used, the number of elementary
machine steps performed, and so on. In computers that do only a fixed number
of steps at a time, the time required to evaluate an expression will be
proportional to the number of elementary steps performed in the process of
evaluation.

We say that $R(n)$ has order of growth $\Theta(f(n))$, written $R(n) =
\Theta(f(n))$ (pronounced "theta of $f(n)$"), if there are positive constants
$k_1$ and $k_2$ independent of $n$ such that

\begin{equation*} k_1 \cdot f(n) \leq R(n) \leq k_2 \cdot f(n) \end{equation*}

for any value of $n$ larger than some minimum $m$. In other words, for large
$n$, the value $R(n)$ is always sandwiched between two values that both scale
with $f(n)$:

  * A lower bound $k_1 \cdot f(n)$ and
  * An upper bound $k_2 \cdot f(n)$

We can apply this definition to show that the number of steps required to
evaluate `count_factors(n)` grows as $\Theta(\sqrt{n})$ by inspecting the
function body.

First, we choose $k_1=1$ and $m=0$, so that the lower bound states that
`count_factors(n)` requires at least $1 \cdot \sqrt{n}$ steps for any $n>0$.
There are at least 4 lines executed outside of the `while` statement, each of
which takes at least 1 step to execute. There are at least two lines executed
within the `while` body, along with the while header itself. All of these
require at least one step. The `while` body is evaluated at least $\sqrt{n}-1$
times. Composing these lower bounds, we see that the process requires at least
$4 + 3 \cdot (\sqrt{n}-1)$ steps, which is always larger than $k_1 \cdot
\sqrt{n}$.

Second, we can verify the upper bound. We assume that any single line in the
body of `count_factors` requires at most `p` steps. This assumption isn't true
for every line of Python, but does hold in this case. Then, evaluating
`count_factors(n)` can require at most $p \cdot (5 + 4 \sqrt{n})$, because
there are 5 lines outside of the `while` statement and 4 within (including the
header). This upper bound holds even if every `if` header evaluates to true.
Finally, if we choose $k_2=5p$, then the steps required is always smaller than
$k_2 \cdot \sqrt{n}$. Our argument is complete.

### 2.8.4 Example: Exponentiation

Consider the problem of computing the exponential of a given number. We would
like a function that takes as arguments a base `b` and a positive integer
exponent `n` and computes $b^n$. One way to do this is via the recursive
definition

\begin{align*} b^n &= b \cdot b^{n-1} \\\ b^0 &= 1 \end{align*}

which translates readily into the recursive function


​    

    >>> def exp(b, n):
            if n == 0:
                return 1
            return b * exp(b, n-1)


This is a linear recursive process that requires $\Theta(n)$ steps and
$\Theta(n)$ space. Just as with factorial, we can readily formulate an
equivalent linear iteration that requires a similar number of steps but
constant space.


​    

    >>> def exp_iter(b, n):
            result = 1
            for _ in range(n):
                result = result * b
            return result


We can compute exponentials in fewer steps by using successive squaring. For
instance, rather than computing $b^8$ as

\begin{equation*} b \cdot (b \cdot (b \cdot (b \cdot (b \cdot (b \cdot (b
\cdot b)))))) \end{equation*}

we can compute it using three multiplications:

\begin{align*} b^2 &= b \cdot b \\\ b^4 &= b^2 \cdot b^2 \\\ b^8 &= b^4 \cdot
b^4 \end{align*}

This method works fine for exponents that are powers of 2. We can also take
advantage of successive squaring in computing exponentials in general if we
use the recursive rule

\begin{equation*} b^n = \begin{cases} (b^{\frac{1}{2} n})^2 & \text{if $n$ is
even} \\\ b \cdot b^{n-1} & \text{if $n$ is odd} \end{cases} \end{equation*}

We can express this method as a recursive function as well:


​    

    >>> def square(x):
            return x*x


​    
​    

    >>> def fast_exp(b, n):
            if n == 0:
                return 1
            if n % 2 == 0:
                return square(fast_exp(b, n//2))
            else:
                return b * fast_exp(b, n-1)


​    
​    

    >>> fast_exp(2, 100)
    1267650600228229401496703205376


The process evolved by `fast_exp` grows logarithmically with `n` in both space
and number of steps. To see this, observe that computing $b^{2n}$ using
`fast_exp` requires only one more multiplication than computing $b^n$. The
size of the exponent we can compute therefore doubles (approximately) with
every new multiplication we are allowed. Thus, the number of multiplications
required for an exponent of `n` grows about as fast as the logarithm of `n`
base 2. The process has $\Theta(\log n)$ growth. The difference between
$\Theta(\log n)$ growth and $\Theta(n)$ growth becomes striking as $n$ becomes
large. For example, `fast_exp` for `n` of 1000 requires only 14
multiplications instead of 1000.

### 2.8.5 Growth Categories

Orders of growth are designed to simplify the analysis and comparison of
computational processes. Many different processes can all have equivalent
orders of growth, which indicates that they scale in similar ways. It is an
essential skill of a computer scientist to know and recognize common orders of
growth and identify processes of the same order.

**Constants.** Constant terms do not affect the order of growth of a process.
So, for instance, $\Theta(n)$ and $\Theta(500 \cdot n)$ are the same order of
growth. This property follows from the definition of theta notation, which
allows us to choose arbitrary constants $k_1$ and $k_2$ (such as
$\frac{1}{500}$) for the upper and lower bounds. For simplicity, constants are
always omitted from orders of growth.

**Logarithms.** The base of a logarithm does not affect the order of growth of
a process. For instance, $\log_2 n$ and $\log_{10} n$ are the same order of
growth. Changing the base of a logarithm is equivalent to multiplying by a
constant factor.

**Nesting.** When an inner computational process is repeated for each step in
an outer process, then the order of growth of the entire process is a product
of the number of steps in the outer and inner processes.

For example, the function `overlap` below computes the number of elements in
list `a` that also appear in list `b`.


​    

    >>> def overlap(a, b):
            count = 0
            for item in a:
                if item in b:
                    count += 1
            return count


​    
​    

    >>> overlap([1, 3, 2, 2, 5, 1], [5, 4, 2])
    3


The `in` operator for lists requires $\Theta(n)$ time, where $n$ is the length
of the list `b`. It is applied $\Theta(m)$ times, where $m$ is the length of
the list `a`. The `item in b` expression is the inner process, and the `for
item in a` loop is the outer process. The total order of growth for this
function is $\Theta(m \cdot n)$.

**Lower-order terms.** As the input to a process grows, the fastest growing
part of a computation dominates the total resources used. Theta notation
captures this intuition. In a sum, all but the fastest growing term can be
dropped without changing the order of growth.

For instance, consider the `one_more` function that returns how many elements
of a list `a` are one more than some other element of `a`. That is, in the
list `[3, 14, 15, 9]`, the element 15 is one more than 14, so `one_more` will
return 1.


​    

    >>> def one_more(a):
            return overlap([x-1 for x in a], a)


​    
​    

    >>> one_more([3, 14, 15, 9])
    1


There are two parts to this computation: the list comprehension and the call
to `overlap`. For a list `a` of length $n$, list comprehension requires
$\Theta(n)$ steps, while the call to `overlap` requires $\Theta(n^2)$ steps.
The sum of steps is $\Theta(n + n^2)$, but this is not the simplest way of
expressing the order of growth.

$\Theta(n^2 + k \cdot n)$ and $\Theta(n^2)$ are equivalent for any constant
$k$ because the $n^2$ term will eventually dominate the total for any $k$. The
fact that bounds must hold only for $n$ greater than some minimum $m$
establishes this equivalence. For simplicity, lower-order terms are always
omitted from orders of growth, and so we will never see a sum within a theta
expression.

**Common categories.** Given these equivalence properties, a small set of
common categories emerge to describe most computational processes. The most
common are listed below from slowest to fastest growth, along with
descriptions of the growth as the input increases. Examples for each category
follow.

| **Category** | **Theta Notation** | **Growth Description**                  | **Example** |
| ------------ | ------------------ | --------------------------------------- | ----------- |
| Constant     | $\Theta(1)$        | Growth is independent of the input      | `abs`       |
| Logarithmic  | $\Theta(\log{n})$  | Multiplying input increments resources  | `fast_exp`  |
| Linear       | $\Theta(n)$        | Incrementing input increments resources | `exp`       |
| Quadratic    | $\Theta(n^2)$      | Incrementing input adds n resources     | `one_more`  |
| Exponential  | $\Theta(b^n)$      | Incrementing input multiplies resources | `fib`       |

Other categories exist, such as the $\Theta(\sqrt{n})$ growth of
`count_factors`. However, these categories are particularly common.

Exponential growth describes many different orders of growth, because changing
the base $b$ does affect the order of growth. For instance, the number of
steps in our tree-recursive Fibonacci computation `fib` grows exponentially in
its input $n$. In particular, one can show that the nth Fibonacci number is
the closest integer to

\begin{equation*} \frac{\phi^{n-2}}{\sqrt{5}} \end{equation*}

where $\phi$ is the golden ratio:

\begin{equation*} \phi = \frac{1 + \sqrt{5}}{2} \approx 1.6180 \end{equation*}

We also stated that the number of steps scales with the resulting value, and
so the tree-recursive process requires $\Theta(\phi^n)$ steps, a function that
grows exponentially with $n$.

_Continue_ : [ 2.9 Recursive Objects ](../pages/29-recursive-objects.html)

## 2.9 Recursive Objects

Objects can have other objects as attribute values. When an object of some
class has an attribute value of that same class, it is a recursive object.

### 2.9.1 Linked List Class

A linked list, introduced earlier in this chapter, is composed of a first
element and the rest of the list. The rest of a linked list is itself a linked
list  a recursive definition. The empty list is a special case of a linked
list that has no first element or rest. A linked list is a sequence: it has a
finite length and supports element selection by index.

We can now implement a class with the same behavior. In this version, we will
define its behavior using special method names that allow our class to work
with the built-in `len` function and element selection operator (square
brackets or `operator.getitem`) in Python. These built-in functions invoke
special method names of a class: length is computed by `__len__` and element
selection is computed by `__getitem__`. The empty linked list is represented
by an empty tuple, which has length 0 and no elements.


​    

    >>> class Link:
            """A linked list with a first element and the rest."""
            empty = ()
            def __init__(self, first, rest=empty):
                assert rest is Link.empty or isinstance(rest, Link)
                self.first = first
                self.rest = rest
            def __getitem__(self, i):
                if i == 0:
                    return self.first
                else:
                    return self.rest[i-1]
            def __len__(self):
                return 1 + len(self.rest)


​    
​    

    >>> s = Link(3, Link(4, Link(5)))
    >>> len(s)
    3
    >>> s[1]
    4


The definitions of `__len__` and `__getitem__` are in fact recursive. The
built-in Python function `len` invokes a method called `__len__` when applied
to a user-defined object argument. Likewise, the element selection operator
invokes a method called `__getitem__`. Thus, bodies of these two methods will
call themselves indirectly. For `__len__`, the base case is reached when
`self.rest` evaluates to the empty tuple, `Link.empty`, which has a length of
0.

The built-in `isinstance` function returns whether the first argument has a
type that is or inherits from the second argument. `isinstance(rest, Link)` is
true if `rest` is a `Link` instance or an instance of some sub-class of
`Link`.

Our implementation is complete, but an instance of the `Link` class is
currently difficult to inspect. To help with debugging, we can also define a
function to convert a `Link` to a string expression.


​    

    >>> def link_expression(s):
            """Return a string that would evaluate to s."""
            if s.rest is Link.empty:
                rest = ''
            else:
                rest = ', ' + link_expression(s.rest)
            return 'Link({0}{1})'.format(s.first, rest)


​    
​    

    >>> link_expression(s)
    'Link(3, Link(4, Link(5)))'


This way of displaying an `Link` is so convenient that we would like to use it
whenever an `Link` instance is displayed. We can ensure this behavior by
setting the `link_expression` function as the value of the special class
attribute `__repr__`. Python displays instances of user-defined classes by
invoking their `__repr__` method.


​    

    >>> Link.__repr__ = link_expression
    >>> s
    Link(3, Link(4, Link(5)))


The `Link` class has the closure property. Just as an element of a list can
itself be a list, a `Link` can contain a `Link` as its `first` element.


​    

    >>> s_first = Link(s, Link(6))
    >>> s_first
    Link(Link(3, Link(4, Link(5))), Link(6))


The `s_first` linked list has only two elements, but its first element is a
linked list with three elements.


​    

    >>> len(s_first)
    2
    >>> len(s_first[0])
    3
    >>> s_first[0][2]
    5


Recursive functions are particularly well-suited to manipulate linked lists.
For instance, the recursive `extend_link` function builds a linked list
containing the elements of one `Link` instance `s` followed by the elements of
another `Link` instance `t`. Installing this function as the `__add__` method
of the `Link` class emulates the addition behavior of a built-in list.


​    

    >>> def extend_link(s, t):
            if s is Link.empty:
                return t
            else:
                return Link(s.first, extend_link(s.rest, t))
    >>> extend_link(s, s)
    Link(3, Link(4, Link(5, Link(3, Link(4, Link(5))))))
    >>> Link.__add__ = extend_link
    >>> s + s
    Link(3, Link(4, Link(5, Link(3, Link(4, Link(5))))))


Rather than list comprehensions, one linked list can be generated from another
using two higher-order functions: `map_link` and `filter_link`. The `map_link`
function defined below applies a function `f` to each element of a linked list
`s` and constructs a linked list containing the results.


​    

    >>> def map_link(f, s):
            if s is Link.empty:
                return s
            else:
                return Link(f(s.first), map_link(f, s.rest))
    >>> map_link(square, s)
    Link(9, Link(16, Link(25)))


The `filter_link` function returns a linked list containing all elements of a
linked list `s` for which `f` returns a true value. The combination of
`map_link` and `filter_link` can express the same logic as a list
comprehension.


​    

    >>> def filter_link(f, s):
            if s is Link.empty:
                return s
            else:
                filtered = filter_link(f, s.rest)
                if f(s.first):
                    return Link(s.first, filtered)
                else:
                    return filtered
    >>> odd = lambda x: x % 2 == 1
    >>> map_link(square, filter_link(odd, s))
    Link(9, Link(25))
    >>> [square(x) for x in [3, 4, 5] if odd(x)]
    [9, 25]


The `join_link` function recursively constructs a string that contains the
elements of a linked list seperated by some `separator` string. The result is
much more compact than the output of `link_expression`.


​    

    >>> def join_link(s, separator):
            if s is Link.empty:
                return ""
            elif s.rest is Link.empty:
                return str(s.first)
            else:
                return str(s.first) + separator + join_link(s.rest, separator)
    >>> join_link(s, ", ")
    '3, 4, 5'


**Recursive Construction.** Linked lists are particularly useful when
constructing sequences incrementally, a situation that arises often in
recursive computations.

The `count_partitions` function from Chapter 1 counted the number of ways to
partition an integer `n` using parts up to size `m` via a tree-recursive
process. With sequences, we can also enumerate these partitions explicitly
using a similar process.

We follow the same recursive analysis of the problem as we did while counting:
partitioning `n` using integers up to `m` involves either

    1. partitioning `n-m` using integers up to `m`, or
    2. partitioning `n` using integers up to `m-1`.

For base cases, we find that 0 has an empty partition, while partitioning a
negative integer or using parts smaller than 1 is impossible.


​    

    >>> def partitions(n, m):
            """Return a linked list of partitions of n using parts of up to m.
            Each partition is represented as a linked list.
            """
            if n == 0:
                return Link(Link.empty) # A list containing the empty partition
            elif n < 0 or m == 0:
                return Link.empty
            else:
                using_m = partitions(n-m, m)
                with_m = map_link(lambda s: Link(m, s), using_m)
                without_m = partitions(n, m-1)
                return with_m + without_m


In the recursive case, we construct two sublists of partitions. The first uses
`m`, and so we add `m` to each element of the result `using_m` to form
`with_m`.

The result of `partitions` is highly nested: a linked list of linked lists.
Using `join_link` with appropriate separators, we can display the partitions
in a human-readable manner.


​    

    >>> def print_partitions(n, m):
            lists = partitions(n, m)
            strings = map_link(lambda s: join_link(s, " + "), lists)
            print(join_link(strings, "\n"))


​    
​    

    >>> print_partitions(6, 4)
    4 + 2
    4 + 1 + 1
    3 + 3
    3 + 2 + 1
    3 + 1 + 1 + 1
    2 + 2 + 2
    2 + 2 + 1 + 1
    2 + 1 + 1 + 1 + 1
    1 + 1 + 1 + 1 + 1 + 1


### 2.9.2 Tree Class

Trees can also be represented by instances of user-defined classes, rather
than nested instances of built-in sequence types. A tree is any data structure
that has as an attribute a sequence of branches that are also trees.

**Internal values.** Previously, we defined trees in such a way that all
values appeared at the leaves of the tree. It is also common to define trees
that have internal values at the roots of each subtree. An internal value is
called an `label` in the tree. The `Tree` class below represents such trees,
in which each tree has a sequence of branches that are also trees.


​    

    >>> class Tree:
            def __init__(self, label, branches=()):
                self.label = label
                for branch in branches:
                    assert isinstance(branch, Tree)
                self.branches = branches
            def __repr__(self):
                if self.branches:
                    return 'Tree({0}, {1})'.format(self.label, repr(self.branches))
                else:
                    return 'Tree({0})'.format(repr(self.label))
            def is_leaf(self):
                return not self.branches


The `Tree` class can represent, for instance, the values computed in an
expression tree for the recursive implementation of `fib`, the function for
computing Fibonacci numbers. The function `fib_tree(n)` below returns a `Tree`
that has the nth Fibonacci number as its `label` and a trace of all previously
computed Fibonacci numbers within its branches.


​    

    >>> def fib_tree(n):
            if n == 1:
                return Tree(0)
            elif n == 2:
                return Tree(1)
            else:
                left = fib_tree(n-2)
                right = fib_tree(n-1)
                return Tree(left.label + right.label, (left, right))


​    
​    

    >>> fib_tree(5)
    Tree(3, (Tree(1, (Tree(0), Tree(1))), Tree(2, (Tree(1), Tree(1, (Tree(0), Tree(1)))))))


Trees represented in this way are also processed using recursive functions.
For example, we can sum the labels of a tree. As a base case, we return that
an empty branch has no labels.


​    

    >>> def sum_labels(t):
            """Sum the labels of a Tree instance, which may be None."""
            return t.label + sum([sum_labels(b) for b in t.branches])


​    
​    

    >>> sum_labels(fib_tree(5))
    10


We can also apply `memo` to construct a Fibonacci tree, where repeated
subtrees are only created once by the memoized version of `fib_tree`, but are
used multiple times as branches of different larger trees.


​    

    >>> fib_tree = memo(fib_tree)
    >>> big_fib_tree = fib_tree(35)
    >>> big_fib_tree.label
    5702887
    >>> big_fib_tree.branches[0] is big_fib_tree.branches[1].branches[1]
    True
    >>> sum_labels = memo(sum_labels)
    >>> sum_labels(big_fib_tree)
    142587180


The amount of computation time and memory saved by memoization in these cases
is substantial. Instead of creating 18,454,929 different instances of the
`Tree` class, we now create only 35.

### 2.9.3 Sets

In addition to the list, tuple, and dictionary, Python has a fourth built-in
container type called a `set`. Set literals follow the mathematical notation
of elements enclosed in braces. Duplicate elements are removed upon
construction. Sets are unordered collections, and so the printed ordering may
differ from the element ordering in the set literal.


​    

    >>> s = {3, 2, 1, 4, 4}
    >>> s
    {1, 2, 3, 4}


Python sets support a variety of operations, including membership tests,
length computation, and the standard set operations of union and intersection


​    

    >>> 3 in s
    True
    >>> len(s)
    4
    >>> s.union({1, 5})
    {1, 2, 3, 4, 5}
    >>> s.intersection({6, 5, 4, 3})
    {3, 4}


In addition to `union` and `intersection`, Python sets support several other
methods. The predicates `isdisjoint`, `issubset`, and `issuperset` provide set
comparison. Sets are mutable, and can be changed one element at a time using
`add`, `remove`, `discard`, and `pop`. Additional methods provide multi-
element mutations, such as `clear` and `update`. The Python [documentation for
sets](http://docs.python.org/py3k/library/stdtypes.html#set) should be
sufficiently intelligible at this point of the course to fill in the details.

**Implementing sets.** Abstractly, a set is a collection of distinct objects
that supports membership testing, union, intersection, and adjunction.
Adjoining an element and a set returns a new set that contains all of the
original set's elements along with the new element, if it is distinct. Union
and intersection return the set of elements that appear in either or both
sets, respectively. As with any data abstraction, we are free to implement any
functions over any representation of sets that provides this collection of
behaviors.

In the remainder of this section, we consider three different methods of
implementing sets that vary in their representation. We will characterize the
efficiency of these different representations by analyzing the order of growth
of set operations. We will use our `Link` and `Tree` classes from earlier in
this section, which allow for simple and elegant recursive solutions for
elementary set operations.

**Sets as unordered sequences.** One way to represent a set is as a sequence
in which no element appears more than once. The empty set is represented by
the empty sequence. Membership testing walks recursively through the list.


​    

    >>> def empty(s):
            return s is Link.empty


​    
​    

    >>> def set_contains(s, v):
            """Return True if and only if set s contains v."""
            if empty(s):
                return False
            elif s.first == v:
                return True
            else:
                return set_contains(s.rest, v)


​    
​    

    >>> s = Link(4, Link(1, Link(5)))
    >>> set_contains(s, 2)
    False
    >>> set_contains(s, 5)
    True


This implementation of `set_contains` requires $\Theta(n)$ time on average to
test membership of an element, where $n$ is the size of the set `s`. Using
this linear-time function for membership, we can adjoin an element to a set,
also in linear time.


​    

    >>> def adjoin_set(s, v):
            """Return a set containing all elements of s and element v."""
            if set_contains(s, v):
                return s
            else:
                return Link(v, s)


​    
​    

    >>> t = adjoin_set(s, 2)
    >>> t
    Link(2, Link(4, Link(1, Link(5))))


In designing a representation, one of the issues with which we should be
concerned is efficiency. Intersecting two sets `set1` and `set2` also requires
membership testing, but this time each element of `set1` must be tested for
membership in `set2`, leading to a quadratic order of growth in the number of
steps, $\Theta(n^2)$, for two sets of size $n$.


​    

    >>> def intersect_set(set1, set2):
            """Return a set containing all elements common to set1 and set2."""
            return keep_if_link(set1, lambda v: set_contains(set2, v))


​    
​    

    >>> intersect_set(t, apply_to_all_link(s, square))
    Link(4, Link(1))


When computing the union of two sets, we must be careful not to include any
element twice. The `union_set` function also requires a linear number of
membership tests, creating a process that also includes $\Theta(n^2)$ steps.


​    

    >>> def union_set(set1, set2):
            """Return a set containing all elements either in set1 or set2."""
            set1_not_set2 = keep_if_link(set1, lambda v: not set_contains(set2, v))
            return extend_link(set1_not_set2, set2)


​    
​    

    >>> union_set(t, s)
    Link(2, Link(4, Link(1, Link(5))))


**Sets as ordered sequences.** One way to speed up our set operations is to
change the representation so that the set elements are listed in increasing
order. To do this, we need some way to compare two objects so that we can say
which is bigger. In Python, many different types of objects can be compared
using `<` and `>` operators, but we will concentrate on numbers in this
example. We will represent a set of numbers by listing its elements in
increasing order.

One advantage of ordering shows up in `set_contains`: In checking for the
presence of an object, we no longer have to scan the entire set. If we reach a
set element that is larger than the item we are looking for, then we know that
the item is not in the set:


​    

    >>> def set_contains(s, v):
            if empty(s) or s.first > v:
                return False
            elif s.first == v:
                return True
            else:
                return set_contains(s.rest, v)


​    
​    

    >>> u = Link(1, Link(4, Link(5)))
    >>> set_contains(u, 0)
    False
    >>> set_contains(u, 4)
    True


How many steps does this save? In the worst case, the item we are looking for
may be the largest one in the set, so the number of steps is the same as for
the unordered representation. On the other hand, if we search for items of
many different sizes we can expect that sometimes we will be able to stop
searching at a point near the beginning of the list and that other times we
will still need to examine most of the list. On average we should expect to
have to examine about half of the items in the set. Thus, the average number
of steps required will be about $\frac{n}{2}$. This is still $\Theta(n)$
growth, but it does save us some time in practice over the previous
implementation.

We can obtain a more impressive speedup by re-implementing `intersect_set`. In
the unordered representation, this operation required $\Theta(n^2)$ steps
because we performed a complete scan of `set2` for each element of `set1`. But
with the ordered representation, we can use a more clever method. We iterate
through both sets simultaneously, tracking an element `e1` in `set1` and `e2`
in `set2`. When `e1` and `e2` are equal, we include that element in the
intersection.

Suppose, however, that `e1` is less than `e2`. Since `e2` is smaller than the
remaining elements of `set2`, we can immediately conclude that `e1` cannot
appear anywhere in the remainder of `set2` and hence is not in the
intersection. Thus, we no longer need to consider `e1`; we discard it and
proceed to the next element of `set1`. Similar logic advances through the
elements of `set2` when `e2 < e1`. Here is the function:


​    

    >>> def intersect_set(set1, set2):
            if empty(set1) or empty(set2):
                return Link.empty
            else:
                e1, e2 = set1.first, set2.first
                if e1 == e2:
                    return Link(e1, intersect_set(set1.rest, set2.rest))
                elif e1 < e2:
                    return intersect_set(set1.rest, set2)
                elif e2 < e1:
                    return intersect_set(set1, set2.rest)


​    
​    

    >>> intersect_set(s, s.rest)
    Link(4, Link(5))


To estimate the number of steps required by this process, observe that in each
step we shrink the size of at least one of the sets. Thus, the number of steps
required is at most the sum of the sizes of `set1` and `set2`, rather than the
product of the sizes, as with the unordered representation. This is
$\Theta(n)$ growth rather than $\Theta(n^2)$ \-- a considerable speedup, even
for sets of moderate size. For example, the intersection of two sets of size
100 will take around 200 steps, rather than 10,000 for the unordered
representation.

Adjunction and union for sets represented as ordered sequences can also be
computed in linear time. These implementations are left as an exercise.

**Sets as binary search trees.** We can do better than the ordered-list
representation by arranging the set elements in the form of a tree with
exactly two branches. The `entry` of the root of the tree holds one element of
the set. The entries within the `left` branch include all elements smaller
than the one at the root. Entries in the `right` branch include all elements
greater than the one at the root. The figure below shows some trees that
represent the set `{1, 3, 5, 7, 9, 11}`. The same set may be represented by a
tree in a number of different ways. In all binary search trees, all elements
in the `left` branch be smaller than the `entry` at the root, and that all
elements in the `right` subtree be larger.

![](http://www.composingprograms.com/img/set_trees.png)

The advantage of the tree representation is this: Suppose we want to check
whether a value `v` is contained in a set. We begin by comparing `v` with
`entry`. If `v` is less than this, we know that we need only search the `left`
subtree; if `v` is greater, we need only search the `right` subtree. Now, if
the tree is "balanced," each of these subtrees will be about half the size of
the original. Thus, in one step we have reduced the problem of searching a
tree of size $n$ to searching a tree of size $\frac{n}{2}$. Since the size of
the tree is halved at each step, we should expect that the number of steps
needed to search a tree grows as $\Theta(\log n)$. For large sets, this will
be a significant speedup over the previous representations. This
`set_contains` function exploits the ordering structure of the tree-structured
set.


​    

    >>> def set_contains(s, v):
            if s is None:
                return False
            elif s.entry == v:
                return True
            elif s.entry < v:
                return set_contains(s.right, v)
            elif s.entry > v:
                return set_contains(s.left, v)


Adjoining an item to a set is implemented similarly and also requires
$\Theta(\log n)$ steps. To adjoin a value `v`, we compare `v` with `entry` to
determine whether `v` should be added to the `right` or to the `left` branch,
and having adjoined `v` to the appropriate branch we piece this newly
constructed branch together with the original `entry` and the other branch. If
`v` is equal to the `entry`, we just return the node. If we are asked to
adjoin `v` to an empty tree, we generate a `Tree` that has `v` as the `entry`
and empty `right` and `left` branches. Here is the function:


​    

    >>> def adjoin_set(s, v):
            if s is None:
                return Tree(v)
            elif s.entry == v:
                return s
            elif s.entry < v:
                return Tree(s.entry, s.left, adjoin_set(s.right, v))
            elif s.entry > v:
                return Tree(s.entry, adjoin_set(s.left, v), s.right)


​    
​    

    >>> adjoin_set(adjoin_set(adjoin_set(None, 2), 3), 1)
    Tree(2, Tree(1), Tree(3))


Our claim that searching the tree can be performed in a logarithmic number of
steps rests on the assumption that the tree is "balanced," i.e., that the left
and the right subtree of every tree have approximately the same number of
elements, so that each subtree contains about half the elements of its parent.
But how can we be certain that the trees we construct will be balanced? Even
if we start with a balanced tree, adding elements with `adjoin_set` may
produce an unbalanced result. Since the position of a newly adjoined element
depends on how the element compares with the items already in the set, we can
expect that if we add elements "randomly" the tree will tend to be balanced on
the average.

But this is not a guarantee. For example, if we start with an empty set and
adjoin the numbers 1 through 7 in sequence we end up with a highly unbalanced
tree in which all the left subtrees are empty, so it has no advantage over a
simple ordered list. One way to solve this problem is to define an operation
that transforms an arbitrary tree into a balanced tree with the same elements.
We can perform this transformation after every few `adjoin_set` operations to
keep our set in balance.

Intersection and union operations can be performed on tree-structured sets in
linear time by converting them to ordered lists and back. The details are left
as an exercise.

**Python set implementation.** The `set` type that is built into Python does
not use any of these representations internally. Instead, Python uses a
representation that gives constant-time membership tests and adjoin operations
based on a technique called _hashing_ , which is a topic for another course.
Built-in Python sets cannot contain mutable data types, such as lists,
dictionaries, or other sets. To allow for nested sets, Python also includes a
built-in immutable `frozenset` class that shares methods with the `set` class
but excludes mutation methods and operators.