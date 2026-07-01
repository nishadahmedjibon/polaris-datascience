# NumPy Lecture 02 - Slicing, Indexing, and 2D Arrays

Almost everything you will ever do with scientific image data comes down to one skill:
**grabbing the right piece of an array, correctly and fast.**

This lecture goes from the very basics of indexing all the way to real beam-image operations.
By the end, you will be able to extract regions from images, filter data by condition, find
peak locations, project 2D data into 1D, and average stacks of frames — all without a single
Python loop.

---

## Part 1 : Indexing 1D Arrays

Indexing means picking out one specific element by its position number.
NumPy arrays, like Python lists, start counting from **0**.

```
Array:    [10,  20,  30,  40,  50]
Index:      0    1    2    3    4
```

```python
import numpy as np

a = np.array([10, 20, 30, 40, 50])

print(a[0])    # 10  → first element
print(a[2])    # 30  → third element
print(a[4])    # 50  → last element
```

### Negative indexing

Negative indices count from the end. `-1` is always the last element.

```python
print(a[-1])   # 50  → last
print(a[-2])   # 40  → second to last
print(a[-5])   # 10  → same as a[0] for this array
```

**Why this matters:** In real code you often don't know the exact length of an array
(it depends on how many images you loaded, or how many events passed a filter).
`a[-1]` always safely gets the last element regardless of size.

---

## Part 2 : Slicing 1D Arrays

Slicing extracts a **range** of elements in one shot.

### The syntax

```
array[start : stop : step]
```

- `start` — index to begin at (included)
- `stop` — index to end at (**not included** — this is the most common beginner mistake)
- `step` — how many positions to jump each time (default is 1)

```python
a = np.array([10, 20, 30, 40, 50, 60, 70, 80])
#              0   1   2   3   4   5   6   7

print(a[2:5])     # [30 40 50]       → index 2, 3, 4  (5 is excluded)
print(a[:4])      # [10 20 30 40]    → from start up to index 3
print(a[4:])      # [50 60 70 80]    → from index 4 to the end
print(a[:])       # [10 20 30 40 50 60 70 80]  → entire array
print(a[::2])     # [10 30 50 70]    → every 2nd element
print(a[1::2])    # [20 40 60 80]    → every 2nd element starting at index 1
print(a[::-1])    # [80 70 60 50 40 30 20 10]  → reversed
```

### The most common beginner mistake

`stop` is **excluded**. So `a[2:5]` gives you indices 2, 3, 4 — not 5.

```python
a = np.array([10, 20, 30, 40, 50])
print(a[1:4])   # [20 30 40]  → indices 1, 2, 3  (NOT 4)
```

---

## Part 3 : The View Trap (Critical)

This is the **number one bug** that beginners hit with NumPy. It causes silent,
hard-to-find errors in real code.

### Slices are views, not copies

When you slice an array, NumPy does NOT create a new copy of the data.
It creates a **view** — a window into the same block of memory.

```python
a = np.array([1, 2, 3, 4, 5])
b = a[1:4]       # b is a view of a, not a copy

b[0] = 999       # modifying b...
print(a)          # [  1 999   3   4   5]  ← a changed too!
```

This is not a bug in NumPy — it is a deliberate design choice for performance.
Views avoid copying large arrays unnecessarily. But if you don't know about it,
it will corrupt your data.

### Always use `.copy()` when you want independence

```python
a = np.array([1, 2, 3, 4, 5])
c = a[1:4].copy()   # c is a true independent copy

c[0] = 999
print(a)             # [1, 2, 3, 4, 5]  ← a is unaffected
print(c)             # [999, 3, 4]
```

### Rule for scientific work

When you load a raw beam image and want to process a copy of it,
always `.copy()` first so the raw data stays intact for reference.

```python
raw_image = load_image(...)          # your original data
working_copy = raw_image.copy()      # safe to modify
```

### How to check if two arrays share memory

```python
a = np.array([1, 2, 3, 4, 5])
b = a[1:4]
c = a[1:4].copy()

print(np.shares_memory(a, b))   # True  → view
print(np.shares_memory(a, c))   # False → independent copy
```

---

## Part 4 : Boolean Indexing (Masking)

Boolean indexing lets you filter an array by condition, not by position.
This is one of the most powerful and frequently used features in NumPy.

### The concept

