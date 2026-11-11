# Project: managing a research programme
An alternative project to the Game of Life, for the scientific Python crash
course.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

Python is not only for computing: it is a great tool to **coordinate**,
**check** and **summarize** administrative data. In this project, we manage
the list of participants of a research programme with 50 projects and about
1000 participants:

* reading and writing CSV files, the format every spreadsheet and tool
  (including [Grist]) can import and export;
* counting participants per project and per institute;
* finding problems: duplicated registrations, missing emails;
* producing a report: summary files and a figure;
* writing formulas in [Grist], which uses Python.

Everything uses what we've seen in the course (functions, loops, dictionaries,
matplotlib, `argparse`, pytest), plus the standard module `csv`.

**Duration**: about 1h45.


Before you start
-------------------------------------------------------------------------------

Activate the course environment and create a folder for the project:

``` bash
(base) $ conda activate sci
(sci) $ cd Documents/python-course
(sci) $ mkdir programme
(sci) $ cd programme
```

Then save this script as `generate_participants.py` and run it. It creates a
**fake** list of participants (`participants.csv`), with a few problems hidden
inside, as in real life:

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

``` bash
(sci) $ python generate_participants.py
Written participants.csv with 1005 rows
```

Open `participants.csv` in your spreadsheet software or in a text editor: one
line per participant, with columns separated by commas. With your real data,
export your spreadsheet or your Grist table as CSV and use it instead.

> **Note on accents**: always read and write CSV files with
> `encoding="utf-8"`, as in this project. Otherwise, names like *Gaëlle* or
> *Océane* may be corrupted, especially on Windows.


Step 1: reading the data and counting
-------------------------------------------------------------------------------

The standard module `csv` reads a CSV file. `csv.DictReader` returns each row
as a **dictionary**, whose keys are the column names: much more readable than
column numbers.

Create a file `programme.py`:

``` python
"""Tools to analyse the participants of a research programme."""

import csv


def load_participants(filename):
    """Read a CSV file and return a list of dictionaries, one per participant."""
    with open(filename, newline="", encoding="utf-8") as f:
        return list(csv.DictReader(f))


def count_by(participants, column):
    """Count the participants for each value of a column.

    Parameters
    ----------
    participants : list of dict
        The participants, as returned by load_participants.
    column : str
        Name of the column, e.g. "project" or "institute".

    Returns
    -------
    dict
        Value of the column -> number of participants, sorted by decreasing count.

    Examples
    --------
    >>> count_by([{"role": "PI"}, {"role": "Engineer"}, {"role": "PI"}], "role")
    {'PI': 2, 'Engineer': 1}
    """
    counts = {}
    for participant in participants:
        value = participant[column]
        counts[value] = counts.get(value, 0) + 1
    sorted_counts = {}
    for value in sorted(counts, key=counts.get, reverse=True):
        sorted_counts[value] = counts[value]
    return sorted_counts


def normalize_email(email):
    """Return the email in a canonical form (no spaces, lower case)."""
    return email.strip().lower()


def find_duplicates(participants):
    """Return the emails appearing more than once (ignoring case and spaces)."""
    seen = set()
    duplicates = []
    for participant in participants:
        email = normalize_email(participant["email"])
        if email == "":
            continue
        if email in seen and email not in duplicates:
            duplicates.append(email)
        seen.add(email)
    return duplicates


def find_missing(participants, column):
    """Return the participants for which a column is empty."""
    return [p for p in participants if p[column].strip() == ""]
```

Let's look at `count_by`, the heart of the project:

* `counts.get(value, 0)` returns the count of `value` if it is already in the
  dictionary, and `0` otherwise. So `counts[value] = counts.get(value, 0) + 1`
  adds one to the count, whether the value has already been seen or not;
* `sorted(counts, key=counts.get, reverse=True)` sorts the keys of the
  dictionary according to their count (`key=counts.get`), from the largest to
  the smallest (`reverse=True`).

For duplicates, emails are first **normalized** (spaces removed, lower case):
`Alice@CNRS.fr ` and `alice@cnrs.fr` are the same person. The set `seen`
remembers the emails already met: testing `email in seen` is very fast, even
with thousands of emails.

Try it in the interpreter:

