# Project: analysing mouse behaviour in an open field
An alternative project to the Game of Life, for the scientific Python crash
course.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

Behavioural setups and tracking software (EthoVision, DeepLabCut, Bonsai,
home-made Python setups...) all end up producing the same kind of data: the
**position of the animal over time**, usually as CSV files. In this project,
we analyse an **open-field test**, where a mouse freely explores a square
arena:

* simulating realistic trajectories for two groups of mice;
* computing classic measures: **distance travelled** and **time spent in the
  center** of the arena (an indicator of anxiety);
* processing all the recordings of an experiment with one script, and
  comparing the groups;
* testing the measures on trajectories where we know the answer.

Everything uses what we've seen in the course: functions, loops, numpy,
matplotlib, `argparse` and pytest.

**Duration**: about 1h45.


Before you start
-------------------------------------------------------------------------------

Activate the course environment and create a folder for the project:

``` bash
(base) $ conda activate sci
(sci) $ cd Documents/python-course
(sci) $ mkdir openfield
(sci) $ cd openfield
```

Many behavioural setups come with **Jupyter notebooks**: once this project is
done, try to rewrite step 4 as a notebook in JupyterLab, and compare.


Step 1: simulating a mouse
-------------------------------------------------------------------------------

Real recordings are not always at hand, and simulated data have a big
advantage: we know exactly how they were produced. Create a file
`openfield.py`:

``` python
"""Analysis of mouse trajectories in an open-field test."""

import numpy as np


def simulate_trajectory(rng, duration=300, fps=10, arena=50, speed=8):
    """Simulate the trajectory of a mouse exploring a square arena.

    Parameters
    ----------
    rng : numpy.random.Generator
        Random generator.
    duration : float
        Duration of the recording (s).
    fps : int
        Number of positions per second.
    arena : float
        Side of the square arena (cm).
    speed : float
        Mean walking speed (cm/s).

    Returns
    -------
    t, x, y : numpy.ndarray
        Time (s) and position (cm) of the mouse.
    """
    n = int(duration * fps)
    t = np.arange(n) / fps
    x = np.empty(n)
    y = np.empty(n)
    x[0], y[0] = arena / 2, arena / 2
    angle = rng.uniform(0, 2 * np.pi)
    for i in range(1, n):
        angle += rng.normal(0, 0.4)                          # smooth turns
        step = rng.exponential(speed / fps)                  # variable speed
        x[i] = x[i - 1] + step * np.cos(angle)
        y[i] = y[i - 1] + step * np.sin(angle)
        if not (0 <= x[i] <= arena and 0 <= y[i] <= arena):  # hit a wall
            x[i] = np.clip(x[i], 0, arena)
            y[i] = np.clip(y[i], 0, arena)
            angle += np.pi / 2 + rng.uniform(0, np.pi)       # turn away
    return t, x, y


def distance_travelled(x, y):
    """Return the total length of the path (same unit as x and y)."""
    return np.sum(np.hypot(np.diff(x), np.diff(y)))


def time_in_center(t, x, y, arena=50, center=0.5):
    """Return the time spent in the central zone of the arena.

    The central zone is a square of side center * arena, in the middle.
    """
    margin = arena * (1 - center) / 2
    inside = ((x > margin) & (x < arena - margin) &
              (y > margin) & (y < arena - margin))
    dt = t[1] - t[0]
    return inside.sum() * dt
```

`simulate_trajectory` is a **random walk**: at each frame, the mouse turns a
little (a random change of `angle`) and moves forward by a random step. When
it hits a wall, it is put back on the wall (`np.clip` limits values to an
interval) and turns away. The parameter `speed` controls how much the mouse
moves: we'll use it to simulate a "treated" group less active than the
control group.

This function uses a loop over frames, because each position depends on the
previous one. On the other hand, the analysis functions work on whole arrays,
without loops:

* `np.diff(x)` gives the differences between consecutive values, i.e. the
  displacements between frames, and `np.hypot(dx, dy)` computes
  `sqrt(dx**2 + dy**2)`, the length of each displacement. Their sum is the
  distance travelled;
* in `time_in_center`, a boolean mask tells, for each frame, whether the mouse
  is in the central square. `inside.sum()` counts the `True` values, i.e. the
  frames spent in the center; multiplied by the duration of a frame, it gives
  a time.

Look at a trajectory:

``` python
>>> import matplotlib.pyplot as plt
>>> import numpy as np
>>> from openfield import simulate_trajectory
>>> t, x, y = simulate_trajectory(np.random.default_rng(0))
>>> plt.plot(x, y, linewidth=0.5)
>>> plt.gca().set_aspect("equal")    # same scale on both axes
>>> plt.show()
```


