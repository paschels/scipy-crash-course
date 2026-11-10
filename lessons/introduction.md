# A Gentle introduction to Python
**Pascal Helson** - [pascal.helson@u-bordeaux.fr](mailto:pascal.helson@u-bordeaux.fr)  
Lecture notes from the scientific Python crash course taught by Nicolas Rougier at the [University of Bordeaux] for the academic year 2021,
adapted in 2026 by Pascal Helson (Miniforge, system text editor, reading error messages).  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

The primary goal of this lesson is twofold:

* To discover Python basic syntax through the interpreter
* To learn how to read Python error messages

**Note**: This lesson only covers the very basics of Python. If you're already
          familiar with Python, you can probably skip it and target the next
          lesson, but who knows, you might discover some tips while reading
          this lesson.



Before you start
-------------------------------------------------------------------------------

This lesson assumes you followed the previous lesson: Miniforge is installed
and the `sci` environment has been created and tested.

Open a shell:

* **Linux and macOS**: the *Terminal* application.
* **Windows**: the *Miniforge Prompt* (Start menu, type "Miniforge").

Then activate the course environment. The prompt should now start with
`(sci)`:

``` bash
(base) $ conda activate sci
(sci) $
```

**Don't forget this step each time you open a new shell**: without it, you
would use another Python, without the packages of the course.

You can now start the Python interpreter:

``` bash
(sci) $ python
Python 3.12.7 | packaged by conda-forge | ...
Type "help", "copyright", "credits" or "license" for more information.
>>> |
```

or the [IPython] interpreter, which is far more powerful than the vanilla
interpreter:

``` bash
(sci) $ ipython
Python 3.12.7 | packaged by conda-forge | ...
Type 'copyright', 'credits' or 'license' for more information
IPython 8.29.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: |
```

In the following examples, the `>>>` is the prompt and does not need to be
typed. If you chose the IPython interpreter, your prompt is more likely
something like `In [12]:`. There is also a second prompt (`...`) meaning the
previous line is not ended and needs to be terminated. This is for example the
case when you enter an unbalanced expression (i.e. the number of opening
parentheses is greater than the number of closing parentheses).

To leave the interpreter, type `exit()`.


Python is a calculator
-------------------------------------------------------------------------------

### Arithmetic computation

Now it's time to experiment a little bit with Python. Let's start with simple
arithmetic operations because Python can be used as a regular calculator with
standard arithmetic operations (addition, subtraction, multiplication,
division, etc.)

Addition
``` python
>>> 2 + 3
5
```

Subtraction
``` python
>>> 11 - 3
8
```

Multiplication
``` python
>>> 3 * 4
12
```

Division
``` python
>>> 11 / 5
2.2
```

Integer division
``` python
>>> 11 // 5
2
```

Modulo operation
``` python
>>> 11 % 5
1
```

Power
``` python
>>> 2**3
8
```

Note that you cannot have spaces between digits of a number:

``` python
>>> 1 0 + 2
  File "<stdin>", line 1
    1 0 + 2
      ^
SyntaxError: invalid syntax
```

