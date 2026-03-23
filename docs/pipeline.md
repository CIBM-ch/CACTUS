# Pipeline Guide

CACTUS generates white matter substrates through a three-stage pipeline. Each stage reads the output of the previous one.

```
config file ──> [init] ──> .init files ──> [optimize] ──> optimized_final.txt ──> [grow] ──> .ply meshes
```

---

## Stage 1: Initialization

Places fibres in a cubic voxel using a Watson distribution for angular dispersion and a gamma distribution for radii.

```bash
cactus1-substrates init -config_file config_files/cactus_single.txt
```

**What happens:**
1. Reads the configuration file and computes all parameter combinations
2. For each combination, creates a directory (e.g., `tutorial_single_00000/`)
3. Generates a `.init` file containing initial fibre positions and radii

**Output:**
```
tutorial_single_00000.init
tutorial_single_00000/
    optimized_final.txt    # copy of .init as starting point
    profiler_ovlp.log      # initial overlap metric
    profiler_non_ovlp.log  # initial non-overlap metric
tutorial_single_00001.init
tutorial_single_00001/
    ...
```

> The number of `.init` files depends on the parameter sweep. Two ICVF values with one repetition = 2 substrates.

---

## Stage 2: Optimization

Runs the C++ joint fibre optimizer to minimize fibre overlaps using Adagrad gradient descent with OpenMP parallelization.

```bash
cactus1-substrates optimize -config_file config_files/cactus_single.txt
```

To optimize a specific `.init` file:

```bash
cactus1-substrates optimize -config_file config_files/cactus_single.txt -file tutorial_single_00000.init
```

**What happens:**
1. Reads the overlap metric from `profiler_ovlp.log`
2. Skips if already below `optimization_tolerance`
3. Runs the C++ optimizer, which saves `.partial` checkpoints every 50 iterations
4. Writes `optimized_final.txt` when done

**Output (inside each experiment folder):**
```
tutorial_single_00000/
    optimized_final.txt     # final optimized fibre configuration
    *.partial               # intermediate checkpoints
    profiler_ovlp.log       # updated overlap metric
```

**Monitoring progress:**

While optimization runs (it can take hours for dense substrates), use the monitor in a separate terminal:

```bash
cactus1-substrates monitor -folder tutorial_single_00000/
```

---

## Stage 3: Growth & Meshing

This stage has two substeps that must run in order.

### Substep 1: Growth

Discretizes fibres into a 3D voxel grid and applies the Fibre Radial Growth (FRG) algorithm.

```bash
cactus1-substrates grow -config_file config_files/cactus_single.txt -substep growth -run_case test
```

**What happens:**
1. Reads `optimized_final.txt`
2. For each fibre, creates a voxel-space representation
3. Propagates radial growth over `growth_iterations` iterations
4. Saves compressed pickle files (`.bz2`) with the voxel data

**Output:**
```
tutorial_single_00000/
    meshes/
        pickles/
            strand_00000_dict.bz2
            strand_00000_mask.bz2
            ...
```

### Substep 2: Meshing

Converts voxel data into triangle meshes using marching cubes.

```bash
cactus1-substrates grow -config_file config_files/cactus_single.txt -substep mesh -run_case test
```

**Output:**
```
tutorial_single_00000/
    meshes/
        simulations/
            strand_00000_outer_erode_0.ply
            strand_00000_outer_erode_0.vol
            ...
```

### Run cases

| Option | Description | When to use |
|--------|-------------|-------------|
| `test` | Process ~5 fibres spread across the range | Validate parameters before full run |
| `missing` | Process only fibres that failed or are missing | Resume after interruption |
| `all` | Process all fibres | Final production run |

> Always run `test` first to verify your `grid_size` and `growth_iterations` produce good results before committing to `all`.

### Processing a specific file

```bash
cactus1-substrates grow -config_file config_files/cactus_single.txt \
    -substep growth -run_case test -file tutorial_single_00000.init
```

---

## Quick mesh

For fast visualization without the full growth pipeline, generate a mesh directly from any strand file:

```bash
cactus1-substrates quick-mesh -file tutorial_single_00000/optimized_final.txt
```

Parallel mode (faster, produces a single combined mesh):

```bash
cactus1-substrates quick-mesh -file tutorial_single_00000/optimized_final.txt \
    --parallel -output output.ply
```

---

## Combining individual meshes

After Stage 3, each fibre is a separate `.ply` file. To combine them into a single mesh:

```bash
cactus1-substrates paste-mesh -file tutorial_single_00000.init \
    -sim_vol simulations -inn_out outer -n_erode 0
```

---

## Visualizing results

Open PLY files in [MeshLab](https://www.meshlab.net/) or any 3D viewer:

```bash
meshlab tutorial_single_00000/meshes/simulations/*.ply
```

Or with Python using PyVista:

```python
import pyvista as pv
mesh = pv.read("tutorial_single_00000/meshes/simulations/strand_00000_outer_erode_0.ply")
mesh.plot()
```

---

## Complete workflow example

```bash
# 1. Initialize substrates
cactus1-substrates init -config_file config_files/cactus_single.txt

# 2. Optimize fibre arrangement
cactus1-substrates optimize -config_file config_files/cactus_single.txt

# 3a. Test growth on a few fibres
cactus1-substrates grow -config_file config_files/cactus_single.txt -substep growth -run_case test

# 3b. Test meshing
cactus1-substrates grow -config_file config_files/cactus_single.txt -substep mesh -run_case test

# 3c. Inspect results in MeshLab, then run all
cactus1-substrates grow -config_file config_files/cactus_single.txt -substep growth -run_case all
cactus1-substrates grow -config_file config_files/cactus_single.txt -substep mesh -run_case all

# 4. Combine into single mesh
cactus1-substrates paste-mesh -file tutorial_single_00000.init
```