Step 2: tests
-------------------------------------------------------------------------------

Measures on trajectories are easy to test with paths we can compute by hand:
a square of side 10 has a perimeter of 40, a 3-4-5 triangle has a hypotenuse
of 5... Create `test_openfield.py`:

``` python
"""Tests for openfield.py."""

import numpy as np

from openfield import distance_travelled, simulate_trajectory, time_in_center


def test_distance_of_a_square():
    x = np.array([0, 10, 10, 0, 0])
    y = np.array([0, 0, 10, 10, 0])
    assert distance_travelled(x, y) == 40


def test_distance_diagonal():
    assert np.isclose(distance_travelled(np.array([0, 3]), np.array([0, 4])), 5)


def test_time_in_center():
    t = np.arange(4) * 0.5                  # 4 frames, 0.5 s apart
    x = np.array([25, 25, 2, 2])            # 2 frames in the center
    y = np.array([25, 25, 2, 2])
    assert time_in_center(t, x, y) == 1.0


def test_simulated_mouse_stays_in_arena():
    rng = np.random.default_rng(0)
    t, x, y = simulate_trajectory(rng, duration=60)
    assert x.min() >= 0 and x.max() <= 50
    assert y.min() >= 0 and y.max() <= 50
```

``` bash
(sci) $ pytest
...
test_openfield.py ....                                                   [100%]

============================== 4 passed in 0.05s ===============================
```

The last test checks a property of the simulation itself: the mouse never
leaves the arena, whatever the random numbers.


Step 3: an experiment
-------------------------------------------------------------------------------

Save this script as `generate_mice.py` and run it. It simulates a 5-minute
recording at 10 frames per second for 6 control mice and 6 treated mice,
less active, and saves each recording in a CSV file with three columns
(`time,x,y`), like a tracking software would:

``` python
"""Simulate open-field recordings for two groups of mice."""

from pathlib import Path

import numpy as np

from openfield import simulate_trajectory

rng = np.random.default_rng(seed=11)
folder = Path("tracking")
folder.mkdir(exist_ok=True)
for group, speed in [("control", 8), ("treated", 5)]:
    for mouse in range(1, 7):
        t, x, y = simulate_trajectory(rng, speed=speed)
        filename = folder / f"{group}_mouse{mouse}.csv"
        np.savetxt(filename, np.column_stack([t, x, y]), delimiter=",",
                   fmt="%.2f", header="time,x,y", comments="")
print("Written 12 files in", folder)
```

``` bash
(sci) $ python generate_mice.py
Written 12 files in tracking
```


Step 4: analysing all the recordings
-------------------------------------------------------------------------------

Create `analyze_mice.py`:

``` python
"""Analyse all the open-field recordings of a folder."""

import argparse
from pathlib import Path

import matplotlib.pyplot as plt
import numpy as np

from openfield import distance_travelled, time_in_center


def main():
    parser = argparse.ArgumentParser(description="Open-field analysis.")
    parser.add_argument("folder", help="folder with time,x,y CSV files")
    args = parser.parse_args()

    results = {}                                  # group -> list of distances
    with open("summary.csv", "w") as output:
        output.write("mouse,distance_cm,center_s\n")
        for filename in sorted(Path(args.folder).glob("*.csv")):
            t, x, y = np.loadtxt(filename, delimiter=",", skiprows=1, unpack=True)
            distance = distance_travelled(x, y)
            center = time_in_center(t, x, y)
            output.write(f"{filename.stem},{distance:.1f},{center:.1f}\n")
            print(f"{filename.stem:16s} distance {distance:7.1f} cm   "
                  f"center {center:5.1f} s")
            group = filename.stem.split("_")[0]
            if group not in results:
                results[group] = []
            results[group].append(distance)

    fig, axes = plt.subplots(1, 2, figsize=(10, 4))
    t, x, y = np.loadtxt(sorted(Path(args.folder).glob("*.csv"))[0],
                         delimiter=",", skiprows=1, unpack=True)
    axes[0].plot(x, y, linewidth=0.5)
    axes[0].set_aspect("equal")
    axes[0].set_title("Trajectory of the first mouse")
    axes[1].boxplot(list(results.values()))
    axes[1].set_xticks(range(1, len(results) + 1), list(results))
    axes[1].set_ylabel("Distance travelled (cm)")
    fig.tight_layout()
    fig.savefig("openfield.png", dpi=150)
    plt.show()


if __name__ == "__main__":
    main()
```

A few details:

* `np.loadtxt(..., unpack=True)` returns the columns separately, so that
  `t, x, y = ...` gets the three columns directly;
