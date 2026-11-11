# Project: a whole behavioural experiment with pandas
An alternative project to the Game of Life, for the scientific Python crash
course. It uses [pandas], introduced at the beginning of the last session.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

A real experiment is never one file: it is **many recordings** (one per
animal, per session...) plus a **metadata** table describing the conditions
(sex, treatment, cage, genotype...). The heart of the analysis is to bring
them together correctly. In this project, we analyse an open-field
experiment with 16 mice:

* combining all the tracking files into **one long table** (`concat`);
* computing per-animal measures with `groupby`, without mixing animals;
* attaching the experimental conditions with `merge`;
* comparing groups, and following the activity **over time**;
* thinking about the **design** of the experiment.

It extends the project [Mouse behaviour in an open field](open_field.md), but
can be done on its own. The same approach applies to any experiment with many
recordings: electrophysiology sessions, plate reader runs, patients...

**Duration**: about 1h45.


Before you start
-------------------------------------------------------------------------------

Activate the course environment, check that pandas and openpyxl are installed
(they were installed at the beginning of the session), and create a folder for
the project:

``` bash
(base) $ conda activate sci
(sci) $ conda install pandas openpyxl
(sci) $ cd Documents/python-course
(sci) $ mkdir experiment
(sci) $ cd experiment
```

Save this script as `generate_experiment.py` and run it. It simulates a
10-minute open-field test at 5 frames per second for 16 mice: 8 treated with
a drug, 8 with the vehicle (control), half females and half males, housed in
4 cages. The mice are a bit less active at the end of the test, as real mice
getting used to the arena:

``` python
"""Simulate an open-field experiment: 16 mice, tracking files and metadata."""

from pathlib import Path

import numpy as np
import pandas as pd

rng = np.random.default_rng(seed=4)
folder = Path("tracking")
folder.mkdir(exist_ok=True)


def simulate(speed, duration=600, fps=5, arena=50):
    """Random walk of a mouse in a square arena (positions in cm)."""
    n = duration * fps
    x, y = np.empty(n), np.empty(n)
    x[0] = y[0] = arena / 2
    angle = rng.uniform(0, 2 * np.pi)
    for i in range(1, n):
        habituation = 1.5 - i / n                  # mice slow down over time
        angle += rng.normal(0, 0.4)
        step = rng.exponential(speed * habituation / fps)
        x[i] = np.clip(x[i - 1] + step * np.cos(angle), 0, arena)
        y[i] = np.clip(y[i - 1] + step * np.sin(angle), 0, arena)
        if x[i] in (0, arena) or y[i] in (0, arena):
            angle += np.pi / 2 + rng.uniform(0, np.pi)
    return np.arange(n) / fps, x.round(2), y.round(2)


rows = []
for k in range(1, 17):
    mouse = f"M{k:02d}"
    sex = "F" if k % 2 else "M"
    treatment = "drug" if k > 8 else "vehicle"
    speed = (6 if treatment == "drug" else 9) + (1 if sex == "F" else 0)
    t, x, y = simulate(speed)
    pd.DataFrame({"time": t, "x": x, "y": y}).to_csv(folder / f"{mouse}.csv",
                                                      index=False)
    rows.append({"mouse": mouse, "sex": sex, "treatment": treatment,
                 "cage": f"C{(k - 1) // 4 + 1}"})

pd.DataFrame(rows).to_csv("metadata.csv", index=False)
print("Written 16 tracking files and metadata.csv")
```

``` bash
(sci) $ python generate_experiment.py
Written 16 tracking files and metadata.csv
```

You get a folder `tracking` with one file per mouse (`M01.csv`, ...), with the
columns `time,x,y`, and a table `metadata.csv`:

```
mouse,sex,treatment,cage
M01,F,vehicle,C1
M02,M,vehicle,C1
M03,F,vehicle,C1
...
```


Step 1: one long table
-------------------------------------------------------------------------------

Create a file `behaviour.py`:

