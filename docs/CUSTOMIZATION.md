# Customization Guide

This guide explains how to customize `cpp-devkit` for an individual C++ repository while keeping shared defaults consistent across projects.

## 1. How customization works

`cpp-devkit` provides shared defaults for:

- `.clang-format` — source-code formatting.
- `.clang-tidy` — static analysis checks.
- `.github/workflows/cpp-quality.yml` — reusable GitHub Actions workflow.
- `.gitignore` — common C++ and CMake-generated files.

A consuming repository can use the shared defaults directly or keep its own configuration files.

### Configuration precedence

The reusable workflow follows this rule:

1. If the consuming repository contains its own `.clang-format`, that file is used.
2. Otherwise, the workflow downloads `.clang-format` from the selected `cpp-devkit` release.
3. The same rule applies independently to `.clang-tidy`.

**Important:** Configuration files are not merged automatically. A local file replaces the shared default for that tool. If you need to customize only a few settings, copy the complete shared file into your repository and edit the relevant settings. This preserves the other defaults.

## 2. Customizing clang-format

### 2.1 Use the shared formatting rules unchanged

If your repository does not contain a `.clang-format` file, the reusable workflow downloads the shared configuration.

You can also run clang-format locally:

```bash
clang-format --version
clang-format --dry-run --Werror src/example.cpp
```

To format files in place:

```bash
clang-format -i src/example.cpp
```

To check all tracked C++ source and header files:

```bash
git ls-files '*.cpp' '*.cc' '*.cxx' '*.h' '*.hpp' '*.hh' '*.hxx' \
  | xargs -r clang-format --dry-run --Werror
```

### 2.2 Override selected formatting settings

For a project-specific style, copy the complete `.clang-format` file from `cpp-devkit` into your repository:

```bash
curl -L \
  https://raw.githubusercontent.com/rathoddinesh14/cpp-devkit/v1.0.0/.clang-format \
  -o .clang-format
```

Then edit the settings you want to change. For example, to use a 120-character line limit and a two-space indentation width, change these entries in the copied file:

```yaml
ColumnLimit: 120
IndentWidth: 2
```

These are edits to the full copied configuration—not a partial overlay. Keep the rest of the file so that all other shared formatting decisions remain in effect.

### 2.3 Validate your formatting configuration

Run:

```bash
clang-format --style=file --dump-config
clang-format --dry-run --Werror path/to/source.cpp
```

The first command prints the effective configuration. The second checks whether the file already follows it.

## 3. Customizing clang-tidy

The shared `.clang-tidy` enables selected checks from these families:

- `bugprone-*`
- `clang-analyzer-*`
- `misc-*`
- selected `modernize-*`
- `performance-*`
- selected `readability-*`

The configuration also contains exclusions to avoid noisy or low-value diagnostics.

### 3.1 Use a project-specific header filter

`HeaderFilterRegex` controls which headers are included in diagnostics. For example, a project with public headers under `include/aryabhatta/` can use:

```yaml
HeaderFilterRegex: 'include/aryabhatta/.*'
```

A graphics library with headers under `include/vishwakarma/` can use:

```yaml
HeaderFilterRegex: 'include/vishwakarma/.*'
```

Copy the complete shared `.clang-tidy` file first, then change the filter to match your repository's public include layout.

### 3.2 Enable additional checks

You can broaden the enabled check set. For example:

```yaml
Checks: >
  bugprone-*,
  clang-analyzer-*,
  cppcoreguidelines-*,
  misc-*,
  modernize-*,
  performance-*,
  readability-*
```

Be aware that enabling entire check families can produce a large number of diagnostics, especially in an established codebase. Enable broader groups gradually and review the results before making them mandatory in CI.

### 3.3 Disable noisy or unsuitable checks

Use a negative pattern prefixed with `-` to exclude a check. For example:

```yaml
Checks: >
  bugprone-*,
  clang-analyzer-*,
  modernize-*,
  performance-*,
  readability-*,
  -readability-magic-numbers
```

This keeps the selected check families enabled except for `readability-magic-numbers`.

### 3.4 Treat warnings as errors

To make clang-tidy diagnostics fail the command, add:

```yaml
WarningsAsErrors: '*'
```

This is best introduced after the project has addressed existing warnings. Otherwise, CI may fail immediately on legacy diagnostics.

### 3.5 Run clang-tidy locally

The build must generate a compilation database:

```bash
cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Debug \
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

Then run clang-tidy against a source file:

```bash
clang-tidy -p build src/example.cpp
```

The `-p build` option tells clang-tidy where to find `compile_commands.json`.

## 4. Customizing the GitHub Actions workflow

The reusable workflow is stored at:

```text
.github/workflows/cpp-quality.yml
```

A consuming repository calls it from its own workflow. For example, create `.github/workflows/ci.yml` in the consuming repository:

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
      config_ref: v1.0.0
      run_format: true
      run_tidy: true
      run_build_and_test: true
      source_dir: .
      build_dir: build
      build_type: Debug
      clang_version: 17
```

