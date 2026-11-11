# Alternative projects

Projects for the last session of the scientific Python crash course, as
alternatives to the Game of Life. Pick the one closest to your own work; the
others make good exercises to practice at home.

All the projects have the same level and the same structure as the Game of
Life: a small module of documented functions, tests with pytest, a script
with command-line options producing results and figures, and a *Going
further* section with answers. They all start from **simulated data**, so
that everybody can work without an instrument or a dataset, and they all
explain how to switch to **real data**.

This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


## Projects with numpy and matplotlib

### [Conway's Game of Life](documentation_tests_game_of_life.md#project-the-game-of-life) <img src="https://img.shields.io/badge/1H45-numpy-306998.svg?style=flat-square" align="right"/>

The reference project of the course. Simulate a cellular automaton, first in
pure Python with lists, then vectorized with numpy (about 50 times faster),
test it on known patterns, and animate it with matplotlib.  
**For you if** you want to learn vectorization, or simply have fun.

### [Automating an instrument](instrument_logging.md) <img src="https://img.shields.io/badge/1H45-numpy-306998.svg?style=flat-square" align="right"/>

Read a (simulated) temperature probe at regular intervals, raise an alarm,
store every measurement in an SQLite database, query it in SQL and plot the
results. Explains how to drive real instruments with pyserial and PyVISA.  
**For you if** you want to automate equipment, collect data, interface
experimental setups or build databases.

### [Managing a research programme](research_programme.md) <img src="https://img.shields.io/badge/1H45-numpy-306998.svg?style=flat-square" align="right"/>

Check and summarize the participants of a programme of 50 projects (CSV
files): counts per project and institute, duplicated registrations, missing
emails, a report with a figure. Includes Python formulas in Grist.  
**For you if** you coordinate projects, manage administrative data, or use
Grist.

### [Counting cells in images](counting_cells.md) <img src="https://img.shields.io/badge/1H45-numpy-306998.svg?style=flat-square" align="right"/>

Threshold, label and measure bright objects in microscopy images with
`scipy.ndimage`, process a whole folder, and check the results visually.
Explains how to read real TIFF images.  
**For you if** you process images or build analysis pipelines, as with
ImageJ/Fiji macros.

### [Mouse behaviour in an open field](open_field.md) <img src="https://img.shields.io/badge/1H45-numpy-306998.svg?style=flat-square" align="right"/>

Analyse animal trajectories: distance travelled, time spent in the center of
the arena, comparison of two groups, speed and occupancy heatmaps.  
**For you if** you analyse behavioural or tracking data (EthoVision,
DeepLabCut, notebooks provided with a setup).

### [Heat diffusion (and waves)](heat_diffusion.md) <img src="https://img.shields.io/badge/1H45-numpy-306998.svg?style=flat-square" align="right"/>

Solve the heat equation with finite differences, discover numerical
instability, validate the simulation against exact solutions, then extend it
to 2D and to the wave equation.  
**For you if** you run simulations or numerical computations (physics, earth
sciences, modelling), especially coming from MATLAB.


## Projects with pandas

### [A research programme with pandas](programme_pandas.md) <img src="https://img.shields.io/badge/1H45-pandas-150458.svg?style=flat-square" align="right"/>

Merge the participants with the description of the projects, clean the data,
summarize with `groupby` and cross tables, and produce an Excel report with
several sheets.  
**For you if** you prepare reports or exchange tables with spreadsheets and
Grist.

### [Monitoring with time series](timeseries_pandas.md) <img src="https://img.shields.io/badge/1H45-pandas-150458.svg?style=flat-square" align="right"/>

Check two weeks of data-logger measurements: detect gaps and glitches,
resample by hour, day or week, compute time above a limit, and see how
missing data can bias statistics.  
**For you if** you work with data loggers, sensors, instruments or
monitoring stations.

### [A whole behavioural experiment](experiment_pandas.md) <img src="https://img.shields.io/badge/1H45-pandas-150458.svg?style=flat-square" align="right"/>

Combine 16 tracking files with a metadata table, compute per-animal measures,
compare groups, follow the activity over time, and spot a flaw in the design
of the experiment.  
**For you if** you analyse experiments with many recordings and conditions
(behaviour, recording sessions, plates...).


## What they all teach

Whatever the topic, the same habits come back in every project, because they
make scientific code reliable: **isolate** each task in a small documented
function, **test** it on cases where you know the answer, **look** at your
results, and **automate** with a script that can be run again in one command
when the data change.

Many of you already use AI assistants to write Python. They are useful, but
they make mistakes that look plausible. The skills of this course are exactly
what you need to use them safely: read the code, understand the error
messages, and test the result. A good exercise: ask an assistant to write one
of the functions of your project, then check it with the tests of the
project.
