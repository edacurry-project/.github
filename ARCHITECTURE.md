# EDACurry target architecture

This document defines the intended repository boundaries for the EDACurry project. It is a target architecture for the migration from the current monolithic codebase; it does not require the existing code to be reorganized before it is ready.

## Design goal

EDACurry uses a language-independent internal representation as the boundary between parsing and generation:

```text
Eldo / Spectre / JSON / XML
             |
             v
    edacurry-frontend
             |
             v
       EDACurry AST/IR
       edacurry-core
             |
             v
     edacurry-backend
             |
             v
Eldo / Spectre / JSON / XML
```

Higher-level bindings and applications depend on these libraries rather than owning their implementation.

## Repository boundaries

### `edacurry-core`

Owns the language-independent C++ model and algorithms shared by the rest of the project.

Expected responsibilities:

- AST/IR node types and ownership model
- common enums and identifiers
- factories
- visitors and transformations
- language-independent analysis
- common utilities required by the public C++ API
- public headers and CMake package configuration

`edacurry-core` must not depend on:

- ANTLR or parser-generator artifacts
- Eldo or Spectre syntax
- frontend-specific code
- backend-specific code
- pybind11 or Python packaging

Dependencies must point toward core, never from core toward another EDACurry repository.

### `edacurry-frontend`

Owns all input-language parsing and deserialization into the EDACurry AST/IR.

Expected responsibilities:

- Eldo grammar and parser
- Spectre grammar and parser
- generated ANTLR sources when generated sources are intentionally versioned
- Eldo -> EDACurry conversion
- Spectre -> EDACurry conversion
- JSON/XML readers when they represent input formats
- parser-specific diagnostics and frontend tests

ANTLR grammar files are source code of the frontend. There is intentionally no separate `edacurry-grammar` repository.

### `edacurry-backend`

Owns serialization and generation from the EDACurry AST/IR.

Expected responsibilities:

- EDACurry -> Eldo generation
- EDACurry -> Spectre generation
- JSON/XML writers when they represent output formats
- formatting and backend-specific diagnostics
- backend tests

The backend depends on `edacurry-core` and must not depend on the frontend for normal operation.

### `edacurry-python`

Owns the Python-facing distribution layer.

Expected responsibilities:

- pybind11 bindings
- Python package layout
- type stubs
- packaging metadata and wheels
- Python-level convenience API

The Python repository may depend on core, frontend and backend. The underlying C++ implementation must not live in the binding layer.

### `edacurry-regression`

Owns cross-repository integration and regression testing.

Expected coverage:

- Eldo -> AST -> Eldo
- Spectre -> AST -> Spectre
- Eldo -> AST -> Spectre
- Spectre -> AST -> Eldo
- JSON/XML round trips where applicable
- known real-world netlists
- scalability/benchmark inputs and checks
- compatibility checks across released component versions

This repository is the final end-to-end signal for whether the EDACurry toolchain works as a system.

### Tool repositories

Substantial tools and applications built on EDACurry live in independent repositories under the organization. They are not collected inside a generic `edacurry-tools` source repository.

A repository named `edacurry-tools` should only exist if there is a concrete need for small shared development/build/support utilities. It must not become an application dumping ground.

## Dependency direction

The expected dependency graph is:

```text
                         +-------------------+
                         |  edacurry-python  |
                         +---------+---------+
                                   |
                    +--------------+--------------+
                    |              |              |
                    v              v              v
          +---------+------+ +-----+----------+ +--+----------------+
          | edacurry-core  | |   frontend     | |     backend       |
          +----------------+ +--------+--------+ +---------+---------+
                    ^               |                    |
                    +---------------+--------------------+
```

More simply:

```text
frontend -> core <- backend
               ^
               |
             python
```

Tool repositories depend on the public APIs they need. Regression may orchestrate all repositories but should not contain production implementation code.

## Naming conventions

Unless migration evidence requires a different choice, use:

- C++ namespace: `edacurry`
- CMake package namespace: `EDACurry::`
- core target: `EDACurry::Core`
- frontend target: `EDACurry::Frontend`
- backend target: `EDACurry::Backend`
- Python import package: `edacurry`

Targets should expose dependencies through normal CMake package configuration rather than relying on hard-coded sibling paths.

## Versioning

During the initial modularization phase, EDACurry components should use coordinated semantic versions in the `0.x` series. This reduces compatibility ambiguity while repository boundaries and public APIs are still stabilizing.

Independent versioning can be introduced later if components develop genuinely independent release cadences and compatibility guarantees.

## Branch policy

For the new repositories:

- `main` is the stable branch and should be protected.
- `develop` is the integration branch while the project is pre-1.0 and undergoing active cross-repository development.
- migration work should use explicit branches such as `migration/initial-split` until the extracted repository builds and its relevant regression tests pass.
- feature/fix/refactor branches should be short-lived and merged through pull requests.

The current legacy/working monorepo is not required to adopt this policy during ongoing development.

## Generated parser policy

The grammar files are always the source of truth.

The migration must explicitly choose one generated-source policy:

1. generate parser sources during the build/CI, or
2. version generated C++ sources and have CI regenerate them and assert that the working tree remains clean.

For a C++ library intended to build without requiring Java and the ANTLR generator on end-user systems, option 2 may be preferable. The generator version must be pinned by build tooling rather than by an unexplained binary JAR committed beside the grammar.

## Migration principle

Repository splitting happens only after the received monorepo has been audited and the desired boundaries are validated. Preserve Git history where practical by filtering the original repository rather than copying files into fresh unrelated histories.
