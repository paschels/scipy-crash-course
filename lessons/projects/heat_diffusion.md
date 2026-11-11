# Project: simulating heat diffusion (and waves)
An alternative project to the Game of Life, for the scientific Python crash
course.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

Numerical simulation is at the heart of physics, chemistry, earth and
environmental sciences. This project solves a classic **partial differential
equation**, the heat equation, with the simplest numerical method (explicit
finite differences):

* discretizing an equation and writing the update with numpy slices, without
  loops over the points (like the Game of Life!);
* discovering **numerical instability**, and how to avoid it;
* **validating** a simulation against exact solutions with tests;
* running simulations with different parameters from the command line.

In *Going further*, the same method solves the **wave equation**.

**Duration**: about 1h45.


Before you start
-------------------------------------------------------------------------------

Activate the course environment and create a folder for the project:

``` bash
(base) $ conda activate sci
(sci) $ cd Documents/python-course
(sci) $ mkdir heat
(sci) $ cd heat
```

> **Coming from MATLAB?** Three differences matter here. Indices start at
> **0**, not 1. In a slice `u[a:b]`, the element `b` is **not** included. And
> `u[-1]` is the last element (MATLAB's `u(end)`). Finally, `B = A` does not
> copy an array: use `B = A.copy()`.


The model
-------------------------------------------------------------------------------

We consider a metal rod of length L. Its temperature u(x, t) evolves
according to the heat equation:

> ∂u/∂t = D ∂²u/∂x²

where D is the thermal diffusivity (in m²/s). To solve it numerically, the rod
is cut into points spaced by `dx`, and time into steps `dt`. Replacing the
derivatives by finite differences gives the update rule, for each inner point
`i`:

> u_new[i] = u[i] + r (u[i+1] − 2 u[i] + u[i−1]),  with  r = D dt / dx²

Each point moves towards the mean of its two neighbours: heat flows from hot
to cold. The two ends are kept at fixed temperatures (*boundary conditions*).

This scheme is simple, but it has one condition: it is **stable only if
r ≤ 0.5**. We'll see what happens otherwise.


Step 1: one time step
-------------------------------------------------------------------------------

Create a file `heat.py`:

``` python
"""Simulation of heat diffusion in a rod (1D heat equation)."""

import numpy as np


def stability_number(D, dt, dx):
    """Return r = D dt / dx**2. The explicit scheme is stable only if r <= 0.5."""
    return D * dt / dx**2


def step(u, r):
    """Advance the temperature profile by one time step.

    Explicit finite-difference scheme: each point moves towards the mean of
    its two neighbours. The two end points are kept fixed (boundary
    conditions: imposed temperatures).

    Parameters
    ----------
    u : numpy.ndarray
        Temperature at each point of the rod.
    r : float
        Stability number D dt / dx**2.

    Returns
    -------
    numpy.ndarray
        The new temperature profile.
    """
    new = u.copy()
    new[1:-1] = u[1:-1] + r * (u[2:] - 2 * u[1:-1] + u[:-2])
    return new


def simulate(u0, r, n_steps, every=1):
    """Run the simulation and return the profiles saved every `every` steps."""
    u = u0.copy()
    saved = [u.copy()]
    for i in range(1, n_steps + 1):
        u = step(u, r)
        if i % every == 0:
            saved.append(u.copy())
    return np.array(saved)
```

The whole physics is in **one line** of `step`:

``` python
new[1:-1] = u[1:-1] + r * (u[2:] - 2 * u[1:-1] + u[:-2])
```

`u[1:-1]` are the inner points, `u[2:]` their right neighbours and `u[:-2]`
their left neighbours: three arrays of the same size, shifted by one point.
This is exactly the trick used to count the neighbours in the Game of Life.
The first and last points of `new` are not touched: they keep their imposed
temperatures.

Note `u.copy()` in `step` and `simulate`: without it, we would modify the
array given as argument, a classic source of bugs.


Step 2: tests against known solutions
-------------------------------------------------------------------------------

How do you know a simulation is right? By checking it on cases where the
answer is known exactly. The heat equation has several:

* a **uniform** temperature does not change;
* with fixed ends, the temperature tends to a **linear profile** between the
  two end temperatures (the steady state);
* a **sine-shaped** profile `sin(πx/L)` keeps its shape and decays
  exponentially as `exp(−D π² t / L²)`.

Create `test_heat.py`:

``` python
"""Tests for heat.py."""

import numpy as np

from heat import simulate, stability_number, step


def test_uniform_profile_does_not_change():
    u = np.full(20, 37.0)
    assert np.allclose(step(u, 0.4), u)


def test_boundaries_are_fixed():
    u = np.linspace(0, 1, 11) ** 2
    new = step(u, 0.4)
    assert new[0] == u[0] and new[-1] == u[-1]


def test_converges_to_linear_profile():
    u0 = np.zeros(21)
    u0[0] = 100.0
    final = simulate(u0, 0.5, 5000)[-1]
    assert np.allclose(final, np.linspace(100, 0, 21), atol=0.01)


def test_sine_mode_decays_as_expected():
    # u = sin(pi x / L) decays as exp(-D pi^2 t / L^2)
    L, D, n = 1.0, 1e-4, 101
    x = np.linspace(0, L, n)
    dx = x[1] - x[0]
    dt = 0.4
    r = stability_number(D, dt, dx)
    n_steps = 2500
    final = simulate(np.sin(np.pi * x / L), r, n_steps)[-1]
    expected = np.sin(np.pi * x / L) * np.exp(-D * np.pi**2 * n_steps * dt / L**2)
    assert np.allclose(final, expected, atol=1e-3)
```

``` bash
(sci) $ pytest
...
test_heat.py ....                                                        [100%]

============================== 4 passed in 0.07s ===============================
```

The last test compares the simulation to the exact solution after 2500 steps.
It passes with a tolerance of 0.001 (`atol`): the numerical solution is an
approximation, but a good one.


Step 3: running simulations
-------------------------------------------------------------------------------

Create `run_heat.py`: a rod at 20 °C whose left end is suddenly heated to
100 °C, while the right end stays at 20 °C.

``` python
"""Simulate and plot heat diffusion in a rod."""

import argparse

import matplotlib.pyplot as plt
import numpy as np

from heat import simulate, stability_number


def main():
    parser = argparse.ArgumentParser(description="1D heat diffusion.")
    parser.add_argument("--length", type=float, default=1.0, help="rod length (m)")
    parser.add_argument("--points", type=int, default=101, help="number of points")
    parser.add_argument("--diffusivity", type=float, default=1e-4,
                        help="thermal diffusivity D (m2/s)")
    parser.add_argument("--dt", type=float, default=0.4, help="time step (s)")
    parser.add_argument("--duration", type=float, default=3000,
                        help="simulated duration (s)")
    args = parser.parse_args()

    x = np.linspace(0, args.length, args.points)
    dx = x[1] - x[0]
    r = stability_number(args.diffusivity, args.dt, dx)
    print(f"r = {r:.3f}", "(stable)" if r <= 0.5 else "(UNSTABLE!)")

    u0 = np.full(args.points, 20.0)             # rod at 20 °C...
    u0[0] = 100.0                               # ...left end heated at 100 °C
    n_steps = int(args.duration / args.dt)
    profiles = simulate(u0, r, n_steps, every=n_steps // 5)

    fig, ax = plt.subplots(figsize=(8, 4))
    for k, profile in enumerate(profiles):
        ax.plot(x, profile, label=f"t = {k * (n_steps // 5) * args.dt:.0f} s")
    ax.set_xlabel("Position (m)")
    ax.set_ylabel("Temperature (°C)")
    ax.legend()
    fig.savefig("heat.png", dpi=150)
    plt.show()


if __name__ == "__main__":
    main()
```

`np.full(n, 20.0)` creates an array of `n` values all equal to 20.
`every=n_steps // 5` saves 6 profiles regularly spaced in time (`//` is the
integer division).

``` bash
(sci) $ python run_heat.py
r = 0.400 (stable)
```

The figure shows the heat progressively entering the rod. Run longer
simulations (`--duration 20000`): the profile tends to the straight line
between 100 °C and 20 °C, as our test predicted.


Step 4: numerical instability
-------------------------------------------------------------------------------

To go faster, it is tempting to take a larger time step. Try:

``` bash
(sci) $ python run_heat.py --dt 0.6 --duration 60
r = 0.600 (UNSTABLE!)
```

The figure is absurd: after only 100 steps, temperatures reach ±10¹³ °C!

``` python
>>> import numpy as np
>>> from heat import simulate
>>> u0 = np.full(101, 20.0)
>>> u0[0] = 100
>>> simulate(u0, 0.6, 100)[-1][:4]
array([ 1.00000000e+02, -8.12782141e+12,  1.59773236e+13, -2.32859175e+13])
```

This is not a bug in the code: the **numerical method** itself amplifies small
oscillations at each step when r > 0.5. The program runs without any error
message, and produces nonsense. Lessons for all numerical work:

* know the **stability conditions** of your methods, and check them in the
  code (as `run_heat.py` does with its message);
* **always plot** your results: absurd values are obvious on a figure;
* test against **known solutions** (step 2).


Going further
-------------------------------------------------------------------------------

**1.** Modify the boundary conditions to simulate an **insulated** rod (no
heat flows through the ends): at each step, set `new[0] = new[1]` and
`new[-1] = new[-2]`. Write a test checking that the total amount of heat,
`u.sum()`, is then (almost) conserved.

<details><summary>Answer</summary>

``` python
def step_insulated(u, r):
    """One time step with insulated ends (zero heat flux)."""
    new = u.copy()
    new[1:-1] = u[1:-1] + r * (u[2:] - 2 * u[1:-1] + u[:-2])
    new[0] = new[1]
    new[-1] = new[-2]
    return new


def test_insulated_rod_conserves_heat():
    u = np.zeros(51)
    u[20:30] = 100.0
    total = u.sum()
    for _ in range(1000):
        u = step_insulated(u, 0.4)
    assert abs(u.sum() - total) / total < 0.05
```

The conservation is only approximate with this simple treatment of the
boundaries: a good opportunity to discuss discretization errors.
</details>

**2.** Simulate diffusion in a **2D plate**, displayed with `imshow`. The
update uses the four neighbours (up, down, left, right):

> u_new = u + r (u_up + u_down + u_left + u_right − 4 u)

and is stable for r ≤ 0.25.

<details><summary>Answer</summary>

``` python
import matplotlib.pyplot as plt
import numpy as np


def step2d(U, r):
    """One time step of 2D diffusion, with fixed borders."""
    new = U.copy()
    new[1:-1, 1:-1] = U[1:-1, 1:-1] + r * (
        U[2:, 1:-1] + U[:-2, 1:-1] + U[1:-1, 2:] + U[1:-1, :-2]
        - 4 * U[1:-1, 1:-1])
    return new


U = np.zeros((50, 50))
U[20:30, 20:30] = 100.0           # a hot square in the middle
for _ in range(200):
    U = step2d(U, 0.2)
plt.imshow(U, cmap="inferno")
plt.colorbar(label="Temperature")
plt.show()
```

The slices are exactly those of `step_numpy` in the Game of Life, without the
diagonals.
</details>

**3.** The **wave equation** ∂²u/∂t² = c² ∂²u/∂x² describes vibrating strings,
sound, or waves on water. Its explicit scheme needs the two previous time
steps:

> u_new = 2 u − u_old + C² (u[i+1] − 2 u[i] + u[i−1]),  with  C = c dt / dx

and is stable for C ≤ 1. Simulate a Gaussian pulse in a string with fixed
ends. What do you observe?

<details><summary>Answer</summary>

``` python
import matplotlib.pyplot as plt
import numpy as np


def wave_step(u, u_old, C2):
    """One time step of the 1D wave equation, with fixed ends."""
    new = u.copy()
    new[1:-1] = 2 * u[1:-1] - u_old[1:-1] + C2 * (u[2:] - 2 * u[1:-1] + u[:-2])
    return new


x = np.linspace(0, 1, 201)
u0 = np.exp(-((x - 0.3) / 0.03) ** 2)      # a pulse at x = 0.3
u_old, u = u0.copy(), u0.copy()            # initially at rest
C2 = 0.9 ** 2
plt.plot(x, u0, label="t = 0")
for step in range(1, 301):
    u, u_old = wave_step(u, u_old, C2), u
    if step % 100 == 0:
        plt.plot(x, u, label=f"step {step}")
plt.legend()
plt.show()
```

The pulse splits into two pulses of half height travelling in opposite
directions. When a pulse reaches a fixed end, it is **reflected upside down**.
After 100 steps, one pulse is at x = 0.75 and the other, already reflected,
is at x = 0.15, inverted. Note the line `u, u_old = wave_step(...), u`: both
variables are updated at once, using the old values.
</details>

**4.** Check the precision of the method: for the sine profile of the tests,
compute the maximum error with respect to the exact solution for several
values of `dx` (e.g. 21, 41, 81 and 161 points), keeping r = 0.4. How does
the error decrease when `dx` is divided by 2?

<details><summary>Answer</summary>

``` python
import numpy as np

from heat import simulate

D, L, r, T = 1e-4, 1.0, 0.4, 1000.0
for n in [21, 41, 81, 161]:
    x = np.linspace(0, L, n)
    dx = x[1] - x[0]
    dt = r * dx**2 / D
    n_steps = int(T / dt)
    final = simulate(np.sin(np.pi * x / L), r, n_steps)[-1]
    exact = np.sin(np.pi * x / L) * np.exp(-D * np.pi**2 * n_steps * dt / L**2)
    print(f"{n:4d} points: max error {np.abs(final - exact).max():.2e}")
```

The error is divided by about 4 each time `dx` is divided by 2: the method is
of **order 2** in space. Checking the order of convergence is a standard way
to validate a numerical code.
</details>


Resources
-------------------------------------------------------------------------------

* [**12 steps to Navier–Stokes**](https://lorenabarba.com/blog/cfd-python-12-steps-to-navier-stokes/)
  by Lorena Barba: a famous series of Jupyter notebooks, from the simplest
  equation to fluid dynamics, with the same method as this project.
* [**scipy.integrate**](https://docs.scipy.org/doc/scipy/reference/integrate.html):
  ready-made solvers for ordinary differential equations (`solve_ivp`).
* [**Scientific Python Lectures**](https://lectures.scientific-python.org):
  chapters on SciPy and on optimizing numerical code.
* [**From Python to Numpy**](https://www.labri.fr/perso/nrougier/from-python-to-numpy/)
  by Nicolas P. Rougier: vectorization of simulations (including
  reaction-diffusion and fluids).
* [**NumPy for MATLAB users**](https://numpy.org/doc/stable/user/numpy-for-matlab-users.html):
  a correspondence table between MATLAB and NumPy.