``` python
>>> from programme import load_participants, count_by
>>> participants = load_participants("participants.csv")
>>> len(participants)
1005
>>> participants[0]
{'first_name': 'Samir', 'last_name': 'Laurent', 'email': 'samir.laurent0@example.fr', 'institute': 'INEE', 'role': 'Researcher', 'project': 'P03'}
>>> count_by(participants, "role")
{'Researcher': 373, 'PhD student': 197, 'Engineer': 197, 'Postdoc': 149, 'PI': 89}
```


Step 2: tests
-------------------------------------------------------------------------------

Before trusting a report sent to 50 project leaders, test the functions on
small cases where you know the answer. Create `test_programme.py`:

``` python
"""Tests for programme.py."""

from programme import count_by, find_duplicates, find_missing, normalize_email

PEOPLE = [
    {"email": "a@x.fr", "project": "P01"},
    {"email": " A@X.fr ", "project": "P02"},
    {"email": "b@x.fr", "project": "P01"},
    {"email": "", "project": "P03"},
]


def test_count_by():
    assert count_by(PEOPLE, "project") == {"P01": 2, "P02": 1, "P03": 1}


def test_normalize_email():
    assert normalize_email("  Alice.Martin@CNRS.fr ") == "alice.martin@cnrs.fr"


def test_find_duplicates_ignores_case_and_spaces():
    assert find_duplicates(PEOPLE) == ["a@x.fr"]


def test_empty_emails_are_not_duplicates():
    assert find_duplicates([{"email": ""}, {"email": ""}]) == []


def test_find_missing():
    assert find_missing(PEOPLE, "email") == [{"email": "", "project": "P03"}]
```

``` bash
(sci) $ pytest --doctest-modules
...
programme.py .                                                           [ 16%]
test_programme.py .....                                                  [100%]

============================== 6 passed in 0.01s ===============================
```

`PEOPLE`, written in capitals, is a **constant** shared by several tests:
a small, hand-made dataset containing exactly the tricky cases (same email
with different case and spaces, empty email).


Step 3: the report
-------------------------------------------------------------------------------

Create `report.py`:

``` python
"""Produce a report on the participants of the programme."""

import argparse
import csv

import matplotlib.pyplot as plt

from programme import count_by, find_duplicates, find_missing, load_participants


def save_counts(counts, filename, column):
    """Save a dictionary of counts as a CSV file."""
    with open(filename, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow([column, "participants"])
        for value, count in counts.items():
            writer.writerow([value, count])


def main():
    parser = argparse.ArgumentParser(description="Programme report.")
    parser.add_argument("filename", help="CSV file of participants")
    args = parser.parse_args()

    participants = load_participants(args.filename)
    print(f"{len(participants)} rows")

    duplicates = find_duplicates(participants)
    print(f"{len(duplicates)} duplicated emails:", ", ".join(duplicates))
    missing = find_missing(participants, "email")
    print(f"{len(missing)} participants without email, saved in missing_emails.csv")
    with open("missing_emails.csv", "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=list(participants[0]))
        writer.writeheader()
        writer.writerows(missing)

    by_project = count_by(participants, "project")
    by_institute = count_by(participants, "institute")
    save_counts(by_project, "by_project.csv", "project")
    save_counts(by_institute, "by_institute.csv", "institute")
    print("Largest projects:", list(by_project.items())[:3])

    fig, axes = plt.subplots(1, 2, figsize=(12, 4))
    axes[0].bar(list(by_project), list(by_project.values()))
    axes[0].set_title("Participants per project")
    axes[0].tick_params(axis="x", labelrotation=90, labelsize=6)
    axes[1].barh(list(by_institute), list(by_institute.values()))
    axes[1].set_title("Participants per institute")
    axes[1].invert_yaxis()
    fig.tight_layout()
    fig.savefig("report.png", dpi=150)
    plt.show()


if __name__ == "__main__":
    main()
```

``` bash
(sci) $ python report.py participants.csv
1005 rows
5 duplicated emails: samir.laurent0@example.fr, david.leroy100@example.fr, theo.robert200@example.fr, pierre.bertrand300@example.fr, pierre.simon400@example.fr
20 participants without email, saved in missing_emails.csv
Largest projects: [('P19', 30), ('P03', 28), ('P15', 28)]
```

The script produces three CSV files (`by_project.csv`, `by_institute.csv`,
`missing_emails.csv`), which you can open in a spreadsheet or import into
Grist, and a figure `report.png`.

Next month, with the updated list, the whole report is produced again with
**one command**, exactly the same way. This is the real gain of automation.


Python in Grist
-------------------------------------------------------------------------------