In such a case, Python complains about a syntax error and points at the
position of the error in the expression (using the `^` character). Why does
Python point at the zero and not at the space? Because you could have written
`1 + 2` and the space would have been legal. The interpreter can only find the
error after it discovers the extra digit and consequently points at it when
reporting the error. This is your first error message: we'll learn how to read
them in the [Reading error messages](#reading-error-messages) section.

Of course, you can compose any number of operations in order to compute a more
complex operation:

``` python
>>> 11 - (5 * (11//5)) #  = 11 % 5
1
```


###  Native numeric types

Python natively offers four main numeric types: `bool`, `int`, `float` and
`complex`. But always keep in mind that they are the poor man's version of
their mathematical equivalents (Boolean ($\mathbb{B}$), Integer
($\mathbb{Z}$), Real ($\mathbb{R}$) and Complex ($\mathbb{C}$)). Python
integers can be as large as your memory allows, but `float` and `complex`
have limited range and precision.

``` python
>>> 2**100
1267650600228229401496703205376
```

In the case of `float` and `complex`, this has very important consequences.

``` python
>>> 0.3 == 0.1*3
False
>>> 0.5 == 0.1*5
True
>>> 0.1*3
0.30000000000000004
```

The reason is that the decimal number `0.1` cannot be represented exactly and
is only approximated[^1]. On most machines, if Python were to print the true
decimal value of the binary approximation stored for 0.1, it would have to
display

``` python
>>> 0.1
0.1000000000000000055511151231257827021181583404541015625
```

Not very convenient... Consequently, Python (as many other languages) chose to
display a rounded value instead. If you want to know more, have a look at the
[Floating Point Arithmetic: Issues and Limitations](https://docs.python.org/3/tutorial/floatingpoint.html)
chapter in the official
[Python 3 tutorial](https://docs.python.org/3/tutorial/index.html). An
immediate and practical consequence is that what you see in the console is not
always what you get in memory, even if they're reasonably close.

[^1]: *What Every Computer Scientist Should Know About Floating-Point Arithmetic*  
      David Goldberg, Computing Surveys, 1991.


For each type, there exist many ways to specify the same number.

``` python
>>> True              # Boolean
>>> 0b1010            # Integer (base  2: binary)
>>> 0o12              # Integer (base  8: octal)
>>> 10                # Integer (base 10: decimal)
>>> 0x0a              # Integer (base 16: hexadecimal)
>>> 10.0              # Float
>>> 1e1               # Float (scientific notation)
>>> float('inf')      # Float (infinity +∞)
>>> float('nan')      # Float (Not A Number: nan)
>>> 10 + 0j           # Complex
```

But you can also force the type of any quantity by casting it.

``` python
>>> bool(0)
False
>>> int(0)
0
>>> float(0)
0.0
>>> complex(0)
0j
```


### Beyond simple arithmetic

If you want to use more elaborate functions, you'll need the help of the
[mathematical module](https://docs.python.org/3/library/math.html) for real
numbers and the
[complex mathematical module](https://docs.python.org/3/library/cmath.html) for
complex numbers.

**Power and logarithmic functions** 

``` python
>>> from math import *
>>> log(exp(1.234))
1.234
```

**Trigonometric functions**

``` python
>>> from math import *
>>> asin(sin(1.234))
1.234
```

**Hyperbolic functions**

``` python
>>> from math import *
>>> asinh(sinh(1.234))
1.234
```

**Special functions**

``` python
>>> from math import *
>>> gamma(2.0)
1.0
```

**Constants**

``` python
>>> from math import *
>>> pi
3.141592653589793
>>> e
2.718281828459045
>>> nan
nan
>>> inf
inf
```


### Logical operations

Logic is an important part of Python because it allows you to manipulate and
compare quantities, including numbers, and we'll see later that it works for
all kinds of objects.

``` python
>>> True and True # Logical and
>>> 42 or 57      # Logical or
>>> 1 == 2-1      # Equality test
>>> 1 != 2        # Inequality test
>>> not 24        # Negation
```

There is also an identity test, `is`, which is **not** a test for equality: it
checks whether two names refer to the very same object.

``` python
>>> a = [1, 2]
>>> b = [1, 2]
>>> a == b
True
>>> a is b
False
```

If you use `is` with a literal number, Python even warns you that you probably
meant `==`:

``` python
>>> 1 is 1.0
<stdin>:1: SyntaxWarning: "is" with 'int' literal. Did you mean "=="?
False
```

A *warning* is not an error: Python still computes the result, but tells you
that something looks suspicious.


### Bitwise operations

Bitwise operations are logical operations that operate at the bit level. They
might be useful in some situations but we won't use them in this course.

``` python
>>> 1 | 2   # bitwise or
>>> 1 & 2   # bitwise and
>>> 1 ^ 2   # bitwise xor
>>> 8 << 2  # bitwise left shift
>>> 8 >> 2  # bitwise right shift
>>> ~8      # bitwise negation
```




Python is much more
-------------------------------------------------------------------------------

Besides being a convenient calculator, Python is also (and mostly) a powerful
programming language with an elegant and intuitive syntax. Furthermore, you
have to know that Python is an interpreted language, meaning each time you
enter a set of instructions, they need to be interpreted by the Python
interpreter. This can make Python quite slow in some situations but we'll see
later how to overcome most of Python's slowness.

### Variables

Until now, we have been playing in the console, throwing some expressions in
the interpreter and checking the result. The problem is that those expressions
cannot be re-used. It's thus time to save us some trouble and assign those
expressions to variables. This can be done quite naturally.

``` python
>>> width = 1
>>> height = 2
```

What is really cool though is that you can assign several variables at once:

``` python
>>> width, height = 2, 1
>>> width
2
>>> height
1
```

However, you cannot refer to a new variable on the same line (in a fresh
interpreter, where `width` does not exist yet):

``` python
>>> width, height = 2, 2*width
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'width' is not defined
```

In this case, you have to split the expression in two distinct lines.

``` python
>>> width = 2
>>> height = 2*width
```

Variables can be manipulated just as any expression but they need to have been
defined previously.

``` python
>>> a = 2*b
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'b' is not defined
```

In interactive mode, that is, the console mode we've been using from the start,
there is a special variable whose name is `_` and that contains the last
**printed** expression.

``` python
>>> very_long_name = 10
>>> very_long_name
10
>>> b = 2 + _ # here, _ = very_long_name
>>> b
12
```

Don't explicitly assign a value to the `_` variable or you'll kill the magic,
and you don't want to do that, do you?



### Containers

Besides the numeric types, Python also offers native container types (also
known as collections). One is dedicated to storing ordered sequences of
characters (i.e. strings) while the others allow to store just anything and
offer different properties. In a nutshell:

``` python
>>> "1,2,3,4"  # String (ordered, characters only, immutable, indexable)
>>> (1,2,3,4)  # Tuple (ordered, immutable, indexable)
>>> [1,2,3,4]  # List (ordered, mutable, indexable)
>>> {1,2,3,4}  # Set (unordered, mutable, unique elements)
>>> {1:2,3:4}  # Dictionary (insertion ordered, mutable, indexed by keys)
```

**Strings**

Strings are expressed by enclosing a text using pairs of `"` or `'`
characters. Depending on the character you chose, you can use the other inside
the string.

``` python
>>> "Hello 'world!'"
"Hello 'world!'"
>>> 'Hello "world"!'
'Hello "world"!'
>>> "" # empty string
''
```

Note that you can split a string using spaces and Python will concatenate all
the pieces together.

``` python
>>> "Hello"  " "  "world!"
'Hello world!'
```


But you can also explicitly concatenate the different pieces together.

``` python
>>> "Hello" + " " + "world!"
'Hello world!'
```

For multiline strings, you need to triple the enclosing quotes or to use
parentheses.

``` python
>>> """Hello
... world!"""
'Hello\nworld!'

>>> '''Hello
... world!'''
'Hello\nworld!'

>>> ("Hello"
... " world!")
'Hello world!'
```

In the output above, you can see the "\n" character has been added: it
expresses the newline character. There are several backslashed characters:

Escape Sequence | Meaning 
----------------|---------------------------
\\\\            | Backslash (\)
\\\'            | Single quote (')
\\\"            | Double quote (")
\a              | ASCII Bell (BEL)
\b              | ASCII Backspace (BS)
\n              | ASCII Linefeed (LF)
\r              | ASCII Carriage Return (CR)
\t              | ASCII Horizontal Tab (TAB)
\v              | ASCII Vertical Tab (VT)

This means the `\n` will be interpreted as a new line character, but what if we
want to really have `\n` in our string, as in a Windows path like
`C:\some\name`? Either we "escape" all the backslashes or we prefix the string
with `r` (for *raw*) meaning special characters won't be interpreted.

``` python
>>> print('C:\some\name')
<stdin>:1: SyntaxWarning: invalid escape sequence '\s'
C:\some
ame
>>> print('C:\\some\\name')
C:\some\name
>>> print(r'C:\some\name')
C:\some\name
```

In the first case, Python warns that `\s` is not a known escape sequence, and
`\n` has been turned into a new line.

Strings in Python 3 are [Unicode] strings, meaning you can encode pretty much
any glyph from any language (and emojis as well).

``` python
>>> "ℕ ⊂ ℤ ⊂ ℚ ⊂ ℝ ⊂ ℂ" 
'ℕ ⊂ ℤ ⊂ ℚ ⊂ ℝ ⊂ ℂ'
```

**Tuples**

Tuples are immutable containers, meaning they cannot be changed after they've
been created. They allow to store pretty much anything, including other tuples.
To create a tuple, you simply write a comma-separated list of values,
optionally enclosed by parentheses.

``` python
>>> 1,2,3
(1, 2, 3)
>>> 1, "2", (1,2) 
(1, '2', (1, 2))
>>> () # empty tuple
()
```

**Lists**

Lists are mutable containers quite similar to tuples, but they can be modified
after creation. They also allow to store pretty much anything. To create a
list, you need to write a comma-separated list of values enclosed by square
brackets.

``` python
>>> [1,2,3]
[1, 2, 3]

>>> [1, "2", (1,2)]
[1, '2', (1, 2)]

>>> [] # empty list
[]
```

**Sets**

Sets are mutable containers (they can be modified) and contain only unique
elements, i.e. they prevent duplicated elements.

``` python
>>> {1, 2, 2}
{1, 2}

>>> {1, 2, "2"}  # the display order may vary
{1, 2, '2'}
```

**Dictionaries**

A dictionary can be considered as a kind of associative memory where items are
indexed by a key (instead of an integer). The type of the key can be pretty
much anything immutable (numbers, strings, tuples...). Items are kept in the
order they were inserted.

``` python
>>> { "item 1" : 1, "item 2" : 2}
{'item 1': 1, 'item 2': 2}
>>> { 1 : 2, 3 : 4}
{1: 2, 3: 4}
```


### Indexing and slicing

Individual items of a list, tuple and string can be accessed individually
using their position as index. Note that the first element has index 0.

``` python
>>> d = [1,2,3,4,5]
>>> d[0]
1
```

This also works using negative indices, meaning the position has to be taken
from the end. Note that the last element has index -1.

``` python
>>> s = "Hello world!"
>>> s[-1]
'!'
```

Furthermore, we can also access a **range** of items using the slice notation
`start:end`. Note that both start and end are optional, and that the `end`
item is **not** included.

``` python
>>> d = [1,2,3,4,5]
>>> d[1:3]
[2, 3]
```

If start is missing, Python will implicitly replace it by the start of the
list. If end is missing, Python will implicitly replace it by the end of the
list.


``` python
>>> d = [1,2,3,4,5]

>>> d[1:]
[2, 3, 4, 5]

>>> d[:2]
[1, 2]

>>> d[:]
[1, 2, 3, 4, 5]
```

We can further refine our slice by giving the step between elements. The new
syntax is thus `start:end:step`.

``` python
>>> d = [1,2,3,4,5,6]
>>> d[0:5:2]
[1, 3, 5]

# Can be abbreviated into
>>> d[::2]
[1, 3, 5]
```

What if we use a negative step? We get the reversed sequence.

``` python
>>> d = [1,2,3,4,5,6]
>>> d[::-1] 
[6, 5, 4, 3, 2, 1]
```

Because strings are indexable, indexing and slicing work just the same.


``` python
>>> s = "Hello world!"
>>> s[6:-1]
'world'
>>> s[6:]
'world!'
```


### Adding and removing

Adding an element to a mutable container can be done in two distinct
ways. Either by creating a new container that is the container plus the new
item, or by inserting the new item into the container, hence modifying it.

``` python
>>> l1 = [1, 2, 3]
>>> l2 = l1 + [4]
>>> l2
[1, 2, 3, 4]
>>> l1.append(4)
>>> l1
[1, 2, 3, 4]
```

For removing items, we need to use the **del** keyword and give indices where
to delete items.

``` python
>>> l1 = [1, 2, 3, 4, 5, 6]
>>> del l1[0]
>>> l1
[2, 3, 4, 5, 6]
```

But we can also delete a range of indices at once.

``` python
>>> l1 = [1, 2, 3, 4, 5, 6]
>>> del l1[::2]
>>> l1
[2, 4, 6]
```



Running scripts
-------------------------------------------------------------------------------

Until now, we have been mostly typing some Python expressions directly into
the interpreter, meaning we had to type and retype the same expressions again
and again. To save us some time, it's time to put all these commands into a
file that will be run by the Python interpreter.

As in the previous lesson, open the text editor of your system (*Notepad* on
Windows, *TextEdit* in plain text mode on macOS, *Text Editor*, *gedit* or
*Kate* on Linux) and type:

``` python
width = 2
height = 3
print("Area:", width * height)
```

Then save it with the name `script.py` in your course folder (on Windows,
choose *"All files (\*.\*)"* as type so that the file is not saved as
`script.py.txt`). The `.py` is the regular file extension used for Python
programs. You are free to use any extension you like, but using `.py` is a
damn good idea since the operating system can make the connection with Python
based on this extension.

To run this script, go to your course folder in the shell (with the `cd`
command, as in the previous lesson) and start it using the regular Python
interpreter.

``` bash
(sci) $ python script.py
Area: 6
(sci) $
```

If you run the above command, Python will terminate as soon as your program
has ended. If you want to stay within the Python interpreter (for example to
inspect your variables), you'll have to use the `-i` switch (interactive mode)
that tells Python not to exit once the program has finished.

``` bash
(sci) $ python -i script.py
Area: 6
>>> width
2
```

Another option is to use [IPython], which allows you to run a script from
within the interpreter.

``` python
In [1]: %run script.py
Area: 6

In [2]: width
Out[2]: 2
```

### Indentation

Something you will discover soon enough (or maybe you've already discovered
it) is that Python is rather nitpicking about indentation.

``` python
>>> a = 1
>>>  a = 1
  File "<stdin>", line 1
    a = 1
IndentationError: unexpected indent
```

The reason is that indentation has a semantic meaning but we'll see that later.
When you indent, always use **4 spaces** and never tabulations.



Reading error messages
-------------------------------------------------------------------------------

You've already met a few error messages. Don't be afraid of them: they are
not a punishment, they are Python's way of telling you **what** went wrong and
**where**. Learning to read them is one of the most useful skills you can
acquire.

### Anatomy of an error message

Let's write a small script, `area.py`, with a mistake in it:

``` python
width = 10
height = "5"
area = width * height + 1
print("Area:", area)
```

and run it:

``` bash
(sci) $ python area.py
Traceback (most recent call last):
  File "/home/user/python-course/area.py", line 3, in <module>
    area = width * height + 1
           ~~~~~~~~~~~~~~~^~~
TypeError: can only concatenate str (not "int") to str
```

**Read it from the bottom up:**

1. **The last line is the most important one.** It gives the *type* of the
   error (`TypeError`) followed by a *message* explaining it: here, Python
   cannot add (*concatenate*) an integer to a string.
2. **The lines above tell you where** it happened: in the file `area.py`, at
   line 3, and they show the faulty line.
3. **The `^` and `~` characters** point at the exact operation that failed:
   the `+`.

Now, **think about the cause**, which is not always on the reported line.
Python complains at line 3, but the real mistake is at line 2: `height` is the
string `"5"`, not the number `5`. So `width * height` is the string `"5"`
repeated 10 times, and adding `1` to a string fails. The fix:
`height = 5`.

The first line, `Traceback (most recent call last)`, simply announces that
what follows is the path Python followed until the error. With longer
programs, there will be several `File ..., line ...` blocks: the last one is
where the error actually occurred.

### Syntax errors

Some errors are detected **before** your program even starts: Python cannot
understand what you wrote. There is no `Traceback` line in this case:

``` bash
(sci) $ python mean.py
  File "/home/user/python-course/mean.py", line 2
    print("Mean:", mean
         ^
SyntaxError: '(' was never closed
```

Here the file name and line number are given, the `^` points at the opening
parenthesis, and the message is explicit: it was never closed.

> **Tip**: with syntax errors, if the reported line looks correct, look at the
> line just **before**: a missing parenthesis or quote is often detected one
> line too late.

### Warnings are not errors

You've also met lines like:

``` python
<stdin>:1: SyntaxWarning: invalid escape sequence '\s'
```

A **warning** does not stop the program: Python tells you that something looks
suspicious, but still runs your code. Don't ignore warnings though, they
often reveal a bug.

### The most common errors

Error type | Typical cause
-----------|---------------------------------------------------------------
`SyntaxError` | Python cannot understand the code (missing parenthesis, quote, colon...)
`IndentationError` | Wrong indentation (unexpected spaces at the beginning of a line...)
`NameError` | A variable is used before being defined, or its name is misspelled
`TypeError` | An operation is applied to the wrong type (e.g. adding a string and a number)
`ValueError` | The type is right but the value is not (e.g. `float("abc")`)
`IndexError` | An index is out of the range of a list, tuple or string
`KeyError` | A key does not exist in a dictionary
`AttributeError` | An object does not have the requested method (e.g. `append` on a tuple)
`ZeroDivisionError` | Division by zero
`ModuleNotFoundError` | The module to import is not installed in the current environment, or its name is misspelled

When you don't understand a message, copy its **last line** into a search
engine: you'll almost always find an explanation.



Exercises
-------------------------------------------------------------------------------

We'll stop here with this very short introduction to Python. It's now time to
move on to programming with Python. So now, shut down your computer, grab a
pen and a sheet of paper and try to answer these exercises without typing them
in a Python interpreter. Once you've answered every question, you can check
them.

### Find the type of the following expressions

``` python
.0
```

``` python
-1
```

``` python
1,
```

``` python
'4.0 + 5.0'
```

``` python
1e2
```

``` python
1j
```

``` python
[{()}]
```

``` python
float('nan')
```


### Are these legal python expressions?

```python
1 + 1 == 2
```

```python
1 = 2
```

``` python
(1,)[0] 
```

``` python
1 + 1i
```

``` python
1 <- 2
```

``` python
0.+.0
```

``` python
3***3
```

``` python
3 <<2>> 3
```

``` python
[({})]
```


### Find the result of the following expressions

``` python
1.+1,
```

``` python
1,+1.
```

``` python
(1,)*3
```

``` python
1e1000 - 1e1000
```

``` python
'abc'*3
```

``` python
3 or 10
```

``` python
3 <2> 3
```


### Read the error messages

For each case below, read the error message, then answer: what is the
**type** of error, **where** does it happen, what is the **cause**, and how
would you **fix** it? Click on *Answer* to check.

**1.**

``` python
>>> print("Hello world!"
  File "<stdin>", line 1
    print("Hello world!"
         ^
SyntaxError: '(' was never closed
```

<details><summary>Answer</summary>

`SyntaxError`: the opening parenthesis of `print(` is never closed. Fix:
`print("Hello world!")`.
</details>

**2.**

``` python
>>> lenght = 5
>>> length * 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'length' is not defined. Did you mean: 'lenght'?
```

<details><summary>Answer</summary>

`NameError`: the variable was defined as `lenght` (typo) but used as
`length`. Python even suggests the right name. Fix: use the same spelling
everywhere (preferably the correct one, `length`).
</details>

**3.**

``` python
>>> "2" + 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only concatenate str (not "int") to str
```

<details><summary>Answer</summary>

`TypeError`: `"2"` is a string, `2` is an integer, and they cannot be added.
Fix: `int("2") + 2` (gives `4`) or `"2" + str(2)` (gives `'22'`), depending
on what you want.
</details>

**4.**

``` python
>>> d = [1, 2, 3]
>>> d[3]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
```

<details><summary>Answer</summary>

`IndexError`: the list has 3 elements, with indices 0, 1 and 2. Index 3 does
not exist. To get the last element, use `d[2]` or `d[-1]`.
</details>

**5.**

``` python
>>> float("1,2")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: could not convert string to float: '1,2'
```

<details><summary>Answer</summary>

`ValueError`: Python uses a dot as decimal separator, not a comma. Fix:
`float("1.2")`.
</details>

**6.**

``` python
>>> t = (1, 2, 3)
>>> t.append(4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'tuple' object has no attribute 'append'
```

<details><summary>Answer</summary>

`AttributeError`: a tuple is immutable, so it has no `append` method. Fix: use
a list, `t = [1, 2, 3]`, or create a new tuple, `t = t + (4,)`.
</details>

**7.**

``` python
>>> ages = {"alice": 31}
>>> ages["bob"]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'bob'
```

<details><summary>Answer</summary>

`KeyError`: the dictionary has no key `"bob"`. The message only gives the
missing key. Fix: add it first, e.g. `ages["bob"] = 25`.
</details>

**8.**

``` python
>>> s = "hello'
  File "<stdin>", line 1
    s = "hello'
        ^
SyntaxError: unterminated string literal (detected at line 1)
```

<details><summary>Answer</summary>

`SyntaxError`: the string starts with `"` but ends with `'`, so it is never
closed. Fix: use the same quote on both sides, `s = "hello"`.
</details>

**9.** You open a new shell, start `python` and type:

``` python
>>> import numpy
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'numpy'
```

But numpy was installed during the previous lesson!

<details><summary>Answer</summary>

`ModuleNotFoundError`: this Python does not know numpy. The prompt did not
show `(sci)`: the course environment was not activated, so another Python was
used. Fix: exit Python, type `conda activate sci`, then start `python` again.
</details>

**10.** The script `convert.py` contains:

``` python
celsius = input("Temperature in Celsius: ")
fahrenheit = celsius * 9 / 5 + 32
print(fahrenheit)
```

and running it gives:

``` bash
(sci) $ python convert.py
Temperature in Celsius: 20
Traceback (most recent call last):
  File "/home/user/python-course/convert.py", line 2, in <module>
    fahrenheit = celsius * 9 / 5 + 32
                 ~~~~~~~~~~~~^~~
TypeError: unsupported operand type(s) for /: 'str' and 'int'
```

<details><summary>Answer</summary>

`TypeError` at line 2, on the division. But the cause is at line 1: `input`
always returns a **string** (`"20"`), so `celsius * 9` gives the string
`"202020202020202020"`, which cannot be divided by 5. Fix: convert the input
into a number, `celsius = float(input("Temperature in Celsius: "))`.
</details>


### How to...

* Build the empty set?
* Get the list of unique letters composing "abracadabra"
* Build a tuple of two empty lists
* Generate a list of all even numbers between 20 and 40 (included)
* Check if a word is a palindrome?
* Transform the string "1.2" into a float?
* Count the number of unique elements in a list?
* Find the maximum representable integer?
* Print the string `"Isn't, he said"` (including quotes)?
* Swap the content of two variables?



Resources
-------------------------------------------------------------------------------

Here are a set of resources for those who want to go further in their knowledge
of Python.

* [**The (official) Python tutorial**](https://docs.python.org/3/tutorial/index.html)
  does not attempt to be comprehensive and cover every single feature, or even
  every commonly used feature. Instead, it introduces many of Python's most
  noteworthy features, and will give you a good idea of the language's flavor
  and style. Its chapter on
  [errors and exceptions](https://docs.python.org/3/tutorial/errors.html)
  complements the section on reading error messages.

* [**Dive into python**](http://www.diveintopython3.net) is a teach-by-example
  guide to the paradigms of programming in Python and modern software
  development techniques. It assumes some preexisting knowledge of programming,
  although not necessarily in Python.  
  
* [**Programming with Python**](https://github.com/vsego/python-lecture-notes) are
  the lecture notes from the course taught at the University of Manchester in
  the academic year 2014/15. The aim of the course is to lay a strong
  foundation for your future programming requirements, be they in Python or
  some other language or a similar tool.  

* [**Scientific Python Lectures**](https://lectures.scientific-python.org) (formerly
  Scipy Lecture Notes) are a set of tutorials on the scientific Python
  ecosystem: a quick introduction to central tools and techniques. The
  different chapters each correspond to a 1 to 2 hours course with increasing
  level of expertise, from beginner to expert.


<!----------------------------- External links ------------------------------->
[Python]:     http://www.python.org
[Numpy]:      http://www.numpy.org
[Scipy]:      http://www.scipy.org
[Pandas]:     http://pandas.pydata.org
[Matplotlib]: http://matplotlib.org
[IPython]:    http://ipython.org
[Jupyter]:    http://jupyter.org
[Git]:        https://git-scm.com
[OpenGL]:     https://www.opengl.org
[Glumpy]:     https://glumpy.github.io
[Bokeh]:      https://bokeh.org
[Cython]:     http://cython.org
[EDMI]:       http://www.math.u-bordeaux1.fr/ED/ecole_doctorale/
[University of Bordeaux]: http://www.u-bordeaux.com
[Unicode]:    https://en.wikipedia.org/wiki/Unicode
<!---------------------------------------------------------------------------->
