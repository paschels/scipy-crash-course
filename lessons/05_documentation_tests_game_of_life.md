# Tables, tests and the Game of Life
**Pascal Helson** - [pascal.helson@u-bordeaux.fr](mailto:pascal.helson@u-bordeaux.fr)  
Lecture notes from the scientific Python crash course, following
*Computing, visualizing and automating*.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

The primary goal of this lesson is threefold:

* To analyse tables of data with [pandas]
* To document and test your code, so that others (and your future self) can
  understand it and trust it
* To program [Conway's Game of Life], a small project using everything seen
  in this course: functions, loops, numpy, matplotlib and scripts

**Contents** (3.5 hours, including a break)

* [Tables of data with pandas](#tables-of-data-with-pandas) (45 min)
* [Documenting code](#documenting-code) (25 min)
* [Testing code](#testing-code) (35 min)
* [Project: the Game of Life](#project-the-game-of-life) (1h35)
* [Going further](#going-further) (for the fastest, or at home)



Before you start
-------------------------------------------------------------------------------

Open a shell (*Terminal* on Linux and macOS, *Miniforge Prompt* on Windows),
activate the course environment and install [pandas], to work with tables
of data, [openpyxl], which pandas uses to read and write Excel files, and
[pytest], the tool we'll use to run tests:

``` bash
(base) $ conda activate sci
(sci) $ conda install pandas openpyxl pytest
```

Then create two folders in your course folder: `tables` for the first part,
and `life` for the project:

``` bash
(sci) $ cd Documents/python-course
(sci) $ mkdir tables life
```

Use the interface you prefer (text editor and shell, Spyder or JupyterLab),
but **run the tests and the final script from the shell**.



Tables of data with pandas
-------------------------------------------------------------------------------

numpy is perfect for arrays of numbers. But much scientific data comes as
**tables**: one row per sample, measurement or animal, and columns of
different types (dates, names, categories, numbers), often with missing
values. This is what [pandas] is made for: think of it as a spreadsheet you
drive with Python. And since pandas is built on numpy, everything you learned
(masks, `mean`, `sum`...) still applies.

### Some data

Go to the `tables` folder, save this script as `generate_experiments.py` and
run it. It creates a table of 60 fake chemistry experiments: for each one, its
date, the sample, the operator, the catalyst used, the temperature, and the
yield obtained (with 3 failed measurements):

``` python
"""Generate a fake table of experiments."""

import numpy as np
import pandas as pd

rng = np.random.default_rng(seed=5)
n = 60
catalyst = rng.choice(["A", "B", "C"], n)
temperature = rng.choice([40, 60, 80], n)
effect = {"A": 0, "B": 8, "C": -5}
yields = (50 + 0.3 * temperature + np.array([effect[c] for c in catalyst])
          + rng.normal(0, 4, n)).round(1)
yields[rng.choice(n, 3, replace=False)] = np.nan        # failed measurements

experiments = pd.DataFrame({
    "date": pd.date_range("2026-01-05", periods=n, freq="B").strftime("%Y-%m-%d"),
    "sample": [f"S{i:03d}" for i in range(1, n + 1)],
    "operator": rng.choice(["Ana", "Ben", "Chloé"], n),
    "catalyst": catalyst,
    "temperature": temperature,
    "yield": yields,
})
experiments.to_csv("experiments.csv", index=False)
print("Written experiments.csv with", n, "experiments")
```

``` bash
(sci) $ python generate_experiments.py
Written experiments.csv with 60 experiments
```

By convention, pandas is imported as `pd`. The script already shows the
main object of pandas, the **DataFrame** (a table), built here from a
dictionary of columns, and written to a CSV file with `to_csv`.

The rest of this section is best done interactively, in IPython, Spyder or
JupyterLab: pandas tables are displayed nicely there.

### Reading and exploring

``` python
>>> import pandas as pd
>>> df = pd.read_csv("experiments.csv")
>>> df.head()
         date sample operator catalyst  temperature  yield
0  2026-01-05   S001    Chloé        C           80   72.4
1  2026-01-06   S002      Ben        C           80   72.4
2  2026-01-07   S003      Ben        A           80   71.6
3  2026-01-08   S004    Chloé        C           40   56.7
4  2026-01-09   S005      Ben        B           40   75.4
>>> df.shape
(60, 6)
```

`read_csv` reads a CSV file into a DataFrame (`pd.read_excel` does the same
for Excel files). `head()` shows the first rows; on the left, the **index**
numbers the rows. Two other commands to run first on any new dataset:

``` python
>>> df.info()
<class 'pandas.DataFrame'>
RangeIndex: 60 entries, 0 to 59
Data columns (total 6 columns):
 #   Column       Non-Null Count  Dtype
---  ------       --------------  -----
 0   date         60 non-null     str
 1   sample       60 non-null     str
 2   operator     60 non-null     str
 3   catalyst     60 non-null     str
 4   temperature  60 non-null     int64
 5   yield        57 non-null     float64
dtypes: float64(1), int64(1), str(4)
memory usage: 2.9 KB
>>> df.describe()
       temperature      yield
count    60.000000  57.000000
mean     59.000000  68.050877
std      17.822096   7.370577
min      40.000000  53.000000
25%      40.000000  62.800000
50%      60.000000  68.900000
75%      80.000000  72.500000
max      80.000000  88.900000
```

`info()` gives the type of each column and the number of non-missing values:
we immediately see that 3 yields are missing. `describe()` gives the main
statistics of the numerical columns.

### Columns and rows

A column is selected by its name, and is a **Series**: a 1D array with an
index. All the numpy operations work on it:

``` python
>>> df["yield"].mean()
np.float64(68.05087719298245)
>>> df[["sample", "yield"]].head(3)        # several columns: a list of names
  sample  yield
0   S001   72.4
1   S002   72.4
2   S003   71.6
>>> df["catalyst"].value_counts()          # how many times each value appears
catalyst
A    23
C    19
B    18
Name: count, dtype: int64
```

Rows are selected with **boolean masks**, exactly as with numpy arrays, and
conditions are combined with `&` and `|`, with parentheses:

``` python
>>> df[(df["catalyst"] == "B") & (df["yield"] > 80)]
          date sample operator catalyst  temperature  yield
36  2026-02-24   S037      Ben        B           80   80.6
48  2026-03-12   S049      Ben        B           80   88.9
```

New columns are computed from existing ones, like numpy arrays:

``` python
>>> df["yield_fraction"] = df["yield"] / 100
```

To access a single value, use `df.loc[row, column]`, e.g.
`df.loc[0, "yield"]`; `sort_values("yield", ascending=False)` sorts the
table.

### Missing values

Missing values appear as `NaN`. pandas statistics ignore them (the mean above
was computed on 57 values), but you should always know where they are:

``` python
>>> df["yield"].isna().sum()
np.int64(3)
>>> df[df["yield"].isna()]
          date sample operator catalyst  temperature  yield
12  2026-01-21   S013    Chloé        B           40    NaN
19  2026-01-30   S020      Ana        C           80    NaN
50  2026-03-16   S051    Chloé        A           40    NaN
>>> clean = df.dropna(subset=["yield"])    # remove the rows without yield
>>> len(clean)
57
```

### Grouping: the killer feature

The questions we ask about tables are often *"what is the mean of X for each
value of Y?"*. This is what `groupby` does, in one line:

``` python
>>> df.groupby("catalyst")["yield"].agg(["count", "mean", "std"]).round(1)
          count  mean  std
catalyst
A            22  65.5  6.0
B            17  74.5  5.4
C            18  65.0  6.9
```

Catalyst B gives better yields. Does the temperature matter too? A **pivot
table** crosses two columns:

``` python
>>> table = df.pivot_table(values="yield", index="temperature",
...                        columns="catalyst", aggfunc="mean")
>>> table.round(1)
catalyst        A     B     C
temperature
40           60.3  70.6  56.6
60           66.2  74.0  66.6
80           71.8  80.4  70.6
```

The yield increases with the temperature, for all catalysts. Doing this with
loops, or by hand in a spreadsheet, would take much longer, and could not be
re-run in one command when new experiments are added.

### Plotting and saving

DataFrames and Series have a `plot` method, which uses matplotlib:

``` python
>>> import matplotlib.pyplot as plt
>>> ax = table.plot(marker="o")             # one line per catalyst
>>> ax.set_ylabel("Mean yield (%)")
>>> plt.show()
>>> df.boxplot(column="yield", by="catalyst")
>>> plt.show()
```

Results are saved with `to_csv`, or `to_excel` for Excel files:

``` python
>>> table.round(1).to_csv("summary.csv")
>>> table.round(1).to_excel("summary.xlsx")
```

### Reading a pandas error

A typo in a column name gives a long error message:

``` python
>>> df["Yield"]
Traceback (most recent call last):
  File ".../pandas/core/indexes/base.py", line 3641, in get_loc
    return self._engine.get_loc(casted_key)
  ...
KeyError: 'Yield'

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  ...
KeyError: 'Yield'
```

As usual, the **last line** says it all: there is no column `'Yield'` (the
column is `yield`, in lower case: names are case sensitive). The message
contains two tracebacks because pandas catches an internal error and raises
a clearer one: read the last one. `df.columns` lists the exact column names.

> **When to use numpy, when to use pandas?** numpy for arrays of numbers
> (images, grids, signals, simulations: the Game of Life below); pandas for
> tables with named columns of different types (experiments, measurements,
> participants...). Both work together: `df["yield"].to_numpy()` gives a numpy
> array.



Documenting code
-------------------------------------------------------------------------------

Code is read much more often than it is written. Six months from now, you
won't remember why you wrote a line: documentation is a message to your
future self, and to anyone who will use or review your code.

### Names

The first documentation is the choice of good names. Compare:

``` python
def f(a, b):
    return a * b

def rectangle_area(width, height):
    return width * height
```

Both work, but only the second explains itself. Use explicit names for
variables and functions, even if they are longer.

### Comments

A comment starts with `#` and is ignored by Python. Use comments to explain
**why** you do something, not **what** the code does (the code already says
it):

``` python
x = x + 1   # add 1 to x                         <- useless
x = x + 1   # indices start at 1 in the file     <- useful
```

### Docstrings

A **docstring** is a string placed on the first line of a function (or of a
file). Unlike comments, docstrings are kept by Python and displayed by
`help()`, by Spyder (`Ctrl+I`) and by JupyterLab (`Shift+Tab`).

For a short function, one line is enough:

``` python
def rectangle_area(width, height):
    """Return the area of a rectangle."""
    return width * height
```

For more complex functions, the scientific Python community uses the
[numpy docstring style][numpydoc], with sections for the parameters, the
returned values and examples:

``` python
def count_neighbours(grid, i, j):
    """Count the living neighbours of a cell.

    Parameters
    ----------
    grid : list of lists of int
        The grid, with 1 for a living cell and 0 for a dead cell.
    i, j : int
        Row and column of the cell.

    Returns
    -------
    int
        Number of living cells among the 8 neighbours (0 to 8).

    Examples
    --------
    >>> count_neighbours([[1, 1, 0], [0, 1, 0], [0, 0, 0]], 1, 1)
    2
    """
```

Writing the docstring **before** the code is a good habit: if you can't
explain simply what a function does, it probably does too many things.

A docstring at the top of a file documents the whole module (see
`analyze.py` in the previous lesson).

### README

A project folder should contain a `README.md` file, written in
[Markdown] (the format of these lecture notes), explaining what the project
does, how to install it and how to use it. It is the first file people read,
and GitHub displays it automatically on the project page. We'll write one at
the end of the project.



Testing code
-------------------------------------------------------------------------------

How do you know your code is correct? By trying it on cases where you know the
answer. Instead of doing it by hand once, a **test** does it automatically,
every time you want: after each modification, you can check in one second that
nothing is broken.

### assert

The `assert` statement checks that a condition is true. If it is, nothing
happens; otherwise, Python raises an `AssertionError`:

``` python
>>> assert 1 + 1 == 2
>>> assert 0.1 + 0.2 == 0.3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError
```

Remember the floating point lesson: never compare floats with `==`. Use
`pytest.approx` or `numpy.isclose`, which allow a small tolerance:

``` python
>>> import pytest
>>> 0.1 + 0.2 == pytest.approx(0.3)
True
>>> import numpy as np
>>> np.isclose(0.1 + 0.2, 0.3)
np.True_
```

### Writing tests with pytest

With pytest, tests are simply functions:

* whose name starts with `test_`,
* placed in files whose name starts with `test_`,
* containing `assert` statements.

For example, a file `test_area.py`:

``` python
from geometry import rectangle_area


def test_rectangle_area():
    assert rectangle_area(2, 3) == 6


def test_rectangle_area_zero():
    assert rectangle_area(0, 5) == 0
```

Running `pytest` in the folder finds and runs all the tests automatically:

``` bash
(sci) $ pytest
============================= test session starts ==============================
collected 2 items

test_area.py ..                                                          [100%]

============================== 2 passed in 0.01s ===============================
```

Each dot is a test that passed. `pytest -v` shows the name of each test.

### Reading a failed test

When a test fails, pytest shows which test failed, the faulty line (marked
with `>`), and the values that were compared (lines starting with `E`). We'll
see a real example during the project.

### What to test?

* **Simple cases** where you know the answer by heart.
* **Edge cases**: zero, empty list, first and last element, borders of a
  grid... This is where bugs hide.
* **Known results** from the literature, or from another implementation.
* **Properties** that must always hold, e.g. "the input is not modified".

A test that never fails is useless: check that your tests **can** fail, by
introducing a bug on purpose.

### Doctests

The `Examples` section of a docstring is also a test: pytest can run the
examples and check that the result is the one written. Use the
`--doctest-modules` option:

``` bash
(sci) $ pytest --doctest-modules
```

This way, the examples in your documentation are always correct.



Project: the Game of Life
-------------------------------------------------------------------------------

### The rules

The [Game of Life][Conway's Game of Life], invented by the mathematician John
Conway in 1970, is not really a game: it is a *cellular automaton*. The
universe is a grid of cells, each cell being either **alive** or **dead**.
Each cell has 8 neighbours (horizontally, vertically and diagonally). At each
generation, all the cells are updated **at the same time**, according to the
number of living neighbours:

* a dead cell with **exactly 3** living neighbours becomes alive (**birth**);
* a living cell with **2 or 3** living neighbours stays alive (**survival**);
* in all other cases, the cell dies or stays dead (by loneliness or
  overpopulation).

From these very simple rules emerge surprisingly complex behaviours. Some
patterns never change, some oscillate, some move across the grid:

```
 Block (stable)    Blinker (period 2)      Glider (moves)
   ....              .....    .....          .#....
   .##.              ..#..    .....          ..#...
   .##.              ..#.. -> .###.          ###...
   ....              ..#..    .....          ......
                     .....    .....
```

In this project, cells outside the grid are considered dead.

All the files of the project go in the `life` folder created at the
beginning of the lesson:

``` bash
(sci) $ cd Documents/python-course/life
```

We'll proceed in steps, writing tests as we go:

1. a pure Python version, with lists of lists;
2. tests for it;
3. a numpy version, tested against the first one;
4. an animation with matplotlib, in a script.

### Step 1: counting the neighbours

We represent the grid as a list of lists, with `1` for a living cell and `0`
for a dead one. `grid[i][j]` is the cell at row `i`, column `j`.

Create a file `life.py` with a module docstring and a first function, which
counts the living neighbours of a cell. Try to write it yourself first, from
its docstring, before reading the solution!

``` python
"""Conway's Game of Life.

Two implementations of the same rules: a pure Python one, working on lists
of lists, and a numpy one, working on 2D arrays. Cells outside the grid are
considered dead.
"""

import numpy as np


def count_neighbours(grid, i, j):
    """Count the living neighbours of a cell.

    Parameters
    ----------
    grid : list of lists of int
        The grid, with 1 for a living cell and 0 for a dead cell.
    i, j : int
        Row and column of the cell.

    Returns
    -------
    int
        Number of living cells among the 8 neighbours (0 to 8).

    Examples
    --------
    >>> count_neighbours([[1, 1, 0], [0, 1, 0], [0, 0, 0]], 1, 1)
    2
    """
    n_rows, n_cols = len(grid), len(grid[0])
    count = 0
    for di in (-1, 0, 1):
        for dj in (-1, 0, 1):
            if di == 0 and dj == 0:
                continue  # the cell itself is not a neighbour
            ni, nj = i + di, j + dj
            if 0 <= ni < n_rows and 0 <= nj < n_cols:
                count += grid[ni][nj]
    return count
```

The two nested loops visit the 9 cells of the 3×3 square around `(i, j)`,
skip the cell itself, and ignore positions outside the grid.

Now write the first tests, in a file `test_life.py`. A full grid is a good
case: we know how many neighbours each cell has, including on the edges.

``` python
"""Tests for life.py, to be run with pytest."""

import numpy as np

from life import count_neighbours


def test_count_neighbours_full_grid():
    grid = [[1, 1, 1],
            [1, 1, 1],
            [1, 1, 1]]
    assert count_neighbours(grid, 1, 1) == 8   # center
    assert count_neighbours(grid, 0, 0) == 3   # corner
    assert count_neighbours(grid, 0, 1) == 5   # edge


def test_count_neighbours_empty_grid():
    grid = [[0, 0, 0],
            [0, 0, 0],
            [0, 0, 0]]
    assert count_neighbours(grid, 1, 1) == 0
```

``` bash
(sci) $ pytest
...
test_life.py ..                                                          [100%]

============================== 2 passed in 0.06s ===============================
```

### Step 2: computing the next generation

Add the `step` function to `life.py`. Note that it builds a **new** grid:
since all the cells are updated at the same time, we must not modify the grid
while we are still reading it.

``` python
def step(grid):
    """Compute the next generation (pure Python version).

    Parameters
    ----------
    grid : list of lists of int
        The current generation.

    Returns
    -------
    list of lists of int
        The next generation. The input grid is not modified.
    """
    new_grid = []
    for i in range(len(grid)):
        new_row = []
        for j in range(len(grid[0])):
            n = count_neighbours(grid, i, j)
            if grid[i][j] == 1 and n in (2, 3):
                new_row.append(1)   # survival
            elif grid[i][j] == 0 and n == 3:
                new_row.append(1)   # birth
            else:
                new_row.append(0)   # death or stays dead
        new_grid.append(new_row)
    return new_grid
```

The patterns shown above give us perfect tests: the block must not change,
the blinker must alternate between its two positions, and `step` must not
modify its input. Add to `test_life.py` (and update the import line to
`from life import count_neighbours, step`):

``` python
def test_block_is_stable():
    block = [[0, 0, 0, 0],
             [0, 1, 1, 0],
             [0, 1, 1, 0],
             [0, 0, 0, 0]]
    assert step(block) == block


def test_blinker_oscillates():
    vertical = [[0, 0, 0, 0, 0],
                [0, 0, 1, 0, 0],
                [0, 0, 1, 0, 0],
                [0, 0, 1, 0, 0],
                [0, 0, 0, 0, 0]]
    horizontal = [[0, 0, 0, 0, 0],
                  [0, 0, 0, 0, 0],
                  [0, 1, 1, 1, 0],
                  [0, 0, 0, 0, 0],
                  [0, 0, 0, 0, 0]]
    assert step(vertical) == horizontal
    assert step(horizontal) == vertical


def test_step_does_not_modify_input():
    grid = [[0, 1, 0],
            [0, 1, 0],
            [0, 1, 0]]
    copy = [row[:] for row in grid]
    step(grid)
    assert grid == copy
```

``` bash
(sci) $ pytest -v
...
test_life.py::test_count_neighbours_full_grid PASSED                     [ 20%]
test_life.py::test_count_neighbours_empty_grid PASSED                    [ 40%]
test_life.py::test_block_is_stable PASSED                                [ 60%]
test_life.py::test_blinker_oscillates PASSED                             [ 80%]
test_life.py::test_step_does_not_modify_input PASSED                     [100%]

============================== 5 passed in 0.06s ===============================
```

**See a test fail.** Introduce a bug on purpose in `step`: replace
`n in (2, 3)` by `n == 2`, and run `pytest` again:

``` bash
(sci) $ pytest
...
=================================== FAILURES ===================================
_____________________________ test_block_is_stable _____________________________

    def test_block_is_stable():
        block = [[0, 0, 0, 0],
                 [0, 1, 1, 0],
                 [0, 1, 1, 0],
                 [0, 0, 0, 0]]
>       assert step(block) == block
E       assert [[0, 0, 0, 0]... [0, 0, 0, 0]] == [[0, 0, 0, 0]... [0, 0, 0, 0]]
E
E         At index 1 diff: [0, 0, 0, 0] != [0, 1, 1, 0]
E         Use -v to get more diff

test_life.py:29: AssertionError
=========================== short test summary info ============================
FAILED test_life.py::test_block_is_stable - assert [[0, 0, 0, 0]... [0, 0, 0,...
========================= 1 failed, 4 passed in 0.07s ==========================
```

Read it like any error message: the test `test_block_is_stable` failed at line
29 (marked with `>`), and the `E` lines say that row 1 of the result is
`[0, 0, 0, 0]` instead of `[0, 1, 1, 0]`: the block died. Indeed, each cell of
the block has 3 neighbours, and our buggy rule only lets cells with 2
neighbours survive. The tests caught the bug immediately. Fix it and check
that all tests pass again.

### Step 3: watching it live (in text mode)

Before any graphics, a simple text display is enough to check that the glider
moves. Create a script `show.py`:

``` python
"""Display a few generations of a glider in text mode."""

from life import step


def show(grid):
    """Print the grid, with # for living cells and . for dead cells."""
    for row in grid:
        print("".join("#" if cell else "." for cell in row))
    print()


glider = [[0, 1, 0, 0, 0, 0],
          [0, 0, 1, 0, 0, 0],
          [1, 1, 1, 0, 0, 0],
          [0, 0, 0, 0, 0, 0],
          [0, 0, 0, 0, 0, 0],
          [0, 0, 0, 0, 0, 0]]

grid = glider
for generation in range(5):
    show(grid)
    grid = step(grid)
```

``` bash
(sci) $ python show.py
.#....
..#...
###...
......
......
......

......
#.#...
.##...
.#....
......
......

...
```

After 4 generations, the glider has its initial shape again, shifted by one
cell down and to the right.

### Step 4: a numpy version

The pure Python version works, but it is slow: for each cell, Python loops
over its 8 neighbours. With numpy, we can count the neighbours of **all** the
cells at once.

The idea: add a border of dead cells around the grid (`np.pad`), then take 8
slices of this padded grid, each one shifted by one cell in one direction.
Adding these 8 shifted grids gives, for each cell, the number of its living
neighbours. Then the rules are applied with boolean masks.

Add this function to `life.py`:

``` python
def step_numpy(Z):
    """Compute the next generation (numpy version).

    Parameters
    ----------
    Z : numpy.ndarray
        2D array of 0 and 1, the current generation.

    Returns
    -------
    numpy.ndarray
        The next generation, with the same shape as Z.
    """
    P = np.pad(Z, 1)  # add a border of dead cells
    N = (P[:-2, :-2] + P[:-2, 1:-1] + P[:-2, 2:] +
         P[1:-1, :-2]               + P[1:-1, 2:] +
         P[2:, :-2]  + P[2:, 1:-1]  + P[2:, 2:])
    birth = (Z == 0) & (N == 3)
    survival = (Z == 1) & ((N == 2) | (N == 3))
    return (birth | survival).astype(int)
```

Take the time to understand the slices: `P[1:-1, 1:-1]` would be the grid
itself; `P[:-2, :-2]` is the grid shifted so that each cell sees its
upper-left neighbour; `P[2:, 2:]` its lower-right neighbour, and so on.

How do we test it? We have a reference implementation that we trust, since
it passes all our tests: the numpy version must give **exactly the same
result** on many random grids. Add to `test_life.py` (and import
`step_numpy`):

``` python
def test_numpy_matches_python():
    rng = np.random.default_rng(0)
    for _ in range(10):
        Z = rng.integers(0, 2, size=(20, 30))
        expected = step(Z.tolist())
        assert step_numpy(Z).tolist() == expected
```

`Z.tolist()` converts an array into a list of lists, so that both versions
can be compared. Run the tests, including the doctest of `count_neighbours`:

``` bash
(sci) $ pytest --doctest-modules
...
life.py .                                                                [ 14%]
test_life.py ......                                                      [100%]

============================== 7 passed in 0.40s ===============================
```

Was it worth it? In IPython, Spyder or JupyterLab, compare the two versions on
a 100×100 grid:

``` python
In [1]: import numpy as np
In [2]: from life import step, step_numpy
In [3]: Z = np.random.default_rng(0).integers(0, 2, size=(100, 100))
In [4]: L = Z.tolist()
In [5]: %timeit step(L)
In [6]: %timeit step_numpy(Z)
```

The numpy version is typically **about 50 times faster**.

### Step 5: animating with matplotlib

To animate the game, we display the grid with `imshow` (as in the previous
lesson), and use matplotlib's `FuncAnimation`, which calls a function to
update the image at each frame. Create a script `run_life.py`:

``` python
"""Run and display Conway's Game of Life."""

import argparse

import matplotlib.pyplot as plt
import numpy as np
from matplotlib.animation import FuncAnimation

from life import step_numpy


def main():
    parser = argparse.ArgumentParser(description="Conway's Game of Life.")
    parser.add_argument("--size", type=int, default=100, help="grid size")
    parser.add_argument("--density", type=float, default=0.2,
                        help="initial proportion of living cells")
    parser.add_argument("--seed", type=int, default=None, help="random seed")
    parser.add_argument("--save", default=None,
                        help="save the animation to this file (e.g. life.gif)")
    args = parser.parse_args()

    rng = np.random.default_rng(args.seed)
    Z = (rng.random((args.size, args.size)) < args.density).astype(int)

    fig, ax = plt.subplots(figsize=(6, 6))
    image = ax.imshow(Z, cmap="gray_r", interpolation="nearest")
    ax.set_xticks([])
    ax.set_yticks([])

    def update(frame):
        nonlocal Z
        Z = step_numpy(Z)
        image.set_data(Z)
        ax.set_title(f"Generation {frame + 1} - {Z.sum()} living cells")
        return [image]

    animation = FuncAnimation(fig, update, frames=200, interval=50)
    if args.save:
        animation.save(args.save, fps=20)
        print("Saved", args.save)
    else:
        plt.show()


if __name__ == "__main__":
    main()
```

A few explanations:

* `rng.random(...) < args.density` gives `True` for a proportion `density` of
  the cells, converted to `1` and `0` by `astype(int)`.
* `update` is a function defined **inside** `main`. The line `nonlocal Z`
  tells Python that `Z` is the variable of `main`, which `update` modifies at
  each frame.
* `FuncAnimation` calls `update` 200 times, every 50 milliseconds.
  **Keep the result in a variable** (`animation`): otherwise, the animation is
  deleted immediately and nothing moves.

Run it from the shell, with different options:

``` bash
(sci) $ python run_life.py
(sci) $ python run_life.py --size 50 --density 0.3 --seed 1
(sci) $ python run_life.py --seed 1 --save life.gif
Saved life.gif
```

The last command saves the animation as an image file, which you can open in
your web browser or put in a presentation.

> **Note**: animations work best from the shell. In Spyder, the *Plots* pane
> only shows still images: set *Preferences > IPython console > Graphics >
> Graphics backend* to *Automatic* to open figures in a separate window. In
> JupyterLab, use the `--save` option and open the GIF file, or display the
> animation in the notebook with
> `from IPython.display import HTML; HTML(animation.to_jshtml())`.

### Step 6: wrapping up

Your project folder now looks like this:

```
life/
├── README.md        (to write now)
├── life.py          the rules: count_neighbours, step, step_numpy
├── test_life.py     the tests
├── show.py          text display of a glider
└── run_life.py      the animation script
```

Finish by writing the `README.md`: one or two sentences explaining what the
project is, the rules of the game, how to run the tests (`pytest`) and how to
run the animation, with its options (copy the output of
`python run_life.py -h`). Then run `pytest --doctest-modules` one last time.

Congratulations: you've written a small but real scientific program,
documented, tested, fast, and usable from the command line!



Going further
-------------------------------------------------------------------------------

Try to answer before clicking on *Answer*.

### pandas

**P1.** With the table of experiments, compute the mean yield of each
operator. Ben seems to be the best chemist. Is he really? Hint:
`pd.crosstab(df["operator"], df["catalyst"])` counts the experiments for
each pair (operator, catalyst).

<details><summary>Answer</summary>

``` python
>>> df.groupby("operator")["yield"].mean().round(1)
operator
Ana      63.8
Ben      71.8
Chloé    68.4
Name: yield, dtype: float64
>>> pd.crosstab(df["operator"], df["catalyst"])
catalyst   A  B  C
operator
Ana       11  1  7
Ben        4  9  5
Chloé      8  8  7
>>> df.pivot_table(values="yield", index="operator", columns="catalyst",
...                aggfunc="mean").round(1)
catalyst     A     B     C
operator
Ana       65.6  70.1  59.5
Ben       65.4  76.1  69.4
Chloé     65.6  73.1  66.7
```

Ben used the best catalyst (B) much more often than Ana, who used it only
once: this is why his mean yield is higher. With catalyst A, the three
operators get the same yield. This is a classic trap called **confounding**:
before concluding from a mean, check how the groups were made. (In fact, our
generator gives no role at all to the operator!)
</details>

**P2.** Convert the column `date` into real dates with
`pd.to_datetime(df["date"])`, then compute the mean yield per month.

<details><summary>Answer</summary>

``` python
>>> df["date"] = pd.to_datetime(df["date"])
>>> df.groupby(df["date"].dt.month)["yield"].mean().round(1)
date
1    67.7
2    66.6
3    69.9
Name: yield, dtype: float64
```

Once converted, a date column gives access to `.dt.year`, `.dt.month`,
`.dt.day_name()`... and can be used to sort or filter by period.
</details>

**P3.** For each catalyst, find the experiment with the best yield (sample,
temperature and yield).

<details><summary>Answer</summary>

``` python
>>> best = df.groupby("catalyst")["yield"].idxmax()
>>> df.loc[best, ["catalyst", "sample", "temperature", "yield"]]
   catalyst sample  temperature  yield
46        A   S047           80   75.9
48        B   S049           80   88.9
57        C   S058           80   73.3
```

`idxmax` gives the index (row label) of the maximum in each group, and
`df.loc` selects these rows and the requested columns.
</details>

### Game of Life

**1.** Add a test checking that after 4 generations, the glider has moved by
one cell down and one cell to the right.

<details><summary>Answer</summary>

``` python
def test_glider_moves():
    glider = [[0, 1, 0, 0, 0, 0],
              [0, 0, 1, 0, 0, 0],
              [1, 1, 1, 0, 0, 0],
              [0, 0, 0, 0, 0, 0],
              [0, 0, 0, 0, 0, 0],
              [0, 0, 0, 0, 0, 0]]
    grid = glider
    for _ in range(4):
        grid = step(grid)
    # shifted: an empty first row, then each row shifted right by one cell
    expected = [[0] * 6] + [[0] + row[:-1] for row in glider[:-1]]
    assert grid == expected
```
</details>

**2.** Plot the number of living cells as a function of the generation, for
a random 100×100 grid over 300 generations. What do you observe?

<details><summary>Answer</summary>

``` python
import matplotlib.pyplot as plt
import numpy as np

from life import step_numpy

rng = np.random.default_rng(1)
Z = (rng.random((100, 100)) < 0.2).astype(int)
population = []
for generation in range(300):
    population.append(Z.sum())
    Z = step_numpy(Z)

fig, ax = plt.subplots()
ax.plot(population)
ax.set_xlabel("Generation")
ax.set_ylabel("Living cells")
plt.show()
```

The population first drops quickly (many cells die of loneliness or
overpopulation), then fluctuates and slowly stabilizes when only stable and
oscillating patterns remain.
</details>

**3.** Write a function `load_pattern(filename, size)` reading a pattern
from a text file using `#` for living cells and `.` for dead cells (like the
output of `show`), and returning a `size`×`size` numpy array containing the
pattern near its upper-left corner.

<details><summary>Answer</summary>

``` python
def load_pattern(filename, size=20):
    """Load a pattern from a text file (# alive, . dead) into a square grid."""
    with open(filename) as f:
        lines = f.read().split()
    Z = np.zeros((size, size), dtype=int)
    for i, line in enumerate(lines):
        for j, char in enumerate(line):
            if char == "#":
                Z[i + 1, j + 1] = 1
    return Z
```

The pattern is placed at row 1, column 1, so that it does not touch the
border. Thousands of patterns are available on the [LifeWiki].
</details>

**4.** In our version, the border of the grid is a wall of dead cells, and
gliders crash into it. Write a version `step_torus` where the grid wraps
around: a cell on the right edge has neighbours on the left edge, and so on.
Hint: `np.roll`.

<details><summary>Answer</summary>

``` python
def step_torus(Z):
    """Compute the next generation on a grid that wraps around."""
    N = sum(np.roll(np.roll(Z, di, axis=0), dj, axis=1)
            for di in (-1, 0, 1) for dj in (-1, 0, 1)
            if (di, dj) != (0, 0))
    birth = (Z == 0) & (N == 3)
    survival = (Z == 1) & ((N == 2) | (N == 3))
    return (birth | survival).astype(int)
```

`np.roll` shifts an array, and the elements going out on one side come back on
the other. A good test: on a 6×6 torus, a glider gets back to its exact
initial position after 24 generations (4 generations per cell, 6 cells).
</details>

**5.** Many variants of the Game of Life exist, written as `B3/S23` for the
original (birth with 3 neighbours, survival with 2 or 3). Generalize
`step_numpy` into `step_rule(Z, birth=(3,), survival=(2, 3))`, and try
*HighLife* (`B36/S23`). How do you check that your generalization is correct?

<details><summary>Answer</summary>

``` python
def step_rule(Z, birth=(3,), survival=(2, 3)):
    """Compute the next generation for any B/S rule."""
    P = np.pad(Z, 1)
    N = (P[:-2, :-2] + P[:-2, 1:-1] + P[:-2, 2:] +
         P[1:-1, :-2]               + P[1:-1, 2:] +
         P[2:, :-2]  + P[2:, 1:-1]  + P[2:, 2:])
    born = (Z == 0) & np.isin(N, birth)
    survive = (Z == 1) & np.isin(N, survival)
    return (born | survive).astype(int)
```

HighLife: `step_rule(Z, birth=(3, 6))`. To check it, write a test comparing
`step_rule` with the default rule to `step_numpy` on random grids, exactly as
we did for `step_numpy` and `step`.
</details>

**6.** Read this pytest output. Which test failed, where, and what does it
tell you?

``` bash
__________________________ test_numpy_matches_python ___________________________

    def test_numpy_matches_python():
        rng = np.random.default_rng(0)
        for _ in range(10):
            Z = rng.integers(0, 2, size=(20, 30))
            expected = step(Z.tolist())
>           assert step_numpy(Z).tolist() == expected
E           assert [[1, 1, 1, 0,... 0, ...], ...] == [[1, 0, 1, 0,... 0, ...], ...]
E
E             At index 0 diff: [1, 1, 1, 0, 0, ...] != [1, 0, 1, 0, 0, ...]

test_life.py:61: AssertionError
```

<details><summary>Answer</summary>

`test_numpy_matches_python` failed at line 61: on a random grid, the numpy
version and the Python version give different results (already on the first
row). The test does not say **which** one is wrong. Here, the other tests
also failed (e.g. `test_block_is_stable`), which points to the Python
version: indeed, this output was obtained with the bug of step 2
(`n == 2`). When several tests fail, start with the simplest one.
</details>



Resources
-------------------------------------------------------------------------------

* [**10 minutes to pandas**](https://pandas.pydata.org/docs/user_guide/10min.html)
  and the [**pandas getting started tutorials**](https://pandas.pydata.org/docs/getting_started/intro_tutorials/index.html):
  the official introductions, very visual.

* [**Comparison with spreadsheets**](https://pandas.pydata.org/docs/getting_started/comparison/comparison_with_spreadsheets.html):
  how to do in pandas what you do in Excel (filters, pivot tables,
  VLOOKUP...).

* [**Conway's Game of Life**][Conway's Game of Life] on Wikipedia, and the
  [**LifeWiki**][LifeWiki], an encyclopedia of patterns, with thousands of
  patterns to download.

* [**From Python to Numpy**](https://www.labri.fr/perso/nrougier/from-python-to-numpy/)
  by Nicolas P. Rougier: its chapter on vectorization uses the Game of Life
  as a main example, and goes much further (reaction-diffusion, boids...).

* [**Numpy tutorial**](https://github.com/rougier/numpy-tutorial) by Nicolas
  P. Rougier, which builds the Game of Life step by step, from Python to
  numpy.

* [**pytest: Get Started**](https://docs.pytest.org/en/stable/getting-started.html),
  the official introduction to pytest.

* [**numpydoc style guide**][numpydoc]: the complete description of the numpy
  docstring format.

* [**doctest**](https://docs.python.org/3/library/doctest.html) in the
  official Python documentation.

* [**Matplotlib animations**](https://matplotlib.org/stable/users/explain/animations/animations.html):
  how `FuncAnimation` works, and how to save animations.

* [**Software Carpentry: Defensive Programming**](https://swcarpentry.github.io/python-novice-inflammation/10-defensive.html):
  assertions and tests in a scientific context.


<!----------------------------- External links ------------------------------->
[Conway's Game of Life]: https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life
[LifeWiki]:   https://conwaylife.com/wiki/
[pandas]:     https://pandas.pydata.org
[openpyxl]:   https://openpyxl.readthedocs.io
[pytest]:     https://docs.pytest.org
[numpydoc]:   https://numpydoc.readthedocs.io/en/latest/format.html
[Markdown]:   https://www.markdownguide.org/basic-syntax/
<!---------------------------------------------------------------------------->

<br/><br/>
---

Copyright © 2026 [Pascal Helson](https://sites.google.com/view/page-pers-phelson) •
Released under a [CC-BY 4.0 International](https://creativecommons.org/licenses/by/4.0/legalcode) license.
