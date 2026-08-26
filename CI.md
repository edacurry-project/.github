# EDACurry CI conventions

The organization provides a reusable CMake workflow at:

```text
edacurry-project/.github/.github/workflows/cmake-ci.yml
```

C++ repositories can call it from their own `.github/workflows/ci.yml`.

## Minimal caller

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  cmake:
    uses: edacurry-project/.github/.github/workflows/cmake-ci.yml@main
```

This runs configure/build/test jobs with both GCC and Clang on `ubuntu-latest`.

## Additional system packages

A repository can request additional Ubuntu packages:

```yaml
jobs:
  cmake:
    uses: edacurry-project/.github/.github/workflows/cmake-ci.yml@main
    with:
      apt_packages: "libexample-dev another-package"
```

Keep repository-specific dependencies in the caller rather than hard-coding frontend/backend dependencies into the shared workflow.

## Additional CMake arguments

The default configure arguments are:

```text
-DSTRICT_WARNINGS=ON -DWARNINGS_AS_ERRORS=ON
```

Override them only when necessary:

```yaml
jobs:
  cmake:
    uses: edacurry-project/.github/.github/workflows/cmake-ci.yml@main
    with:
      cmake_args: >-
        -DSTRICT_WARNINGS=ON
        -DWARNINGS_AS_ERRORS=ON
        -DBUILD_EXAMPLES=OFF
```

## Repositories with cross-repository dependencies

The shared workflow intentionally assumes that the repository can configure from its own checkout plus normal installed/fetched dependencies.

During migration, a component may temporarily need a sibling EDACurry checkout. In that case, use a repository-specific workflow until the component exposes a stable CMake package interface. Do not permanently encode the monorepo layout into the organization-wide workflow.

`edacurry-regression` is expected to use its own orchestration workflow because its purpose is explicitly to check out and test several repositories together.

## Parser regeneration

If `edacurry-frontend` versions generated ANTLR sources, add a repository-specific CI job that:

1. installs/obtains the pinned ANTLR generator version;
2. regenerates parser sources from the grammar files;
3. fails if `git diff --exit-code` reports changes.

This keeps grammar files authoritative while allowing normal consumers to build without running the generator.

## Python CI

`edacurry-python` will need a dedicated workflow for Python versions, wheel builds and import/API tests. The CMake reusable workflow may still be used for its native-library stage if useful, but should not replace Python packaging tests.

## Pinning

During early development callers may reference `@main`. Once the shared workflow stabilizes, prefer a versioned tag or immutable commit SHA for release branches so changes to organization CI cannot unexpectedly alter an old release line.
