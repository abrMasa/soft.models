# soft.models: contributing

> How to set up a development environment, the Git and Jira workflow and the conventions of `soft.models`. Back to the [README](README.md) · [Architecture](ARCHITECTURE.md)

## Development environment

The build of the repository is described in the [README](README.md#build); the tooling of the ecosystem is on the transverse pages:

- Go: the version of `go.mod`, `GOPRIVATE=github.com/masagroup/*` with `gh auth setup-git` or an SSH `insteadOf` rewrite for the private modules; `staticcheck` installed by the build.
- Node and yarn: the Node version of the Dockerfile or of `package.json` (`engines`), yarn with the lockfile of the repository, a `%USERPROFILE%\.npmrc` with a `read:packages` PAT for `npm.pkg.github.com` (`@masagroup` scope).
- C++: the Windows toolset above, or the development container of the repository on Linux (`docker compose --profile dev up -d`, then `cmake` and `ninja` inside it).
- Python: Python 3 (Python 2.7 only where the repository still uses legacy scripts).
- Editor rules for Claude Code: the shared `claude/` folder of [cicd](https://github.com/masagroup/cicd) installs the same `CLAUDE.md` for every developer (rules, roles, build pages, conventions).

## Git and Jira workflow

- The organisation repository `masagroup/soft.models` is the reference (CI, releases); work on a fork or on a branch of it, never directly on `master` nor on a release branch (`6.28`...).
- Branch name: `<TICKET>-<short-description>` (for example `SWNG-421-tests-and-fixes`), created from the target branch of the ticket.
- Commit message: `<type>(<scope>): <TICKET-ID>: <imperative description in English>`, with `type` in `feat`, `fix`, `refactor`, `test`, `build`, `ci`, `perf`, `docs`, `chore`; the body says what and why, and the CI impact when there is one. One commit is one coherent change; no `Co-Authored-By` nor other automatic signature.
- Pull request: masabot reads the Jira keys (`SW`, `SWNG`, `IO`, `AI`, `GEFORA`, `COA`) in the title, the branch, the body and the commits, comments the Jira issue and moves it (*In Progress*, *In Review* with `[DONE]`, *In QA* or *In Validation* after the merge). The PR is built rebased on its base branch; the comment commands `/rebase`, `/rebase-all` and `/automerge` are available. See [CI/CD pipelines](https://github.com/masagroup/wiki/wiki/cicd-pipelines).
- Never push to `master` by hand, never create a release or push an image: the CI does it.

## Conventions

- Code, comments, commit messages and file names in English; documentation of the repository in English.
- Formatting of the repository (`clang-format`, `gofmt`/`goimports`, `prettier`/`eslint`) on the lines you touch; no reformatting of a whole file for a one-line change; no dead code, no `TODO` without a ticket; logs at the right level, never a secret.
- C++: C++17 as accepted by both MSVC v140 (`vc140_x64`) and recent GCC/Clang; never replace a forward declaration by an include in a header; the frozen libraries (Boost 1.56, Qt 4.8.7, protobuf 2.x on the old gen) bound what is available; determinism of the simulation (no iteration over unordered containers for observable results, no dependency on addresses, randomness and clock from the simulation only); modern CMake, `std::filesystem`, no hard-coded path.
- Go: the version of `go.mod` and of the Docker image, `GOPRIVATE=github.com/masagroup/*`; `go vet`, `staticcheck`, `gofmt` and `go test -race ./...` green before a commit; wrapped errors (`fmt.Errorf("...: %w", err)`), no `log.Fatal` outside `main`, clean stop on `SIGINT` and `SIGTERM`, timeouts on every network I/O; REST APIs from an OpenAPI contract (`oapi-codegen`).
- TypeScript: Angular with Nx, the `sw.design-system` components before custom ones, yarn with a frozen lockfile, the lint and the tests of the repository, no new `any`, no dependency added without a reason.
- Python: Python 3 for scripts and tests, no dependency outside the standard library without a `requirements` file.
- Shared contracts (`.proto`, `.ecore`, OpenAPI, exported Go modules, file formats) are changed with the consumers in mind: grep the workspace, state the compatibility strategy, update the dependants.
- Documentation: this README set (README, ARCHITECTURE, CONTRIBUTING, README.LEGACY) is the source of the [SWORD wiki](https://github.com/masagroup/wiki/wiki/soft.models); keep the metadata block at the top of the README up to date (the `apps` list feeds the Interfaces section).

## CI/CD

* No `.github/` directory: no workflows, no artifacts, no releases built here.
* Versioning is by git commit: `soft.go` and `soft.ts` pin a commit SHA in their CI workflows (`.github/workflows/build_and_test.yaml`); `soft.cpp` and the `soft` umbrella use a local path / submodule. A change here only takes effect in a consumer's CI after a SHA bump in that consumer's workflow.
* Observed pins: `soft.go` at `90286c3...` (merge of PR #22 `rename.library.index`, 9 commits behind HEAD, missing PRs #23, #24, #25); `soft.ts` at `f0dd72e...` (= HEAD).
* Tags: 9 local tags `v1.1.0` to `v1.8.0`; latest `v1.8.0` = `91fcdc5` (2023-09-22), 15 commits behind HEAD. HEAD `f0dd72e` (merge of PR #25 `ecore.factory`, 2024-09-19) is untagged.
* General pipeline documentation: [cicd](https://github.com/masagroup/cicd) and the ecosystem wiki page [CI/CD pipelines](https://github.com/masagroup/wiki/wiki/cicd-pipelines).

## Known issues and open questions

They are not kept in the repository: see the [private artifact](https://claude.ai/artifact/UfvKmky7e8LwEXf192hQjQ#soft.models) that holds them for every page of the wiki.
