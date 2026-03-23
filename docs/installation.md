# Installation

## Requirements

| Dependency | Version | Purpose |
|------------|---------|---------|
| Python | >= 3.9 | Runtime |
| g++ | C++20 support | Compiling the fibre optimizer |
| OpenMP | (bundled with g++) | Parallel optimization |

### System packages (Ubuntu/Debian)

```bash
sudo apt install build-essential g++ libomp-dev
```

### System packages (macOS)

```bash
brew install gcc libomp
```

> On macOS, Apple Clang does not ship OpenMP. Install GCC via Homebrew and ensure `g++` points to the Homebrew version, not Apple Clang.

## Install from source

```bash
git clone https://github.com/Juanitovh/CACTUS.git
cd CACTUS
make
```

This runs two steps:
1. Compiles the C++ optimizer binary (`source_fibre_optimization/joint_fibre_optimizer`)
2. Installs the Python package in editable mode (`pip install -e .`)

After installation, the `cactus1-substrates` command is available in your terminal.

### Manual install

If you prefer to run each step separately:

```bash
make build-cpp       # Compile C++ optimizer only
pip install -e .     # Install Python package in editable mode
```

### Verify installation

```bash
cactus1-substrates --help
```

You should see the list of available subcommands: `init`, `optimize`, `grow`, `quick-mesh`, `monitor`, `convert`, `paste-mesh`.

## Optimizer binary resolution

The Python package finds the C++ optimizer binary automatically through a fallback chain:

1. `CACTUS_OPTIMIZER_PATH` environment variable (if set)
2. `joint_fibre_optimizer` on your system `PATH`
3. `<repo>/source_fibre_optimization/joint_fibre_optimizer` (default build location)

No manual path configuration is needed. If you move the binary, set the environment variable:

```bash
export CACTUS_OPTIMIZER_PATH=/path/to/joint_fibre_optimizer
```

## Troubleshooting

### `omp.h not found` during C++ compilation

Your compiler does not have OpenMP headers. On Ubuntu: `sudo apt install libomp-dev`. On macOS, make sure you are using Homebrew GCC, not Apple Clang.

### `clang` is used instead of `g++`

If your system sets `CXX=clang++`, the Makefile overrides it to `g++`. If you still have issues, explicitly set:

```bash
CXX=g++ make build-cpp
```

### Python version mismatch

CACTUS requires Python >= 3.9. Check your version:

```bash
python --version
```

### First run is slow

Numba JIT-compiles several numerical functions on first invocation. Subsequent runs will be fast.
