<!--
sword:
  schema: 1
  name: soft.models
  family: ecore-esync
  kind: ecore-model
  generation: shared
  shipped: false # UNVERIFIED
  languages: [XML, Ecore]
  build: [none]
  ng: [ecore-model]
  apps: []
-->

# soft.models

> Ecore meta-models (`.ecore` XMI files) that the SOFT code generators consume to produce the `soft.cpp`, `soft.go` and `soft.ts` Ecore runtime libraries and their tests; data only, no executable code.

| | |
|---|---|
| **Family** | Ecore / esync |
| **Languages** | XML (Ecore/XMI), Eclipse launch XML. No Go, C++, TypeScript or Python source. |
| **Build systems** | None (no `build.xml`, Dockerfile, CMake, `go.mod`, Makefile or `package.json`) |

**[Architecture](ARCHITECTURE.md)** · [Contributing](CONTRIBUTING.md) · [known issues and open questions](https://claude.ai/artifact/UfvKmky7e8LwEXf192hQjQ#soft.models) · [repository on GitHub](https://github.com/masagroup/soft.models)

## Overview

`soft.models` holds the input models of the SOFT generator chain (`soft.gen`). Its main file, `ecore.ecore`, is the SOFT port of the Eclipse EMF Ecore meta-model, annotated with per-language generator hints (C++, Go, TypeScript). Running the generators on it produces the Ecore runtime itself in the three languages (`soft.cpp`, `soft.go`, `soft.ts`). The repository also ships sample models (`library`, `tournament`, `empty`) used to generate tests and examples.

<details>
<summary>More</summary>

The repository contains no service and speaks no protocol. Its only "clients" are the generators and the CI of the runtime repositories, which use it through a pinned-commit CI checkout, a local path or a git submodule.

NG-specific models (`physical`, `simulation`) are NOT here: they live in `sw/sw.api.ng/ecore/` (see [sw.api.ng](https://github.com/masagroup/sw.api.ng)). The NG link is indirect: NG Go services depend on `soft.go`, which is generated from `ecore.ecore`.

</details>

The diagram of this overview is in [Architecture](ARCHITECTURE.md#overview).

## Build

There is nothing to build in this repository. The models are consumed by other repositories; the regeneration commands below are the real "build" of the data.

### Linux (Docker)

Not applicable to this repository. The consumers run the generators of [soft.gen](https://github.com/masagroup/soft.gen):

<details>
<summary>Details</summary>

| Generator | Used by |
|---|---|
| soft.gen v2: the Go command `soft.gen` (`go`, `ts`), at the version `SOFT_GEN_VERSION` of each Makefile | `soft.go/Makefile`, `soft.ts/Makefile`; the `release/v2` branches of soft.sync.go and sw.api.ng.go |
| v1 image `masagroup/soft.generator.cpp` (Java, Acceleo), no tag | `soft.cpp/build.xml` |
| v1 image `masagroup/soft.generator.go` (Java, Acceleo), no tag | `sw.api.ng.go/generate.sh` and `soft.sync.go/Makefile` on their `master` |

soft.go and soft.ts no longer use the v1 images `masagroup/soft.generator.go|ts`: they generate with soft.gen v2 (`go run .../cmd/soft-gen@v2.0.0-alpha.2` until 2026-10-22, the image `masagroup/soft.gen:<SOFT_GEN_VERSION>` since). Their Makefiles read the models from a sibling `../models` checkout (mounted on `/models` when the generator runs in a container), so a local `make generate` expects one.

</details>

### Code generation

The `generate.*.launch` files run the generator main classes from inside the `soft.gen` workspace, where `../../../models` resolves to this repository and `../../../cpp|go|ts` to the runtime repositories (umbrella `soft` layout). All use the Maven classpath provider (`org.eclipse.m2e.launchconfig.classpathProvider`), i.e. they assume the generators are Maven-built Java projects (`soft.gen` has a `pom.xml`, per the scan).

<details>
<summary>Details</summary>

| Launch file | Main class | Program arguments |
|---|---|---|
| `generate.ecore.cpp.launch` | `soft.generator.cpp.Generate` | `-m ../../../models/ecore.ecore -o ../../../cpp -t generateModel -t generateLibraryCMakeSources -t generateTests -t generateTestsCMakeSources -t generateTestsCMakeDependencies` |
| `generate.ecore.go.launch` | `soft.generator.go.Generate` | `-m ../../../models/ecore.ecore -o ../../../go` |
| `generate.ecore.ts.launch` | `soft.generator.ts.Generate` | `-m ../../../models/ecore.ecore -o ../../../ts -P ../../../ts/generator.properties -t generateModel -t generateTests` |
| `generate.library.cpp.launch` | `soft.generator.cpp.Generate` | `-m ../../../models/library.ecore -o ../../../cpp` |
| `generate.library.go.launch` | `soft.generator.go.Generate` | `-m ../../../models/library.ecore -o ../../../go/test` |
| `generate.library.ts.launch` | `soft.generator.ts.Generate` | `-m ../../../models/library.ecore -o ../../../ts/examples -P ../../../ts/generator.properties -p accessors=property` |
| `generate.tournament.go.launch` | `soft.generator.go.Generate` | `-m ../../../models/tournament.ecore -o ../../../go/test` |
| `generate.tournament-reflective.go.launch` | `soft.generator.go.Generate` | `-m ../../../models/tournament.ecore -o ../../../go -P ../../../go/generator.properties -p featureDelegation=Reflective -p module=github.com/masagroup/soft.go -p packages=github.com/masagroup/soft.go/test/tournament-reflective/tournament#github.com/masagroup/soft.go/test/tournament-reflective/internal/impls/impls#github.com/masagroup/soft.go/test/tournament-reflective/internal/mocks/mocks` |

Generator CLI flags visible in these files: `-m` model, `-o` output directory, `-P` properties file, `-p key=value` single property, `-t` template/task (repeatable; the form `-t !name` is used in `soft/cpp/build.xml`). The `generate.ecore.ts.launch` configuration sets `STOP_IN_MAIN=true`.

</details>

## Quickstart

Not applicable: this repository produces no executable, service, port, environment variable or runtime configuration.

## Interfaces and access methods

No executable is produced by this repository (library, contract, data or tooling): see the build and the dependencies.

## Documentation and guides

- [Architecture](ARCHITECTURE.md): position in the ecosystem, repository layout, internal structure, how it works (diagrams), dependencies.
- [Contributing](CONTRIBUTING.md): development environment, Git and Jira workflow, conventions, CI/CD.
- [Known issues and open questions](https://claude.ai/artifact/UfvKmky7e8LwEXf192hQjQ#soft.models): the private artifact that holds them (every page of the wiki).
- Related repositories: [soft](https://github.com/masagroup/soft), [soft.cpp](https://github.com/masagroup/soft.cpp), [soft.gen](https://github.com/masagroup/soft.gen), [soft.go](https://github.com/masagroup/soft.go), [soft.ts](https://github.com/masagroup/soft.ts).
- Transverse pages: [Build on Windows](https://github.com/masagroup/wiki/wiki/build-windows), [Build on Linux](https://github.com/masagroup/wiki/wiki/build-linux), [CI/CD pipelines](https://github.com/masagroup/wiki/wiki/cicd-pipelines), [Ecosystem today](https://github.com/masagroup/wiki/wiki/ecosystem-current), [Ecosystem next gen](https://github.com/masagroup/wiki/wiki/ecosystem-nextgen), [Glossary](https://github.com/masagroup/wiki/wiki/glossary).
- Repository on GitHub: [masagroup/soft.models](https://github.com/masagroup/soft.models).
- The same documentation in the SWORD wiki: [soft.models](https://github.com/masagroup/wiki/wiki/soft.models), [architecture](https://github.com/masagroup/wiki/wiki/soft.models-architecture), [contributing](https://github.com/masagroup/wiki/wiki/soft.models-contributing).
