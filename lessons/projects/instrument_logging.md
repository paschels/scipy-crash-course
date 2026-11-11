# Project: automating an instrument
An alternative project to the Game of Life, for the scientific Python crash
course.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

Many of you want to **drive instruments**, **collect their measurements**,
**process** them and **store** them in a **database**. This project builds a
complete small data logger:

* a function reading a measurement from an instrument (simulated here, so that
  everybody can work without hardware);
* an acquisition script, run from the shell, with options;
* storage of every measurement in an [SQLite] database, a single file that
  can be queried in SQL;
* a script plotting the data, and tests.

Everything uses what we've seen in the course (functions, loops, numpy,
matplotlib, `argparse`, pytest), plus the standard module `sqlite3`.

**Duration**: about 1h45.


Before you start
-------------------------------------------------------------------------------

Activate the course environment and create a folder for the project:

``` bash
(base) $ conda activate sci
(sci) $ cd Documents/python-course
(sci) $ mkdir logger
(sci) $ cd logger
```

Nothing to install: `sqlite3` is part of Python's standard library.


Step 1: a (simulated) instrument
-------------------------------------------------------------------------------

Our instrument is a temperature probe in a reactor that is being heated: the
temperature rises from 20 °C towards 80 °C, with some measurement noise.

The key idea of this step is **design**, not physics: the reading of the
instrument is isolated in **one function**. The rest of the program only calls
`read_temperature`. When you connect a real instrument, you will only have to
rewrite this function (see [Using a real instrument](#using-a-real-instrument)).

Create a file `instrument.py`:

``` python
"""Simulated instrument and storage of its measurements in an SQLite database."""

import sqlite3

import numpy as np


def read_temperature(rng, elapsed):
    """Simulate the reading of a temperature probe in a heated reactor.

    Parameters
    ----------
    rng : numpy.random.Generator
        Random generator used for the measurement noise.
    elapsed : float
        Time since the start of the heating, in seconds.

    Returns
    -------
    float
        Temperature in °C: rises from 20 °C towards 80 °C, plus noise.
    """
    return 20 + 60 * (1 - np.exp(-elapsed / 5)) + rng.normal(0, 0.3)


def create_database(filename):
    """Open (or create) the database and make sure the table exists."""
    connection = sqlite3.connect(filename)
    connection.execute(
        "CREATE TABLE IF NOT EXISTS measurements "
        "(timestamp TEXT, elapsed REAL, temperature REAL)")
    connection.commit()
    return connection


def save_measurement(connection, timestamp, elapsed, temperature):
    """Store one measurement in the database."""
    connection.execute("INSERT INTO measurements VALUES (?, ?, ?)",
                       (timestamp, elapsed, temperature))
    connection.commit()


def load_measurements(connection):
    """Return all measurements as a 2D array: elapsed time, temperature."""
    rows = connection.execute(
        "SELECT elapsed, temperature FROM measurements ORDER BY elapsed").fetchall()
    return np.array(rows)
```

Besides `read_temperature`, this module contains three functions to work with
the database:

* `sqlite3.connect(filename)` opens the database file, and creates it if it
  does not exist yet;
* a database contains **tables**, with named and typed columns. The SQL
  command `CREATE TABLE IF NOT EXISTS` creates our table `measurements` the
  first time only;
* `INSERT INTO ... VALUES (?, ?, ?)` adds a row. The `?` are replaced by the
  values given in the tuple: **never** build an SQL command by pasting values
  into a string yourself;
* `commit()` writes the changes to the file;
* `SELECT ... ORDER BY elapsed` reads the rows back, sorted by time, and
  `fetchall()` returns them as a list of tuples, which we convert into a
  numpy array.

Try it in the interpreter (or in Spyder, or a notebook):

``` python
>>> import numpy as np
>>> from instrument import read_temperature
>>> rng = np.random.default_rng(0)
>>> read_temperature(rng, 0)       # about 20 °C
>>> read_temperature(rng, 30)      # close to 80 °C
```


Step 2: tests
-------------------------------------------------------------------------------

Even a simulated instrument deserves tests: they document what we expect, and
they'll still be useful to test the database functions when the instrument is
real. Create `test_instrument.py`:

``` python
"""Tests for instrument.py."""

import numpy as np

from instrument import (create_database, load_measurements, read_temperature,
                        save_measurement)


def test_temperature_starts_near_20():
    rng = np.random.default_rng(0)
    assert abs(read_temperature(rng, 0) - 20) < 2


def test_temperature_tends_to_80():
    rng = np.random.default_rng(0)
    assert abs(read_temperature(rng, 1000) - 80) < 2


def test_database_round_trip(tmp_path):
    connection = create_database(tmp_path / "test.db")
    save_measurement(connection, "2026-01-01T10:00:00", 0.0, 20.5)
    save_measurement(connection, "2026-01-01T10:00:01", 1.0, 25.0)
    data = load_measurements(connection)
    assert data.shape == (2, 2)
    assert data[1, 1] == 25.0
```

`tmp_path` is a **fixture** of pytest: if a test function has a parameter with
this name, pytest gives it a temporary folder, created for the test and
deleted afterwards. This way, the test does not leave files behind, nor mess
with your real database.

``` bash
(sci) $ pytest
...
test_instrument.py ...                                                   [100%]

============================== 3 passed in 0.08s ===============================
```


Step 3: the acquisition script
-------------------------------------------------------------------------------

The acquisition loop reads the instrument at regular intervals, prints each
measurement, stores it, and warns when the temperature exceeds an alarm level.
Create `acquire.py`:

``` python
"""Acquire measurements from the (simulated) instrument and store them."""

import argparse
import time
from datetime import datetime

import numpy as np

from instrument import create_database, read_temperature, save_measurement


def main():
    parser = argparse.ArgumentParser(description="Temperature data logger.")
    parser.add_argument("--duration", type=float, default=20,
                        help="acquisition duration (s)")
    parser.add_argument("--interval", type=float, default=0.5,
                        help="time between two measurements (s)")
    parser.add_argument("--alarm", type=float, default=75,
                        help="print a warning above this temperature (°C)")
    parser.add_argument("--db", default="measurements.db",
                        help="database file")
    args = parser.parse_args()

    rng = np.random.default_rng()
    connection = create_database(args.db)
    start = time.time()
    elapsed = 0.0
    while elapsed < args.duration:
        temperature = read_temperature(rng, elapsed)
        timestamp = datetime.now().isoformat(timespec="seconds")
        save_measurement(connection, timestamp, elapsed, temperature)
        message = f"{timestamp}  t = {elapsed:5.1f} s  T = {temperature:5.1f} °C"
        if temperature > args.alarm:
            message += "  ALARM!"
        print(message)
        time.sleep(args.interval)
        elapsed = time.time() - start
    connection.close()
    print("Data saved in", args.db)


if __name__ == "__main__":
    main()
```

New here:

* the standard module `time`: `time.time()` gives the current time in seconds,
  and `time.sleep(0.5)` waits half a second;
* the standard module `datetime`: `datetime.now().isoformat(...)` gives the
  date and time as a text like `2026-11-11T10:44:33`, a format that sorts
  correctly and that every tool understands;
* `{elapsed:5.1f}` in an f-string displays a number with 1 decimal, on 5
  characters, so that the columns are aligned.

Run it:

``` bash
(sci) $ python acquire.py --duration 20 --interval 0.5
2026-11-11T10:44:33  t =   0.0 s  T =  20.2 °C
2026-11-11T10:44:34  t =   0.5 s  T =  25.6 °C
2026-11-11T10:44:34  t =   1.0 s  T =  31.0 °C
...
2026-11-11T10:44:48  t =  15.1 s  T =  77.2 °C  ALARM!
...
Data saved in measurements.db
```

(Your values will differ slightly: the noise is random.) Press `Control-C` to
stop the acquisition before the end if needed.

Run the script a second time: the new measurements are **added** to the same
database. A database keeps the whole history of your experiments in a single
file.


Step 4: querying the database
-------------------------------------------------------------------------------

The power of a database is that you can ask it questions in SQL, without
loading everything in memory. In the interpreter:

``` python
>>> import sqlite3
>>> connection = sqlite3.connect("measurements.db")
>>> connection.execute("SELECT COUNT(*), AVG(temperature), MAX(temperature) "
...                    "FROM measurements").fetchone()
(40, 61.3..., 79.9...)
>>> connection.execute("SELECT timestamp, temperature FROM measurements "
...                    "WHERE temperature > 75 LIMIT 3").fetchall()
[('2026-11-11T10:44:48', 77.2...), ...]
>>> connection.close()
```

A few SQL keywords go a long way: `SELECT` (which columns), `FROM` (which
table), `WHERE` (which rows), `ORDER BY` (sort), `COUNT`, `AVG`, `MIN`, `MAX`
(statistics), `LIMIT` (how many rows).

> **Tip**: to explore a database with a graphical interface, use the free
> [DB Browser for SQLite](https://sqlitebrowser.org/). It opens the
> `measurements.db` file like a spreadsheet.


Step 5: plotting
-------------------------------------------------------------------------------

Create `plot_measurements.py`:

``` python
"""Plot the measurements stored in the database."""

import argparse

import matplotlib.pyplot as plt

from instrument import create_database, load_measurements


def main():
    parser = argparse.ArgumentParser(description="Plot logged temperatures.")
    parser.add_argument("--db", default="measurements.db", help="database file")
    parser.add_argument("--alarm", type=float, default=75, help="alarm level (°C)")
    args = parser.parse_args()

    connection = create_database(args.db)
    data = load_measurements(connection)
    connection.close()
    elapsed, temperature = data[:, 0], data[:, 1]

    fig, ax = plt.subplots(figsize=(8, 4))
    ax.plot(elapsed, temperature, ".-")
    ax.axhline(args.alarm, color="red", linestyle="--", label="alarm")
    ax.set_xlabel("Time (s)")
    ax.set_ylabel("Temperature (°C)")
    ax.legend()
    fig.savefig("measurements.png", dpi=150)
    plt.show()


if __name__ == "__main__":
    main()
```

``` bash
(sci) $ python plot_measurements.py
```

`ax.axhline` draws a horizontal line, here the alarm level. The `".-"` format
draws both the points and the lines between them: with real data, it is
important to **see** the measurements, not only a smooth line.

Your project folder now contains:

```
logger/
├── instrument.py          the instrument and the database functions
├── test_instrument.py     the tests
├── acquire.py             the acquisition script
├── plot_measurements.py   the plotting script
└── measurements.db        the database (created by acquire.py)
```


Using a real instrument
-------------------------------------------------------------------------------

Most laboratory instruments communicate in one of two ways, each with a Python
library available on conda-forge (no administrator rights needed):

* **Serial port** (RS-232, or USB seen as a serial port: `COM3` on Windows,
  `/dev/ttyUSB0` on Linux): use [pyserial].

  ``` python
  import serial

  port = serial.Serial("COM3", baudrate=9600, timeout=1)
  port.write(b"MEAS?\n")                 # the command depends on the instrument
  answer = port.readline().decode()      # e.g. "23.47\r\n"
  temperature = float(answer)
  ```

* **Measurement instruments** (oscilloscopes, multimeters, generators...)
  through USB, GPIB or Ethernet, using the standard SCPI commands: use
  [PyVISA].

  ``` python
  import pyvisa

  manager = pyvisa.ResourceManager()
  print(manager.list_resources())        # find the address of your instrument
  instrument = manager.open_resource("USB0::0x0957::0x1796::MY123::INSTR")
  print(instrument.query("*IDN?"))       # the instrument introduces itself
  ```

The commands themselves (`MEAS?`, ...) are given in the **manual of your
instrument**. Many manufacturers also provide Python examples or libraries.

Thanks to the design of step 1, connecting a real instrument only means
rewriting `read_temperature`. Install the libraries with
`conda install pyserial` or `conda install pyvisa pyvisa-py`.


Going further
-------------------------------------------------------------------------------

**1.** Add a column `sample` to the database, given by a new option
`--sample` of `acquire.py`, so that you can tell experiments apart. Then
plot only the measurements of one sample.

<details><summary>Answer</summary>

In `create_database`, the table gets a new column:

``` python
    connection.execute(
        "CREATE TABLE IF NOT EXISTS measurements "
        "(sample TEXT, timestamp TEXT, elapsed REAL, temperature REAL)")
```

`save_measurement` takes a `sample` parameter and uses four `?`, and
`load_measurements` filters with `WHERE`:

``` python
def load_measurements(connection, sample):
    rows = connection.execute(
        "SELECT elapsed, temperature FROM measurements "
        "WHERE sample = ? ORDER BY elapsed", (sample,)).fetchall()
    return np.array(rows)
```

Note `(sample,)`: a tuple with one element needs a comma. Delete the old
`measurements.db` first: an existing table is not modified by
`CREATE TABLE IF NOT EXISTS`.
</details>

**2.** Instead of printing `ALARM!` at every measurement above the
threshold, print it only when the temperature **crosses** the threshold
(once), and print `back to normal` when it goes back below.

<details><summary>Answer</summary>

Keep the state in a variable, created before the loop:

``` python
    alarm_on = False
    while elapsed < args.duration:
        ...
        if temperature > args.alarm and not alarm_on:
            print("ALARM!")
            alarm_on = True
        elif temperature <= args.alarm and alarm_on:
            print("back to normal")
            alarm_on = False
```
</details>

**3.** Add a test checking that `load_measurements` returns the measurements
sorted by time, even if they were inserted in a different order.

<details><summary>Answer</summary>

``` python
def test_measurements_are_sorted(tmp_path):
    connection = create_database(tmp_path / "test.db")
    save_measurement(connection, "t2", 2.0, 30.0)
    save_measurement(connection, "t1", 1.0, 25.0)
    data = load_measurements(connection)
    assert list(data[:, 0]) == [1.0, 2.0]
```
</details>

**4.** Write a script `export.py` exporting the whole database into a CSV
file, to open it in a spreadsheet or to share it.

<details><summary>Answer</summary>

``` python
import csv
import sqlite3

connection = sqlite3.connect("measurements.db")
rows = connection.execute("SELECT * FROM measurements").fetchall()
connection.close()

with open("measurements.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["timestamp", "elapsed", "temperature"])
    writer.writerows(rows)
print(len(rows), "rows exported")
```
</details>

**5.** Real instruments sometimes fail to answer. Make `acquire.py` robust:
if `read_temperature` raises an error, print a message, skip this measurement
and continue. Hint: look for `try` / `except` in the Python tutorial.

<details><summary>Answer</summary>

``` python
        try:
            temperature = read_temperature(rng, elapsed)
        except Exception as error:
            print("Reading failed:", error)
            time.sleep(args.interval)
            elapsed = time.time() - start
            continue
```

`try` runs the block; if an error occurs, the `except` block runs instead of
stopping the program. Catch errors only where you know how to handle them:
elsewhere, an error message is more useful than a silent failure.
</details>


Resources
-------------------------------------------------------------------------------

* [**sqlite3**](https://docs.python.org/3/library/sqlite3.html) in the
  official Python documentation, with a tutorial.
* [**SQLite tutorial**](https://www.sqlitetutorial.net/) to learn SQL step by
  step.
* [**pyserial**][pyserial] and [**PyVISA**][PyVISA] documentations, for real
  instruments.
* [**Errors and exceptions**](https://docs.python.org/3/tutorial/errors.html)
  in the Python tutorial, for `try` / `except`.
* [**pytest fixtures**](https://docs.pytest.org/en/stable/how-to/tmp_path.html):
  how `tmp_path` works.


<!----------------------------- External links ------------------------------->
[SQLite]:   https://www.sqlite.org
[pyserial]: https://pyserial.readthedocs.io
[PyVISA]:   https://pyvisa.readthedocs.io
<!---------------------------------------------------------------------------->
