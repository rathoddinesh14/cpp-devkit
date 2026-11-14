# Customization

`cpp-devkit` supplies defaults, not mandatory project-wide configuration.

## Local clang-format override

Commit a `.clang-format` file in the root of the consumer repository. The reusable
workflow detects it and does not download the central default.

To start from the shared configuration and adjust only a few settings:

```yaml
BasedOnStyle: file
ColumnLimit: 120
```

For this to inherit the central configuration, the consumer must also make the central
file available locally (for example, by copying it and changing the required settings).
A standalone `.clang-format` file is otherwise treated as the complete configuration.

## Local clang-tidy override

Commit a `.clang-tidy` file in the consumer repository. The workflow preserves it.
The consumer can change enabled checks, exclusions, warning policy and header filters.

## Disable individual jobs

```yaml
with:
  run_format: true
  run_tidy: false
  run_build_and_test: true
```

## CMake customization

Pass project-specific options through `cmake_args`:

```yaml
with:
  cmake_args: |
    -DMYLIB_BUILD_TESTS=ON
    -DMYLIB_BUILD_EXAMPLES=OFF
```

The project must define CTest tests for the test step to execute meaningful checks.

## Important

- Pin the reusable workflow to a release tag.
- Keep local overrides under version control.
- The initial workflow targets Linux and CMake projects.
- clang-tidy requires all relevant translation units to be represented in
  `compile_commands.json`; generated or dependency-heavy projects may need extra setup.
