# Project Name

> Replace this with a short description of your project.

## Branching Strategy

| Branch | Purpose |
|--------|---------|
| `main` | Production-ready code. Merges here trigger a release. |
| `development` | Active development integration branch. |
| `test` | QA / staging environment. |
| `hotfix/*` | Emergency fixes branched from `main`. |
| `feature/*` | Individual feature development, branched from `development`. |

## Release Process

Releases are **fully automated** via GitHub Actions:

1. Merge a PR into `main`.
2. The **Release** workflow calculates the next semantic version using [GitVersion](https://gitversion.net) (Conventional Commits mode).
3. A Git tag (`vX.Y.Z`) is created and pushed.
4. A **GitHub Release** is published with commit messages since the previous tag as the release notes.

### Version Bumping (Conventional Commits)

| Commit prefix | Version bump |
|---------------|-------------|
| `feat:` | Minor |
| `fix:`, `perf:`, `refactor:` | Patch |
| `feat!:` or `BREAKING CHANGE:` | Major |
| `chore:`, `docs:`, `style:` | No bump |

## Getting Started

```bash
# Clone the repo
git clone https://github.com/<org>/<repo>.git

# Create a feature branch
git checkout development
git checkout -b feature/my-feature
```