[Grist] is a spreadsheet where each table behaves like a database table. Its
**formulas are written in Python**: everything you learned in this course
applies directly.

In a formula column, `$column_name` refers to the value of another column in
the same row. For example, in a table `Participants`:

``` python
# Full name, with the last name in capitals
$first_name + " " + $last_name.upper()

# Normalized email (same idea as normalize_email)
$email.strip().lower()

# Flag missing emails
"MISSING" if $email.strip() == "" else ""
```

A formula can also be several lines long, with `if`, loops and `return`, like
the body of a function:

``` python
if $role == "PI":
    return "Project leader"
elif $role == "PhD student" or $role == "Postdoc":
    return "Early career"
return "Staff"
```

In a table `Projects`, a formula can look up the related rows of another
table, here the participants of each project:

``` python
# Number of participants of this project
len(Participants.lookupRecords(project=$name))
```

Grist and a Python script complement each other well: Grist for collaborative
entry and simple formulas, a script for heavier checks and reports, the two
exchanging data through CSV exports and imports.


Going further
-------------------------------------------------------------------------------

**1.** Write a function `count_by_two(participants, column1, column2)`
returning, for example, the number of participants for each pair
(project, role). Use tuples as dictionary keys.

<details><summary>Answer</summary>

``` python
def count_by_two(participants, column1, column2):
    """Count the participants for each pair of values of two columns."""
    counts = {}
    for participant in participants:
        key = (participant[column1], participant[column2])
        counts[key] = counts.get(key, 0) + 1
    return counts

counts = count_by_two(participants, "project", "role")
print(counts[("P01", "PI")])
```

Tuples can be dictionary keys because they are immutable; lists cannot.
</details>

**2.** Some projects have no PI. Write a function returning the list of
these projects.

<details><summary>Answer</summary>

``` python
def projects_without_pi(participants):
    """Return the sorted list of projects that have no participant with role PI."""
    all_projects = set()
    with_pi = set()
    for participant in participants:
        all_projects.add(participant["project"])
        if participant["role"] == "PI":
            with_pi.add(participant["project"])
    return sorted(all_projects - with_pi)
```

The difference of two sets (`-`) gives the elements of the first that are not
in the second.
</details>

**3.** Write a script producing a **mailing list** per project: one text file
per project (`mailing_P01.txt`, ...), containing the emails of its
participants separated by `; `, ready to paste in an email client. Ignore
missing emails and duplicates.

<details><summary>Answer</summary>

``` python
from pathlib import Path

from programme import load_participants, normalize_email

participants = load_participants("participants.csv")
emails = {}                                   # project -> list of emails
for participant in participants:
    email = normalize_email(participant["email"])
    project = participant["project"]
    if email == "":
        continue
    if project not in emails:
        emails[project] = []
    if email not in emails[project]:
        emails[project].append(email)

folder = Path("mailing")
folder.mkdir(exist_ok=True)
for project, addresses in sorted(emails.items()):
    (folder / f"mailing_{project}.txt").write_text("; ".join(addresses))
print(len(emails), "mailing lists written")
```

`"; ".join(list)` builds one string from a list, with `"; "` between the
elements.
</details>

**4.** Add an option `--project` to `report.py`: when given, the script
prints the list of the participants of this project only, sorted by last name.

<details><summary>Answer</summary>

``` python
    parser.add_argument("--project", default=None,
                        help="list the participants of this project")
    ...
    if args.project:
        members = [p for p in participants if p["project"] == args.project]
        for p in sorted(members, key=lambda p: p["last_name"]):
            print(f"{p['last_name']:12s} {p['first_name']:10s} {p['role']}")
```

`lambda p: p["last_name"]` is a tiny anonymous function, giving the value to
sort by for each participant.
</details>


Resources
-------------------------------------------------------------------------------

* [**csv**](https://docs.python.org/3/library/csv.html) in the official Python
  documentation.
* [**Grist help center**](https://support.getgrist.com/), in particular the
  pages on formulas and their Python functions.
* [**pandas**](https://pandas.pydata.org/docs/getting_started/index.html): the
  reference library for tables of data in Python. Once comfortable with this
  project, it is the natural next step: counting, filtering and grouping
  become one-liners.
* [**Automate the Boring Stuff with Python**](https://automatetheboringstuff.com/),
  a free book on automating everyday office tasks (files, spreadsheets,
  emails, PDF...).


<!----------------------------- External links ------------------------------->
[Grist]: https://www.getgrist.com
<!---------------------------------------------------------------------------->
