# Neutrons Conda Actions for GitHub

## Overview

This repository contains GitHub actions for various conda-related tasks, such as verifying a conda package, and uploading a conda package to Anaconda Cloud.

These actions primarily assumes that you have built a `<package-name>.conda`,  
and that it is located in a conda-style channel directory (see [conda-index](https://github.com/conda/conda-index)).

Available actions:

- [conda-verify](#conda-verify): Verify a conda package by installing it with `micromamba` and ensuring it is importable by Python, and that the version reported by conda and python match.
- [conda-publish](#conda-publish): Publish a conda package to Anaconda Cloud.
- [conda-cleanup](#conda-cleanup): Clean up old conda packages from Anaconda Cloud.

## conda-verify

GitHub action to verify a conda package by installing it with `micromamba` and ensuring it is importable by Python, and that the version reported by conda and python match.

#### Usage

Available inputs are listed in [`conda-verify/action.yaml`](#conda-verify/action.yaml).

Example usage in a GitHub workflow:

```yaml
jobs:
  # First, build your conda package and upload it as an artifact:
  build:
    steps:
      - name: Build conda package
        run: |
          # steps to build your .conda package

      - name: Upload conda package as artifact
        uses: actions/upload-artifact@main
        with:
          name: artifact-conda-package
          path: ${{ env.PKG_NAME }}-*.conda

  # Then, to verify the conda package:
  conda-verify:
    needs: build
    run-on: ubuntu-latest
    defaults:
      run:
        shell: bash -el {0}
    steps:
      - name: Download conda package artifact
        uses: actions/download-artifact@main
        with:
          name: artifact-conda-package
          path: /tmp/local-channel/linux-64

      - name: Verify Conda Package
        uses: neutrons/conda-actions/conda-verify@main
        with:
          local-channel: /tmp/local-channel
          package-name: ${{ env.PKG_NAME }}
          extra-channels: mantid neutrons pyoncat
```

## conda-publish

GitHub action to publish a conda package to Anaconda Cloud.

#### Usage

## conda-cleanup

GitHub action to remove old packages of a specific label from [anaconda.org](https://anaconda.org),
keeping the N most recent versions.

## Usage

Available inputs are listed in [`conda-cleanup/action.yaml`](#conda-cleanup/action.yaml).

```yaml
jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Remove old dev packages
        uses: neutrons/conda-actions/conda-cleanup@main
        with:
          anaconda_token: ${{ secrets.ANACONDA_TOKEN }}
          organization: neutrons
          package_name: my-package
          label: dev
          keep: 5
```

## Inputs

| Input            | Description                                                           | Required | Default |
| ---------------- | --------------------------------------------------------------------- | -------- | ------- |
| `anaconda_token` | Anaconda.org API token                                                | Yes      | -       |
| `organization`   | Anaconda.org organization or user name                                | Yes      | -       |
| `package_name`   | Name of the conda package to clean up                                 | Yes      | -       |
| `label`          | Label to target for cleanup (e.g., `dev`, `nightly`, `rc`)            | Yes      | -       |
| `keep`           | Number of most recent package versions to keep                        | No       | `5`     |
| `dry_run`        | If `true`, only print what would be deleted without actually deleting | No       | `false` |

## Outputs

| Output        |                                       |
| ------------- | ------------------------------------- |
| `num_removed` | Number of files that would be deleted |
