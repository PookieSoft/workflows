# PookieSoft Workflows

Centralised reusable GitHub Actions workflows and composite actions for the PookieSoft organisation.

## Versioning

Pin callers to a major-version tag (`@v1`, `@v2`), which rolls forward to the latest non-breaking release in that line. Breaking changes get a new major version.

```yaml
# Recommended
uses: PookieSoft/workflows/.github/workflows/<workflow>.yml@v1

# Avoid — unpinned, breaks all consumers on any push to main
uses: PookieSoft/workflows/.github/workflows/<workflow>.yml@main

# Strictest — pin to a commit SHA
uses: PookieSoft/workflows/.github/workflows/<workflow>.yml@<sha>
```

## Available reusable workflows

| Workflow | Purpose |
|---|---|
| [`dependabot-auto-label.yml`](.github/workflows/dependabot-auto-label.yml) | Strip `major`/`minor` from Dependabot PRs and ensure only `patch` is set |

Planned additions: `pr-ci.yml`, `dependabot-pr-ci.yml`, `release.yml`, `security-scan.yml`.

## How callers use it

Each consumer repo holds a thin caller workflow. Example:

```yaml
# .github/workflows/dependabot-auto-label.yml in a consumer repo
name: Dependabot auto-label

on:
    pull_request_target:
        types: [opened, reopened, labeled]

permissions:
    pull-requests: write

jobs:
    auto-label:
        uses: PookieSoft/workflows/.github/workflows/dependabot-auto-label.yml@v1
        secrets: inherit
```

## Repository layout

```
.github/
  workflows/   # reusable workflows (callable via `uses:`)
  actions/     # composite actions (callable via `uses: PookieSoft/workflows/.github/actions/<name>@v1`)
```

## Releasing

1. Land changes via PR.
2. Tag the new release: `git tag -a v1.0.1 -m "..."` then `git push origin v1.0.1`.
3. Move the rolling major tag: `git tag -f v1 v1.0.1 && git push origin v1 --force`.
4. For breaking changes, bump to `v2.0.0` and publish migration notes in the release.

## Access

This repo is private to PookieSoft. Org access is enabled so any repo in the org can `uses:` workflows from here.
