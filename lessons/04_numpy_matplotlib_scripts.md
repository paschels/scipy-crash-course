# Computing, visualizing and automating
**Pascal Helson** - [pascal.helson@u-bordeaux.fr](mailto:pascal.helson@u-bordeaux.fr)  
Lecture notes from the scientific Python crash course, following
*Control flow and functions*.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

The primary goal of this lesson is threefold:

* To compute efficiently on arrays of numbers with [numpy]
* To visualize data with [matplotlib]
* To automate a task with a script that processes many files

On the way, we'll install two interfaces that make this work more
comfortable: [Spyder] and [JupyterLab].

**Contents** (3.5 hours, including a break)

* [Two new tools: Spyder and JupyterLab](#two-new-tools-spyder-and-jupyterlab) (20 min)
* [Computing with numpy](#computing-with-numpy) (60 min)
* [Visualizing with matplotlib](#visualizing-with-matplotlib) (45 min)
* [Automating with scripts](#automating-with-scripts) (45 min)
* [Exercises](#exercises) (30 min)



Two new tools: Spyder and JupyterLab
-------------------------------------------------------------------------------

You now know how to write and run Python code with nothing more than a text
editor and a shell. An **IDE** (Integrated Development Environment) puts an
editor, a shell and many helpers (completion, variable explorer, debugger,
figures...) in a single window. But it does not contain any magic of its own:
**it asks the Python of your environment to run your code**, exactly like you
did with `python script.py`. This becomes really handy as soon as we draw
figures.

Open a shell (*Terminal* on Linux and macOS, *Miniforge Prompt* on Windows),
activate the course environment and install them (no administrator rights
needed):

``` bash
(base) $ conda activate sci
(sci) $ conda install spyder jupyterlab ipykernel spyder-kernels
(sci) $ python -m ipykernel install --user --name sci --display-name "sci"

...
Proceed ([y]/n)? y
```

> **Note**: the first time you start Spyder or JupyterLab, your system may show
> a firewall alert about `python`. Both work only on your own computer: you can
> click *Cancel* (or *Deny*), no administrator rights are needed.

### Spyder

Start Spyder from your shell:

``` bash
(sci) $ spyder
```

The window has three main parts: the **editor** on the left, the **IPython
console** at the bottom right, and on the top right the **variable explorer**
and the **plots** pane, where figures are displayed. Open a script from the
previous lessons (*File > Open*) and run it with the green *Run* button or the
`F5` key:

``` python
In [1]: runfile('.../python-course/script.py', wdir='.../python-course')
Area: 6
```

Look at the console: it is an **IPython** shell, the same one you used before.
Spyder simply ran your script in it for you, exactly like `%run script.py`.

### JupyterLab

Go to your course folder and start JupyterLab:

``` bash
(sci) $ cd Documents/python-course
(sci) $ jupyter lab
```

JupyterLab opens in your web browser, with your course folder in the file
browser on the left. Click on *Python 3 (ipykernel)* under *Notebook* to
create a notebook. A notebook is made of **cells**: type some code in a cell
and press `Shift+Enter` to run it. Results and figures appear just below the
cell, and you can add text cells (*Markdown*) to explain what you do.

``` python
[1]: print("Hello world!")
     Hello world!
[2]: %run script.py
     Area: 6
```

Each notebook is backed by an IPython *kernel*: once again, the same Python
runs your code. To stop JupyterLab, use *File > Shut Down*, or press
`Control-C` in the shell.

### One Python, several interfaces

To convince yourself that it is always the same Python, type these two lines
in the Python shell, in the Spyder console and in a notebook:

``` python
>>> import sys
>>> print(sys.executable)
/home/user/miniforge3/envs/sci/bin/python
```

You should get the same path everywhere (on Windows, something like
`C:\Users\name\miniforge3\envs\sci\python.exe`). Use whichever interface you
prefer for the rest of the lesson: Spyder is great for scripts, JupyterLab for
exploring data step by step. Everything also works from the shell.



Computing with numpy
-------------------------------------------------------------------------------

### Why numpy?

Python lists are flexible, but they are not made for computing:

``` python
>>> [1, 2, 3] * 2
[1, 2, 3, 1, 2, 3]
>>> [1, 2, 3] + 1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only concatenate list (not "int") to list
```

To multiply each element by 2, you would need a loop. [numpy] provides a new
type, the **array**, on which operations apply to every element at once, and
much faster than a loop. By convention, numpy is imported as `np`:

``` python
>>> import numpy as np
>>> a = np.array([1, 2, 3])
>>> a * 2
array([2, 4, 6])
>>> a + 1
array([2, 3, 4])
>>> a ** 2
array([1, 4, 9])
```

### Creating arrays

``` python
>>> np.array([1, 2, 3])        # from a list
array([1, 2, 3])
>>> np.arange(0, 10, 2)        # like range, but gives an array
array([0, 2, 4, 6, 8])
>>> np.linspace(0, 1, 5)       # 5 values evenly spaced from 0 to 1 (included)
array([0.  , 0.25, 0.5 , 0.75, 1.  ])
>>> np.zeros(3)
array([0., 0., 0.])
>>> np.ones((2, 3))            # 2 rows, 3 columns
array([[1., 1., 1.],
       [1., 1., 1.]])
```

Random numbers are obtained from a *generator*. Giving it a *seed* makes the
results reproducible (you'll get exactly the same numbers as below):

``` python
>>> rng = np.random.default_rng(42)
>>> rng.integers(0, 2, size=(3, 4))     # random integers 0 or 1
array([[0, 1, 1, 0],
       [0, 1, 0, 1],
       [0, 0, 1, 1]])
>>> rng.normal(0, 1, size=3).round(3)  # normal distribution, mean 0, std 1
array([ 0.128, -0.316, -0.017])
```

Unlike lists, all the elements of an array have the same type, called its
`dtype`:

``` python
>>> np.array([1, 2.5])
array([1. , 2.5])
>>> np.array([1, 2.5]).dtype
dtype('float64')
```

### Shape and dimensions

Arrays can have several dimensions. A 2D array is a table (or a matrix, or an
image):

``` python
>>> m = np.array([[1, 2, 3],
...               [4, 5, 6]])
>>> m.shape        # (rows, columns)
(2, 3)
>>> m.ndim         # number of dimensions
2
>>> m.size         # number of elements
6
>>> np.arange(6).reshape(2, 3)
array([[0, 1, 2],
       [3, 4, 5]])
```

### Indexing and slicing

Indexing and slicing work like for lists, with one index per dimension,
separated by commas:

``` python
>>> m[0, 1]        # row 0, column 1
np.int64(2)
>>> m[1]           # row 1
array([4, 5, 6])
>>> m[:, 0]        # all rows, column 0
array([1, 4])
>>> m[:, 1:]       # all rows, columns 1 to the end
array([[2, 3],
       [5, 6]])
```

`np.int64(2)` is simply the number 2, stored as a numpy integer.

### Boolean masks

Comparing an array gives an array of booleans, which can be used to **select**
elements. This is one of the most useful features of numpy:

``` python
>>> x = np.array([3, 8, 1, 4, 6])
>>> x > 3
array([False,  True, False,  True,  True])
>>> x[x > 3]                    # elements greater than 3
array([8, 4, 6])
>>> x[x > 3].size               # how many are greater than 3
3
>>> x[x > 3] = 0                # set them to 0
>>> x
array([3, 0, 1, 0, 0])
```

Conditions are combined with `&` (and), `|` (or) and `~` (not), with
parentheses: `x[(x > 2) & (x < 7)]`.

### Computing

numpy provides the usual mathematical functions, which apply to every element:

``` python
>>> np.sin(np.pi / 2)
np.float64(1.0)
>>> t = np.linspace(0, 2 * np.pi, 5)
>>> np.cos(t).round(3)
array([ 1.,  0., -1., -0.,  1.])
```

and many statistics. With 2D arrays, the `axis` argument tells along which
dimension to compute (`axis=0`: down the rows, giving one result per column;
`axis=1`: across the columns, giving one result per row):

``` python
>>> m = np.array([[1, 2, 3], [4, 5, 6]])
>>> m.sum()
np.int64(21)
>>> m.sum(axis=0)    # sum of each column
array([5, 7, 9])
>>> m.sum(axis=1)    # sum of each row
array([ 6, 15])
>>> m.mean(), m.min(), m.max()
(np.float64(3.5), np.int64(1), np.int64(6))
```

Remember the `stats` exercise of the previous lesson, with its two loops? With
numpy:

``` python
>>> data = np.array([2, 4, 4, 4, 5, 5, 7, 9])
>>> np.mean(data), np.std(data)
(np.float64(5.0), np.float64(2.0))
```

### Operations between arrays

Operations between two arrays of the same shape are done element by element:

``` python
>>> np.array([1, 2, 3]) + np.array([10, 20, 30])
array([11, 22, 33])
```

When shapes differ, numpy tries to **broadcast** them, i.e. to repeat the
smaller array to match the larger one. This works for example between a
2D array and a row:

``` python
>>> m + np.array([10, 20, 30])
array([[11, 22, 33],
       [14, 25, 36]])
```

When shapes are not compatible, you get an error worth reading carefully:

``` python
>>> np.array([1, 2, 3]) + np.array([1, 2])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: operands could not be broadcast together with shapes (3,) (2,)
```

The message gives the two shapes: here, 3 elements and 2 elements. Most numpy
errors come from shapes: when in doubt, **print the `shape` of your arrays**.

### Copies

Be careful: assigning an array to another name does **not** copy it. Both
names refer to the same data:

``` python
>>> a = np.array([1, 2, 3])
>>> b = a
>>> b[0] = 100
>>> a
array([100,   2,   3])
```

Use `b = a.copy()` when you need an independent copy.

### Why it's fast

numpy operations are run by compiled code, without Python loops. In IPython
(or in Spyder or Jupyter), the magic command `%timeit` measures the execution
time of a line. Compare:

``` python
In [1]: numbers = list(range(1_000_000))
In [2]: %timeit [x * 2 for x in numbers]
In [3]: a = np.arange(1_000_000)
In [4]: %timeit a * 2
```

The numpy version is typically **tens of times faster**. Rule of thumb: with
numpy, avoid loops over the elements of an array, and write operations on whole
arrays instead.



Visualizing with matplotlib
-------------------------------------------------------------------------------

[matplotlib] is the reference library for figures in Python. We use its
`pyplot` module, imported as `plt` by convention.

### A first plot

``` python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 2 * np.pi, 100)

fig, ax = plt.subplots()
ax.plot(x, np.sin(x), label="sin")
ax.plot(x, np.cos(x), label="cos", linestyle="--")
ax.set_xlabel("x")
ax.set_ylabel("y")
ax.set_title("Trigonometric functions")
ax.legend()
fig.savefig("trigo.png", dpi=150)
plt.show()
```

* `plt.subplots()` creates a **figure** (`fig`, the whole image) containing
  **axes** (`ax`, the plotting area with its axes, labels, title...).
* `ax.plot(x, y)` draws a line through the points `(x, y)`. `x` and `y` must
  have the same size.
* `fig.savefig` saves the figure; the format is given by the extension
  (`.png`, `.pdf`, `.svg`...).
* `plt.show()` displays the figure.

Where does the figure appear?

* **From the shell** (`python trigo.py`): in a new window. The script waits
  until you close it.
* **In Spyder**: in the *Plots* pane (top right).
* **In JupyterLab**: just below the cell. `plt.show()` is not even needed.

### Other kinds of plots

``` python
rng = np.random.default_rng(0)
values = rng.normal(0, 1, 1000)

fig, axes = plt.subplots(1, 3, figsize=(12, 4))   # 1 row, 3 plots

axes[0].scatter(values[:-1], values[1:], s=5)     # point cloud
axes[0].set_title("Scatter")

axes[1].hist(values, bins=30)                     # histogram
axes[1].set_title("Histogram")

image = rng.random((20, 20))
im = axes[2].imshow(image, cmap="viridis")        # 2D array as an image
fig.colorbar(im, ax=axes[2])
axes[2].set_title("Image")

fig.tight_layout()                                # avoid overlaps
plt.show()
```

`imshow` displays a 2D array as an image, each element being one pixel: we'll
use it to display the Game of Life in the next lesson.

The [matplotlib gallery](https://matplotlib.org/stable/gallery/index.html)
shows hundreds of examples: find a figure close to what you want, and adapt
its code.

### Reading a long error message

Errors raised inside a library produce long tracebacks:

``` python
>>> x = np.linspace(0, 1, 50)
>>> plt.plot(x, x[1:])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File ".../matplotlib/pyplot.py", line 4043, in plot
    return gca().plot(
           ^^^^^^^^^^^
  File ".../matplotlib/axes/_axes.py", line 1792, in plot
    lines = [*self._get_lines(self, *args, data=data, **kwargs)]
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File ".../matplotlib/axes/_base.py", line 509, in _plot_args
    raise ValueError(f"x and y must have same first dimension, but "
ValueError: x and y must have same first dimension, but have shapes (50,) and (49,)
```

Don't panic. As usual, read the **last line**: `x` has 50 elements and `y` 49.
Then find **your** code in the traceback: it is the first `File` line
(`<stdin>`, or the name of your script), not the files of matplotlib. The bug
is almost always in your code, not in the library.



Automating with scripts
-------------------------------------------------------------------------------

Interfaces like Spyder and JupyterLab are great to explore. But when the same
analysis must be applied to 10, 100 or 1000 files, or run again every week, a
**script run from the shell** is the right tool: it needs no interface and
does exactly the same thing every time.

### Preparing some data

Save this script as `generate_data.py` and run it. It creates a `data` folder
with fake temperature measurements from three sensors:

``` python
"""Generate fake temperature measurements for three sensors."""

from pathlib import Path

import numpy as np

rng = np.random.default_rng(seed=1)
folder = Path("data")
folder.mkdir(exist_ok=True)

hours = np.arange(0, 48, 0.5)  # two days, every 30 minutes
for name, offset in [("sensor_a", 18), ("sensor_b", 21), ("sensor_c", 15)]:
    temperature = offset + 4 * np.sin(2 * np.pi * hours / 24) + rng.normal(0, 0.5, hours.size)
    data = np.column_stack([hours, temperature])
    np.savetxt(folder / f"{name}.csv", data, delimiter=",", fmt="%.2f",
               header="hour,temperature", comments="")
    print("Written", folder / f"{name}.csv")
```

``` bash
(sci) $ python generate_data.py
Written data/sensor_a.csv
Written data/sensor_b.csv
Written data/sensor_c.csv
```

A few new things here:

* `Path` (from the standard module `pathlib`) represents a file or folder
  path, and works on all systems: `folder / "name.csv"` builds the path with
  the right separator (`/` or `\`).
* `np.savetxt` writes an array to a text file; `np.loadtxt` reads it back.
* `f"{name}.csv"` is an **f-string**: the expressions between braces are
  replaced by their value. Very convenient to build file names or messages.

Open one of the files with your text editor: it is a simple text file, with
one measurement per line.

### Arguments on the command line

A script becomes reusable when it takes its inputs from the command line
rather than having them written in its code. The simplest way is `sys.argv`,
the list of the words typed after `python`:

``` python
import sys

print(sys.argv)
name = sys.argv[1]
print("Hello", name)
```

``` bash
(sci) $ python hello_args.py Alice
['hello_args.py', 'Alice']
Hello Alice
(sci) $ python hello_args.py
['hello_args.py']
Traceback (most recent call last):
  File "/home/user/python-course/hello_args.py", line 4, in <module>
    name = sys.argv[1]
           ~~~~~~~~^^^
IndexError: list index out of range
```

Without argument, the list has only one element, hence the error. The
standard module `argparse` does the job properly: it checks the arguments,
converts them, and generates a help message for free.

### The analysis script

Save this script as `analyze.py`:

``` python
"""Analyse all temperature files of a folder.

For each CSV file, print a summary and save a figure.
"""

import argparse
from pathlib import Path

import matplotlib.pyplot as plt
import numpy as np


def load(filename):
    """Return the hours and temperatures stored in a CSV file."""
    data = np.loadtxt(filename, delimiter=",", skiprows=1)
    return data[:, 0], data[:, 1]


def summarize(temperature):
    """Return the mean, minimum and maximum temperature."""
    return temperature.mean(), temperature.min(), temperature.max()


def plot(hours, temperature, title, filename):
    """Save a plot of the temperature as a function of time."""
    fig, ax = plt.subplots(figsize=(8, 4))
    ax.plot(hours, temperature)
    ax.set_xlabel("Time (hours)")
    ax.set_ylabel("Temperature (°C)")
    ax.set_title(title)
    fig.savefig(filename, dpi=100)
    plt.close(fig)


def main():
    parser = argparse.ArgumentParser(description="Analyse temperature files.")
    parser.add_argument("folder", help="folder containing the CSV files")
    args = parser.parse_args()

    folder = Path(args.folder)
    for filename in sorted(folder.glob("*.csv")):
        hours, temperature = load(filename)
        mean, low, high = summarize(temperature)
        print(f"{filename.stem}: mean {mean:.1f} °C, min {low:.1f} °C, max {high:.1f} °C")
        plot(hours, temperature, filename.stem, filename.with_suffix(".png"))


if __name__ == "__main__":
    main()
```

and run it from the shell:

``` bash
(sci) $ python analyze.py data
sensor_a: mean 18.0 °C, min 13.2 °C, max 22.2 °C
sensor_b: mean 21.0 °C, min 16.7 °C, max 25.9 °C
sensor_c: mean 14.9 °C, min 10.2 °C, max 19.7 °C
```

The `data` folder now also contains one figure per sensor. Thanks to
`argparse`, the script also documents itself and reports wrong usage clearly:

``` bash
(sci) $ python analyze.py -h
usage: analyze.py [-h] folder

Analyse temperature files.

positional arguments:
  folder      folder containing the CSV files

options:
  -h, --help  show this help message and exit
(sci) $ python analyze.py
usage: analyze.py [-h] folder
analyze.py: error: the following arguments are required: folder
```

### Anatomy of a good script

Look at how `analyze.py` is organized. This structure is worth reusing for
all your scripts:

1. **A docstring** at the top, saying what the script does.
2. **The imports**, all at the top: standard modules first (`argparse`,
   `pathlib`), then external libraries (`matplotlib`, `numpy`).
3. **Small functions**, each doing one thing (`load`, `summarize`, `plot`),
   with a docstring. They are easy to understand, to reuse and to test (next
   lesson!).
4. **A `main` function** that reads the arguments and calls the other
   functions.
5. **The last two lines**, which call `main()` only when the file is run as a
   script:

   ``` python
   if __name__ == "__main__":
       main()
   ```

   When you run `python analyze.py`, Python sets the special variable
   `__name__` to `"__main__"`, so `main()` is called. But when another script
   (or a notebook) does `import analyze` to reuse the `load` function,
   `__name__` is `"analyze"`, and nothing is executed: only the functions are
   defined.

A few more details used in this script: `folder.glob("*.csv")` gives all the
files ending with `.csv` in the folder; `filename.stem` is the name without
extension; `filename.with_suffix(".png")` is the same path with another
extension; and `{mean:.1f}` in an f-string displays a number with 1 decimal.
Note also `plt.close(fig)`: when a script creates many figures, closing them
frees the memory.

> **Tip**: the script processes 3 files in a fraction of a second. It would
> process 3000 with no more effort from you, and exactly the same way. This is
> the real power of automation: **reproducibility**.



Exercises
-------------------------------------------------------------------------------

Write each exercise in a script (or a notebook), run it, and only then compare
with the answer.

### numpy

**1.** Create the array of the squares of the integers from 0 to 9, then
compute the mean of its even elements, without any loop.

<details><summary>Answer</summary>

``` python
x = np.arange(10) ** 2
print(x[x % 2 == 0].mean())   # 24.0
```
</details>

**2.** Convert the temperatures `[-10, 0, 20, 37, 100]` from Celsius to
Fahrenheit (`F = C × 9/5 + 32`), without any loop.

<details><summary>Answer</summary>

``` python
celsius = np.array([-10, 0, 20, 37, 100])
print(celsius * 9 / 5 + 32)   # [ 14.   32.   68.   98.6 212. ]
```

Compare with the function of the previous lesson: the same formula now works
on a whole array at once.
</details>

**3.** Build a 5×5 array containing the integers from 1 to 25, then compute
the sum of each row and the mean of each column.

<details><summary>Answer</summary>

``` python
m = np.arange(1, 26).reshape(5, 5)
print(m.sum(axis=1))    # [ 15  40  65  90 115]
print(m.mean(axis=0))   # [11. 12. 13. 14. 15.]
```
</details>

**4.** Draw 10 000 values from a normal distribution (mean 0, standard
deviation 1, seed 0). Which proportion of them is greater than 1?

<details><summary>Answer</summary>

``` python
rng = np.random.default_rng(0)
values = rng.normal(0, 1, 10_000)
print((values > 1).mean())   # 0.161
```

The mean of an array of booleans is the proportion of `True`. The theoretical
value is about 0.159.
</details>

**5.** In the array `[3, -1, 4, -1, 5]`, replace all negative values by 0.

<details><summary>Answer</summary>

``` python
y = np.array([3, -1, 4, -1, 5])
y[y < 0] = 0
print(y)   # [3 0 4 0 5]
```
</details>

### matplotlib

**6.** Plot the function `f(x) = exp(-x/5) × cos(x)` for `x` between 0 and
20, with axis labels and a title, and save it as `damped.png`.

<details><summary>Answer</summary>

``` python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 20, 500)
fig, ax = plt.subplots()
ax.plot(x, np.exp(-x / 5) * np.cos(x))
ax.set_xlabel("x")
ax.set_ylabel("f(x)")
ax.set_title("Damped oscillation")
fig.savefig("damped.png")
plt.show()
```

With too few points (e.g. `np.linspace(0, 20, 20)`), the curve looks broken:
matplotlib only draws straight lines between the points.
</details>

**7.** Using `analyze.py`'s `load` function, plot the three sensors on the
**same** figure, with a legend.

<details><summary>Answer</summary>

``` python
from pathlib import Path

import matplotlib.pyplot as plt

from analyze import load

fig, ax = plt.subplots(figsize=(8, 4))
for filename in sorted(Path("data").glob("*.csv")):
    hours, temperature = load(filename)
    ax.plot(hours, temperature, label=filename.stem)
ax.set_xlabel("Time (hours)")
ax.set_ylabel("Temperature (°C)")
ax.legend()
plt.show()
```

`from analyze import load` works because of the
`if __name__ == "__main__":` line: importing `analyze` defines its functions
without running the analysis.
</details>

### Automation

**8.** Write a script `squares.py` taking an integer `n` as argument, and
printing the squares of the integers from 1 to `n`. What happens with
`python squares.py abc`?

<details><summary>Answer</summary>

``` python
import argparse

parser = argparse.ArgumentParser(description="Print squares.")
parser.add_argument("n", type=int, help="how many squares")
args = parser.parse_args()

for i in range(1, args.n + 1):
    print(i, i**2)
```

``` bash
(sci) $ python squares.py 3
1 1
2 4
3 9
(sci) $ python squares.py abc
usage: squares.py [-h] n
squares.py: error: argument n: invalid int value: 'abc'
```

With `type=int`, argparse converts the argument and refuses invalid values
with a clear message.
</details>

**9.** Add to `analyze.py` an optional argument `--threshold`: for each file,
the script also prints how many measurements are above this temperature.
Usage: `python analyze.py data --threshold 22`.

<details><summary>Answer</summary>

In `main`, add the argument:

``` python
parser.add_argument("--threshold", type=float, default=None,
                    help="count the measurements above this temperature")
```

and, in the loop, after the summary:

``` python
        if args.threshold is not None:
            count = (temperature > args.threshold).sum()
            print(f"  {count} measurements above {args.threshold} °C")
```

Arguments starting with `--` are optional; without `default`, their value is
`None` when they are not given.
</details>

**10.** Modify `analyze.py` so that it also writes a file `summary.csv`
containing one line per sensor: name, mean, minimum and maximum.

<details><summary>Answer</summary>

Before the loop, open the file; in the loop, write one line per sensor:

``` python
    with open(folder / "summary.csv", "w") as output:
        output.write("sensor,mean,min,max\n")
        for filename in sorted(folder.glob("sensor_*.csv")):
            hours, temperature = load(filename)
            mean, low, high = summarize(temperature)
            output.write(f"{filename.stem},{mean:.2f},{low:.2f},{high:.2f}\n")
```

The `with` statement closes the file automatically at the end of the block.
Note the pattern `sensor_*.csv`: otherwise, running the script twice would
also try to analyse `summary.csv` itself!
</details>



Resources
-------------------------------------------------------------------------------

* [**NumPy: the absolute basics for beginners**](https://numpy.org/doc/stable/user/absolute_beginners.html)
  is the official introduction to numpy, with many illustrations of arrays,
  indexing and broadcasting.

* [**100 numpy exercises**](https://github.com/rougier/numpy-100) by Nicolas
  P. Rougier: a collection of exercises of increasing difficulty, with
  solutions, to practice numpy.

* [**From Python to Numpy**](https://www.labri.fr/perso/nrougier/from-python-to-numpy/)
  by Nicolas P. Rougier, a free book on vectorization: how to replace loops by
  numpy operations. It includes the Game of Life, our project for the next
  lesson.

* [**Matplotlib tutorials**](https://matplotlib.org/stable/tutorials/index.html)
  and the [**Matplotlib gallery**](https://matplotlib.org/stable/gallery/index.html):
  the official introduction, and hundreds of examples to adapt.

* [**Scientific Visualization: Python + Matplotlib**](https://github.com/rougier/scientific-visualization-book)
  by Nicolas P. Rougier, a free book on making good scientific figures.

* [**Argparse tutorial**](https://docs.python.org/3/howto/argparse.html) and
  [**pathlib**](https://docs.python.org/3/library/pathlib.html) in the official
  Python documentation.

* [**Scientific Python Lectures**](https://lectures.scientific-python.org)
  have detailed chapters on numpy and matplotlib.

* [**Spyder documentation**](https://docs.spyder-ide.org/) and
  [**JupyterLab documentation**](https://jupyterlab.readthedocs.io/).


<!----------------------------- External links ------------------------------->
[numpy]:      https://numpy.org
[matplotlib]: https://matplotlib.org
[Spyder]:     https://www.spyder-ide.org
[JupyterLab]: https://jupyterlab.readthedocs.io
<!---------------------------------------------------------------------------->

<br/><br/>
---

Copyright © 2026 [Pascal Helson](https://sites.google.com/view/page-pers-phelson) •
Released under a [CC-BY 4.0 International](https://creativecommons.org/licenses/by/4.0/legalcode) license.
