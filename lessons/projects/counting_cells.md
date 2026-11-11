# Project: counting cells in microscopy images
An alternative project to the Game of Life, for the scientific Python crash
course.  
This work is licensed under [Creative Commons Attribution 4.0 International
License](http://creativecommons.org/licenses/by/4.0/).


Objectives
-------------------------------------------------------------------------------

For numpy, an image is just a **2D array** of numbers: everything you learned
about arrays applies to images. In this project, we build a small image
analysis pipeline, like the ones you would write as a macro in ImageJ/Fiji:

* creating synthetic fluorescence images, whose content we know;
* separating the objects from the background (**thresholding**);
* finding and numbering the objects (**labelling**), and measuring them;
* processing a whole folder of images with a script, and checking the
  results visually;
* testing the pipeline on images where we know the right answer.

Everything uses what we've seen in the course (numpy, boolean masks,
matplotlib, `argparse`, pytest), plus the module `ndimage` of [SciPy],
already installed in the `sci` environment.

**Duration**: about 1h45.


Before you start
-------------------------------------------------------------------------------

Activate the course environment and create a folder for the project:

``` bash
(base) $ conda activate sci
(sci) $ cd Documents/python-course
(sci) $ mkdir cells
(sci) $ cd cells
```

This project is very visual: Spyder or JupyterLab are convenient to look at
the images at each step.


Step 1: synthetic images
-------------------------------------------------------------------------------

To check that an analysis is right, start with images whose content you know.
Create a file `cells.py`, with a first function drawing bright disks (our
"nuclei") on a dark background, then blurring the image and adding noise, as
a microscope and a camera would:

``` python
"""Detect and measure bright objects (e.g. cell nuclei) in an image."""

import numpy as np
from scipy import ndimage


def make_image(rng, n_cells=30, size=256, noise=0.15):
    """Create a synthetic fluorescence image with bright disks on a dark background.

    Returns
    -------
    image : numpy.ndarray
        2D float array of shape (size, size).
    """
    y, x = np.mgrid[0:size, 0:size]
    image = np.zeros((size, size))
    for _ in range(n_cells):
        cy, cx = rng.integers(10, size - 10, 2)
        radius = rng.uniform(4, 9)
        brightness = rng.uniform(0.6, 1.0)
        image[(x - cx)**2 + (y - cy)**2 <= radius**2] = brightness
    image = ndimage.gaussian_filter(image, 1.5)      # blur, as a microscope
    image += rng.normal(0.1, noise, image.shape)       # camera noise
    return image


def threshold(image, k=3):
    """Return a mask of the pixels brighter than the background.

    The background level is estimated with the median and the noise with
    the median absolute deviation, which are not influenced much by the
    bright objects.
    """
    background = np.median(image)
    noise = np.median(np.abs(image - background)) * 1.4826
    return image > background + k * noise


def measure(mask, min_area=20):
    """Label the objects of a mask and measure them.

    Parameters
    ----------
    mask : numpy.ndarray of bool
        Pixels belonging to objects.
    min_area : int
        Objects smaller than this (in pixels) are ignored, as noise.

    Returns
    -------
    labels : numpy.ndarray of int
        Image where each kept object has its own number (0 = background).
    areas : numpy.ndarray
        Area of each kept object, in pixels.
    centers : list of (float, float)
        Center (row, column) of each kept object.
    """
    labels, n = ndimage.label(mask)
    areas = np.bincount(labels.ravel())[1:]          # pixels per object
    keep = np.flatnonzero(areas >= min_area) + 1      # labels of large objects
    labels = np.where(np.isin(labels, keep), labels, 0)
    centers = ndimage.center_of_mass(mask, labels, keep)
    return labels, areas[keep - 1], centers
```

We'll come back to `threshold` and `measure` in the next steps. In
`make_image`:

* `np.mgrid[0:size, 0:size]` gives two arrays `y` and `x` containing, for each
  pixel, its row and its column. `(x - cx)**2 + (y - cy)**2 <= radius**2` is
  then a **boolean mask** of the pixels inside a disk, and
  `image[mask] = brightness` paints the disk, without any loop over pixels;
* `ndimage.gaussian_filter` blurs the image, like the optics of a microscope;
* `rng.normal(...)` adds random noise to every pixel.

Look at an image:

``` python
>>> import matplotlib.pyplot as plt
>>> import numpy as np
>>> from cells import make_image
>>> image = make_image(np.random.default_rng(0), n_cells=30)
>>> image.shape, image.dtype
((256, 256), dtype('float64'))
>>> plt.imshow(image, cmap="gray")
>>> plt.colorbar()
>>> plt.show()
```


Step 2: thresholding
-------------------------------------------------------------------------------

To separate the cells from the background, we keep the pixels brighter than
a **threshold**. A good threshold is "the background level plus a few times
the noise". But computing them with the mean and the standard deviation of
the image would be biased by the bright cells themselves. The function
`threshold` uses **robust** statistics instead:

* the **median** of the image estimates the background level, since most
  pixels belong to the background;
* the **median absolute deviation** (MAD), multiplied by 1.4826, estimates the
  standard deviation of the noise.

The result is a boolean mask, `True` for the pixels of the objects:

``` python
>>> from cells import threshold
>>> mask = threshold(image)
>>> mask.mean()            # proportion of "object" pixels
>>> plt.imshow(mask)
>>> plt.show()
```

Try different values of `k` (`threshold(image, k=1)`, `k=5`...) and look at
the masks: too low, noise becomes objects; too high, faint cells disappear.


Step 3: labelling and measuring
-------------------------------------------------------------------------------

A mask tells which pixels belong to objects, but not **which** object. The
function `ndimage.label` gives a different number to each group of connected
pixels: `0` for the background, then `1`, `2`, `3`... for the objects. This
is the equivalent of *Analyze Particles* in ImageJ.

The function `measure` then:

* counts the pixels of each object with `np.bincount`, which counts how many
  times each number appears in an array (`ravel()` flattens the 2D image into
  1D). We drop the first count, which is the background;
* keeps only the objects larger than `min_area`: isolated noisy pixels are
  not cells. `np.flatnonzero` gives the positions of the `True` values, and
  `np.isin` / `np.where` set the labels of the small objects to 0;
* computes the center of each object with `ndimage.center_of_mass`.

``` python
>>> from cells import measure
>>> labels, areas, centers = measure(mask)
>>> len(areas)
>>> plt.imshow(labels, cmap="nipy_spectral")   # one colour per object
>>> plt.show()
```


Step 4: tests
-------------------------------------------------------------------------------

Image analysis is a domain where it is easy to get **plausible but wrong**
results. Tests on simple, noise-free images, where we know the answer exactly,
are precious. Create `test_cells.py`:

``` python
"""Tests for cells.py."""

import numpy as np

from cells import measure, threshold


def disk_image(centers, radius=6, size=100):
    """Image with bright disks of the given radius, no noise."""
    y, x = np.mgrid[0:size, 0:size]
    image = np.zeros((size, size))
    for cy, cx in centers:
        image[(x - cx)**2 + (y - cy)**2 <= radius**2] = 1.0
    return image


def test_counts_separated_disks():
    mask = disk_image([(20, 20), (50, 70), (80, 30)]) > 0.5
    labels, areas, centers = measure(mask)
    assert len(areas) == 3


def test_area_of_a_disk():
    mask = disk_image([(50, 50)], radius=10) > 0.5
    labels, areas, centers = measure(mask)
    assert abs(areas[0] - np.pi * 10**2) < 0.05 * np.pi * 10**2


def test_center_of_a_disk():
    mask = disk_image([(30, 60)]) > 0.5
    labels, areas, centers = measure(mask)
    assert np.allclose(centers[0], (30, 60))


def test_small_objects_are_ignored():
    mask = np.zeros((50, 50), dtype=bool)
    mask[10, 10] = True                     # one isolated pixel
    labels, areas, centers = measure(mask, min_area=5)
    assert len(areas) == 0


def test_noise_only_gives_few_pixels():
    rng = np.random.default_rng(0)
    image = rng.normal(0.1, 0.15, (200, 200))
    assert threshold(image).mean() < 0.01
```

