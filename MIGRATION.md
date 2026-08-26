# EDACurry monorepo migration playbook

This document describes how to receive the final working monorepo, audit it, split it into the new EDACurry repositories, and preserve relevant Git history without modifying the legacy repository's `main` branch.

## Safety rules

1. The source monorepo is treated as read-only during extraction.
2. Never rewrite or force-push its `main` branch.
3. Perform filtering only on disposable local clones or mirrors.
4. Record the exact source commit SHA used for the migration.
5. Do not push an extracted repository until it builds independently enough to validate its boundary.
6. Do not delete the legacy monorepo after migration. It remains the authoritative record of the complete pre-split history.

## Phase 0: receive and freeze the source snapshot

When the working package is ready:

```sh
git clone --mirror <SOURCE_REPOSITORY_URL> edacurry-source.git
cd edacurry-source.git
git fsck --full

git rev-parse main
```

Record the resulting SHA in the migration notes.

If development continues after the agreed migration point, do not silently repeat the split from a different SHA. Treat later synchronization as a separate migration step.

## Phase 1: audit before splitting

Before running `git filter-repo`, inspect:

- complete directory tree
- CMake targets and include paths
- public headers
- C++ namespace usage
- dependency graph between AST, parser, writer and Python code
- generated parser sources and their grammar inputs
- bundled third-party libraries
- tests and fixtures
- documentation and licensing files
- files shared by multiple future repositories

Produce a final path-to-repository mapping before extraction.

The current public EDACurry layout suggests the following preliminary mapping, but the received codebase takes precedence.

### Preliminary `edacurry-core`

Likely candidates include language-independent portions currently similar to:

```text
sources/include/edacurry/
sources/src/edacurry/structure/
sources/src/edacurry/features/
sources/src/edacurry/utility/
sources/src/edacurry/factory.cpp
sources/src/edacurry/enums.cpp
```

Some parts of `edacurry.cpp` may belong to core and some may belong to bindings. Split by responsibility, not by filename convenience.

### Preliminary `edacurry-frontend`

Likely candidates include:

```text
grammar/
sources/include/antlr4parser/
sources/src/antlr4parser/
sources/src/edacurry/frontend/
```

Any parser-specific public headers belong here. Grammar files and their generated parser implementation move together.

### Preliminary `edacurry-backend`

Likely candidates include:

```text
sources/src/edacurry/backend/
```

Corresponding public headers must be included once identified.

### Preliminary `edacurry-python`

Extract Python/pybind-specific code only after separating it from core implementation. The current monorepo may mix bindings into otherwise general C++ translation units; this is expected to require a normal refactor before or during extraction.

### Preliminary `edacurry-regression`

Regression content should be selected from existing tests and fixtures that validate multiple components together. Unit tests that exercise only one library stay with that library.

### Tools

Each substantial application is evaluated independently and gets its own repository when appropriate. Do not place applications into a generic tools repository merely because they were under a historical `tools/` directory.

## Phase 2: create destination repositories

Create destination repositories empty: no README, license or initial commit. This lets filtered history become the natural repository history.

Initial expected repositories:

```text
edacurry-core
edacurry-frontend
edacurry-backend
edacurry-python
edacurry-regression
```

Additional tool repositories are created only when their migration is actually scheduled.

## Phase 3: extract history with `git filter-repo`

Install `git-filter-repo` and use a fresh clone for each destination.

General pattern:

```sh
git clone <SOURCE_REPOSITORY_URL> work-core
cd work-core

git filter-repo \
  --path <PATH_1> \
  --path <PATH_2> \
  --path <PATH_3>
```

Then reorganize paths either during filtering with `--path-rename` or in a dedicated migration commit after extraction.

Example shape only:

```sh
git filter-repo \
  --path sources/src/edacurry/structure/ \
  --path sources/src/edacurry/features/ \
  --path sources/src/edacurry/utility/ \
  --path sources/src/edacurry/factory.cpp \
  --path sources/src/edacurry/enums.cpp
```

Do not copy this command blindly. The final path list must be generated from the received source snapshot.

### Preserve shared repository metadata intentionally

Files such as these may need to be reintroduced after filtering:

```text
LICENSE
README.md
CITATION.cff
CMake modules
formatting configuration
```

Do not retain historical build files that describe the old monolithic build unless they remain valid for the extracted repository.

## Phase 4: normalize each extracted repository

After history extraction, make a dedicated migration commit that establishes the new repository root and build interface.

For each repository:

- move files into a clean conventional layout
- add a focused top-level `CMakeLists.txt`
- add or adapt README
- retain license and attribution
- define exported CMake targets
- eliminate hard-coded sibling paths where possible
- remove obsolete bundled/generated artifacts only after their replacement is proven
- keep migration-only changes separate from semantic code changes

Prefer commits such as:

```text
migration: extract core history
build: make core standalone
refactor: separate Python bindings from core
```

Avoid a single opaque commit that both restructures the repository and changes parser behavior.

## Phase 5: validate dependency boundaries

Required direction:

```text
frontend -> core <- backend
               ^
               |
             python
```

Validation questions:

- Can core configure without ANTLR, pybind11, Eldo/Spectre parser sources and backend code?
- Can frontend build against only the public core interface plus parser dependencies?
- Can backend build against only the public core interface plus serializer dependencies?
- Can Python bindings consume installed/exported CMake targets instead of compiling every implementation source directly?
- Are frontend and backend independently testable?

Any violation is resolved before the corresponding repository is considered migrated.

## Phase 6: establish regression orchestration

`edacurry-regression` should check out known-compatible revisions of the components and run at least:

```text
Eldo -> AST -> Eldo
Spectre -> AST -> Spectre
Eldo -> AST -> Spectre
Spectre -> AST -> Eldo
JSON/XML round trips where applicable
```

Preserve representative real-world and scalability inputs subject to licensing and repository-size constraints.

The regression repository becomes the cross-repository compatibility gate.

## Phase 7: publish the initial modular baseline

Before calling the migration complete:

- all extracted repositories have CI
- component unit tests pass
- regression tests pass across the selected revisions
- READMEs document local sibling builds and/or installed package usage
- public target names follow the project convention
- migration source SHA is recorded
- legacy repository is clearly marked as legacy only when active development has actually moved

Only then create coordinated pre-1.0 tags such as `v0.x.y` across the new components.

## Synchronizing late changes from the source monorepo

If commits land in the monorepo after the initial extraction but before cutover, do not manually copy files.

Preferred sequence:

1. identify the source commit range after the recorded migration SHA;
2. classify each commit by destination repository;
3. re-run the filter on a fresh clone or cherry-pick only when the mapping is unambiguous;
4. run component and regression tests again;
5. update the recorded cutover SHA.

This keeps the provenance auditable.

## Migration record

When the package arrives, create a migration record containing at least:

```text
Source repository:
Source main SHA:
Migration date:
Core extraction paths:
Frontend extraction paths:
Backend extraction paths:
Python extraction paths:
Regression extraction paths:
Generated parser policy:
ANTLR version:
Known compatibility exceptions:
```

That record should be committed to `edacurry-regression` or another permanent project-governance location after cutover.
