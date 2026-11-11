# Project: monitoring with time series in pandas
An alternative project to the Game of Life, for the scientific Python crash
course. It uses [pandas], introduced at the beginning of the last session.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

Data loggers, sensors, instruments and monitoring stations produce **time
series**: measurements with a timestamp, at regular intervals, for days or
months. pandas is particularly good with them. In this project, we check and
summarize two weeks of measurements from a logger in a lab room:

* reading timestamps as **real dates**, and selecting periods by date;
* detecting the **gaps** in the recording (logger offline);
* removing **spikes** (electrical glitches) with a rolling median;
* **resampling**: hourly, daily or weekly statistics in one line;
* producing a report with a figure, from the command line.

The same tools apply to any time series: reactor temperature, pressure,
weather station, river level, power consumption...

**Duration**: about 1h45.


Before you start
-------------------------------------------------------------------------------

Activate the course environment, check that pandas is installed (it was
installed at the beginning of the session), and create a folder for the
project:

``` bash
(base) $ conda activate sci
(sci) $ conda install pandas
(sci) $ cd Documents/python-course
(sci) $ mkdir logger-pandas
(sci) $ cd logger-pandas
```

Save this script as `generate_logger.py` and run it. It creates `logger.csv`:
a temperature and a humidity every 10 minutes for two weeks, with a daily
cycle, a slow drift of the temperature, 6 glitches at 85 °C, and 6 hours
during which the logger was offline:

``` python
"""Generate two weeks of fake measurements from a data logger in a lab room."""

import numpy as np
import pandas as pd

rng = np.random.default_rng(seed=21)
time = pd.date_range("2026-03-02", "2026-03-16", freq="10min", inclusive="left")
hours = (time.hour + time.minute / 60).to_numpy()
day = np.arange(len(time)) / 144                       # days since the start

temperature = (21 + 2 * np.sin(2 * np.pi * (hours - 9) / 24)  # daily cycle
               + 0.15 * day                                    # slow drift
               + rng.normal(0, 0.2, len(time)))
humidity = 45 - 5 * np.sin(2 * np.pi * (hours - 9) / 24) + rng.normal(0, 1, len(time))

data = pd.DataFrame({"timestamp": time,
                     "temperature": temperature.round(2),
                     "humidity": humidity.round(1)})
spikes = rng.choice(len(data), 6, replace=False)       # electrical glitches
data.loc[spikes, "temperature"] = 85.0
data = data.drop(index=range(1000, 1036))              # logger offline for 6 h
data.to_csv("logger.csv", index=False)
print("Written logger.csv with", len(data), "rows")
```

``` bash
(sci) $ python generate_logger.py
Written logger.csv with 1980 rows
(sci) $ head -3 logger.csv
timestamp,temperature,humidity
2026-03-02 00:00:00,19.66,48.3
2026-03-02 00:10:00,19.83,48.6
```

(On Windows, open the file in a text editor instead of using `head`.)


Step 1: dates as index
-------------------------------------------------------------------------------

Create a file `timeseries.py`:

``` python
"""Tools to check and summarize time series from a data logger."""

import pandas as pd


def load(filename):
    """Read a logger CSV file, with the timestamps as index (sorted)."""
    data = pd.read_csv(filename, parse_dates=["timestamp"], index_col="timestamp")
    return data.sort_index()


def find_gaps(index, expected="10min"):
    """Return the gaps of a time index larger than the expected interval.

    Returns
    -------
    pandas.DataFrame
        One row per gap: the last time before the gap, the first time after
        it, and the duration between them.
    """
    steps = index.to_series().diff()
    gaps = steps[steps > pd.Timedelta(expected)]
    return pd.DataFrame({"start": gaps.index - gaps,
                         "end": gaps.index,
                         "duration": gaps}).reset_index(drop=True)


def remove_spikes(series, window=7, threshold=3.0):
    """Replace isolated outliers by NaN.

    A value is an outlier when it differs from the rolling median of its
    neighbours by more than `threshold` (in the units of the series).
    """
    median = series.rolling(window, center=True, min_periods=1).median()
    return series.where((series - median).abs() <= threshold)


def daily_summary(series):
    """Return the minimum, mean and maximum of each day."""
    return series.resample("D").agg(["min", "mean", "max"]).round(2)


def time_above(series, limit, step="10min"):
    """Return the total time during which the series is above a limit."""
    return (series > limit).sum() * pd.Timedelta(step)
```

