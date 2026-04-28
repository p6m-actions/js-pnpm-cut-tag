# JavaScript PNPM Cut Tag

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/js-pnpm-cut-tag?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action for cutting versioned tags for JavaScript projects using pnpm. This action will:

1. Bump the version in package.json based on the specified version level (patch, minor, or major)
2. Create a Git tag using the format "vMajor.Minor.Patch"
3. Push the new version commit and tag to the repository via `p6m-actions/p6m-release-action`, which produces a verified commit through the p6m GitHub App (compatible with branch rulesets that require signed commits or block direct pushes).

This action assumes that pnpm has been installed and the repository has been checked out with the necessary permissions.

## Usage

Add this action to your workflow after you've set up pnpm and checked out your code. Your workflow must grant `id-token: write` so the underlying release action can perform the OIDC token exchange that mints the GitHub App credential used to push the commit and tag.

```yaml
- name: Cut Tag
  uses: p6m-actions/js-pnpm-cut-tag@v2
  with:
    version-level: "patch" # Options: patch, minor, major
```

## Inputs

| Input           | Description                                    | Required | Default |
| --------------- | ---------------------------------------------- | -------- | ------- |
| `version-level` | Version level to bump (patch, minor, or major) | Yes      | `patch` |

## Outputs

| Output    | Description                  |
| --------- | ---------------------------- |
| `version` | The new version number       |
| `tag`     | The new tag that was created |

## Examples

### Example 1: Build and Release Workflow

This example shows how to use this action as part of a workflow that builds a project and then cuts a new tag:

```yaml
name: Build and Release

on:
  push:
    branches: [main]

permissions:
  contents: write # Required for checkout / fallback git operations
  id-token: write # Required for OIDC token exchange in p6m-release-action

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup pnpm
        uses: p6m-actions/js-pnpm-setup@v1
        with:
          node-version: "20"

      - name: Build
        uses: p6m-actions/js-pnpm-build@v1

      - name: Cut Tag
        uses: p6m-actions/js-pnpm-cut-tag@v2
        with:
          version-level: "patch"
```

### Example 2: Manual Release Workflow

This example shows a standalone workflow for manually releasing a new version:

```yaml
name: Release New Version

on:
  workflow_dispatch:
    inputs:
      version-level:
        description: "Version level to bump"
        required: true
        default: "patch"
        type: choice
        options:
          - patch
          - minor
          - major

permissions:
  contents: write # Required for checkout / fallback git operations
  id-token: write # Required for OIDC token exchange in p6m-release-action

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup pnpm
        uses: p6m-actions/js-pnpm-setup@v1
        with:
          node-version: "20"

      - name: Cut Tag
        uses: p6m-actions/js-pnpm-cut-tag@v2
        with:
          version-level: ${{ github.event.inputs.version-level }}

      - name: Output Version Info
        run: echo "Released version ${{ steps.cut-tag.outputs.version }} with tag ${{ steps.cut-tag.outputs.tag }}"
```

## Required Permissions

This action delegates the commit and tag push to `p6m-actions/p6m-release-action`, which uses an OIDC token exchange to mint a GitHub App credential. Your workflow must therefore grant `id-token: write`. `contents: write` is also recommended to cover `actions/checkout` and any fallback git operations.

```yaml
permissions:
  contents: write
  id-token: write
```
