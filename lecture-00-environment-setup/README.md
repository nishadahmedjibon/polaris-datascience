# Lecture 00 : Setting Up Your Data Science Environment

## Who is this for?

Anyone with zero setup on their computer. By the end of this lecture, you'll have Python, Jupyter Notebook, and every library needed for this entire series installed and verified.

---

## Step 1 : Install Python

Check if Python is already installed. Open your terminal (Command Prompt on Windows, Terminal on Mac/Linux) and type:

```
python --version
```

If you see something like `Python 3.10.x` or higher, skip to Step 2.

If not installed, go to [python.org/downloads](https://www.python.org/downloads/) and download the latest version.

**Windows users:** During installation, check the box that says **"Add Python to PATH"** before clicking install. This step is easy to miss and causes most beginner errors later.

**Mac/Linux users:** Python usually comes pre-installed, but it may be an old version. Installing the latest from python.org is recommended.

---

## Step 2 : Install Jupyter Notebook and required libraries

Open your terminal and run each line below, one at a time, waiting for each to finish:

```
pip install jupyter
pip install numpy
pip install pandas
pip install matplotlib
pip install scipy
pip install opencv-python
pip install tifffile
pip install scikit-learn
```

If `pip` is not recognized, try `pip3` instead, or `python -m pip install ...`.

---

## Step 3 : Create your working folder

```
cd Desktop
mkdir polaris-datascience
cd polaris-datascience
```

This is where all your notebooks will live locally before you push them to GitHub.

---

## Step 4 : Launch Jupyter Notebook

```
jupyter notebook
```

This opens a browser tab. Click **New → Python 3 (ipykernel)** to create your first notebook.

To run any cell inside the notebook: press **Shift + Enter**.

---

## Step 5 : Verify your installation

In the first cell of your new notebook, paste this and run it:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import scipy
import cv2

print("NumPy:", np.__version__)
print("Pandas:", pd.__version__)
print("Matplotlib:", plt.matplotlib.__version__)
print("SciPy:", scipy.__version__)
print("OpenCV:", cv2.__version__)
print("Setup complete!")
```

If you see version numbers printed with no errors, your environment is ready.

---

## Next

Move on to [NumPy Lecture 01 — What & Why NumPy Arrays](../numpy/lecture-01-what-why-numpy-arrays/README.md)
