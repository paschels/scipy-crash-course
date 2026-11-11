# A Gentle introduction to Python
**Pascal Helson** - [pascal.helson@u-bordeaux.fr](mailto:pascal.helson@u-bordeaux.fr)  
Lecture notes from the scientific Python crash course taught by Nicolas Rougier at the [University of Bordeaux] for the academic year 2021,
adapted in 2026 by Pascal Helson (Miniforge, system text editor, reading error messages).  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).

## Installation


This lesson aims at providing the student with a clean development environment,
including a Python installation and essential packages (using the [Miniforge]
installer) and a shell. We'll use the text editor that already comes with your
operating system to write our first script, then introduce the Python &
[IPython] shells, and explain how to run a Python script from the command line
or from inside the [IPython] shell. IDEs such as Spyder and JupyterLab will
come in the next session.

The key idea of this lesson: **Python is a program that runs your code. An
editor or an IDE is just an interface to write code and to ask Python to run
it.** You don't need an IDE to write Python code.

<br/>

**Contents**
* [Objectives](#objectives)
* [Installation](#installation-) ..............................................................................................................................................
  <img src="https://img.shields.io/badge/-Duration:_30mn-black.svg?style=flat-square" align="right"/>
* [Python, IPython & your first script](#python-ipython--your-first-script-) ................................................................................................
  <img src="https://img.shields.io/badge/-Duration:_20mn-black.svg?style=flat-square" align="right"/>
* [Testing your installation](#testing-your-installation-) .........................................................................................................................
  <img src="https://img.shields.io/badge/-Duration:_10mn-black.svg?style=flat-square" align="right"/>


<br/><br/><br/><br/>

## Objectives

The primary goals of this lesson are:

* To ensure you have a clean Python installation, without administrator rights
* To write a Python script with the text editor of your system and run it
* To type a few Python lines in the Python and IPython shells
* To understand that Python runs your code, whatever tool you use to write it

### Convention

During this introduction, we'll use three different kinds of console, namely
`shell`, `Python` and `IPython`. To help you distinguish them, there will be a
small colored icon in front of them:

<img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

``` bash
(base) $ _
```

<img src="https://img.shields.io/badge/-Python-blue.svg?style=flat-square" align="left"/>

``` Pycon
Python 3.12.7 | packaged by conda-forge | ...
Type "help", "copyright", "credits" or "license" for more information.
>>> _
```

<img src="https://img.shields.io/badge/-IPython-orange.svg?style=flat-square" align="left"/>

``` IPython
Python 3.12.7 | packaged by conda-forge | ...
Type 'copyright', 'credits' or 'license' for more information
IPython 8.29.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: _
```

Which shell to use depends on your system:

* **Linux and macOS**: the regular *Terminal* application.
* **Windows**: the *Miniforge Prompt* (installed below). Do **not** use
  PowerShell or the regular command prompt: `conda` won't be found there.

In the examples, `$` is the shell prompt, `>>>` is the Python prompt and
`In [1]:` is the IPython prompt: they must not be typed. The word in
parentheses at the beginning of the shell prompt, such as `(base)` or `(sci)`,
tells you which conda environment is active (more on this below). There is
also a second Python prompt (`...`) meaning the previous line is not ended and
needs to be terminated. This is for example the case when you enter an
unbalanced expression (e.g. the number of opening parentheses is greater than
the number of closing parentheses).

If you type a shell command in a Python shell or a Python command in a shell,
the console will report an error. This is the most common beginner mistake, so
always look at the prompt first!

> **Note**: to exit a Python or IPython shell, type `exit()`.


<br/><br/>
## Installation <img src="https://img.shields.io/badge/-Duration:_30mn-black.svg?style=flat-square" align="right"/>

Python might already be installed on your system. However, we won't use it
because we'll need to install several packages that might interfere with the
system packages. We'll thus install our own Python using [Miniforge], a minimal
installer for the `conda` package manager that uses the community-driven
[conda-forge] repository. Miniforge is free software, installs in your home
folder (**no administrator rights needed**) and does not depend on the
commercial Anaconda repository.

### Is conda already installed?

Before installing anything, check whether conda (Miniforge, Anaconda or
Miniconda) is already on your computer:

* **Linux and macOS**: open a Terminal and type `conda --version`. If a version
  number is displayed, conda is already installed. If you get
  `command not found`, it is not.
* **Windows**: open the Start menu and search for "Miniforge", "Anaconda" and
  "Miniconda". If a *Miniforge Prompt*, *Anaconda Prompt* or *Miniconda Prompt*
  shortcut appears, conda is already installed. Don't rely on PowerShell or the
  regular command prompt to check, conda is usually invisible there.

If conda is **already installed**, don't reinstall it: skip to
[Checking the configuration](#checking-the-configuration) and use the prompt
you found (e.g. *Anaconda Prompt*) wherever this lesson says *Miniforge Prompt*.

### Installing Miniforge

Download the installer for your system from the [Miniforge] page:

* **Windows**: `Miniforge3-Windows-x86_64.exe`. Run it and:
  * choose **"Just Me (recommended)"** (no administrator rights needed);
  * keep the proposed installation folder;
  * do **not** add Miniforge to the `PATH`.

  If Windows displays *"Windows protected your PC"*, click *"More info"* then
  *"Run anyway"* (if your IT policy allows it).

* **Linux and macOS**: open a Terminal and type (the installer matching your
  machine is chosen automatically):

  <img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

  ``` bash
  $ curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
  $ bash Miniforge3-$(uname)-$(uname -m).sh
  ```

  Accept the license, keep the proposed folder (`~/miniforge3`) and answer
  `yes` when asked to initialize conda. Never use `sudo` here.

Then open a **new** shell:

* **Linux and macOS**: close and reopen the Terminal.
* **Windows**: open the Start menu, type "Miniforge" and click on
  **Miniforge Prompt**.

The prompt should now start with `(base)`. Test that you have access to the
`conda` command:

<img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

``` bash
(base) $ conda --version
conda 25.x.x
```

### Checking the configuration

Check that packages come from conda-forge only:

<img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

``` bash
(base) $ conda config --show channels
channels:
  - conda-forge
```

If `defaults` also appears (this happens when Anaconda or Miniconda was already
installed), replace it with conda-forge:

<img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

``` bash
(base) $ conda config --add channels conda-forge
(base) $ conda config --remove channels defaults
(base) $ conda config --set channel_priority strict
```

> **Note**: this only changes where *future* packages come from. Your existing
> environments and the usual `conda` commands keep working as before.

### Creating an environment for the course

We don't install anything in the `base` environment. Instead, we create a
dedicated **environment** named `sci`, with Python and the packages we need:

<img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

``` bash
(base) $ conda create -n sci python=3.12 numpy scipy matplotlib ipython cython numba
...
Proceed ([y]/n)? y
...
(base) $ conda activate sci
(sci) $ _
```

The prompt now starts with `(sci)`: this means that the `python` command will
use the Python of this environment. **Every time you open a new shell**, start
by typing `conda activate sci`.

> **Note**: if downloads fail (`HTTP`, `SSL` or `CondaHTTPError` errors), your
> network probably goes through a proxy: contact your IT department.


<br/><br/>
## Python, IPython & your first script <img src="https://img.shields.io/badge/-Duration:_20mn-black.svg?style=flat-square" align="right"/>

You are now ready to start using Python, and there are two basic ways to do
that:

1. You can start a Python or IPython shell and type some Python commands
2. You can write a Python script in a text editor and ask Python to execute it

No IDE is needed for either of them.

### The Python shell

The Python shell can be started using the command `python` (make sure the
`sci` environment is active):

<img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

``` Pycon
(sci) $ python
Python 3.12.7 | packaged by conda-forge | ...
Type "help", "copyright", "credits" or "license" for more information.
>>> _
```

Once inside the shell, you can start typing commands:

<img src="https://img.shields.io/badge/-Python-blue.svg?style=flat-square" align="left"/>

``` Pycon
>>> print("Hello world!")
Hello world!
>>> exit()
```

### The IPython shell

The IPython shell can be started using the command `ipython`:

<img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

``` Pycon
(sci) $ ipython
Python 3.12.7 | packaged by conda-forge | ...
Type 'copyright', 'credits' or 'license' for more information
IPython 8.29.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: _
```

It works pretty much like the default Python shell and you can type commands:

<img src="https://img.shields.io/badge/-IPython-orange.svg?style=flat-square" align="left"/>

``` IPython
In [1]: print("Hello world!")
Hello world!
```

Both shells allow you to enter Python commands, but IPython offers a set of
supplementary commands as well as some goodies such as code completion (press
`Tab`) and interactive figures. If you don't know which one to use, choose
IPython. Note that IPython is not a different Python: it is a nicer interface
on top of the same Python.

### Writing a script with your system's text editor

Using a shell is fine for short code snippets, but as soon as you want to write
longer programs, you'll first write your code in a file and then ask Python to
execute it. A Python script is just a **plain text file**: any text editor can
write it. Let's use the one that already comes with your system.

**1. Create a folder for the course**, e.g. `python-course`, wherever you like
(for instance in your Documents folder).

**2. Open your system's default text editor:**

* **Windows**: *Notepad* (Start menu, type "Notepad").
* **macOS**: *TextEdit* (Applications folder, or `Cmd+Space` and type
  "TextEdit"). Before typing, choose *Format > Make Plain Text*, and disable
  *Edit > Substitutions > Smart Quotes*: curly quotes `“ ”` would break your
  code.
* **Linux**: *Text Editor* or *gedit* (GNOME, e.g. Ubuntu), *Kate* or *KWrite*
  (KDE).

**3. Type exactly this single line:**

```python
print("Hello world!")
```

**4. Save the file as `hello.py`** in your `python-course` folder. The `.py`
extension tells everyone (including your system) that this is a Python script.
Be careful:

* **Windows (Notepad)**: in the *Save as* dialog, choose *"All files (\*.\*)"*
  as type, otherwise the file will be saved as `hello.py.txt`. Keep the
  *UTF-8* encoding.
* **macOS (TextEdit)**: if asked, uncheck *"If no extension is provided, use
  .txt"*, or choose *"Use .py"* when TextEdit asks which extension to keep.

**5. Run the script with Python.** In your shell, go to the folder where you
saved the file with the `cd` command. Tip: type `cd ` (with a space), then drag
and drop the folder from your file explorer into the shell window: its full
path will be pasted for you. Then press `Enter` and run the script:

<img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

``` bash
# Replace the path with the folder where you saved your script
(sci) $ cd Documents/python-course
(sci) $ python hello.py
Hello world!
```

That's it: you wrote a program with a simple text editor and Python executed
it. **The editor only writes text; Python runs it.**

If you run the above command, Python terminates as soon as your program has
ended. If you want to stay within the Python interpreter afterwards (for
example to inspect your variables), use the `-i` switch (interactive mode):

<img src="https://img.shields.io/badge/-Shell-black.svg?style=flat-square" align="left"/>

``` bash
(sci) $ python -i hello.py
Hello world!
>>> _
```

The IPython shell can also run a script, through the magic command `%run`
(there are many [other magic commands](https://ipython.readthedocs.io/en/stable/interactive/magics.html)).
Start `ipython` from the folder containing `hello.py`, then:

<img src="https://img.shields.io/badge/-IPython-orange.svg?style=flat-square" align="left"/>

``` IPython
In [1]: %run hello.py
Hello world!
```

> **About indentation**: Python uses indentation (spaces at the beginning of
> lines) to structure code. Always indent with **4 spaces**, never with
> tabulations: mixing both makes Python complain. Simple editors like Notepad
> or TextEdit insert a tabulation when you press `Tab`, so type the spaces
> yourself.


<br/><br/>
## Testing your installation <img src="https://img.shields.io/badge/-Duration:_10mn-black.svg?style=flat-square" align="right"/>

It's now time to check our installation to see if everything is ok. At this
point, no need yet to understand what you're typing, but we need to check that
some important packages are present with the proper version. Start `python` in
the `sci` environment and type:

**Checking for numpy**

<img src="https://img.shields.io/badge/-Python-blue.svg?style=flat-square" align="left"/>

``` pycon
>>> import numpy
>>> print(numpy.__version__)
2.1.3
```

**Checking for scipy**

<img src="https://img.shields.io/badge/-Python-blue.svg?style=flat-square" align="left"/>

``` pycon
>>> import scipy
>>> print(scipy.__version__)
1.14.1
```

**Checking for matplotlib**

<img src="https://img.shields.io/badge/-Python-blue.svg?style=flat-square" align="left"/>

``` pycon
>>> import matplotlib
>>> print(matplotlib.__version__)
3.9.2
```

**Checking for cython**

<img src="https://img.shields.io/badge/-Python-blue.svg?style=flat-square" align="left"/>

``` pycon
>>> import cython
>>> print(cython.__version__)
3.0.11
```

**Checking for numba**

<img src="https://img.shields.io/badge/-Python-blue.svg?style=flat-square" align="left"/>

``` pycon
>>> import numba
>>> print(numba.__version__)
0.60.0
```

For each of these packages, the `x.y.z` version should be equal to or greater
than the displayed version. If you get `ModuleNotFoundError`, check that the
prompt showed `(sci)` before you started `python`: you are probably using
another Python.


<br/><br/>

##

Copyright © 2019 [Nicolas P. Rougier](http://www.labri.fr/perso/nrougier) •
Released under a [CC-BY 4.0 International](https://creativecommons.org/licenses/by/4.0/legalcode) license.
Adapted in 2026 (Miniforge instead of Anaconda, system text editor).


<!----------------------------- External links ------------------------------->
[Miniforge]:    https://github.com/conda-forge/miniforge
[conda-forge]:  https://conda-forge.org
[Python]:       https://www.python.org
[Numpy]:        https://numpy.org
[Scipy]:        https://scipy.org
[Matplotlib]:   https://matplotlib.org
[IPython]:      https://ipython.org
[Jupyter]:      https://jupyter.org
[Cython]:       https://cython.org
[Unicode]:      https://en.wikipedia.org/wiki/Unicode
<!---------------------------------------------------------------------------->

<br/><br/>
---

Copyright © 2026 [Pascal Helson](https://sites.google.com/view/page-pers-phelson) •
Released under a [CC-BY 4.0 International](https://creativecommons.org/licenses/by/4.0/legalcode) license.
