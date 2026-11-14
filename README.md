# cpp-devkit

Reusable C++ development tooling for projects built with CMake.

`cpp-devkit` provides shared defaults for:

- `clang-format` source formatting
- `clang-tidy` static analysis
- GitHub Actions quality checks
- Optional CMake build and CTest execution
- Consumer-side configuration overrides

## Repository contents

```text
.
├── .clang-format
├── .clang-tidy
├── .gitignore
├── .github/
│   └── workflows/
│       └── cpp-quality.yml
├── docs/
│   ├── INTEGRATION.md
│   └── CUSTOMIZATION.md
├── LICENSE
└── README.md
```

## Quick integration

Create `.github/workflows/quality.yml` in the consumer repository:

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

The reusable workflow checks out and analyzes the **consumer repository**. If the consumer
does not provide `.clang-format` or `.clang-tidy`, the workflow downloads the defaults
from the selected `cpp-devkit` version.

See [Integration](docs/INTEGRATION.md) and [Customization](docs/CUSTOMIZATION.md).

## Requirements

- A CMake-based C++ project for build and test jobs
- CMake must register tests with CTest for the test step
- A usable compilation database for clang-tidy
- Linux GitHub-hosted runner (`ubuntu-latest`) for this initial workflow

The reusable workflow is intentionally small and Linux-focused. Projects with special
system dependencies should install them in their own workflow or extend this workflow
as the project requirements become clear.

## Versioning

Consumers should pin a release tag, for example `@v1.0.0`, rather than tracking `main`.
Configuration and workflow changes can then be adopted deliberately.

## License

MIT. See [LICENSE](LICENSE).