Use a released tag such as `v1.0.0` rather than a moving branch for reproducible CI. Create that tag after the corresponding changes have been merged into the repository's default branch.

### 4.1 Available workflow inputs

| Input | Default | Purpose |
|---|---|---|
| `config_ref` | `v1.0.0` | Tag or ref used to download shared configuration files |
| `run_format` | `true` | Enables or disables clang-format validation |
| `run_tidy` | `true` | Enables or disables clang-tidy |
| `run_build_and_test` | `true` | Enables or disables CMake build and CTest |
| `source_dir` | `.` | Directory containing the top-level `CMakeLists.txt` |
| `build_dir` | `build` | CMake build directory |
| `build_type` | `Debug` | CMake build configuration |
| `cmake_args` | empty | Additional CMake configure arguments |
| `clang_version` | `17` | LLVM/Clang version used by the workflow |

### 4.2 Disable a check for a repository

For example, if a repository is not ready for clang-tidy yet:

```yaml
jobs:
  quality:
    uses: rathoddinesh14/cpp-devkit/.github/workflows/cpp-quality.yml@v1.0.0
    with:
      run_format: true
      run_tidy: false
      run_build_and_test: true
```

The other checks remain enabled.

### 4.3 Run formatting only

For a repository that is still being set up:

```yaml
jobs:
  quality:
    uses: rathoddinesh14/cpp-devkit/.github/workflows/cpp-quality.yml@v1.0.0
    with:
      run_format: true
      run_tidy: false
      run_build_and_test: false
```

You can enable the remaining checks once the project has a working CMake build and compilation database.

## 5. Customizing CMake configuration

The workflow uses CMake to configure and build the consuming project. The project must provide a valid `CMakeLists.txt`.

### 5.1 Pass project-specific CMake options

Use `cmake_args` to pass options required by your project:

```yaml
jobs:
  quality:
    uses: rathoddinesh14/cpp-devkit/.github/workflows/cpp-quality.yml@v1.0.0
    with:
      cmake_args: |
        -DBUILD_TESTING=ON
        -DMY_PROJECT_ENABLE_EXAMPLES=OFF
        -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

Use the actual option names defined by your project's CMake configuration. The example option `MY_PROJECT_ENABLE_EXAMPLES` is illustrative.

### 5.2 Build from a subdirectory

If the top-level `CMakeLists.txt` is inside a directory such as `engine/`, set:

```yaml
with:
  source_dir: engine
  build_dir: build
```

The path is relative to the root of the consuming repository.

### 5.3 Project dependencies

The reusable workflow cannot infer every project's external dependencies. If your build requires system packages, SDKs, GPU drivers, or third-party libraries, add the required installation steps in a project-owned workflow or extend the reusable workflow deliberately.

For example, a project that depends on a system package may need a setup step before invoking the reusable workflow. Keep project-specific dependency installation separate from the shared quality configuration unless several repositories genuinely need the same setup.

## 6. Recommended adoption strategy

For an existing codebase, do not enable every check as a blocking requirement on day one.

A practical rollout is:

1. **Start with formatting.** Add the shared `.clang-format` and resolve formatting differences.
2. **Enable the build and tests.** Confirm that CMake configuration, compilation, and CTest work in CI.
3. **Introduce clang-tidy.** Review existing diagnostics and fix the most important issues.
4. **Increase strictness gradually.** Add more checks or `WarningsAsErrors` once the baseline is clean.
5. **Pin a release.** Reference a version tag so all consuming repositories use a predictable toolchain configuration.

For a new repository, you can usually enable all three checks immediately, provided its CMake build and tests are ready.

## 7. Updating to a newer cpp-devkit version

When a new version is released, update the reusable workflow reference and the configuration reference together:

```yaml
jobs:
  quality:
    uses: rathoddinesh14/cpp-devkit/.github/workflows/cpp-quality.yml@v1.1.0
    with:
      config_ref: v1.1.0
```

Review the release notes before upgrading. A change to formatting rules may reformat many files, while new clang-tidy checks may introduce additional diagnostics.

## 8. Design principles

- **Shared by default:** Repositories should start with consistent formatting and analysis rules.
- **Local ownership:** A project can maintain its own complete configuration when its needs differ.
- **No hidden merging:** Local configuration replaces the downloaded default; it is not layered over it.
- **Explicit build settings:** Project-specific CMake options and dependencies remain visible in the consuming repository.
- **Controlled upgrades:** Version tags make changes to shared tooling deliberate and reviewable.

---

**Quick reference**

| Requirement | Recommended approach |
|---|---|
| Use standard formatting | Do not add a local `.clang-format` |
| Change formatting rules | Copy the complete shared file, then edit it |
| Change clang-tidy checks | Copy the complete shared file, then edit it |
| Disable one CI check | Set its `run_*` input to `false` |
| Pass custom CMake options | Use `cmake_args` |
| Use a nested CMake project | Set `source_dir` |
| Keep CI reproducible | Pin the workflow and `config_ref` to a release tag |