In `load`, two options of `read_csv` make all the difference:

* `parse_dates=["timestamp"]` converts the text `2026-03-02 00:10:00` into a
  real date, on which pandas can compute;
* `index_col="timestamp"` uses the dates as the **index** of the table,
  instead of the row numbers. The table is then a time series, and rows can be
  selected by date.

Try it:

``` python
>>> from timeseries import load
>>> data = load("logger.csv")
>>> data.head(3)
                     temperature  humidity
timestamp
2026-03-02 00:00:00        19.66      48.3
2026-03-02 00:10:00        19.83      48.6
2026-03-02 00:20:00        19.11      49.3
>>> data.loc["2026-03-05 12:00":"2026-03-05 12:30"]    # a period
                     temperature  humidity
timestamp
2026-03-05 12:00:00        23.25      39.1
2026-03-05 12:10:00        23.25      40.1
2026-03-05 12:20:00        23.07      42.8
2026-03-05 12:30:00        22.90      40.8
>>> data["temperature"].max()
np.float64(85.0)
```

A maximum of 85 °C in a lab room? Those are the glitches. And `data.loc["2026-03-08"]`
selects a whole day: dates can be given with any precision.

The other functions of the module:

* `find_gaps`: `index.to_series().diff()` computes the time between two
  consecutive measurements. Normally 10 minutes; anything longer is a gap;
* `remove_spikes`: `rolling(7, center=True).median()` computes, for each
  point, the median of the 7 points around it. A glitch is a value far from
  this local median. `series.where(condition)` keeps the values where the
  condition is true and replaces the others by `NaN`;
* `daily_summary`: `resample("D")` groups the measurements **by day**
  (`"h"` by hour, `"W"` by week, `"MS"` by month...), and `agg` computes
  several statistics per group;
* `time_above`: counts the measurements above a limit, and converts the
  count into a duration with `pd.Timedelta`.

``` python
>>> from timeseries import find_gaps, remove_spikes, daily_summary, time_above
>>> find_gaps(data.index)
                start                 end        duration
0 2026-03-08 22:30:00 2026-03-09 04:40:00 0 days 06:10:00
>>> clean = remove_spikes(data["temperature"])
>>> clean.isna().sum(), clean.max()
(np.int64(6), np.float64(25.47))
>>> daily_summary(clean).head(3)
              min   mean    max
timestamp
2026-03-02  18.74  21.08  23.28
2026-03-03  18.97  21.19  23.64
2026-03-04  19.20  21.41  23.74
>>> time_above(clean, 23)
Timedelta('4 days 14:30:00')
```


Step 2: tests
-------------------------------------------------------------------------------

Time series functions are easy to test on tiny series built with
`pd.date_range`, which creates regular timestamps. Create
`test_timeseries.py`:

``` python
"""Tests for timeseries.py."""

import numpy as np
import pandas as pd

from timeseries import daily_summary, find_gaps, remove_spikes, time_above


def regular_index(n, start="2026-01-01"):
    return pd.date_range(start, periods=n, freq="10min")


def test_no_gap_in_regular_index():
    assert len(find_gaps(regular_index(50))) == 0


def test_find_one_gap():
    index = regular_index(50).delete(range(10, 16))      # remove 1 hour
    gaps = find_gaps(index)
    assert len(gaps) == 1
    assert gaps.loc[0, "duration"] == pd.Timedelta("70min")
    assert gaps.loc[0, "start"] == index[9]


def test_remove_spikes():
    series = pd.Series([20.0] * 10, index=regular_index(10))
    series.iloc[4] = 85.0
    clean = remove_spikes(series)
    assert np.isnan(clean.iloc[4])
    assert clean.drop(clean.index[4]).eq(20.0).all()


def test_daily_summary():
    index = pd.date_range("2026-01-01", periods=48, freq="1h")    # two days
    series = pd.Series(np.r_[np.full(24, 10.0), np.full(24, 20.0)], index=index)
    daily = daily_summary(series)
    assert daily["mean"].tolist() == [10.0, 20.0]


def test_time_above():
    series = pd.Series([1, 5, 6, 2], index=regular_index(4))
    assert time_above(series, 4) == pd.Timedelta("20min")
```

