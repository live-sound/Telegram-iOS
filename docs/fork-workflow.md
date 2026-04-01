# Fork Workflow

This repository is maintained as a public fork of `TelegramMessenger/Telegram-iOS`.

The goals of this workflow are:

- keep a clean path for pulling changes from Telegram upstream
- keep product-specific changes isolated from upstream intake
- avoid fragile GitHub Desktop sync behavior around submodules

## Repository Layout

Remotes:

- `origin` -> `https://github.com/live-sound/Telegram-iOS.git`
- `upstream` -> `https://github.com/TelegramMessenger/Telegram-iOS.git`

Branches:

- `master`: upstream intake branch
- `main`: product branch for the custom client
- `feature/*`: short-lived branches created from `main`

Rules:

- Do not put app-specific changes on `master`.
- Keep `master` close to `upstream/master`.
- Put branding, feature work, and product behavior changes on `main`.
- Create feature branches from `main` and merge them back into `main`.

## Why `master` And `main` Both Exist

`master` is used to absorb Telegram updates with minimal fork-specific maintenance.

`main` is the branch that should become the default branch in GitHub and should contain the actual custom client work.

This split makes upstream syncs safer:

1. update `master` from Telegram
2. review the result
3. merge `master` into `main`

## One-Time Setup

Verify remotes:

```bash
git remote -v
```

Recommended GitHub settings:

- set `main` as the default branch
- protect `main`
- protect `master`
- disable force-push on both long-lived branches

Recommended local Git setting for this repo:

```bash
git config fetch.recurseSubmodules false
```

This avoids noisy recursive fetch failures when upstream history references old submodule commits that are no longer fetchable from all remotes.

## Submodule Policy

This fork must not use relative submodule URLs for fork-specific dependencies that are expected to resolve under the parent repository owner.

The fork currently keeps these submodules as absolute URLs:

- `submodules/rlottie/rlottie`
- `submodules/TgVoipWebrtc/tgcalls`

Use this rule:

- if a submodule is consumed without local changes, point it at the original upstream repository
- if a submodule needs local patches, fork that submodule and update `.gitmodules` to point to the fork

After any `.gitmodules` change:

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

## Daily Development

Start new work from `main`:

```bash
git switch main
git pull --ff-only origin main
git switch -c feature/short-description
```

Work normally, then merge the feature branch back into `main`.

Do not branch from `master` for product work.

## Upstream Sync Procedure

Use the command line for upstream syncs. GitHub Desktop is acceptable for browsing history, staging changes, and making commits, but it is not the recommended tool for syncing this repository because submodule recursion can produce misleading fetch failures.

### 1. Update The Upstream Intake Branch

```bash
git switch master
git fetch upstream --prune --tags --no-recurse-submodules
git merge upstream/master
git push origin master
```

If the merge reports conflicts, resolve them on `master` first.

### 2. Update Submodules

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

If a submodule fails because the configured remote does not contain the required commit:

- inspect the affected submodule URL in `.gitmodules`
- confirm whether it should point to Telegram upstream or to a fork
- fix `.gitmodules` if necessary

### 3. Bring Upstream Changes Into The Product Branch

```bash
git switch main
git merge master
git push origin main
```

If the repository is maintained by multiple people, prefer a pull request from `master` into `main` instead of merging directly.

## Handling Conflicts

When `master` conflicts with `upstream/master`:

- prefer upstream for Telegram source changes unless the fork intentionally maintains a divergence
- keep fork-safe submodule URL fixes in `.gitmodules`
- keep `master` focused on maintenance-only changes

When `main` conflicts with `master`:

- preserve intentional product changes from `main`
- pull in upstream structural changes from `master`
- expect conflicts around branding, configuration, build wiring, and app entry points

## GitHub UI Guidance

Do not use GitHub's `Sync fork` button on `main`.

If GitHub UI sync is used at all, use it only for `master`, and only when you understand exactly what will be merged from `upstream/master` into the fork's `master`.

The preferred sync path remains local CLI commands because they make submodule handling explicit.

## Suggested Release Flow

For each upstream intake:

1. merge `upstream/master` into `master`
2. update submodules
3. build and smoke-test locally
4. merge `master` into `main`
5. continue feature development from `main`

Tagging sync points is recommended:

```bash
git tag telegram-sync-YYYY-MM-DD
git push origin telegram-sync-YYYY-MM-DD
```

## Quick Reference

Fetch upstream without recursive submodule noise:

```bash
git fetch upstream --prune --tags --no-recurse-submodules
```

Sync submodules explicitly:

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

Create a feature branch:

```bash
git switch main
git switch -c feature/short-description
```