``` bash
(sci) $ pytest
...
test_cells.py .....                                                      [100%]

============================== 5 passed in 0.19s ===============================
```

Note the tolerances: a disk drawn with pixels does not have exactly the area
πr², so `test_area_of_a_disk` accepts a 5 % difference, and
`test_center_of_a_disk` uses `np.allclose`.


Step 5: processing a folder of images
-------------------------------------------------------------------------------

Save this script as `generate_images.py` and run it. It creates a folder
`images` with three fields of view, containing 15, 30 and 45 cells. The
images are saved in numpy's `.npy` format, which keeps the exact values:

``` python
"""Generate a few synthetic microscopy images."""

from pathlib import Path

import numpy as np

from cells import make_image

rng = np.random.default_rng(seed=3)
folder = Path("images")
folder.mkdir(exist_ok=True)
for i, n_cells in enumerate([15, 30, 45]):
    np.save(folder / f"field_{i + 1}.npy", make_image(rng, n_cells))
    print("Written", folder / f"field_{i + 1}.npy")
```

Then create the analysis script `count_cells.py`:

``` python
"""Count the cells in all the images of a folder."""

import argparse
from pathlib import Path

import matplotlib.pyplot as plt
import numpy as np

from cells import measure, threshold


def save_overlay(image, centers, filename):
    """Save the image with a red cross on each detected object."""
    fig, ax = plt.subplots(figsize=(5, 5))
    ax.imshow(image, cmap="gray")
    rows = [c[0] for c in centers]
    cols = [c[1] for c in centers]
    ax.plot(cols, rows, "r+", markersize=8)
    ax.set_title(f"{len(centers)} objects")
    ax.axis("off")
    fig.savefig(filename, dpi=100)
    plt.close(fig)


def main():
    parser = argparse.ArgumentParser(description="Count bright objects in images.")
    parser.add_argument("folder", help="folder containing .npy images")
    parser.add_argument("--k", type=float, default=3, help="threshold level")
    parser.add_argument("--min-area", type=int, default=20,
                        help="minimum object area (pixels)")
    args = parser.parse_args()

    for filename in sorted(Path(args.folder).glob("*.npy")):
        image = np.load(filename)
        mask = threshold(image, args.k)
        labels, areas, centers = measure(mask, args.min_area)
        print(f"{filename.stem}: {len(areas)} objects, "
              f"mean area {areas.mean():.0f} px")
        save_overlay(image, centers, filename.with_suffix(".png"))


if __name__ == "__main__":
    main()
```

``` bash
(sci) $ python generate_images.py
(sci) $ python count_cells.py images
field_1: 15 objects, mean area 93 px
field_2: 26 objects, mean area 109 px
field_3: 35 objects, mean area 113 px
```

**Look at the result images** (`images/field_3.png`), with a red cross on each
detected object. We created 15, 30 and 45 cells, but found 15, 26 and 35.
Why? Two reasons, visible on the images:

* when two cells **touch**, they form a single object: this is also why the
  mean area increases with the number of cells;
* very **faint** cells may stay below the threshold.

This is the most important lesson of this project: **always check your
analysis visually** on a few images, and validate it on data where you know
the answer. A script that counts wrongly does so very fast, and very
consistently!

Note also the option `--min-area`: options with a dash in their name become
attributes with an underscore (`args.min_area`).


Using real images
-------------------------------------------------------------------------------

