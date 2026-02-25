# GitVersion Configuration Reference

This document explains every setting in [`GitVersion.yml`](./GitVersion.yml) and why it is configured the way it is for this project.

> **GitVersion version:** 6.x (configured for `gittools/actions/gitversion/setup@v3` with `versionSpec: '6.x'`)

---

## Table of Contents

1. [Global Settings](#1-global-settings)
2. [Commit Message Bump Rules](#2-commit-message-bump-rules)
3. [Branch Configuration](#3-branch-configuration)
   - [main](#31-main)
   - [development](#32-development)
   - [test](#33-test)
   - [hotfix](#34-hotfix)
   - [feature](#35-feature)
   - [pull-request](#36-pull-request)
4. [Ignore](#4-ignore)
5. [Merge Message Formats](#5-merge-message-formats)
6. [How Versions Flow Across Branches](#6-how-versions-flow-across-branches)
7. [GitVersion v5 → v6 Migration Notes](#7-gitversion-v5--v6-migration-notes)

---

## 1. Global Settings

These settings apply to the entire repository unless overridden at the branch level.

### `assembly-versioning-scheme`

```yaml
assembly-versioning-scheme: MajorMinorPatch
```

Controls which parts of the version are written into .NET assembly attributes (`AssemblyVersion`).

| Value | Format produced |
|-------|----------------|
| `MajorMinorPatch` | `1.2.3` |
| `MajorMinor` | `1.2.0` |
| `Major` | `1.0.0` |
| `None` | Not set |

**Why `MajorMinorPatch`:** Keeps the assembly version in sync with the full semantic version, making it easy to identify which version of the code is referenced by a compiled artifact.

---

### `assembly-file-versioning-scheme`

```yaml
assembly-file-versioning-scheme: MajorMinorPatch
```

Controls the `AssemblyFileVersion` attribute separately from `AssemblyVersion`. Uses the same value options as `assembly-versioning-scheme`.

**Why `MajorMinorPatch`:** Mirrors the assembly version for consistency. Both attributes will report the same human-readable version.

---

### `workflow`

```yaml
workflow: ContinuousDelivery
```

Defines the global versioning strategy.

> ⚠️ **v6 change:** This key was called `mode` in GitVersion v5. It is now `workflow` in v6. Using the old `mode` key causes GitVersion to output `undefined` and fail JSON parsing.

| Workflow | Behaviour |
|----------|-----------|
| `ContinuousDelivery` | The version is calculated from tags + commit messages. Each commit on a non-mainline branch gets a pre-release label and build metadata. |
| `ContinuousDeployment` | Similar to `ContinuousDelivery` but every commit increments the build number, suitable for pipelines that deploy every commit. |
| `Mainline` | Versions are inferred purely from the git graph without requiring tags. Every merge to `main` is treated as a release. |

**Why `ContinuousDelivery`:** Gives clean `1.2.3` versions on `main` while pre-release branches (e.g. `development`) automatically get labels like `1.2.3-alpha.4`. Tags remain the source of truth for released versions.

---

### `tag-prefix`

```yaml
tag-prefix: 'v'
```

The prefix GitVersion expects on Git tags when looking for version baselines. With `'v'`, a tag of `v1.2.3` represents version `1.2.3`.

**Why `'v'`:** The `v` prefix is the GitHub convention and is expected by most tooling (npm, Go modules, GitHub Releases UI).

---

### `label`

```yaml
label: ci
```

When no tag exists on the branch and GitVersion cannot compute a proper pre-release label, this value is used as the fallback label.

> ⚠️ **v6 change:** This key was called `continuous-delivery-fallback-tag` in GitVersion v5. It is now `label` in v6.

**Example output:** `0.1.0-ci.1`

**Why `ci`:** Makes it obvious that the artefact came from a CI pipeline and has no meaningful release status.

---

## 2. Commit Message Bump Rules

These regular expressions tell GitVersion how to interpret commit messages and decide what part of the version (Major, Minor, Patch) to increment. This implements the **Conventional Commits** specification.

### `commit-message-incrementing`

```yaml
commit-message-incrementing: Enabled
```

Activates parsing of commit messages against the bump-rule patterns below.

| Value | Behaviour |
|-------|-----------|
| `Enabled` | Parse every commit message and bump accordingly |
| `MergeMessageOnly` | Only parse merge commit messages |
| `Disabled` | Ignore commit messages; rely on tags alone |

**Why `Enabled`:** Lets developers control the version bump simply by writing a well-formed commit message — no manual tagging or workflow parameters needed.

---

### `major-version-bump-message`

```yaml
major-version-bump-message: "^(breaking|perf|style|major)(\\([a-z0-9\\-\\.]+\\))?!:|^BREAKING CHANGE:"
```

A regex matched against commit messages. Any matching commit triggers a **Major** bump (`1.x.x → 2.0.0`).

The pattern matches two forms of breaking change notation defined by the Conventional Commits spec:

| Form | Example |
|------|---------|
| `type!: description` | `breaking!: remove legacy auth API` |
| `BREAKING CHANGE:` in footer | `BREAKING CHANGE: renamed config property` |

---

### `minor-version-bump-message`

```yaml
minor-version-bump-message: "^(feat|fix|refactor|minor)(\\([a-z0-9\\-\\.]+\\))?:"
```

Any commit starting with `feat:`, `fix:`, `refactor:`, or `minor:` (with optional scope) triggers a **Minor** bump (`x.1.x → x.2.0`).

| Example commit | Bump |
|----------------|------|
| `feat: add dark mode` | `x.Y.0` Minor |
| `feat(auth): add OAuth2` | `x.Y.0` Minor |

---

### `patch-version-bump-message`

```yaml
patch-version-bump-message: "^(patch|perf|style|docs|chore)(\\([a-z0-9\\-\\.]+\\))?:"
```

Commits starting with `patch:`, `perf:`, `style:`, `docs:`, or `chore:` trigger a **Patch** bump (`x.x.1 → x.x.2`).

| Example commit | Bump |
|----------------|------|
| `patch: resolve null pointer on startup` | `x.x.Z` Patch |
| `perf: cache database queries` | `x.x.Z` Patch |

---

### `no-bump-message`

```yaml
no-bump-message: "^(chore|docs|style)(\\([a-z0-9\\-\\.]+\\))?:"
```

> **Note:** This overlaps with `patch-version-bump-message`. In practice, `commit-message-incrementing` precedence means the first matching rule wins. If you want `chore:`, `docs:`, and `style:` commits to produce **no** version bump at all, this setting is evaluated as a "skip bump" override when the patterns conflict.

---

## 3. Branch Configuration

Each branch key can override any of the global settings. The keys are matched to actual branch names using the `regex` field.

### Common per-branch settings

| Setting | Description |
|---------|-------------|
| `regex` | Pattern matched against the full branch name |
| `workflow` | Versioning workflow for this branch (**v6**: was `mode` in v5) |
| `label` | Pre-release label appended after the version (empty string = no label) (**v6**: was `tag` in v5) |
| `increment` | Default bump type if no commit message rule applies |
| `prevent-increment-of-merged-branch-version` | If `true`, the version from the source branch is preserved rather than bumped again after merge |
| `track-merge-target` | If `true`, this branch tracks the version of its merge target |
| `is-release-branch` | Marks the branch as a release branch |
| `is-mainline` | Marks the branch as the canonical production line |
| `source-branches` | Declares which branches are valid parents of this branch |

> ⚠️ **v6 change:** The per-branch `tracks-release-branches` key was **removed** in GitVersion v6. Remove it from any branch config to avoid parse errors.

---

### 3.1 `main`

```yaml
main:
  regex: ^main$
  workflow: ContinuousDelivery
  label: ''
  increment: Patch
  prevent-increment-of-merged-branch-version: true
  track-merge-target: false
  is-release-branch: true
  is-mainline: true
```

| Setting | Value | Why |
|---------|-------|-----|
| `label` | `''` (empty) | Produces clean versions like `1.2.3` with no pre-release suffix — this is the production release branch |
| `prevent-increment-of-merged-branch-version` | `true` | When a `development` branch at `1.1.0-alpha.3` merges in, `main` adopts `1.1.0` without bumping again |
| `is-mainline` | `true` | Tells GitVersion this is the canonical history line; all version baselines ultimately trace back here |
| `is-release-branch` | `true` | Enables tracking of release tags on this branch |

---

### 3.2 `development`

```yaml
development:
  regex: ^development$
  workflow: ContinuousDelivery
  label: alpha
  increment: Minor
  prevent-increment-of-merged-branch-version: false
  track-merge-target: true
  is-release-branch: false
  source-branches:
    - main
```

| Setting | Value | Why |
|---------|-------|-----|
| `label` | `alpha` | Version looks like `1.2.0-alpha.5` — clearly pre-production |
| `increment` | `Minor` | New features are integrated here, so the default bump is Minor |
| `track-merge-target` | `true` | Stays in sync with `main`'s version when `main` is merged back in (e.g. after a hotfix) |
| `source-branches` | `[main]` | `development` is always branched from or merges originate from `main` |

---

### 3.3 `test`

```yaml
test:
  regex: ^test$
  workflow: ContinuousDelivery
  label: beta
  increment: Patch
  prevent-increment-of-merged-branch-version: false
  track-merge-target: false
  is-release-branch: false
  source-branches:
    - development
```

| Setting | Value | Why |
|---------|-------|-----|
| `label` | `beta` | Version looks like `1.2.0-beta.2` — code is in QA/staging |
| `increment` | `Patch` | Only stabilisation/bug fixes are expected here, so Patch is appropriate |
| `source-branches` | `[development]` | `test` always receives code promoted from `development` |

---

### 3.4 `hotfix`

```yaml
hotfix:
  regex: ^hotfix[/-]
  workflow: ContinuousDelivery
  label: hotfix
  increment: Patch
  prevent-increment-of-merged-branch-version: false
  track-merge-target: false
  is-release-branch: false
  source-branches:
    - main
```

| Setting | Value | Why |
|---------|-------|-----|
| `regex` | `^hotfix[/-]` | Matches `hotfix/login-crash` or `hotfix-login-crash` |
| `label` | `hotfix` | Version looks like `1.2.4-hotfix.1` while the fix is being developed |
| `increment` | `Patch` | Hotfixes should only ever increment the Patch component |
| `source-branches` | `[main]` | Hotfixes branch directly from `main` and merge back to `main` (and optionally `development`) |

---

### 3.5 `feature`

```yaml
feature:
  regex: ^feature[/-]
  workflow: ContinuousDelivery
  label: feature
  increment: Minor
  prevent-increment-of-merged-branch-version: false
  track-merge-target: false
  is-release-branch: false
  source-branches:
    - development
```

| Setting | Value | Why |
|---------|-------|-----|
| `regex` | `^feature[/-]` | Matches `feature/user-auth` or `feature-user-auth` |
| `label` | `feature` | Version looks like `1.3.0-feature.7` |
| `increment` | `Minor` | Features are new functionality — the default bump aligns with Minor |
| `source-branches` | `[development]` | All feature work originates from and merges back into `development` |

---

### 3.6 `pull-request`

```yaml
pull-request:
  regex: ^(pull|pull\-requests|pr)[/-]
  workflow: ContinuousDelivery
  label: PullRequest
  increment: Inherit
  track-merge-target: false
  is-release-branch: false
  source-branches:
    - development
    - main
    - hotfix
    - feature
```

| Setting | Value | Why |
|---------|-------|-----|
| `regex` | Matches GitHub's internal PR ref format (`refs/pull/123/merge`) | Allows GitVersion to version the temporary merge commit that GitHub creates for PRs |
| `label` | `PullRequest` | Version looks like `1.2.0-PullRequest.123` |
| `increment` | `Inherit` | Inherits the increment from the source branch so the PR version is consistent with what it will produce after merging |

---

## 4. Ignore

```yaml
ignore:
  sha: []
```

A list of specific commit SHAs to exclude from version calculation. Useful when a commit introduced incorrect version data (e.g. a wrong tag was accidentally created and then deleted but the commit remains).

**Current value:** Empty — no commits are excluded.

---

## 5. Merge Message Formats

```yaml
merge-message-formats: {}
```

Custom regex patterns to parse merge commit messages for version information. GitVersion has a set of built-in patterns for GitHub, GitLab, and Azure DevOps merge messages.

**Current value:** Empty object — uses GitVersion's built-in merge message detection, which covers the standard GitHub PR merge message format (`Merge pull request #123 from branch`).

---

## 6. How Versions Flow Across Branches

```
main          v1.0.0 ──────────────────────────────────> v1.1.0 (Release)
                │                                              ↑
development     └──> 1.1.0-alpha.1 → 1.1.0-alpha.2 ──────────┘
                          │
test                      └──> 1.1.0-beta.1 → 1.1.0-beta.2
                     │
feature/login        └──> 1.1.0-feature.1 → 1.1.0-feature.3
```

- `feature/*` branches off `development` and produces `feature` pre-release builds.
- When merged to `development`, the alpha counter advances.
- When `development` is promoted to `test`, it becomes a `beta` build.
- When a PR from `development` (or `test`) is merged to `main`, the release workflow fires, calculates `v1.1.0`, creates the tag, and publishes the GitHub Release.
- `hotfix/*` branches directly off `main`, produces `hotfix` builds, and merges back to `main` for an immediate Patch release.

---

## 7. GitVersion v5 → v6 Migration Notes

This config was updated to be compatible with **GitVersion 6.x** (`gittools/actions/gitversion/setup@v3`, `versionSpec: '6.x'`). Using v5 keys with a v6 tool causes the tool to output `undefined` instead of JSON, resulting in the error:

```
Error: GitVersion output is not valid JSON
dotnet-gitversion Output:
-------------------
undefined
-------------------
```

### Key renames

| v5 key (old) | v6 key (new) | Scope |
|---|---|---|
| `mode` | `workflow` | Global & per-branch |
| `continuous-delivery-fallback-tag` | `label` | Global |
| `tag` | `label` | Per-branch |
| `tracks-release-branches` | **Removed** | Per-branch |
