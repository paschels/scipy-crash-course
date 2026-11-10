# Control flow and functions
Lecture notes from the scientific Python crash course, following
*A Gentle introduction to Python*.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

The primary goal of this lesson is threefold:

* To assess what you remember from the previous lesson (self-assessment)
* To control the execution of a program with conditionals and loops
* To define and call your own functions

**Note**: From now on, programs get longer than one line. Typing them in the
          interpreter quickly becomes tedious: write them in a script with your
          text editor and run them with `python script.py`, as in the previous
          lesson.



Before you start
-------------------------------------------------------------------------------

Open a shell (*Terminal* on Linux and macOS, *Miniforge Prompt* on Windows),
activate the course environment and go to your course folder:

``` bash
(base) $ conda activate sci
(sci) $ cd Documents/python-course
```

In this lesson, some examples span several lines. When you type them in the
interpreter, Python displays the secondary prompt `...` to tell you the block
is not finished. **Press `Enter` on an empty line** to end the block and run
it:

``` python
>>> if 2 > 1:
...     print("yes")
...
yes
```



Self-assessment
-------------------------------------------------------------------------------

Before going further, take 15 minutes to check where you stand. Be honest with
yourself: the goal is not to get a grade, but to know what to review.

### Check your environment

Each of these commands must work without error:

``` bash
(sci) $ python --version
Python 3.12.7
(sci) $ python -c "import numpy, scipy, matplotlib"
(sci) $
```

If the prompt does not start with `(sci)`, or if you get
`ModuleNotFoundError`, go back to the installation lesson.

### Check your knowledge

Answer on a sheet of paper first, **then** check your answers in the
interpreter, and finally click on *Answer*.

**1.** What are the results of `7 // 2` and `7 % 2`?

<details><summary>Answer</summary>

`3` (integer division) and `1` (remainder).
</details>

**2.** What is the type of `6 / 3`?

<details><summary>Answer</summary>

`float`: the result is `2.0`. The `/` operator always returns a float.
</details>

**3.** Why is `0.1 + 0.2 == 0.3` false?

<details><summary>Answer</summary>

Because `0.1`, `0.2` and `0.3` cannot be represented exactly in binary: they
are approximated, and `0.1 + 0.2` gives `0.30000000000000004`.
</details>

**4.** With `s = "Python"`, what are `s[1:4]`, `s[-1]` and `s[::-1]`?

<details><summary>Answer</summary>

`'yth'` (the end index is not included), `'n'` and `'nohtyP'`.
</details>

**5.** Which of these containers can be modified after creation: string,
tuple, list, set, dictionary?

<details><summary>Answer</summary>

Lists, sets and dictionaries. Strings and tuples are immutable.
</details>

**6.** What is `{1, 1, 2}`?

<details><summary>Answer</summary>

`{1, 2}`: a set only keeps unique elements.
</details>

**7.** How do you run the script `script.py` and stay in the interpreter
afterwards to inspect its variables?

<details><summary>Answer</summary>

`python -i script.py`, or `%run script.py` from IPython.
</details>

**8.** What is wrong here?

``` python
>>> d = {"a": 1}
>>> d["b"]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'b'
```

<details><summary>Answer</summary>

The dictionary has no key `"b"`.
</details>

**9.** And here?

``` python
>>> int("3.5")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: invalid literal for int() with base 10: '3.5'
```

<details><summary>Answer</summary>

`"3.5"` is not the text of an integer. Use `float("3.5")`, or
`int(float("3.5"))` to get `3`.
</details>

**10.** In an error message, which line should you read first?

<details><summary>Answer</summary>

The last one: it gives the type of the error and its explanation. The lines
above tell you where it happened.
</details>

**How did you do?** With 8 correct answers or more, you're ready. Otherwise,
review the corresponding sections of *A Gentle introduction to Python* before
continuing: this lesson builds on all of them.



Control flow
-------------------------------------------------------------------------------

