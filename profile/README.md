# EDACurry

EDACurry is an open-source framework for parsing, representing, transforming, and generating transistor-level netlists across multiple SPICE-family languages and interchange formats.

The project is built around a shared, language-independent Abstract Syntax Tree (AST), allowing netlists to be translated and manipulated without coupling transformations to a specific input or output syntax.

## Project structure

EDACurry is being organized as a modular toolchain:

- **edacurry-core** — shared AST/IR, data model, transformations, and common C++ APIs.
- **edacurry-frontend** — parsers and readers that translate supported input formats into the EDACurry AST. Parser grammars belong here.
- **edacurry-backend** — generators and writers that translate the EDACurry AST into supported output formats.
- **edacurry-python** — Python bindings and packaging for the EDACurry libraries.
- **edacurry-regression** — cross-repository integration, conversion, regression, and scalability tests.
- **Individual tool repositories** — substantial applications built on EDACurry live in their own repositories rather than in a common tools repository.

```text
Eldo / Spectre / JSON / XML
             |
             v
    edacurry-frontend
             |
             v
       EDACurry AST
       edacurry-core
             |
             v
     edacurry-backend
             |
             v
Eldo / Spectre / JSON / XML
```

Python bindings and higher-level applications are built on top of these components.

## Migration status

The modular repository structure is currently being prepared while development of the existing EDACurry codebase continues independently. Components will be migrated once their interfaces and cross-repository dependencies are ready to be stabilized.

Until then, repositories in this organization may be incomplete or intentionally empty.
