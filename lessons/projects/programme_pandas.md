# Project: a research programme with pandas
An alternative project to the Game of Life, for the scientific Python crash
course. It uses [pandas], introduced at the beginning of the last session.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

This project is the pandas version of
[Managing a research programme](research_programme.md): the participants of
a programme of 50 projects, about 1000 people. With pandas, each question
becomes one or two lines, and we can go further:

* **merging** two tables: the participants, and the description of the
  projects (theme, budget);
* cleaning the data: normalized emails, duplicates, missing values;
* summarizing with `groupby` and cross tables;
* producing an **Excel report** with several sheets, ready to be shared, or
  imported into [Grist].

You can do this project without having done the version without pandas.

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
(sci) $ mkdir programme-pandas
(sci) $ cd programme-pandas
```

We need two tables. Save this first script as `generate_participants.py` and
run it: it creates a **fake** list of participants, with a few problems hidden
inside, as in real life (duplicated registrations, missing emails):

``` python
"""Generate a fake list of participants of a research programme."""

import csv
import unicodedata

import numpy as np

rng = np.random.default_rng(seed=7)

first_names = ["Alice", "Bruno", "Chloé", "David", "Emma", "Farid", "Gaëlle",
               "Hugo", "Inès", "Julien", "Karima", "Louis", "Manon", "Nicolas",
               "Océane", "Pierre", "Quentin", "Rose", "Samir", "Théo"]
last_names = ["Martin", "Bernard", "Dubois", "Thomas", "Robert", "Richard",
              "Petit", "Durand", "Leroy", "Moreau", "Simon", "Laurent",
              "Lefebvre", "Michel", "Garcia", "David", "Bertrand", "Roux"]
institutes = ["INC", "INSB", "INP", "INSU", "INEE", "INSMI", "IN2P3", "INSIS"]
roles = ["PI", "Researcher", "Engineer", "PhD student", "Postdoc"]

rows = []
for i in range(1000):
    first = str(rng.choice(first_names))
    last = str(rng.choice(last_names))
    ascii_name = unicodedata.normalize("NFKD", f"{first}.{last}").encode("ascii", "ignore").decode()
    email = f"{ascii_name.lower()}{i}@example.fr"
    if rng.random() < 0.02:
        email = ""                                  # missing email
    rows.append({
        "first_name": first,
        "last_name": last,
        "email": email,
        "institute": str(rng.choice(institutes, p=[0.25, 0.2, 0.15, 0.1, 0.1, 0.05, 0.1, 0.05])),
        "role": str(rng.choice(roles, p=[0.1, 0.35, 0.2, 0.2, 0.15])),
        "project": f"P{rng.integers(1, 51):02d}",
    })

# a few people registered twice, with different capitalisation
for k in range(5):
    duplicate = dict(rows[k * 100])
    duplicate["email"] = duplicate["email"].upper()
    rows.append(duplicate)

