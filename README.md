# JavaScript PNPM Cut Tag

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/js-pnpm-cut-tag?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action for cutting versioned tags for JavaScript projects using pnpm. This action will:

1. Bump the version in package.json based on the specified version level (patch, minor, or major)
2. Create a Git tag using the format "vMajor.Minor.Patch"
3. Push the new version and tag to the repository

This action assumes that pnpm has been installed and the repository has been checked out with the necessary permissions.

## Usage

Add this action to your workflow after you've set up pnpm and checked out your code. Make sure your workflow has write permissions to push tags and commits to your repository.

```yaml
- name: Cut Tag
  uses: p6m-actions/js-pnpm-cut-tag@v1
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
  contents: write # Required for pushing tags

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
        uses: p6m-actions/js-pnpm-cut-tag@v1
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
  contents: write # Required for pushing tags

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
        uses: p6m-actions/js-pnpm-cut-tag@v1
        with:
          version-level: ${{ github.event.inputs.version-level }}

      - name: Output Version Info
        run: echo "Released version ${{ steps.cut-tag.outputs.version }} with tag ${{ steps.cut-tag.outputs.tag }}"
```

## Required Permissions

This action requires write permissions to the repository contents to push commits and tags. Make sure to include the following permissions in your workflow:

```yaml
permissions:
  contents: write
```