``` python
"""Combine and summarize tracking files with pandas."""

from pathlib import Path

import numpy as np
import pandas as pd


def load_tracking(folder):
    """Read all the tracking files of a folder into one long table.

    Each file is named after the mouse (e.g. M01.csv). The result has one row
    per frame, with the columns mouse, time, x and y.
    """
    tables = []
    for filename in sorted(Path(folder).glob("*.csv")):
        table = pd.read_csv(filename)
        table.insert(0, "mouse", filename.stem)
        tables.append(table)
    return pd.concat(tables, ignore_index=True)


def add_steps(tracking):
    """Add the column step: distance travelled since the previous frame (cm).

    The differences are computed within each mouse, so that the first frame
    of a mouse is not compared with the last frame of the previous one.
    """
    tracking = tracking.copy()
    dx = tracking.groupby("mouse")["x"].diff()
    dy = tracking.groupby("mouse")["y"].diff()
    tracking["step"] = np.hypot(dx, dy).fillna(0)
    return tracking


def summarize(tracking, arena=50, center=0.5):
    """Return one row per mouse: distance travelled (cm) and time in center (s)."""
    margin = arena * (1 - center) / 2
    tracking = tracking.assign(
        in_center=tracking["x"].between(margin, arena - margin)
        & tracking["y"].between(margin, arena - margin))
    dt = tracking["time"].iloc[1] - tracking["time"].iloc[0]
    summary = tracking.groupby("mouse").agg(distance=("step", "sum"),
                                            center_frames=("in_center", "sum"))
    summary["center_time"] = summary["center_frames"] * dt
    return summary.drop(columns="center_frames").round(1)
```

The three functions are the core of the project:

* `load_tracking` reads every file, adds a column `mouse` with the name of the
  file (`insert(0, ...)` puts it first), and stacks all the tables with
  `pd.concat`. The result is a **long** table: one row per frame and per
  mouse, which is the format pandas works best with;
* `add_steps` computes the distance travelled between two frames. The key is
  `groupby("mouse")["x"].diff()`: the differences are computed **within each
  mouse**. A simple `tracking["x"].diff()` would compute, at the first frame of
  each mouse, a meaningless jump from the last position of the previous mouse;
* `summarize` computes, for each mouse, the total distance and the time spent
  in the center. `between(a, b)` tests whether values are in an interval.

``` python
>>> import pandas as pd
>>> from behaviour import load_tracking, add_steps, summarize
>>> tracking = load_tracking("tracking")
>>> tracking.shape
(48000, 4)
>>> tracking.head(3)
  mouse  time      x      y
0   M01   0.0  25.00  25.00
1   M01   0.2  33.84  20.97
2   M01   0.4  36.62  20.51
>>> tracking = add_steps(tracking)
>>> summary = summarize(tracking)
>>> summary.head(3)
       distance  center_time
mouse
M01      5712.8        138.0
M02      5220.4        114.4
M03      5827.8        127.0
```


Step 2: tests
-------------------------------------------------------------------------------

Create `test_behaviour.py`. The test table contains **two** mice, so that the
first test checks precisely the trap described above: the first step of mouse
B must be 0, not a jump from the last position of mouse A.

``` python
"""Tests for behaviour.py."""

import pandas as pd

from behaviour import add_steps, load_tracking, summarize


def two_mice():
    return pd.DataFrame({
        "mouse": ["A", "A", "A", "B", "B"],
        "time":  [0.0, 1.0, 2.0, 0.0, 1.0],
        "x":     [0.0, 3.0, 3.0, 25.0, 25.0],
        "y":     [0.0, 4.0, 14.0, 25.0, 25.0],
    })


def test_steps_are_computed_within_each_mouse():
    steps = add_steps(two_mice())["step"].tolist()
    assert steps == [0.0, 5.0, 10.0, 0.0, 0.0]


def test_summary_distance_and_center_time():
    summary = summarize(add_steps(two_mice()))
    assert summary.loc["A", "distance"] == 15.0
    assert summary.loc["A", "center_time"] == 0.0     # A stays near a corner
    assert summary.loc["B", "center_time"] == 2.0     # B: 2 frames of 1 s


def test_load_tracking(tmp_path):
    pd.DataFrame({"time": [0, 1], "x": [1, 2], "y": [3, 4]}).to_csv(
        tmp_path / "M01.csv", index=False)
    pd.DataFrame({"time": [0], "x": [5], "y": [6]}).to_csv(
        tmp_path / "M02.csv", index=False)
    tracking = load_tracking(tmp_path)
    assert tracking["mouse"].tolist() == ["M01", "M01", "M02"]
    assert list(tracking.columns) == ["mouse", "time", "x", "y"]
```