with open("participants.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=list(rows[0]))
    writer.writeheader()
    writer.writerows(rows)
print("Written participants.csv with", len(rows), "rows")
```

Then save this second script as `generate_projects.py` and run it: it
creates the table describing the projects.

``` python
"""Generate a fake table describing the 50 projects of the programme."""

import numpy as np
import pandas as pd

rng = np.random.default_rng(seed=8)
projects = pd.DataFrame({
    "project": [f"P{i:02d}" for i in range(1, 51)],
    "theme": rng.choice(["Energy", "Health", "Environment", "Materials"], 50),
    "budget_keur": rng.integers(50, 400, 50) // 10 * 10,
    "start_year": rng.choice([2024, 2025, 2026], 50),
})
projects.to_csv("projects.csv", index=False)
print("Written projects.csv with", len(projects), "projects")
```

``` bash
(sci) $ python generate_participants.py
Written participants.csv with 1005 rows
(sci) $ python generate_projects.py
Written projects.csv with 50 projects
```

With your real data, export your spreadsheets or your Grist tables as CSV
(or read Excel files directly with `pd.read_excel`).


Step 1: loading and merging
-------------------------------------------------------------------------------

Create a file `programme_pd.py`:

``` python
"""Analyse the participants of a research programme with pandas."""

import pandas as pd


def load(participants_file, projects_file):
    """Read the two tables and merge them.

    Returns
    -------
    pandas.DataFrame
        One row per participant, with the information of their project.
    """
    participants = pd.read_csv(participants_file, keep_default_na=False)
    projects = pd.read_csv(projects_file)
    return participants.merge(projects, on="project", how="left")


def clean_emails(df):
    """Return a copy of the table with normalized emails (no spaces, lower case)."""
    df = df.copy()
    df["email"] = df["email"].str.strip().str.lower()
    return df


def duplicated_emails(df):
    """Return the rows whose email appears more than once (empty emails ignored)."""
    with_email = df[df["email"] != ""]
    return with_email[with_email.duplicated("email", keep=False)].sort_values("email")


def remove_duplicates(df):
    """Keep only the first registration of each email (empty emails are all kept)."""
    repeated = df.duplicated("email") & (df["email"] != "")
    return df[~repeated]


def summary_by_project(df):
    """Return one row per project: theme, budget, participants, budget per participant."""
    df = df.assign(is_pi=(df["role"] == "PI"))
    summary = df.groupby("project").agg(
        theme=("theme", "first"),
        budget_keur=("budget_keur", "first"),
        participants=("role", "size"),
        pis=("is_pi", "sum"),
    )
    summary["keur_per_participant"] = (summary["budget_keur"]
                                       / summary["participants"]).round(1)
    return summary.sort_values("participants", ascending=False)
```

Let's go through the functions:

* `load` reads the two tables and **merges** them with
  `participants.merge(projects, on="project", how="left")`: for each
  participant, pandas looks for the row of `projects` with the same value in
  the column `project`, and adds its columns (theme, budget, start year).
  This is the equivalent of a `VLOOKUP` in a spreadsheet, or of a join in a
  database. `how="left"` keeps all the participants, even if their project
  were missing from the second table.
* `keep_default_na=False` tells pandas to read empty cells as empty strings
  `""` rather than `NaN`: simpler to handle for text columns like emails.
* `clean_emails` uses the **string methods** of pandas: `.str.strip()` and
  `.str.lower()` apply `strip` and `lower` to every value of the column, in
  one go. It works on a **copy**, to leave the original table untouched.
* `duplicated("email", keep=False)` marks all the rows whose email appears
  more than once.
* `remove_duplicates` keeps the first registration of each person. Note the
  subtlety: `df.drop_duplicates("email")` would also merge all the people
  **without email** into a single row, since their emails are all `""`!
* `summary_by_project` uses a **named aggregation**: each argument of `agg`
  creates a column of the summary, from a column of the table and an
  operation (`"first"`, `"size"`, `"sum"`...). The PIs are counted by summing
  a boolean column, created with `assign`.

Try it interactively:

``` python
>>> import pandas as pd
>>> from programme_pd import load, clean_emails, summary_by_project
>>> df = clean_emails(load("participants.csv", "projects.csv"))
>>> df.shape
(1005, 9)
>>> summary_by_project(df).head()
               theme  budget_keur  participants  pis  keur_per_participant
project
P19      Environment          350            30    2                  11.7
P15           Health          100            28    3                   3.6
P03           Energy          200            28    5                   7.1
P29        Materials          160            27    1                   5.9
P49        Materials          360            27    1                  13.3
>>> pd.crosstab(df["institute"], df["role"])
role       Engineer  PI  PhD student  Postdoc  Researcher
institute
IN2P3            21   9           14       12          38
INC              51  17           47       47          96
...
```


Step 2: tests
-------------------------------------------------------------------------------

Tests with pandas are written exactly as before: build a **small** table by
hand, containing the tricky cases, and check the results. Create
`test_programme_pd.py`:

``` python
"""Tests for programme_pd.py."""

import pandas as pd

from programme_pd import (clean_emails, duplicated_emails, remove_duplicates,
                          summary_by_project)


def make_table():
    return pd.DataFrame({
        "email": ["a@x.fr", " A@X.fr ", "b@x.fr", "", ""],
        "project": ["P01", "P02", "P01", "P03", "P03"],
        "role": ["PI", "Engineer", "Researcher", "PI", "Postdoc"],
        "theme": ["Energy", "Health", "Energy", "Materials", "Materials"],
        "budget_keur": [100, 50, 100, 30, 30],
    })


def test_clean_emails():
    df = clean_emails(make_table())
    assert df["email"].tolist() == ["a@x.fr", "a@x.fr", "b@x.fr", "", ""]


def test_clean_emails_does_not_modify_input():
    df = make_table()
    clean_emails(df)
    assert df.loc[1, "email"] == " A@X.fr "


def test_duplicated_emails_ignores_empty_emails():
    duplicates = duplicated_emails(clean_emails(make_table()))
    assert duplicates["email"].tolist() == ["a@x.fr", "a@x.fr"]


def test_remove_duplicates_keeps_empty_emails():
    df = remove_duplicates(clean_emails(make_table()))
    assert df["email"].tolist() == ["a@x.fr", "b@x.fr", "", ""]


def test_summary_by_project():
    summary = summary_by_project(make_table())
    assert summary.loc["P01", "participants"] == 2
    assert summary.loc["P01", "pis"] == 1
    assert summary.loc["P02", "pis"] == 0
    assert summary.loc["P01", "keur_per_participant"] == 50.0
```

``` bash
(sci) $ pytest
...
test_programme_pd.py .....                                               [100%]

============================== 5 passed in 0.20s ===============================
```

`test_remove_duplicates_keeps_empty_emails` protects us against the
`drop_duplicates` trap described above: a classic bug, invisible on the
figures, that silently removes people from your report.


Step 3: the Excel report
-------------------------------------------------------------------------------

Create `report_pd.py`:

``` python
"""Produce an Excel report on the participants of the programme."""

import argparse

import matplotlib.pyplot as plt
import pandas as pd

from programme_pd import (clean_emails, duplicated_emails, load,
                          remove_duplicates, summary_by_project)


def main():
    parser = argparse.ArgumentParser(description="Programme report with pandas.")
    parser.add_argument("participants", help="CSV file of participants")
    parser.add_argument("projects", help="CSV file of projects")
    parser.add_argument("--output", default="report.xlsx", help="Excel file")
    args = parser.parse_args()

    df = clean_emails(load(args.participants, args.projects))
    duplicates = duplicated_emails(df)
    missing = df[df["email"] == ""]
    df = remove_duplicates(df)
    summary = summary_by_project(df)
    roles = pd.crosstab(df["institute"], df["role"], margins=True)

    print(f"{len(df)} participants in {df['project'].nunique()} projects")
    print(f"{len(duplicates) // 2} duplicated registrations removed")
    print(f"{len(missing)} participants without email")
    print("Projects without PI:", ", ".join(summary[summary["pis"] == 0].index))

    with pd.ExcelWriter(args.output) as writer:
        summary.to_excel(writer, sheet_name="Projects")
        roles.to_excel(writer, sheet_name="Institutes x roles")
        duplicates.to_excel(writer, sheet_name="Duplicates", index=False)
        missing.to_excel(writer, sheet_name="Missing emails", index=False)
    print("Report written in", args.output)

    fig, axes = plt.subplots(1, 2, figsize=(11, 4))
    df["theme"].value_counts().plot.bar(ax=axes[0], rot=0)
    axes[0].set_ylabel("Participants")
    summary.plot.scatter(x="participants", y="budget_keur", ax=axes[1])
    axes[1].set_ylabel("Budget (k€)")
    fig.tight_layout()
    fig.savefig("report.png", dpi=150)
    plt.show()


if __name__ == "__main__":
    main()
```

``` bash
(sci) $ python report_pd.py participants.csv projects.csv
1000 participants in 50 projects
5 duplicated registrations removed
20 participants without email
Projects without PI: P17, P32, P40, P06, P18, P27, P25, P50, P35
Report written in report.xlsx
```

Open `report.xlsx`: it contains four sheets (projects, institutes × roles,
duplicates, missing emails). `pd.ExcelWriter` opens the file once, and each
`to_excel(writer, sheet_name=...)` adds a sheet. In the cross table,
`margins=True` adds the totals. The figure `report.png` shows the
participants per theme, and the budget of each project against its number of
participants.

When the lists are updated next month: one command, and the whole report is
up to date.


Going further
-------------------------------------------------------------------------------

**1.** Build a table with the number of participants for each institute
(rows) and each theme (columns).

<details><summary>Answer</summary>

``` python
>>> df.pivot_table(index="institute", columns="theme", values="email",
...                aggfunc="count")
theme      Energy  Environment  Health  Materials
institute
IN2P3          19           27      32         16
INC            61           49      79         68
...
```

`pd.crosstab(df["institute"], df["theme"])` gives the same table.
</details>

**2.** Produce, for each project, the list of the emails of its participants
separated by `; `, ready to paste in an email client, and save it as a CSV
file.

<details><summary>Answer</summary>

``` python
>>> with_email = df[df["email"] != ""]
>>> mailing = with_email.groupby("project")["email"].agg("; ".join)
>>> mailing.to_csv("mailing.csv")
```

`agg` accepts any function taking a Series: here `"; ".join`, which glues the
emails together.
</details>

**3.** You receive an updated list of participants every month. How do you
find the **new** participants, i.e. the emails present in the new list but
not in the old one?

<details><summary>Answer</summary>

Merge the new list with the emails of the old one, asking pandas to indicate
where each row comes from:

``` python
comparison = new.merge(old[["email"]], on="email", how="left", indicator=True)
newcomers = comparison[comparison["_merge"] == "left_only"]
```

With `indicator=True`, the column `_merge` says `both` (present in both
lists) or `left_only` (only in the new one). Swap the two tables to find the
people who left.
</details>

**4.** What is the total budget of each theme? Be careful: the budget appears
once **per participant** in the merged table.

<details><summary>Answer</summary>

``` python
>>> df.drop_duplicates("project").groupby("theme")["budget_keur"].sum()
theme
Energy         3010
Environment    1990
Health         3070
Materials      2300
Name: budget_keur, dtype: int64
```

Summing `budget_keur` directly over the participants would count the budget
of each project as many times as it has participants. Keep one row per
project first (or work on `projects.csv` directly). After a merge, always
ask yourself what one row represents.
</details>


Resources
-------------------------------------------------------------------------------

* [**10 minutes to pandas**](https://pandas.pydata.org/docs/user_guide/10min.html)
  and the [**pandas getting started tutorials**](https://pandas.pydata.org/docs/getting_started/intro_tutorials/index.html).
* [**Merge, join and concatenate**](https://pandas.pydata.org/docs/user_guide/merging.html)
  in the pandas user guide.
* [**Comparison with spreadsheets**](https://pandas.pydata.org/docs/getting_started/comparison/comparison_with_spreadsheets.html):
  filters, pivot tables and VLOOKUP in pandas.
* [**Grist help center**](https://support.getgrist.com/), for importing and
  exporting CSV and Excel files, and writing Python formulas.


<!----------------------------- External links ------------------------------->
[pandas]: https://pandas.pydata.org
[Grist]:  https://www.getgrist.com
<!---------------------------------------------------------------------------->