* the group of each mouse is read from its file name:
  `"control_mouse1".split("_")` gives `["control", "mouse1"]`, and `[0]` takes
  the first part. **Choosing good file names** makes analyses much easier;
* `results` is a dictionary whose values are lists: for each group, the list
  of the distances of its mice;
* `ax.boxplot` draws one box per group, summarizing the distribution of the
  values (median, quartiles, extremes).

``` bash
(sci) $ python analyze_mice.py tracking
control_mouse1   distance  2331.9 cm   center  60.1 s
control_mouse2   distance  2357.7 cm   center  78.8 s
...
treated_mouse1   distance  1495.2 cm   center  57.8 s
treated_mouse2   distance  1503.4 cm   center  93.2 s
...
```

The summary is saved in `summary.csv`, and the figure in `openfield.png`.
The treated mice travel clearly less, as expected since we simulated them
slower. On the other hand, the time spent in the center does not differ
clearly between the groups: our simulated mice have no anxiety!

With real recordings, only the reading of the files may need to be adapted
(column names, units, header lines): the analysis functions stay the same.


Going further
-------------------------------------------------------------------------------

**1.** Compute the instantaneous **speed** of the mouse at each frame (in
cm/s), and plot it as a function of time. What is the mean speed?

<details><summary>Answer</summary>

``` python
import matplotlib.pyplot as plt
import numpy as np

t, x, y = np.loadtxt("tracking/control_mouse1.csv", delimiter=",",
                     skiprows=1, unpack=True)
speed = np.hypot(np.diff(x), np.diff(y)) / np.diff(t)
print("Mean speed:", speed.mean(), "cm/s")
plt.plot(t[1:], speed)
plt.xlabel("Time (s)")
plt.ylabel("Speed (cm/s)")
plt.show()
```

`speed` has one element less than `t`: there is one displacement between two
consecutive frames, hence `t[1:]`.
</details>

**2.** Plot an **occupancy heatmap**: the time spent by the mouse in each
region of the arena. Hint: `np.histogram2d` or `ax.hist2d`.

<details><summary>Answer</summary>

``` python
fig, ax = plt.subplots()
counts, xedges, yedges, image = ax.hist2d(x, y, bins=20, range=[[0, 50], [0, 50]])
fig.colorbar(image, ax=ax, label="frames")
ax.set_aspect("equal")
plt.show()
```

Mice usually spend more time along the walls (*thigmotaxis*): our simulated
mice do too, because they are blocked by the walls.
</details>

**3.** Write a function `immobility_time(t, x, y, threshold=1.0)` returning
the total time during which the speed of the mouse is below a threshold (in
cm/s). Add a test for it.

<details><summary>Answer</summary>

``` python
def immobility_time(t, x, y, threshold=1.0):
    """Return the time (s) during which the speed is below threshold (cm/s)."""
    speed = np.hypot(np.diff(x), np.diff(y)) / np.diff(t)
    return (speed < threshold).sum() * (t[1] - t[0])
```

and the test, with a mouse that does not move during the first second:

``` python
def test_immobility_time():
    t = np.arange(5) * 0.5
    x = np.array([10, 10, 10, 20, 30])
    y = np.zeros(5)
    assert immobility_time(t, x, y) == 1.0
```
</details>

**4.** Real tracking data contain errors: frames where the animal was not
detected (often saved as `NaN`, *Not a Number*), or sudden jumps. What happens
to `distance_travelled` if one position is `NaN`? How can you fix it?

<details><summary>Answer</summary>

Any computation involving `NaN` gives `NaN`, so the total distance becomes
`NaN`. Two solutions: ignore the missing displacements with
`np.nansum` instead of `np.sum`, or fill the gaps by interpolation before
computing. For example, to remove the frames where `x` or `y` is missing:

``` python
valid = ~np.isnan(x) & ~np.isnan(y)
x, y, t = x[valid], y[valid], t[valid]
```

Always count how many frames are missing: if there are too many, the measure
is not reliable.
</details>


Resources
-------------------------------------------------------------------------------

* [**DeepLabCut**](https://deeplabcut.github.io/DeepLabCut/), a deep-learning
  tool to track body parts of animals in videos, written in Python. Its
  outputs (CSV files of positions) can be analysed exactly like in this
  project.
* [**Bonsai**](https://bonsai-rx.org/), a visual programming tool widely used
  to run behavioural experiments, which can export tracking data.
* [**NumPy for absolute beginners**](https://numpy.org/doc/stable/user/absolute_beginners.html),
  for `np.diff`, boolean masks and statistics.
* [**Matplotlib gallery**](https://matplotlib.org/stable/gallery/index.html),
  for heatmaps and box plots.