Microscopy images are usually TIFF files. The library [tifffile] (available
on conda-forge: `conda install tifffile`) reads them into numpy arrays:

``` python
import tifffile

image = tifffile.imread("my_image.tif")
print(image.shape, image.dtype)   # e.g. (1024, 1024) uint16
image = image.astype(float)       # convert before computing
```

Multi-channel or time-lapse images have more dimensions: check `image.shape`
first, and select the channel you want with slicing (e.g. `image[0]`).

For more advanced processing, [scikit-image] provides the classic
algorithms (automatic thresholds such as Otsu's, morphological operations,
watershed to separate touching cells...), and [napari] is an interactive
image viewer for Python, close to the spirit of Fiji.


Going further
-------------------------------------------------------------------------------

**1.** Plot the histogram of the areas of all the objects found in
`field_3`. Which objects are probably two merged cells?

<details><summary>Answer</summary>

``` python
import matplotlib.pyplot as plt
import numpy as np

from cells import measure, threshold

image = np.load("images/field_3.npy")
labels, areas, centers = measure(threshold(image))
plt.hist(areas, bins=20)
plt.xlabel("Area (pixels)")
plt.ylabel("Number of objects")
plt.show()
```

Single cells have areas around 100 pixels; objects about twice larger (or
more) are probably merged cells.
</details>

**2.** Modify `count_cells.py` so that it also saves, for each image, a CSV
file with one line per object: its number, area, and center coordinates.

<details><summary>Answer</summary>

After `measure`, in the loop:

``` python
        with open(filename.with_suffix(".csv"), "w") as output:
            output.write("object,area,row,column\n")
            for i in range(len(areas)):
                row, col = centers[i]
                output.write(f"{i + 1},{areas[i]},{row:.1f},{col:.1f}\n")
```
</details>

**3.** Write a test checking that `measure` finds **one** object when two
disks touch. This test documents a known limitation of the method.

<details><summary>Answer</summary>

In `test_cells.py`:

``` python
def test_touching_disks_are_merged():
    # two disks of radius 6 whose centers are 10 pixels apart: they overlap
    mask = disk_image([(50, 45), (50, 55)]) > 0.5
    labels, areas, centers = measure(mask)
    assert len(areas) == 1
```

Documenting limitations with tests is good practice: if you improve the method
one day (e.g. with a watershed), this test will tell you that the behaviour
changed.
</details>

**4.** The brightness of a cell is also informative. Using
`ndimage.mean(image, labels, index)`, compute the mean intensity of each
object, and plot the intensity as a function of the area.

<details><summary>Answer</summary>

``` python
import matplotlib.pyplot as plt
import numpy as np
from scipy import ndimage

from cells import measure, threshold

image = np.load("images/field_2.npy")
labels, areas, centers = measure(threshold(image))
index = np.unique(labels)[1:]                  # labels of the objects, without 0
intensities = ndimage.mean(image, labels, index)
plt.scatter(areas, intensities)
plt.xlabel("Area (pixels)")
plt.ylabel("Mean intensity")
plt.show()
```
</details>


Resources
-------------------------------------------------------------------------------

* [**scipy.ndimage**](https://docs.scipy.org/doc/scipy/reference/ndimage.html)
  documentation: filters, labelling, measurements.
* [**Image processing with numpy and scipy**](https://lectures.scientific-python.org/advanced/image_processing/index.html)
  in the Scientific Python Lectures.
* [**scikit-image**][scikit-image] documentation and gallery of examples.
* [**napari**][napari], an interactive viewer for multi-dimensional images.
* [**Image Processing with Python**](https://datacarpentry.github.io/image-processing/)
  by the Data Carpentry, a complete lesson for researchers.


<!----------------------------- External links ------------------------------->
[SciPy]:        https://scipy.org
[tifffile]:     https://github.com/cgohlke/tifffile
[scikit-image]: https://scikit-image.org
[napari]:       https://napari.org
<!---------------------------------------------------------------------------->