``` bash
(sci) $ pytest
...
test_timeseries.py .....                                                 [100%]

============================== 5 passed in 0.19s ===============================
```

`index.delete(range(10, 16))` removes 6 timestamps (one hour) to create a gap;
the gap then lasts 70 minutes between the last measurement before it and the
first one after it.


Step 3: the report
-------------------------------------------------------------------------------

Create `report_logger.py`:

``` python
"""Check and summarize the measurements of a data logger."""

import argparse

import matplotlib.pyplot as plt

from timeseries import daily_summary, find_gaps, load, remove_spikes, time_above


def main():
    parser = argparse.ArgumentParser(description="Data logger report.")
    parser.add_argument("filename", help="CSV file: timestamp, temperature, ...")
    parser.add_argument("--limit", type=float, default=23,
                        help="temperature limit (°C)")
    args = parser.parse_args()

    data = load(args.filename)
    print(f"{len(data)} measurements from {data.index[0]} to {data.index[-1]}")

    gaps = find_gaps(data.index)
    print(f"{len(gaps)} gap(s) in the recording:")
    for _, gap in gaps.iterrows():
        print(f"  no data from {gap['start']} to {gap['end']} ({gap['duration']})")

    temperature = remove_spikes(data["temperature"])
    n_spikes = temperature.isna().sum()
    print(f"{n_spikes} spike(s) removed")
    print("Time above the limit:", time_above(temperature, args.limit))

    daily = daily_summary(temperature)
    daily.to_csv("daily_temperature.csv")
    print("Daily summary saved in daily_temperature.csv")

    fig, axes = plt.subplots(2, 1, figsize=(10, 6), sharex=True)
    axes[0].plot(data.index, data["temperature"], color="lightgray", label="raw")
    axes[0].plot(temperature.index, temperature, label="without spikes")
    axes[0].plot(temperature.rolling("6h").mean(), label="6 h rolling mean")
    axes[0].axhline(args.limit, color="red", linestyle="--", label="limit")
    for _, gap in gaps.iterrows():
        axes[0].axvspan(gap["start"], gap["end"], color="orange", alpha=0.3)
    axes[0].set_ylim(15, 30)
    axes[0].set_ylabel("Temperature (°C)")
    axes[0].legend(loc="upper left")
    axes[1].fill_between(daily.index, daily["min"], daily["max"], alpha=0.3,
                         step="post", label="daily min-max")
    axes[1].step(daily.index, daily["mean"], where="post", label="daily mean")
    axes[1].set_ylabel("Temperature (°C)")
    axes[1].legend(loc="upper left")
    fig.tight_layout()
    fig.savefig("logger_report.png", dpi=150)
    plt.show()


if __name__ == "__main__":
    main()
```

``` bash
(sci) $ python report_logger.py logger.csv
1980 measurements from 2026-03-02 00:00:00 to 2026-03-15 23:50:00
1 gap(s) in the recording:
  no data from 2026-03-08 22:30:00 to 2026-03-09 04:40:00 (0 days 06:10:00)
6 spike(s) removed
Time above the limit: 4 days 14:30:00
Daily summary saved in daily_temperature.csv
```

A few new things in the figure:

* `rolling("6h").mean()`: a rolling mean over a **duration** (6 hours) rather
  than a number of points, which stays correct even around gaps;
