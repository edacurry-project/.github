# EDACurry repository bootstrap checklist

Use this checklist whenever a new repository is created under `edacurry-project`.

## Repository creation

- Create the repository with the agreed public/private visibility.
- For history-preserving migrations, create it empty: no README, license or initial commit.
- Set a concise description that states the repository's single responsibility.
- Add appropriate repository topics once the component exists.

## Initial branches

After the initial history/import is pushed:

- `main` is the stable branch.
- create `develop` from the accepted modular baseline;
- protect `main` from direct pushes;
- require pull requests for protected branches;
- require relevant CI checks before merge;
- avoid force pushes on protected branches.

Migration work may initially live on `migration/initial-split` until the extracted component is independently valid.

## Required repository content

Each production repository should eventually include:

```text
README.md
LICENSE
CMakeLists.txt               # for C++ repositories
.github/workflows/ci.yml
```

Organization-wide contribution, security, issue and pull-request templates are inherited from `edacurry-project/.github` where GitHub supports inheritance; do not duplicate them unless the repository needs an explicit override.

## README minimum

The README should state:

- what the repository owns;
- what it explicitly does not own where ambiguity is likely;
- dependencies on other EDACurry repositories;
- system/build dependencies;
- configure/build/test instructions;
- install/use instructions where applicable;
- current support status;
- relationship to the rest of the EDACurry project.

## CMake conventions

For C++ libraries:

- provide a normal top-level CMake project;
- avoid hard-coded absolute or developer-specific paths;
- export/install targets where practical;
- expose public dependencies correctly;
- prefer imported target dependencies over raw library paths;
- use the `EDACurry::` namespace for exported targets;
- keep build-tree sibling discovery as a developer convenience, not the only supported integration mechanism.

Expected public target names are documented in `ARCHITECTURE.md`.

## CI

Each C++ repository should call the organization's reusable CMake workflow or provide an equivalent repository-specific workflow.

Baseline checks should include:

- GCC build;
- Clang build;
- strict warnings where supported;
- tests with failure output;
- no dependency on unpublished files from the old monorepo.

Additional repository-specific jobs may test parser regeneration, Python wheels, sanitizers or integration scenarios.

## Tests

Keep tests with the component they validate:

- core unit tests -> `edacurry-core`;
- parser/reader tests -> `edacurry-frontend`;
- writer tests -> `edacurry-backend`;
- Python API tests -> `edacurry-python`;
- cross-component conversions -> `edacurry-regression`.

Do not make a component's unit test suite depend on another component merely to reuse a fixture if that fixture can be made local and focused.

## Migration provenance

For repositories extracted from the monorepo, record:

- source repository URL;
- source commit SHA;
- extraction date;
- `git filter-repo` path mapping;
- post-extraction migration commits;
- known history omissions or rewritten paths.

Preserve the legacy repository as the complete pre-split historical record.

## Release readiness

Before the first coordinated tag:

- CI passes on all supported compilers/platforms;
- public CMake/import interfaces are documented;
- cross-repository regression passes;
- dependency versions/ranges are documented;
- generated-code policy is enforced;
- license and attribution files are present;
- repository README no longer describes the component as an incomplete migration unless that is still true.
