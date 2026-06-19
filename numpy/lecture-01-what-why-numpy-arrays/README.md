# NumPy Lecture 01 — What & Why NumPy Arrays

## What is a NumPy array?

A NumPy array (called an `ndarray`, short for "n-dimensional array") is a grid of values, all of the **same type**, stored together in one continuous block of memory.

Compare this to a normal Python list, which can hold mixed types (`[1, "hello", 3.5]`) and stores each element as a separate object scattered in memory with pointers connecting them.

```
Python list:   [ptr→1] [ptr→2] [ptr→3]   ← scattered in memory, slow to process
NumPy array:   [1][2][3][4][5]            ← packed together, one block, fast
```

## Why does this matter?

Because everything in a NumPy array is the same type and sits next to each other in memory, NumPy can hand off entire calculations to highly optimized C code under the hood. Instead of looping through values one at a time (like a Python list forces you to), NumPy processes the **whole array at once** — this is called **vectorization**.

For 10 numbers, this difference is invisible. For a 1000×1000 pixel beam image (1 million numbers), the difference is the gap between code that runs instantly and code that takes minutes.

## dtype — the type of data inside the array

Every array has a `dtype` (data type), which tells NumPy exactly how many bytes each value takes up.

| dtype | Meaning | Bytes per value |
|---|---|---|
| `int32` | 32-bit integer | 4 |
| `int64` | 64-bit integer | 8 |
| `float32` | 32-bit decimal | 4 |
| `float64` | 64-bit decimal | 8 |
| `uint16` | unsigned 16-bit integer (0 to 65535) | 2 |

`uint16` matters for real scientific work — 16-bit TIFF camera images (like the ones used in beam profiling experiments) store each pixel as a `uint16` value.

## shape, ndim, size — the three properties you'll use constantly

- **shape** — size of the array along each dimension, given as a tuple
- **ndim** — number of dimensions
- **size** — total number of elements

```
1D array shape (5,)       → a single row of 5 values
2D array shape (3, 4)     → 3 rows, 4 columns (like a spreadsheet)
3D array shape (2, 3, 4)  → 2 layers, each 3×4
```

A grayscale image is naturally a **2D array** — rows and columns of pixel brightness values. This is the foundation for everything in image-based scientific analysis.

## Why dtype choice is a real engineering decision

Choosing `float64` everywhere "to be safe" can double or quadruple your memory and processing time for no benefit, especially with large datasets like camera images across many experimental runs. Picking the right dtype (`uint16` for raw camera data, `float32` for most calculations) is a meaningful practical skill, not just trivia.

## Next

Open `lecture01.ipynb` in this folder to run the code examples and practice questions.