A **boolean mask** is an array of `True`/`False` values with the same shape as
your data. When you use it to index, you get back only the elements where the
mask is `True`.

```python
a = np.array([10, 20, 30, 40, 50])

mask = a > 25
print(mask)        # [False False  True  True  True]

print(a[mask])      # [30 40 50]
```

You can also write it in one line without naming the mask:

```python
print(a[a > 25])    # [30 40 50]
```

### Combining conditions

Use `&` for AND, `|` for OR. Do **not** use `and`/`or` — they don't work
element-wise on arrays.

```python
a = np.array([5, 10, 15, 20, 25, 30, 35, 40])

print(a[(a > 10) & (a < 30)])    # [15 20 25]  → between 10 and 30
print(a[(a < 10) | (a > 30)])    # [5 35 40]   → below 10 OR above 30
```

Always wrap each condition in its own parentheses when combining — `(a > 10) & (a < 30)`
not `a > 10 & a < 30` (the second form gives wrong results due to Python operator precedence).

### Modifying values with a mask

```python
a = np.array([5, 10, 200, 40, 500])

# Zero out anything above 100 (noise threshold)
a[a > 100] = 0
print(a)   # [  5  10   0  40   0]
```

### Real use case - signal vs noise separation

```python
# Simulated pixel values from a beam detector
pixels = np.array([3, 2, 4890, 1, 4700, 2, 5100, 3])

# Background threshold from a dark frame measurement
threshold = 100

# Extract only signal pixels
signal = pixels[pixels > threshold]
print("Signal pixels:", signal)   # [4890 4700 5100]

# Count how many signal pixels exist
print("Signal count:", signal.size)   # 3

# Compute signal-to-noise related stats
print("Mean signal:", signal.mean())
print("Mean background:", pixels[pixels <= threshold].mean())
```

---

## Part 5 : Fancy Indexing

Instead of a range (slice) or a condition (mask), you can index with a
**list of specific positions** you want.

```python
a = np.array([10, 20, 30, 40, 50])

idx = [0, 2, 4]
print(a[idx])       # [10 30 50]

# You can repeat indices or reorder freely
print(a[[4, 0, 2, 0]])   # [50 10 30 10]
```

### Key difference from slicing

Fancy indexing always returns a **copy**, never a view.

```python
a = np.array([10, 20, 30, 40, 50])
b = a[[0, 1, 2]]
b[0] = 999
print(a)   # [10, 20, 30, 40, 50]  ← unchanged, because b is a copy
```

### Comparison table

| Method | Syntax | Returns | Shared memory? |
|---|---|---|---|
| Single index | `a[2]` | scalar | No |
| Slicing | `a[1:4]` | view | Yes |
| Boolean mask | `a[a > 5]` | copy | No |
| Fancy indexing | `a[[0, 2, 4]]` | copy | No |

---

## Part 6 : 2D Arrays: Indexing

A 2D array is rows and columns — exactly like a spreadsheet or a grayscale image
where each element is a pixel brightness value.

```
          col0  col1  col2  col3
row 0  [  10,   20,   30,   40 ]
row 1  [  50,   60,   70,   80 ]
row 2  [  90,  100,  110,  120 ]
```

### Accessing a single element

```python
image = np.array([
    [ 10,  20,  30,  40],
    [ 50,  60,  70,  80],
    [ 90, 100, 110, 120]
])

print(image[0, 0])    # 10   → row 0, col 0
print(image[1, 2])    # 70   → row 1, col 2
print(image[2, 3])    # 120  → row 2, col 3
print(image[-1, -1])  # 120  → last row, last col
```

Always use the comma syntax `[row, col]`. While `image[1][2]` also works,
it is slower because it creates an intermediate 1D array first.

### Accessing whole rows and columns

```python
# Entire row
print(image[1])        # [50 60 70 80]  → row 1
print(image[1, :])     # [50 60 70 80]  → same, written explicitly

# Entire column
print(image[:, 2])     # [ 30  70 110]  → column 2

# Last column
print(image[:, -1])    # [ 40  80 120]

# Last row
print(image[-1, :])    # [ 90 100 110 120]
```

Memorize: `array[:, col]` for a column, `array[row, :]` for a row.
The `:` means "everything along this dimension."

---

## Part 7 : 2D Slicing (Cropping Sub-regions)

This is how you cut out a **Region of Interest (ROI)** from an image —
for example, the area around the beam spot.

