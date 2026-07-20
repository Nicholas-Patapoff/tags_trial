# Branching & Release Strategy

## Branch types

**`main`**
The integration branch. Always reflects the latest, actively-developed code. Never committed to directly — all changes land here via pull request. Every meaningful milestone on `main` gets tagged (see Tagging below).

**`feature/<short-description>`** (short-lived)
Cut from `main` for a single feature, fix, or change. Naming convention: `feature/add-dark-mode`, `feature/fix-login-crash`. Opened as a PR back into `main`. Deleted automatically on merge (repo has "auto-delete head branches" enabled) — these are disposable, not meant to persist.

**`release/<major>.<minor>.x`** (long-lived, created as needed)
Cut from a release tag (e.g. `v1.2.0`) only when that version line needs a fix after `main` has already moved past it. Exists to keep receiving patches for a version line no longer at the tip of `main`. Not created proactively for every release — only when the first backport is actually needed.

## Tagging

- Format: `vMAJOR.MINOR.PATCH` (semver).
  - **MAJOR** — breaking change: existing consumer code could stop working, or behave differently, after upgrading without any changes on their end (removed/renamed public function, changed signature, changed default behavior, changed output format, raised minimum dependency version, etc.).
  - **MINOR** — new backward-compatible feature.
  - **PATCH** — bug fix only, backward-compatible.
  - Bumping a more-significant number resets everything to its right to `0` (e.g. `v1.4.7` + breaking change → `v2.0.0`, not `v2.4.7`).
- Tags are **annotated** (`git tag -a`) and **immutable** — once pushed, a tag's commit never changes. A bug found in a released version is fixed by cutting the *next* patch tag, never by moving the existing one.
- New major/minor/patch tags for the current line are cut from `main`. Patch tags for old lines are cut from their `release/x.x` branch.
- Each tag gets a corresponding GitHub Release, with notes auto-generated from merged PR titles (categorized via `.github/release.yml` based on PR labels — label PRs with `enhancement`, `bug`, `documentation`, or `breaking-change` so they group correctly, and so the right version number gets bumped).

## How consumers use this

- **Want a frozen, stable version, no surprises:** checkout a specific tag — `git checkout v1.2.0` — or download that version's Release asset. This version never changes underneath them.
- **Want to stay current within an old supported line and auto-receive its bug fixes:** track the corresponding `release/x.x` branch instead of a tag — `git pull origin release/1.2.x` — and get every patch merged into that line without re-pinning each time.
- **Just want the newest stable release, no version number to think about:** use the "Latest" release — GitHub always flags the most recent non-prerelease release as "Latest" on the Releases page. It's also a stable, bookmarkable URL: `https://github.com/<org>/<repo>/releases/latest` (or `GET /repos/<org>/<repo>/releases/latest` via the API) — always resolves to whatever's currently newest, without needing to know the version number.
- **Want the absolute bleeding edge:** track `main` directly. This is *not* the same as "latest stable" — `main` can include unreleased, in-progress, or untagged work. Only for people who specifically want that.

## How bug fixes get branched

1. Bug reported against an old, already-superseded version (e.g. `v1.2.0`, while `main` is on `v2.x`).
2. If `release/1.2.x` doesn't already exist, cut it from the `v1.2.0` tag: `git checkout -b release/1.2.x v1.2.0`.
3. Cut a normal fix branch from there: `git checkout -b fix/crash-on-startup release/1.2.x`.
4. PR the fix branch into `release/1.2.x` (not `main`).
5. Once merged, tag the new commit as the next patch: `v1.2.1`.
6. If the bug also exists on `main`/the current line, cherry-pick the same fix commit forward (or use a backport bot / label-driven automation, if set up) so it isn't re-written from scratch.