* `axvspan` colours the gaps in orange;
* `sharex=True` gives the same time axis to the two plots;
* `fill_between` and `step` draw the daily minimum–maximum band and the daily
  mean.

**Look at the figure carefully.** Two things deserve attention:

* during the gap, matplotlib draws a straight line between the last point
  before and the first point after: a figure can **hide** missing data;
* the daily mean of March 8 is higher than the trend. Why? The 6 missing
  hours are night hours, the coldest ones: the mean of this day is computed on
  the warmer part of the day only. Missing data can **bias** statistics,
  without any error message.

This is why we detect and report gaps before computing anything.


Going further
-------------------------------------------------------------------------------

**1.** Are the temperature and the humidity related? Compute their
correlation, and plot one against the other.

<details><summary>Answer</summary>

``` python
>>> data["temperature"] = remove_spikes(data["temperature"])
>>> data.corr().round(2)
             temperature  humidity
temperature         1.00     -0.88
humidity           -0.88      1.00
>>> data.plot.scatter(x="temperature", y="humidity", s=2)
```

Strong negative correlation: when the room warms up, the relative humidity
drops. (Note that the spikes must be removed first: a few 85 °C values would
distort the correlation.)
</details>

**2.** Instead of the total time above 23 °C, count the number of
**episodes**: how many times did the temperature go above the limit? Compare
the result on the raw cleaned series and on its 1-hour rolling mean.

<details><summary>Answer</summary>

An episode starts where the temperature is above the limit while it was not
at the previous measurement:

``` python
def count_episodes(series, limit):
    """Return the number of times the series goes above a limit."""
    above = series > limit
    starts = above & ~above.shift(1, fill_value=False)
    return starts.sum()
```

`shift(1)` moves the values by one step, so that each point is compared with
the previous one. We get 48 episodes on the cleaned series, but 15 on its
1-hour rolling mean: with noise, the temperature crosses the limit back and
forth several times each afternoon. Smoothing before counting events is
often necessary.
</details>

**3.** Many analyses require regular timestamps. Use
`data.resample("10min").mean()` to get a regular series (the missing times
appear as `NaN`), then fill only the **short** holes with
`interpolate(limit=3)`. How many values remain missing?

<details><summary>Answer</summary>

``` python
>>> regular = data["temperature"].pipe(remove_spikes).resample("10min").mean()
>>> regular.isna().sum()
np.int64(42)
>>> regular.interpolate(limit=3).isna().sum()
np.int64(33)
```

The 6 removed spikes are isolated holes, filled by interpolation. The 6-hour
gap (36 values) is too long: only its first 3 values are filled, the rest
stays missing, which is what we want. Inventing 6 hours of data would be
much worse than admitting they are missing.
</details>

**4.** You have a second logger, in another room, recording every
15 minutes. How would you compare the two rooms hour by hour?

<details><summary>Answer</summary>

Resample both series to the same frequency, then put them side by side; pandas
aligns the rows on the dates automatically:

``` python
room1 = load("logger.csv")["temperature"].resample("h").mean()
room2 = load("logger2.csv")["temperature"].resample("h").mean()
both = pd.DataFrame({"room 1": room1, "room 2": room2})
both.plot()
```

Hours present in only one series get `NaN` in the other column.
</details>


Resources
-------------------------------------------------------------------------------

* [**Time series / date functionality**](https://pandas.pydata.org/docs/user_guide/timeseries.html)
  in the pandas user guide: dates, resampling, time zones.
* [**Windowing operations**](https://pandas.pydata.org/docs/user_guide/window.html):
  rolling and expanding statistics.
* [**How to handle time series data with ease**](https://pandas.pydata.org/docs/getting_started/intro_tutorials/09_timeseries.html),
  one of the pandas getting started tutorials.
* [**Automating an instrument**](instrument_logging.md): the companion
  project, to acquire such data yourself (`pd.read_sql` reads an SQLite table
  directly into a DataFrame).


<!----------------------------- External links ------------------------------->
[pandas]: https://pandas.pydata.org
<!---------------------------------------------------------------------------->