```python
image = np.array([
    [ 1,  2,  3,  4,  5],
    [ 6,  7,  8,  9, 10],
    [11, 12, 13, 14, 15],
    [16, 17, 18, 19, 20],
    [21, 22, 23, 24, 25]
])

# Extract center 3x3
center = image[1:4, 1:4]
print(center)
# [[ 7  8  9]
#  [12 13 14]
#  [17 18 19]]

# Top two rows, all columns
top = image[:2, :]
print(top)
# [[ 1  2  3  4  5]
#  [ 6  7  8  9 10]]

# Bottom-right 3x3
bottom_right = image[-3:, -3:]
print(bottom_right)
# [[13 14 15]
#  [18 19 20]
#  [23 24 25]]
```

### Cropping around a known beam center — real use case

```python
# Simulated large detector image
detector = np.random.randint(0, 50, size=(500, 500))

# Known beam center from alignment
center_row, center_col = 250, 300
half = 50

# Crop 100x100 ROI centered on beam
roi = detector[center_row - half : center_row + half,
               center_col - half : center_col + half]

print(roi.shape)   # (100, 100)
```

### Modifying a sub-region

```python
image = np.zeros((5, 5), dtype=np.uint16)

image[0, :]     = 100    # set entire top row
image[:, 4]     = 200    # set entire right column
image[2, 2]     = 500    # set center pixel
image[1:3, 1:3] = 999    # set 2x2 block

print(image)
```

---

## Part 8 : 2D Boolean Masking

Boolean masks work identically on 2D arrays.

```python
image = np.array([
    [ 10, 200,  30],
    [400,  50, 600],
    [ 70, 800,  90]
])

mask = image > 100
print(mask)
# [[False  True False]
#  [ True False  True]
#  [False  True False]]

# Extracts matching values as a 1D array
print(image[mask])   # [200 400 600 800]

# Zero out everything above 100 (background suppression)
image[mask] = 0
print(image)
# [[ 10   0  30]
#  [  0  50   0]
#  [ 70   0  90]]
```

---

## Part 9 : The `axis` Parameter (Critical for Projections)

The `axis` parameter controls which dimension an operation collapses.
This is the concept most beginners get confused about — but it's essential
for beam profile extraction.

### The mental model

For a 2D array with shape `(rows, cols)`:
- `axis=0` → collapse the **rows** → result has one value per **column**
- `axis=1` → collapse the **columns** → result has one value per **row**

Think of it as: **the axis you name is the axis that disappears.**

```python
image = np.array([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
])

print(np.sum(image, axis=0))   # [12 15 18]  → summed down each column
print(np.sum(image, axis=1))   # [ 6 15 24]  → summed across each row
print(np.sum(image))            # 45           → sum everything
```

Verify `axis=0` by hand: column 0 = 1+4+7 = 12 ✓, column 1 = 2+5+8 = 15 ✓.
Verify `axis=1` by hand: row 0 = 1+2+3 = 6 ✓, row 1 = 4+5+6 = 15 ✓.

### Same logic applies to `np.mean`, `np.max`, `np.min`, `np.std`

```python
print(np.mean(image, axis=0))   # mean of each column
print(np.max(image,  axis=1))   # max of each row
print(np.min(image,  axis=0))   # min of each column
print(np.std(image,  axis=1))   # std of each row
```

### Direct application - beam profile projection

Your proposal's data analysis section defines:

```
P_h(x) = sum over y of I(x, y)    ← horizontal projection
P_v(y) = sum over x of I(x, y)    ← vertical projection
```

In NumPy, for an image array of shape `(height, width)`:

```python
P_h = np.sum(image, axis=0)   # shape (width,)  → profile along x axis
P_v = np.sum(image, axis=1)   # shape (height,) → profile along y axis
```

These two lines are the **core of your beam profile extraction**.
Everything else (Gaussian fitting, FWHM extraction) operates on P_h and P_v.

---

## Part 10 : `np.where()` — Conditional Logic Without Loops

`np.where` has two forms. Both are extremely useful.

### Form 1 : replace values conditionally

```python
np.where(condition, value_if_true, value_if_false)
```

```python
a = np.array([5, 150, 30, 400, 80])

# Replace values above 100 with 1, others with 0 (binary threshold)
result = np.where(a > 100, 1, 0)
print(result)   # [0 1 0 1 0]

# Replace values below threshold with the threshold value (clipping)
clipped = np.where(a < 50, 50, a)
print(clipped)  # [ 50 150  50 400  80]
```

