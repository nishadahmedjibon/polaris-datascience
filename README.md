# Polaris Data Science

A free, structured, beginner-to-advanced learning path for Data Science and Scientific Python — built while preparing for a real high-energy physics experiment at ELSA (University of Bonn) and CERN, as part of Team Polaris's BL4S 2026 win.

**Who this is for:** Anyone — from a 6th grader curious about data, to a professional or PhD holder who wants a clean, practical refresher. No prior coding background assumed beyond basic logic.

**Why this exists:** While preparing for real beam-profile data analysis (Gaussian fitting, background subtraction, flat-field correction, SNR computation), I'm documenting every lecture, every concept, and every line of code I learn — in public, so anyone else starting from zero can follow the exact same path.

---

## How this repo is organized

```
polaris-datascience/
├── lecture-00-environment-setup/   ← Install Python, Jupyter, and libraries
├── numpy/                            ← Arrays, the foundation of everything
├── pandas/                           ← Tabular data handling
├── matplotlib/                       ← Visualization
├── scipy/                            ← Curve fitting, statistics, signal processing
├── opencv/                           ← Image processing (beam image analysis)
└── capstone-beam-analysis/           ← Full pipeline tying it all together
```

Each lecture folder contains:
- `README.md` — theory, explained in plain language, with the "why" before the "how"
- `lectureXX.ipynb` — runnable Jupyter notebook with code examples and practice questions

## Suggested order

1. [Lecture 00 — Environment Setup](./lecture-00-environment-setup/README.md)
2. [NumPy Lecture 01 — What & Why NumPy Arrays](./numpy/lecture-01-what-why-numpy-arrays/README.md)
3. [NumPy Lecture 02 — Slicing, Indexing, 2D Arrays](./numpy/lecture-02-slicing-indexing-2d-arrays/README.md)
4. ... (more added as the series progresses)

## Background

This series is built alongside real prep work for Team Polaris (CERN Beamline for Schools 2026 winner — first-ever Bangladeshi win), studying polysiloxane scintillators for high-energy electron beam profiling. Every example in this series is chosen because it's something genuinely needed for that experiment — not abstract textbook filler.

## License

Free to use, share, and learn from. If this helped you, a star ⭐ on the repo is appreciated but never required.