Until now, Python executed our instructions one after the other, from top to
bottom. *Control flow* statements allow us to execute some instructions only
under some conditions (**conditionals**), or to repeat them (**loops**).

### Conditions

A condition is any expression whose value is `True` or `False`. You already
know the comparison operators:

``` python
>>> x = 3
>>> x == 3        # equal
True
>>> x != 3        # different
False
>>> x > 2         # also <, <=, >=
True
>>> 1 < x < 5     # comparisons can be chained
True
>>> "y" in "Python"   # membership test (strings, lists, tuples, sets, dict keys)
True
```

Conditions can be combined with `and`, `or` and `not`:

``` python
>>> x > 0 and x < 10
True
>>> not (x > 0)
False
```

Any value can be used as a condition. Zero, empty containers and `None` are
considered *false*; everything else is considered *true*:

``` python
>>> bool(0), bool(""), bool([]), bool(None)
(False, False, False, False)
>>> bool(42), bool("abc"), bool([0])
(True, True, True)
```

### if, elif, else

The `if` statement executes a block of code only if a condition is true:

``` python
temperature = 25

if temperature > 30:
    print("It's hot")
elif temperature > 15:
    print("It's mild")
else:
    print("It's cold")
```

``` bash
(sci) $ python weather.py
It's mild
```

Note the syntax carefully:

* each line `if`, `elif` and `else` ends with a colon `:`;
* the block that follows is **indented** (4 spaces): this indentation is what
  tells Python which instructions belong to the block. This is the semantic
  meaning of indentation announced in the previous lesson;
* `elif` (short for *else if*) and `else` are optional, and you can have as
  many `elif` as you want. Python executes **only the first** block whose
  condition is true.

The block ends where the indentation ends:

``` python
x = -2
if x < 0:
    print("negative")
    x = -x
print("absolute value:", x)   # not indented: always executed
```

**Common errors**

Forgetting the colon:

``` python
>>> if x > 2
  File "<stdin>", line 1
    if x > 2
            ^
SyntaxError: expected ':'
```

Using `=` (assignment) instead of `==` (comparison):

``` python
>>> if x = 3:
  File "<stdin>", line 1
    if x = 3:
       ^^^^^
SyntaxError: invalid syntax. Maybe you meant '==' or ':=' instead of '='?
```

Forgetting to indent the block, in a script `check.py`:

``` python
x = 5
if x > 3:
print("big")
```

``` bash
(sci) $ python check.py
  File "/home/user/python-course/check.py", line 3
    print("big")
    ^
IndentationError: expected an indented block after 'if' statement on line 2
```

### while loops

A `while` loop repeats a block **as long as** a condition is true:

``` python
n = 3
while n > 0:
    print(n)
    n = n - 1
print("Go!")
```

``` bash
(sci) $ python countdown.py
3
2
1
Go!
```

Make sure the condition eventually becomes false! If you forget the line
`n = n - 1`, the loop never ends. In such a case, press `Control-C` to
interrupt the program: Python stops and displays a `KeyboardInterrupt`
message.

> **Tip**: `n = n - 1` can be abbreviated `n -= 1`. Similarly, there are `+=`,
> `*=`, `/=`...

### for loops

A `for` loop repeats a block **for each element** of a sequence (list, tuple,
string, set, dictionary...):

``` python
>>> for fruit in ["apple", "pear", "plum"]:
...     print(fruit)
...
apple
pear
plum
>>> for letter in "abc":
...     print(letter)
...
a
b
c
```

To repeat a block a given number of times, or to loop over numbers, use
`range`:

``` python
>>> for i in range(3):
...     print(i)
...
0
1
2
```

`range(stop)` goes from `0` to `stop` **excluded**, `range(start, stop)` from
`start` to `stop` excluded, and `range(start, stop, step)` uses a step. Like
slices! To see its content, convert it into a list:

