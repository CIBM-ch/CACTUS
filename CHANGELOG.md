# Changelog

All notable changes to CACTUS are documented here.

## [1.5.0] - 2024

### Changed
- Full restructuring into a pip-installable Python package (`src/cactus1_substrate/`)
- Unified CLI entry point `cactus1-substrates` with 7 subcommands
- Dynamic path resolution replaces manual `update_paths.py`
- Makefile-based C++ compilation replaces `compile_cpp_code.sh`
- All scripts refactored into proper modules: `core/`, `pipeline/`, `workers/`, `tools/`
- Multiprocessing workers use `initializer`/`initargs` for spawn-safe globals

### Fixed
- Multiprocessing spawn globals bug in `meta_grid.py` and `bake_mesh_pickle.py`
- Numba-visible typo in `propagate_all_heat_2d` (`y` -> `j`)

### Removed
- `cactus_scripts/` directory (migrated to `src/`)
- `update_paths.py`, `compile_cpp_code.sh`, `create_library.py`
- Legacy build scripts (`compile_intel.sh`, `debug.sh`, `debug_perf.sh`)

## [1.0.0] - 2023

Initial release accompanying the Frontiers in Neuroinformatics paper:

> Villarreal-Haro, J.L. et al. "CACTUS: A Computational Framework for Generating Realistic White Matter Microstructure Substrates." *Frontiers in Neuroinformatics*, 2023.

- Three-stage pipeline: initialization, optimization, growth & meshing
- C++ joint fibre optimizer with Adagrad and OpenMP
- Watson distribution for angular dispersion
- Gamma distribution for fibre radii
- Fibre Radial Growth (FRG) algorithm
- Marching cubes meshing to PLY format