### Form 2 : find indices where condition is true

```python
np.where(condition)   # returns tuple of index arrays
```

```python
image = np.array([
    [  5,   6, 5000],
    [4900,   7,    6],
    [  5, 5200,    4]
])

rows, cols = np.where(image > 1000)
print("Rows:", rows)    # [0 1 2]
print("Cols:", cols)    # [2 0 1]

# These are (row, col) pairs: (0,2), (1,0), (2,1)
for r, c in zip(rows, cols):
    print(f"  Hot pixel at ({r},{c}): value = {image[r, c]}")
```

### Finding the peak : beam center detection

```python
image = np.array([
    [  5,    6,   12],
    [  9, 5000,    6],
    [  5,    8,    4]
])

# argmax returns flat index in the "unrolled" array
flat_idx = np.argmax(image)
print("Flat index:", flat_idx)   # 4

# Convert flat index back to (row, col)
row, col = np.unravel_index(flat_idx, image.shape)
print(f"Peak at row={row}, col={col}, value={image[row, col]}")
```

### Full beam-center-finding workflow

```python
# Large simulated detector image with a beam spot
detector = np.random.randint(0, 50, size=(100, 100))
detector[45:55, 45:55] = 5000   # beam spot in center

# Find peak location
flat_idx = np.argmax(detector)
peak_row, peak_col = np.unravel_index(flat_idx, detector.shape)
print(f"Beam center: row={peak_row}, col={peak_col}")

# Crop ROI around the beam center
half = 20
roi = detector[peak_row-half : peak_row+half,
               peak_col-half : peak_col+half]
print("ROI shape:", roi.shape)   # (40, 40)
```

---

## Part 11 : Reshaping and Flattening

### `.flatten()` — collapse any array to 1D

```python
image = np.array([
    [1, 2, 3],
    [4, 5, 6]
])

flat = image.flatten()
print(flat)          # [1 2 3 4 5 6]
print(flat.shape)    # (6,)
```

`.flatten()` always returns a copy.
`.ravel()` does the same but returns a view when possible (faster, but view trap applies).

### `.reshape()` — change shape without changing data

```python
a = np.arange(12)         # [0 1 2 3 4 5 6 7 8 9 10 11]
b = a.reshape(3, 4)        # 3 rows, 4 columns
print(b)

c = a.reshape(4, 3)        # 4 rows, 3 columns
print(c)

d = a.reshape(2, 2, 3)    # 3D: 2 layers of 2x3
print(d.shape)              # (2, 2, 3)
```

Total elements must stay the same: 12 → 3×4 ✓, 12 → 4×3 ✓, 12 → 2×2×3 ✓.

### The `-1` trick — let NumPy calculate one dimension

```python
a = np.arange(24)

b = a.reshape(4, -1)     # 4 rows, NumPy calculates columns → (4, 6)
c = a.reshape(-1, 6)     # NumPy calculates rows, 6 columns → (4, 6)
d = a.reshape(2, 3, -1)  # 2 layers, 3 rows, NumPy figures out cols → (2, 3, 4)

print(b.shape, c.shape, d.shape)
```

You'll use this constantly when processing batches of images where the number
of pixels per image is fixed but the number of images varies.

---

## Part 12 : Stacking and Averaging (Multi-frame Analysis)

In a real beam experiment, you take 100–200 frames at each energy level.
NumPy lets you treat all frames as a single 3D array and process them together.

### Stacking images into a 3D array

```python
img1 = np.zeros((5, 5))
img2 = np.ones((5, 5)) * 2
img3 = np.ones((5, 5)) * 3

# Stack into shape (3, 5, 5) — 3 images, each 5x5
stack = np.stack([img1, img2, img3])
print(stack.shape)   # (3, 5, 5)

# Access individual frames
print(stack[0])   # img1
print(stack[1])   # img2
```

### Averaging across frames - noise reduction

```python
# Simulate 100 noisy beam-on frames
frames = np.random.randint(0, 20, size=(100, 50, 50))

# Average across all 100 frames (axis=0 collapses the frame dimension)
avg_frame = np.mean(frames, axis=0)
print(avg_frame.shape)   # (50, 50) → one clean averaged image
```

