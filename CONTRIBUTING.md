# Contributing

## 1. Local Workflow

Use the Nix development shell for repository work:

```sh
nix develop
```

Run the focused local checks before committing:

```sh
nix fmt
nix flake check --print-build-logs
nix build --print-build-logs
```

For a fast Go-only loop while editing formatter behavior, use:

```sh
nix develop .#ci --command go test ./...
```

Stage newly added files before running the full flake check. The pre-commit
check surface is part of `nix flake check`, and staged files make the local
result match the CI workflow more closely.

## 2. Formatter Development

`mdfmt` is the preferred public CLI. The default formatter pipeline runs
heading numbering, heading spacing normalization, and pipe table alignment.

Useful local commands:

```sh
go test ./...
nix build --print-build-logs
./result/bin/mdfmt --write README.md CONTRIBUTING.md AGENTS.md CLAUDE.md RELEASING.md
```

Use `--no-heading-numbering` when checking spacing and table formatting without
renumbering headings. Keep behavior decisions that affect Markdown output in
`docs/behavior-decisions.md`.

## 3. Nix And CI

The flake owns the Go toolchain, formatter checks, release tooling, and CI
entry points. The module's minimum supported Go version and the Nix build
toolchain are separate policies.

Go version policy:

- Keep `go.mod` at the oldest supported major.minor (`go 1.27`). Raise it only
  when intentionally ending support for that version.
- Use `pkgs.go_1_27` for Nix builds, development shells, CI, and releases.
  The pinned `nixpkgs-26.05-darwin` revision supplies the patch version and
  keeps `x86_64-darwin` available; current unstable no longer evaluates that
  declared system. Updating the pin changes other Nix tools too. Review those
  changes, run `nix flake check` and `nix build`, and confirm the Linux and
  macOS CI jobs before treating a new pin as verified.
- Do not add a `toolchain` directive to `go.mod` without a specific need.
  Nix shells set `GOTOOLCHAIN=local` so Go does not download a different
  compiler automatically.
- Dependabot's Nix flake updates can propose newer `nixpkgs` lock revisions.
  Review the Go patch version in each proposal; these updates can also change
  other packages and are not Go-specific patch proposals.
- Minor-version migrations require updating the Nix package reference and
  reviewing the module minimum separately, then rerunning compatibility checks.
  CI requires the Nix Go version to be at least the `go.mod` minimum.

The primary CI workflow is `.github/workflows/ci.yml` with workflow name `ci`.
It runs Nix checks, Nix build, Go vulnerability checks, and secret scanning.
`AGENTS.md` is the source of truth for agent guidance, and `CLAUDE.md` points
Claude-compatible tooling to it.

## 4. Commit Expectations

Use conventional commit messages such as `docs: update release guide` or
`fix(formatter): preserve fenced code blocks`. Keep docs, behavior changes,
workflow changes, and dependency changes separated when the review surface is
meaningfully different.

Do not include machine-local absolute paths in commits, public docs, issues, or
pull request text. Use repo-relative paths such as `README.md` and
`.github/workflows/ci.yml`.

## 5. Releases

Release work is documented in `RELEASING.md`. Run the pre-release checklist
there before creating a release tag or using the manual release workflow.
