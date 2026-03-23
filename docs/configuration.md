# Configuration Reference

CACTUS uses plain-text configuration files with space-separated key-value pairs. Two templates are provided:

| File | Use case |
|------|----------|
| `config_files/cactus_single.txt` | Single fibre bundle |
| `config_files/cactus_crossing.txt` | Crossing fibre bundles |

## File format

```
# Comments start with #
key value
key value1 value2    # Multiple values = parameter sweep
```

When a key has multiple values, CACTUS generates all combinations across sweepable parameters and creates one substrate per combination.

---

## Stage 1 &mdash; Initialization parameters

### Algorithm parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `outfile` | string | Output filename prefix (e.g., `tutorial_single`) |
| `n_cores` | int | Number of CPU cores for parallel initialization |
| `verbose` | int | Verbosity level (`0` = quiet, `1` = normal) |

### Substrate geometry

| Parameter | Type | Unit | Description |
|-----------|------|------|-------------|
| `n_bundles` | int | &mdash; | Number of fibre bundles (`1` = single, `2` = crossing) |
| `lenght_side` | float | &mu;m | Side length of the cubic voxel |
| `min_rad` | float | &mu;m | Minimum fibre radius |
| `max_rad` | float | &mu;m | Maximum fibre radius |
| `bias` | float | &mu;m | Shift applied to radii distribution (negative = smaller fibres) |

### Sweepable parameters

These parameters accept multiple values to generate substrate combinations:

| Parameter | Type | Unit | Description |
|-----------|------|------|-------------|
| `number_of_repetitions` | int | &mdash; | How many substrates to create per parameter combination |
| `icvf` | float | [0, 1] | Target intra-cellular volume fraction (packing density) |
| `dispersion` | float | degrees | Angular dispersion of fibres (Watson distribution concentration) |
| `gamma_theta` | float | &mdash; | Shape parameter of the gamma distribution for fibre radii |
| `gamma_kappa` | float | &mdash; | Scale parameter of the gamma distribution for fibre radii |

> `gamma_theta` and `gamma_kappa` must have the same number of values. Each pair defines one radii distribution.

**Example &mdash; parameter sweep:**

```
icvf 0.3 0.75
dispersion 10
number_of_repetitions 1
```

This creates 2 substrates: one with ICVF = 0.3, another with ICVF = 0.75.

### Crossing-specific parameters

Only used when `n_bundles >= 2`:

| Parameter | Type | Unit | Description |
|-----------|------|------|-------------|
| `crossing_angle` | float | degrees | Angle between the two fibre bundles |
| `overlap` | float | [0, 1] | Fractional spatial overlap between bundles |
| `depth_lenght_bundle` | float | &mu;m | Depth of each fibre bundle |

---

## Stage 2 &mdash; Optimization parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `optimization_tolerance` | float | Stop when overlap metric drops below this value |
| `max_iterations` | int | Maximum number of Adagrad iterations |
| `alpha` | float | Learning rate for Adagrad gradient descent |
| `subdivisions_number` | int | Number of fibre subdivisions for collision detection |

The optimizer saves `.partial` checkpoint files every 50 iterations, and writes `optimized_final.txt` on completion.

---

## Stage 3 &mdash; Growth & meshing parameters

| Parameter | Type | Unit | Description |
|-----------|------|------|-------------|
| `grid_size` | float | fraction of min radius | Voxel discretization step relative to minimum fibre radius |
| `growth_iterations` | int | &mdash; | Number of radial growth iterations in discrete space |
| `n_segments` | int | &mdash; | Number of segments for fibre subdivision during growth |
| `g_ratio` | float | [0, 1] | Mean g-ratio (inner/outer radius ratio) for myelin modeling |
| `n_erode` | int | &mdash; | Number of morphological erosions to apply |
| `inn_out` | string | &mdash; | Mesh boundary type: `inner`, `outer` |
| `sim_vol` | string | &mdash; | Output mesh type: `simulations`, `volume` |
| `colorless` | int | &mdash; | `1` = no vertex colors, `0` = include colors |

### How `grid_size` and `growth_iterations` interact

- Smaller `grid_size` = finer voxel grid = more memory, more accurate surface
- More `growth_iterations` = thicker radial growth per fibre
- Typical values: `grid_size 0.35`, `growth_iterations 4` for single bundles; `grid_size 0.2`, `growth_iterations 6` for crossing bundles

---

## Full example

```
# --- Stage 1: Initialization ---
outfile tutorial_single
n_cores 10
verbose 1

n_bundles 1
lenght_side 30
min_rad .27
max_rad 2.2
bias -.03

number_of_repetitions 1
icvf 0.3 0.75
dispersion 10
gamma_theta 2.40
gamma_kappa 0.25

# --- Stage 2: Optimization ---
optimization_tolerance 50
max_iterations 3000
alpha 1
subdivisions_number 10

# --- Stage 3: Growth & Meshing ---
grid_size 0.35
growth_iterations 4
n_segments 4
colorless 1
sim_vol simulations
inn_out outer
g_ratio 0.7
n_erode 0
```