**Why averaging reduces noise:** Random noise is different in each frame.
When you average 100 frames, noise cancels out and the real signal (the beam)
stays consistent. Signal-to-noise ratio improves by a factor of √N — so
100 frames gives you 10x better SNR than 1 frame.

### Dark frame subtraction using stacked arrays

```python
# 100 beam-on frames
beam_on  = np.random.randint(100, 200, size=(100, 50, 50))

# 50 dark frames (no beam, just detector noise)
dark     = np.random.randint(0, 20, size=(50, 50, 50))

# Average each set
avg_beam = np.mean(beam_on, axis=0)   # (50, 50)
avg_dark = np.mean(dark,    axis=0)   # (50, 50)

# Subtract dark frame from beam frame
corrected = avg_beam - avg_dark        # (50, 50)
print(corrected.shape)
print("Min after subtraction:", corrected.min())
```

This is your measurement protocol Step 2 (background subtraction) done in three lines.

### `np.vstack` and `np.hstack` - joining arrays

```python
a = np.array([[1, 2, 3]])   # shape (1, 3)
b = np.array([[4, 5, 6]])   # shape (1, 3)

print(np.vstack([a, b]))    # [[1 2 3]   → stack vertically, shape (2, 3)
                             #  [4 5 6]]

print(np.hstack([a, b]))    # [[1 2 3 4 5 6]]  → stack horizontally, shape (1, 6)
```

---

## Part 13 : Putting It All Together: A Mini Beam Analysis

Here is a complete workflow using everything from this lecture —
from a raw simulated image to a cleaned, projected beam profile.

```python
import numpy as np

# 1. Simulate raw beam-on image (noise + beam spot)
raw = np.random.randint(0, 30, size=(100, 100), dtype=np.int32)
raw[45:55, 45:55] += 5000   # add beam spot at center

# 2. Simulate dark frame (detector noise only)
dark = np.random.randint(0, 30, size=(100, 100), dtype=np.int32)

# 3. Dark frame subtraction (use copy to protect raw data)
corrected = raw.copy()
corrected = corrected - dark

# 4. Clip negative values to zero (can't have negative photon counts)
corrected = np.where(corrected < 0, 0, corrected)

# 5. Find beam center
flat_idx = np.argmax(corrected)
peak_row, peak_col = np.unravel_index(flat_idx, corrected.shape)
print(f"Beam center: ({peak_row}, {peak_col})")

# 6. Crop ROI around beam center
half = 20
roi = corrected[peak_row-half : peak_row+half,
                peak_col-half : peak_col+half]
print(f"ROI shape: {roi.shape}")

# 7. Project to 1D profiles (beam profile extraction)
P_h = np.sum(roi, axis=0)   # horizontal projection
P_v = np.sum(roi, axis=1)   # vertical projection
print(f"P_h shape: {P_h.shape}, P_v shape: {P_v.shape}")

# 8. Find peak channel in each projection
print(f"Peak channel x: {np.argmax(P_h)}")
print(f"Peak channel y: {np.argmax(P_v)}")
```

Every line here maps directly to a step in your proposal's measurement protocol.
In later lectures you will replace step 7–8 with actual Gaussian fitting using SciPy.

---

## Summary Table

| Operation | Syntax | Notes |
|---|---|---|
| Single element | `a[i]` or `a[r, c]` | Use comma for 2D |
| Slice | `a[start:stop:step]` | `stop` is excluded |
| Whole row | `a[r, :]` | `:` = all |
| Whole column | `a[:, c]` | `:` = all |
| Sub-region | `a[r1:r2, c1:c2]` | View — use `.copy()` if modifying |
| Boolean mask | `a[a > threshold]` | Returns copy |
| Fancy index | `a[[0, 2, 4]]` | Returns copy |
| Projection | `np.sum(a, axis=0/1)` | axis=0 collapses rows |
| Find peak | `np.unravel_index(np.argmax(a), a.shape)` | Returns (row, col) |
| Stack frames | `np.stack([f1, f2, ...])` | Shape: (N, H, W) |
| Average frames | `np.mean(stack, axis=0)` | Shape: (H, W) |

---

## Next

Open `lecture02.ipynb` in this folder to run the code examples and complete the practice questions.

When ready, move on to
[NumPy Lecture 03 — Broadcasting and Mathematical Operations](../lecture-03-broadcasting-math-operations/README.md)
