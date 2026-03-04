# CXL Test Runner

CXL Test Runner is a GitHub Actions workflow that allows Linux kernel
developers to validate CXL changes using the ndctl CXL test suite.

The workflow builds a kernel, boots it in QEMU using run_qemu.sh, and
executes the CXL unit tests from ndctl.

The goal is to provide a simple automated testing environment that runs
entirely on GitHub-hosted infrastructure. No special hardware is required.


## Quick Start

1. Fork this repository.

2. Open the Actions tab in your fork.

3. Select "CXL test runner".

4. Click "Run workflow".

5. Provide your kernel repository and branch.

Example inputs:

```
kernel_repo: yourname/linux
kernel_branch: cxl-feature-branch
ndctl_repo: pmem/ndctl
ndctl_branch: pending
timeout_min: 35
```

The workflow will:

1. Build the specified kernel
2. Build ndctl
3. Boot the kernel in QEMU
4. Run the CXL unit tests
5. Display results in the GitHub Actions interface


## Example Test Output

The workflow summary displays the results of each CXL test.

Example:

```
1/13 ndctl:cxl / cxl-topology.sh        OK
2/13 ndctl:cxl / cxl-region-sysfs.sh    OK
3/13 ndctl:cxl / cxl-labels.sh          OK
4/13 ndctl:cxl / cxl-create-region.sh   OK
5/13 ndctl:cxl / cxl-xor-region.sh      OK
...
13/13 ndctl:cxl / cxl-poison.sh         OK

Ok:                 13
Fail:               0
Skipped:            0
```
Detailed logs are also uploaded as workflow artifacts.


## Automatically Trigger Tests From Your Kernel Repository

Developers may configure their kernel repository to automatically trigger
the CXL test runner whenever commits are pushed.

Create the file:

.github/workflows/cxl-test.yml

Example workflow:

```
name: run CXL tests

on:
  push:
    branches:
      - cxl-*

jobs:
  trigger:
    runs-on: ubuntu-latest

    steps:
      - name: Trigger CXL test runner
        env:
          GH_TOKEN: ${ { secrets.GITHUB_TOKEN } }

        run: |
          gh workflow run "CXL test runner" \
            --repo yourname/cxl-test-runner \
            -f kernel_repo=${ { github.repository } } \
            -f kernel_branch=${ { github.ref_name } }
```

## Overview

This project builds on the run_qemu.sh testing environment and packages
it into a reproducible GitHub Actions workflow.

The runner performs the following steps:

1. Checkout the requested kernel repository and branch
2. Checkout the requested ndctl repository and branch
3. Build the kernel
4. Boot the kernel using run_qemu.sh
5. Execute the CXL unit tests
6. Publish logs and test summaries


## Test Environment

The workflow currently runs with the following configuration:

```
GitHub runner: ubuntu-24.04
Architecture:  x86_64
mkosi image:   ubuntu noble
```

## Relationship to run_qemu.sh

https://github.com/pmem/run_qemu

This project builds on the run_qemu.sh infrastructure originally
developed by Vishal Verma and expanded upon (including github workflows)
by Marc Herbert.

run_qemu.sh provides a flexible environment for running CXL and NVDIMM
tests in QEMU.

While cxl-test-runner focuses on automated GitHub testing, developers
who want full control over the environment can run run_qemu.sh locally to:

- debug failures interactively
- experiment with different CXL topologies
- develop new tests
- explore advanced QEMU configurations