``` python
>>> list(range(5))
[0, 1, 2, 3, 4]
>>> list(range(2, 10, 3))
[2, 5, 8]
```

Two useful helpers: `enumerate` gives the index together with the element,
and `zip` walks through several sequences at the same time.

``` python
>>> for i, letter in enumerate("abc"):
...     print(i, letter)
...
0 a
1 b
2 c
>>> for name, age in zip(["Alice", "Bob"], [31, 25]):
...     print(name, "is", age)
...
Alice is 31
Bob is 25
```

Looping over a dictionary gives its keys; use `.items()` to get keys and
values:

``` python
>>> ages = {"Alice": 31, "Bob": 25}
>>> for name, age in ages.items():
...     print(name, age)
...
Alice 31
Bob 25
```

### Accumulating results

A very common pattern is to build a result step by step inside a loop:

``` python
total = 0
for number in [3, 1, 4, 1, 5]:
    total += number
print("Total:", total)

squares = []
for i in range(5):
    squares.append(i**2)
print("Squares:", squares)
```

``` bash
(sci) $ python accumulate.py
Total: 14
Squares: [0, 1, 4, 9, 16]
```

### break and continue

`break` leaves the loop immediately; `continue` skips the rest of the block
and goes to the next iteration.

``` python
>>> for i in range(10):
...     if i == 4:
...         break
...     print(i)
...
0
1
2
3
>>> for i in range(5):
...     if i % 2 == 0:
...         continue
...     print(i)
...
1
3
```

Note the two levels of indentation: the `if` block is inside the `for` block.

### Going further: list comprehensions

You'll often meet a compact way to build a list from a loop, called a *list
comprehension*. These two codes are equivalent:

``` python
>>> squares = []
>>> for i in range(5):
...     squares.append(i**2)
...
>>> squares = [i**2 for i in range(5)]
>>> squares
[0, 1, 4, 9, 16]
```

A condition can be added: `[i for i in range(10) if i % 2 == 0]` gives the
even numbers. Use comprehensions for simple cases only; when in doubt, a
regular loop is easier to read.



Functions
-------------------------------------------------------------------------------

As soon as you write the same code twice, it's time to put it in a
**function**: a named block of code that you can call as many times as you
want, with different values. You already used many functions: `print`,
`len`, `range`, `float`...

### Defining and calling a function

``` python
def area(width, height):
    """Return the area of a rectangle."""
    return width * height

print(area(2, 3))
print(area(10, 5))
```

``` bash
(sci) $ python functions.py
6
50
```

Let's decompose:

* `def` starts the definition, followed by the function **name** and its
  **parameters** between parentheses, then a colon;
* the indented block is the **body** of the function;
* the string on the first line is the **docstring**: it documents the
  function. Type `help(area)` in the interpreter to display it;
* `return` gives the result back to the caller and ends the function.

Defining a function does not execute it: its body runs only when the function
is **called**, with parentheses: `area(2, 3)`. Without parentheses, you just
get the function itself:

``` python
>>> area
<function area at 0x...>
```

In a script, a function must be defined **before** it is called, otherwise:

``` bash
(sci) $ python functions.py
Traceback (most recent call last):
  File "/home/user/python-course/functions.py", line 1, in <module>
    print(double(2))
          ^^^^^^
NameError: name 'double' is not defined
```

### return is not print

This is the most common confusion for beginners. `print` **displays** a value
on the screen; `return` **gives back** a value that you can store and reuse.
A function without `return` returns the special value `None`:

``` python
>>> def show(x):
...     print(x)
...
>>> def give(x):
...     return x
...
>>> result = show(3)
3
>>> print(result)
None
>>> result = give(3)
>>> print(result * 2)
6
```

In the interpreter, both seem to display the value, which makes the
difference hard to see. Rule of thumb: **a function should return its
result**, and the caller decides whether to print it.

### Parameters

Parameters can have **default values**, used when the argument is omitted:

``` python
>>> def greet(name, greeting="Hello"):
...     return greeting + ", " + name + "!"
...
>>> greet("Alice")
'Hello, Alice!'
>>> greet("Alice", "Good morning")
'Good morning, Alice!'
```

Arguments can also be passed **by name** (keyword arguments), in any order.
This makes calls easier to read:

``` python
>>> greet(greeting="Hi", name="Bob")
'Hi, Bob!'
```

Calling a function with the wrong number of arguments raises a `TypeError`:

``` python
>>> area(2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: area() missing 1 required positional argument: 'height'
```

### Returning several values

A function can return several values, separated by commas. They are actually
returned as a tuple, which you can unpack:

``` python
>>> def min_max(numbers):
...     return min(numbers), max(numbers)
...
>>> low, high = min_max([3, 1, 4, 1, 5])
>>> low, high
(1, 5)
```

### Local variables

Variables created inside a function are **local**: they only exist while the
function runs, and are not visible outside.

``` python
>>> def f():
...     y = 1
...     return y + 1
...
>>> f()
2
>>> y
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'y' is not defined
```

This is a good thing: functions don't interfere with each other or with the
rest of your program. To get a value out of a function, **return** it.

### Putting it all together

Functions, conditionals and loops combine naturally:

``` python
def is_even(n):
    """Return True if n is even, False otherwise."""
    return n % 2 == 0


def count_even(numbers):
    """Return how many numbers in the list are even."""
    count = 0
    for n in numbers:
        if is_even(n):
            count += 1
    return count


print(count_even([3, 8, 1, 4, 6]))
```

``` bash
(sci) $ python even.py
3
```



Exercises
-------------------------------------------------------------------------------

Write each exercise in a script, run it, and only then compare with the
answer.

### Conditionals and loops

**1.** What does this code display for `x = 7`, `x = 15` and `x = 30`?

``` python
if x > 10:
    print("A")
elif x > 20:
    print("B")
else:
    print("C")
```

<details><summary>Answer</summary>

`C`, `A` and `A`. For `x = 30`, the first condition is already true, so the
`elif` is never tested: Python only executes the first matching block. To get
`B`, the conditions must be tested from the largest to the smallest.
</details>

**2.** Compute the sum of all integers from 1 to 100 with a loop.

<details><summary>Answer</summary>

``` python
total = 0
for i in range(1, 101):
    total += i
print(total)   # 5050
```

Note `range(1, 101)`: the end is excluded. (Shortcut: `sum(range(1, 101))`.)
</details>

**3.** Count the vowels in the word `"programming"`.

<details><summary>Answer</summary>

``` python
count = 0
for letter in "programming":
    if letter in "aeiouy":
        count += 1
print(count)   # 3
```
</details>

**4.** Display the multiplication table of 7, from `7 x 1 = 7` to
`7 x 10 = 70`.

<details><summary>Answer</summary>

``` python
for i in range(1, 11):
    print(7, "x", i, "=", 7 * i)
```
</details>

**5.** Starting from `n = 1`, double `n` until it is greater than 1000. How
many doublings are needed?

<details><summary>Answer</summary>

``` python
n = 1
steps = 0
while n <= 1000:
    n *= 2
    steps += 1
print(steps, n)   # 10 1024
```

The number of iterations is not known in advance: this is a job for `while`.
</details>

**6.** This script never stops. Why? (Stop it with `Control-C`.)

``` python
i = 0
while i < 5:
    print(i)
i += 1
```

<details><summary>Answer</summary>

`i += 1` is not indented, so it is not part of the loop: `i` stays `0`
forever and the condition is always true. Fix: indent `i += 1` by 4 spaces.
</details>

### Functions

**7.** Write a function `celsius_to_fahrenheit(c)` returning the temperature
in Fahrenheit (`F = C × 9/5 + 32`). Check that `celsius_to_fahrenheit(100)`
gives `212.0`.

