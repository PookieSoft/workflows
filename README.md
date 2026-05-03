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

### Reusable workflows

| Workflow | Purpose | Required inputs |
|---|---|---|
| [`security-scan.yml`](.github/workflows/security-scan.yml) | Trivy scan of repo filesystem (always) and Docker image (main only) | `docker-image-name` |
| [`docker-vuln-pr.yml`](.github/workflows/docker-vuln-pr.yml) | On main: snapshot CVE baseline. On schedule: open a PR if new CVEs appeared | `docker-image-name` |
| [`pr-ci.yml`](.github/workflows/pr-ci.yml) | Human-PR CI: tests, coverage comment, optional Docker build/smoke/SSH-deploy | `runtime` |
| [`dependabot-pr-ci.yml`](.github/workflows/dependabot-pr-ci.yml) | Single integrated Dependabot flow: enforce patch label → sync lockfile → tests → smoke | `runtime` |
| [`release.yml`](.github/workflows/release.yml) | Push-to-main release: tests, version bump from PR labels, Docker push, optional npm publish, GitHub Release | `runtime` |

### Composite actions

| Action | Purpose | Inputs |
|---|---|---|
| [`coverage-comment`](.github/actions/coverage-comment/action.yml) | Build a markdown coverage report from `coverage/coverage-summary.json` and emit it as a step output | none |
| [`setup-runtime`](.github/actions/setup-runtime/action.yml) | Install and configure either bun or node based on a runtime input | `runtime` |
| [`install-deps`](.github/actions/install-deps/action.yml) | Install project dependencies via `bun install --frozen-lockfile` or `npm ci` | `runtime` |

## How callers use it

Each consumer repo holds a thin caller workflow. Example:

```yaml
# .github/workflows/pr-ci.yml in a consumer repo
name: PR CI

on:
    pull_request:
        types: [opened, synchronize]
        branches: [main]

permissions:
    pull-requests: write

jobs:
    ci:
        uses: PookieSoft/workflows/.github/workflows/pr-ci.yml@v1
        with:
            runtime: bun
            build-docker: true
            smoke-test: true
            ssh-deploy: true
            docker-image-name: bongbot-develop
            service-name-prefix: bongbot-develop
            environment: Dev
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

This repo is **public** so reusable workflows can be called from any PookieSoft repo. Sharing private reusable workflows across repos requires a paid GitHub plan (Team / Enterprise); the org is on Free, so making the workflows themselves public is the standard pattern. The workflows contain no secrets — those flow via `secrets: inherit` from each consumer repo.
