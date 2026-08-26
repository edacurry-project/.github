# Contributing to EDACurry

Thank you for contributing to EDACurry.

EDACurry is being organized as a multi-repository C++/Python toolchain. Contributions should preserve clear repository boundaries and keep changes reviewable across components.

## Repository responsibilities

Before opening a change, choose the repository that owns the behavior:

- `edacurry-core`: language-independent AST/IR, common data structures, transformations and shared C++ APIs.
- `edacurry-frontend`: grammars, generated parser sources, parsers and readers that produce the EDACurry AST/IR.
- `edacurry-backend`: writers and generators that consume the EDACurry AST/IR.
- `edacurry-python`: Python bindings, Python package structure and packaging.
- `edacurry-regression`: end-to-end and cross-repository regression tests.
- application/tool repositories: substantial applications built on EDACurry, each in its own repository.

If a change spans several repositories, keep each pull request independently understandable and link the related pull requests.

## Branches

For modular EDACurry repositories:

- `main` is the stable branch.
- `develop` is the active integration branch during pre-1.0 development.
- use short-lived branches for changes, for example `feature/...`, `fix/...`, `refactor/...` or `migration/...`.
- merge through pull requests rather than pushing changes directly to protected branches.

The legacy/working monorepo may follow its existing development process until migration is complete.

## Scope changes tightly

Prefer one concern per pull request. Repository extraction, build-system restructuring and semantic behavior changes should normally be separate changes.

In particular, avoid combining parser behavior changes with large file moves unless the behavior change is required to make the move possible.

## Build and test expectations

For C++ repositories, contributions should normally be validated with both GCC and Clang where practical.

A typical local validation sequence is:

```sh
cmake -S . -B build -DSTRICT_WARNINGS=ON -DWARNINGS_AS_ERRORS=ON
cmake --build build --parallel
ctest --test-dir build --output-on-failure
```

Repository-specific READMEs take precedence when additional dependencies or commands are required.

For changes that affect conversions or interactions between repositories, add or update coverage in `edacurry-regression` as appropriate.

## C++ and public API conventions

Unless a repository documents an exception:

- use C++17 or the project-wide standard currently documented by that repository;
- use the `edacurry` C++ namespace;
- exported CMake targets use the `EDACurry::` namespace;
- avoid exposing parser-generator or binding-specific dependencies through the core public API;
- prefer explicit ownership and const-correct interfaces;
- keep language-specific syntax knowledge out of `edacurry-core`.

## Parser and grammar changes

Grammar source files belong to `edacurry-frontend` and are the source of truth for generated parser code.

When generated parser sources are versioned, a grammar change must also update the generated files using the pinned generator version. CI should eventually verify that regeneration does not produce an unexpected diff.

## Commit and pull request quality

Commit messages should state the purpose of the change. Useful prefixes include:

```text
feat:
fix:
refactor:
build:
test:
docs:
migration:
```

They are conventions rather than a strict parser requirement.

A pull request should explain:

- what changed;
- why the change is needed;
- what was validated;
- any cross-repository compatibility impact.

Use the organization pull request template and link related issues or pull requests.

## Versioning and compatibility

During the initial modularization phase, EDACurry components use coordinated pre-1.0 semantic versions where possible.

Changes to public AST/IR interfaces can affect frontend, backend, Python bindings and tool repositories. Call out such changes explicitly and update regression coverage before coordinated release tags are produced.

## Migration contributions

Repository splitting must follow `MIGRATION.md` in the organization `.github` repository.

Do not rewrite the source monorepo's `main` branch as part of migration. Extraction should happen from disposable clones using a recorded source commit SHA, with relevant history preserved where practical.

## Licensing

Contributions must be compatible with the license of the repository receiving the change. Preserve existing copyright, attribution and third-party license information during migration.
