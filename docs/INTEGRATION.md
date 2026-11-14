# Integration

## 1. Add a caller workflow

Create `.github/workflows/quality.yml` in your C++ repository:

```yaml
name: C++ Quality

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    uses: rathoddinesh14/cpp-devkit/.github/workflows/cpp-quality.yml@v1.0.0
    with:
      run_format: true
      run_tidy: true
      run_build_and_test: true
```

The called workflow runs against the caller's repository checkout. It does not build
cpp-devkit itself.

## 2. Optional inputs

```yaml
jobs:
  quality:
    uses: rathoddinesh14/cpp-devkit/.github/workflows/cpp-quality.yml@v1.0.0
    with:
      config_ref: v1.0.0
      run_format: true
      run_tidy: true
      run_build_and_test: true
      source_dir: .
      build_dir: build-ci
      build_type: Release
      clang_version: "17"
      cmake_args: |
        -DMY_PROJECT_BUILD_TESTS=ON
        -DMY_PROJECT_BUILD_EXAMPLES=OFF
```

- `config_ref`: cpp-devkit tag or branch used to fetch missing default config files.
- `run_format`: enable or disable clang-format.
- `run_tidy`: enable or disable clang-tidy.
- `run_build_and_test`: enable or disable CMake build and CTest.
- `source_dir`: CMake source directory relative to the consumer repository root.
- `build_dir`: build directory name.
- `build_type`: CMake build type.
- `clang_version`: LLVM tool version installed by the workflow.
- `cmake_args`: additional CMake arguments, one per line.

## 3. Project dependencies

The default workflow installs CMake, Ninja, build-essential and the requested LLVM tools.
It does not guess project-specific system packages. If your project needs Eigen, OpenGL,
Vulkan, Qt, or other system dependencies, add a setup job or customize the reusable
workflow for that project before enabling the build and clang-tidy jobs.
