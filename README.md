# Neutrons Conda Actions for GitHub

## Overview

This repository contains GitHub actions for various conda-related tasks, such as verifying a conda package, and uploading a conda package to Anaconda Cloud.

These actions primarily assumes that you have built a `<package-name>.conda`,  
and that it is located in a conda-style channel directory (see [conda-index](https://github.com/conda/conda-index)).

Available actions:

- [pkg-verify](#pkg-verify): Verify a conda package by installing it with `micromamba` and ensuring it is importable by Python, and that the version reported by conda and python match.
- [pkg-remove](#pkg-remove): Clean up old conda packages from Anaconda Cloud.
- [publish](#publish): Publish a conda package to Anaconda Cloud.

## pkg-verify

GitHub action to verify a conda package by installing it with `micromamba` and ensuring it is importable by Python, and that the version reported by conda and python match.

#### Usage

Full list of available inputs in [`pkg-verify/action.yaml`](#pkg-verify/action.yaml).

Inputs:

| Input            | Description                                                                       | Required | Default |
| ---------------- | --------------------------------------------------------------------------------- | -------- | ------- |
| `local-channel`  | Path to the local conda channel containing the package to verify                  | No       | -       |
| `package-name`   | Name of the conda package                                                         | Yes      | -       |
| `module-name`    | Name of the Python module to import (if different from package name)              | No       | -       |
| `python-version` | Python version to use for testing (e.g., `3.10`)                                  | No       | `3.10`  |
| `extra-channels` | Additional conda channels to use for dependencies (comma-separated)               | No       | -       |
| `extra-commands` | Additional shell commands to run after installing the package (newline-separated) | No       | -       |

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
  pkg-verify:
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
        uses: neutrons/conda-actions/pkg-verify@main
        with:
          local-channel: /tmp/local-channel
          package-name: ${{ env.PKG_NAME }}
          extra-channels: mantid neutrons pyoncat
```

## pkg-remove

GitHub action to remove old packages of a specific label from [anaconda.org](https://anaconda.org),
keeping the N most recent versions.

#### Usage

Full list of available inputs in [`pkg-remove/action.yaml`](#pkg-remove/action.yaml).

Inputs:

| Input            | Description                                                           | Required | Default |
| ---------------- | --------------------------------------------------------------------- | -------- | ------- |
| `anaconda_token` | Anaconda.org API token                                                | Yes      | -       |
| `organization`   | Anaconda.org organization or user name                                | Yes      | -       |
| `package_name`   | Name of the conda package to clean up                                 | Yes      | -       |
| `label`          | Label to target for cleanup (e.g., `dev`, `nightly`, `rc`)            | No       | -       |
| `keep`           | Number of most recent package versions to keep                        | No       | `5`     |
| `dry_run`        | If `true`, only print what would be deleted without actually deleting | No       | `false` |

Outputs:

| Output        |                                       |
| ------------- | ------------------------------------- |
| `num_removed` | Number of files that would be deleted |

Example:

```yaml
jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Remove old dev packages
        uses: neutrons/conda-actions/pkg-remove@main
        with:
          anaconda_token: ${{ secrets.ANACONDA_TOKEN }}
          organization: neutrons
          package_name: my-package
          label: dev
          keep: 5
```

## publish

GitHub action to publish a conda package to Anaconda Cloud.

This action assumes that:

- The package has already been built and is available in a local conda channel directory
- Either `anaconda-client` or `pixi` is installed in the environment where the action is running

#### Usage

Full list of available inputs in [`publish/action.yaml`](#publish/action.yaml).

Inputs:

| Input            | Description                                                  | Required | Default    |
| ---------------- | ------------------------------------------------------------ | -------- | ---------- |
| `anaconda-token` | Anaconda.org API token                                       | Yes      | -          |
| `organization`   | Anaconda.org organization or user name                       | Yes      | -          |
| `package-path`   | Path to the conda package to publish                         | Yes      | -          |
| `github-ref`     | GitHub ref (e.g., `refs/tags/v1.0.0`) to determine the label | No       | github.ref |
| `label`          | Label to apply to the package (e.g., `dev`, `nightly`, `rc`) | No       | -          |
| `force`          | If `true`, overwrite existing package with the same version  | No       | `false`    |

Example:

```yaml
jobs:
  publish:
    - uses: actions/checkout@main

    - uses: prefix-dev/setup-pixi@main

    - name: Build package
      run: |
        # steps to build your .conda package, for example:
        pixi build

    - name: Publish package to Anaconda Cloud
      uses: neutrons/conda-actions/publish@main
      with:
        anaconda-token: ${{ secrets.ANACONDA_TOKEN }}
        organization: neutrons
        package-path: my-package-*.conda
```