<details><summary>Answer</summary>

``` python
def celsius_to_fahrenheit(c):
    """Convert a temperature from Celsius to Fahrenheit."""
    return c * 9 / 5 + 32

print(celsius_to_fahrenheit(100))   # 212.0
```
</details>

**8.** Write a function `maximum(numbers)` returning the largest element of
a list, **without** using the built-in `max` function.

<details><summary>Answer</summary>

``` python
def maximum(numbers):
    """Return the largest element of a non-empty list."""
    largest = numbers[0]
    for n in numbers:
        if n > largest:
            largest = n
    return largest

print(maximum([3, 8, 1, 4]))   # 8
```
</details>

**9.** Write a function `is_palindrome(word)` returning `True` if the word
reads the same in both directions (e.g. `"radar"`).

<details><summary>Answer</summary>

``` python
def is_palindrome(word):
    """Return True if word is a palindrome."""
    return word == word[::-1]

print(is_palindrome("radar"))   # True
print(is_palindrome("python"))  # False
```

Remember slicing with a negative step from the previous lesson!
</details>

**10.** What does this script display, and why?

``` python
def double(x):
    print(2 * x)

y = double(5)
print(y + 1)
```

<details><summary>Answer</summary>

It displays `10`, then fails:

``` python
TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'
```

`double` **prints** its result but does not **return** it, so `y` is `None`.
Fix: replace `print(2 * x)` with `return 2 * x`.
</details>

**11.** Read this error message. What is the problem?

``` bash
(sci) $ python script.py
Traceback (most recent call last):
  File "/home/user/python-course/script.py", line 4, in <module>
    print(f(3, 4))
          ^^^^^^^
TypeError: f() takes 1 positional argument but 2 were given
```

<details><summary>Answer</summary>

At line 4, the function `f` is called with two arguments (`3` and `4`), but
it was defined with only one parameter, e.g. `def f(x):`. Either call it with
one argument, or add a parameter to its definition.
</details>

**12.** Write a function `stats(numbers)` returning the mean and the
standard deviation of a list of numbers, using only loops (no numpy). Test it
on `[2, 4, 4, 4, 5, 5, 7, 9]`.

<details><summary>Answer</summary>

``` python
def stats(numbers):
    """Return the mean and the (population) standard deviation."""
    n = len(numbers)
    total = 0
    for x in numbers:
        total += x
    mean = total / n
    squares = 0
    for x in numbers:
        squares += (x - mean) ** 2
    std = (squares / n) ** 0.5
    return mean, std

m, s = stats([2, 4, 4, 4, 5, 5, 7, 9])
print(m, s)   # 5.0 2.0
```

In a later lesson, numpy will compute this in one line: `numpy.mean` and
`numpy.std`.
</details>



Resources
-------------------------------------------------------------------------------

Here are a set of tutorials to practice and go further on these topics.

* [**The (official) Python tutorial: More Control Flow Tools**](https://docs.python.org/3/tutorial/controlflow.html)
  covers `if`, `for`, `range`, `break`, `continue` and the definition of
  functions, including all the ways to pass arguments.

* [**Python Tutor**](https://pythontutor.com) lets you run code step by step
  in your browser and **see** what happens at each line: the values of the
  variables, the loops, the function calls. Very useful to understand loops
  and functions, and to find why a program does not do what you expect.

* [**Programming with Python**](https://swcarpentry.github.io/python-novice-inflammation/)
  by the Software Carpentry is a lesson for researchers, using real scientific
  data, with chapters on loops, conditionals and functions.

* [**Think Python**](https://allendowney.github.io/ThinkPython/) by Allen
  Downey is a free online book introducing programming with Python, step by
  step, with many exercises.

* [**Scientific Python Lectures**](https://lectures.scientific-python.org)
  include a chapter on the Python language (control flow, functions) as part
  of their introduction to the scientific Python ecosystem.