``` bash
(sci) $ pytest
...
test_behaviour.py ...                                                    [100%]

============================== 3 passed in 0.18s ===============================
```

`test_load_tracking` writes two tiny files in a temporary folder given by
pytest (`tmp_path`), and checks that `load_tracking` combines them correctly.


Step 3: adding the conditions
-------------------------------------------------------------------------------

The summary knows the mice by their names only. `merge` adds the information
of `metadata.csv`, matching the rows on the column `mouse`:

``` python
>>> metadata = pd.read_csv("metadata.csv")
>>> results = summary.reset_index().merge(metadata, on="mouse")
>>> results.head(3)
  mouse  distance  center_time sex treatment cage
0   M01    5712.8        138.0   F   vehicle   C1
1   M02    5220.4        114.4   M   vehicle   C1
2   M03    5827.8        127.0   F   vehicle   C1
>>> results.groupby("treatment")["distance"].agg(["mean", "std", "count"]).round(1)
             mean    std  count
treatment
drug       3812.8  319.8      8
vehicle    5514.0  309.5      8
>>> results.pivot_table(values="distance", index="treatment", columns="sex").round(1)
sex             F       M
treatment
drug       4100.7  3524.8
vehicle    5798.0  5230.1
```

`reset_index()` turns the index (`mouse`) back into a normal column, needed
for the merge. The treated mice travel about 30 % less, in both sexes.


Step 4: the analysis script
-------------------------------------------------------------------------------

Create `analyze_experiment.py`. Besides the summary, it computes the
**activity over time**: the distance travelled in each minute, averaged for
each group.

``` python
"""Analyse a whole open-field experiment: tracking files + metadata."""

import argparse

import matplotlib.pyplot as plt
import pandas as pd

from behaviour import add_steps, load_tracking, summarize


def main():
    parser = argparse.ArgumentParser(description="Open-field experiment analysis.")
    parser.add_argument("folder", help="folder with one tracking CSV per mouse")
    parser.add_argument("metadata", help="CSV file: mouse, sex, treatment, ...")
    args = parser.parse_args()

    tracking = add_steps(load_tracking(args.folder))
    metadata = pd.read_csv(args.metadata)
    print(f"{tracking['mouse'].nunique()} mice, {len(tracking)} frames")

    results = summarize(tracking).reset_index().merge(metadata, on="mouse")
    results.to_csv("results.csv", index=False)
    print(results.groupby(["treatment", "sex"])[["distance", "center_time"]]
          .mean().round(1))

    tracking["minute"] = (tracking["time"] // 60).astype(int)
    per_minute = (tracking.groupby(["mouse", "minute"])["step"].sum()
                  .reset_index().merge(metadata, on="mouse"))
    activity = per_minute.groupby(["minute", "treatment"])["step"].mean().unstack()

    fig, axes = plt.subplots(1, 2, figsize=(11, 4))
    activity.plot(ax=axes[0], marker="o")
    axes[0].set_xlabel("Minute")
    axes[0].set_ylabel("Distance per minute (cm)")
    axes[0].set_title("Activity over time")
    results.boxplot(column="distance", by=["treatment", "sex"], ax=axes[1])
    axes[1].set_ylabel("Total distance (cm)")
    axes[1].set_title("Distance travelled")
    fig.suptitle("")
    fig.tight_layout()
    fig.savefig("experiment.png", dpi=150)
    plt.show()


if __name__ == "__main__":
    main()
```

``` bash
(sci) $ python analyze_experiment.py tracking metadata.csv
16 mice, 48000 frames
               distance  center_time
treatment sex
drug      F      4100.7        139.0
          M      3524.8        137.6
vehicle   F      5798.0        135.1
          M      5230.1        136.5
```

How the activity over time is computed:

1. `tracking["time"] // 60` gives the minute of each frame;
2. `groupby(["mouse", "minute"])["step"].sum()` gives the distance of each
   mouse in each minute;
3. after merging the metadata, `groupby(["minute", "treatment"])` averages
   the mice of each group, and `unstack()` puts the treatments in columns,
   ready to plot one line per group.

The figure shows that both groups slow down during the test (habituation),
the treated mice being less active all along. The results per mouse are saved
in `results.csv`, ready for a statistical analysis.


Going further
-------------------------------------------------------------------------------

**1.** Compute the mean distance per **cage**. What do you notice? What does
it tell you about the design of the experiment?

<details><summary>Answer</summary>

``` python
>>> results.groupby("cage")["distance"].mean().round(0)
cage
C1    5507.0
C2    5521.0
C3    3860.0
C4    3765.0
Name: distance, dtype: float64
```

Cages C1 and C2 contain only vehicle mice, C3 and C4 only treated mice: the
cage is **confounded** with the treatment. If the mice of cages C3 and C4 were
less active for another reason (noise, position in the room, a dominant
mouse...), we could not tell it apart from an effect of the drug. In a real
experiment, distribute the treatments within each cage. pandas does not
design experiments, but it makes such problems easy to spot:
`pd.crosstab(results["cage"], results["treatment"])`.
</details>

**2.** Is the difference of distance between the treatments statistically
significant? Use a Mann–Whitney test from `scipy.stats`.

<details><summary>Answer</summary>

``` python
>>> from scipy import stats
>>> drug = results.loc[results["treatment"] == "drug", "distance"]
>>> vehicle = results.loc[results["treatment"] == "vehicle", "distance"]
>>> stats.mannwhitneyu(drug, vehicle)
MannwhitneyuResult(statistic=np.float64(0.0), pvalue=np.float64(0.000155...))
```

The p-value is very small: all the treated mice travelled less than all the
control mice (hence a statistic of 0). With 8 animals per group, a
non-parametric test like this one makes few assumptions on the data. (Keep
in mind the remark of exercise 1, though!)
</details>

**3.** Compute the time spent in the center **per minute** for each
treatment, as we did for the distance.

<details><summary>Answer</summary>

``` python
margin = 50 * (1 - 0.5) / 2
tracking["in_center"] = (tracking["x"].between(margin, 50 - margin)
                         & tracking["y"].between(margin, 50 - margin))
dt = 0.2                                        # 5 frames per second
per_minute = (tracking.groupby(["mouse", "minute"])["in_center"].sum() * dt)
per_minute = per_minute.reset_index().merge(metadata, on="mouse")
per_minute.groupby(["minute", "treatment"])["in_center"].mean().unstack().plot()
```
</details>

**4.** Save the results in an Excel file with one sheet per treatment.

<details><summary>Answer</summary>

``` python
with pd.ExcelWriter("results.xlsx") as writer:
    for treatment, group in results.groupby("treatment"):
        group.to_excel(writer, sheet_name=treatment, index=False)
```

Looping over a `groupby` gives, at each iteration, the value of the group and
the sub-table of its rows.
</details>


Resources
-------------------------------------------------------------------------------

* [**Combine data from multiple tables**](https://pandas.pydata.org/docs/getting_started/intro_tutorials/08_combine_dataframes.html)
  and [**Calculate summary statistics**](https://pandas.pydata.org/docs/getting_started/intro_tutorials/06_calculate_statistics.html),
  pandas getting started tutorials.
* [**Group by: split-apply-combine**](https://pandas.pydata.org/docs/user_guide/groupby.html)
  in the pandas user guide.
* [**Tidy data**](https://vita.had.co.nz/papers/tidy-data.html), by Hadley
  Wickham: the paper explaining why "one row per observation" tables are so
  convenient (written for R, valid for pandas).
* [**scipy.stats**](https://docs.scipy.org/doc/scipy/reference/stats.html):
  statistical tests and distributions.
* [**DeepLabCut**](https://deeplabcut.github.io/DeepLabCut/): its tracking
  outputs can be read with pandas and analysed exactly like here.


<!----------------------------- External links ------------------------------->
[pandas]: https://pandas.pydata.org
<!---------------------------------------------------------------------------->
