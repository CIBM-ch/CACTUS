# Examples

## Example 1: Single fibre bundle

Generate a substrate with 151 fibres in a 30 &mu;m voxel at two packing densities (ICVF 0.3 and 0.75).

**Config:** `config_files/cactus_single.txt`

```bash
# Create a working directory
mkdir -p my_experiment && cd my_experiment

# Stage 1: Initialize
cactus1-substrates init -config_file ../config_files/cactus_single.txt

# Stage 2: Optimize (may take 10-60 min depending on density)
cactus1-substrates optimize -config_file ../config_files/cactus_single.txt

# Stage 3: Test on a few fibres first
cactus1-substrates grow -config_file ../config_files/cactus_single.txt -substep growth -run_case test
cactus1-substrates grow -config_file ../config_files/cactus_single.txt -substep mesh -run_case test

# Inspect in MeshLab
meshlab tutorial_single_00000/meshes/simulations/*.ply

# If results look good, run all fibres
cactus1-substrates grow -config_file ../config_files/cactus_single.txt -substep growth -run_case all
cactus1-substrates grow -config_file ../config_files/cactus_single.txt -substep mesh -run_case all
```

**Expected output:** 2 substrates (`tutorial_single_00000`, `tutorial_single_00001`) with PLY meshes for each fibre.

---

## Example 2: Crossing fibre bundles

Generate a substrate with two crossing bundles at 60 degrees.

**Config:** `config_files/cactus_crossing.txt`

```bash
mkdir -p crossing_experiment && cd crossing_experiment

cactus1-substrates init -config_file ../config_files/cactus_crossing.txt
cactus1-substrates optimize -config_file ../config_files/cactus_crossing.txt
cactus1-substrates grow -config_file ../config_files/cactus_crossing.txt -substep growth -run_case test
cactus1-substrates grow -config_file ../config_files/cactus_crossing.txt -substep mesh -run_case test
```

---

## Example 3: Quick visualization

Preview a mesh from an optimized strand file without running the full growth pipeline:

```bash
cd my_experiment

# Single combined mesh (fast)
cactus1-substrates quick-mesh -file tutorial_single_00000/optimized_final.txt --parallel -output preview.ply

# View it
meshlab preview.ply
```

---

## Example 4: Custom radii distribution

Create substrates sweeping over two different gamma distributions:

```
outfile custom_radii
n_cores 10
verbose 1

n_bundles 1
lenght_side 30
min_rad .15
max_rad 2.5
bias 0

number_of_repetitions 1
icvf 0.7
dispersion 10

gamma_theta 2.40 3.50
gamma_kappa 0.25 0.15

optimization_tolerance 50
max_iterations 3000
alpha 1
subdivisions_number 10

grid_size 0.35
growth_iterations 4
n_segments 4
colorless 1
sim_vol simulations
inn_out outer
g_ratio 0.7
n_erode 0
```

This creates 2 substrates: one with `gamma_theta=2.40, gamma_kappa=0.25` and one with `gamma_theta=3.50, gamma_kappa=0.15`.

---

## Example 5: Resume interrupted growth

If growth was interrupted (e.g., machine shutdown), use `missing` to process only incomplete fibres:

```bash
# This checks which pickle files are missing and only processes those
cactus1-substrates grow -config_file ../config_files/cactus_single.txt -substep growth -run_case missing

# Same for meshing
cactus1-substrates grow -config_file ../config_files/cactus_single.txt -substep mesh -run_case missing
```
