<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 📦 Publish Python Project to PyPI

Publishes a Python project to PyPI, the Python Package Index.

## pypi-publish-action

## Usage Example

<!-- markdownlint-disable MD046 -->

```yaml
steps:
  - name: 'Publish to PyPI'
    uses: lfreleng-actions/pypi-publish-action@main
    with:
      environment: 'development'
      attestations: true
```

<!-- markdownlint-enable MD046 -->

## Inputs

<!-- markdownlint-disable MD013 -->

| Name                     | Required | Description                                              |
| ------------------------ | -------- | -------------------------------------------------------- |
| environment              | False    | Mandatory environment, e.g. development, production      |
| tag                      | False    | Tag for this build/release                               |
| artefact_path            | False    | Path/location of build artefacts                         |
| one_password_item        | False    | 1Password vault credential for PyPI publishing           |
| op_service_account_token | False    | 1Password service account credential to access vault     |
| pypi_credential          | False    | PyPI API credential from GitHub secrets                  |
| publish_disable          | False    | Disables the final publishing step that uploads packages |
| attesations              | False    | Enables GitHub support for artefact attestations         |
| no_checkout              | False    | Don't perform a checkout of the local repository         |

<!-- markdownlint-enable MD013 -->

## Implementation Details

Uses the upstream actions:

<!-- markdownlint-disable MD013 -->

- [1password/load-secrets-action](https://github.com/1password/load-secrets-action)
- [modeseven-lfreleng-actions/gh-action-pypi-publish](https://github.com/modeseven-lfreleng-actions/gh-action-pypi-publish)

The second action above is a modified/forked version of another action:

- [pypa/gh-action-pypi-publish](https://github.com/pypa/gh-action-pypi-publish)

<!-- markdownlint-enable MD013 -->

## Authentication Methods

Publishes using three different authentication methods.

In order of preference:

- [Trusted Publishing](https://docs.pypi.org/trusted-publishers/) (Uses an
OIDC token)
- Static credential retrieved from 1Password vault using a service account
- A static credential from GitHub secrets

Note: the first/initial publishing step cannot leverage Trusted Publishing

The first time a given repository gets published to PyPI, a static API key
is necessary. After this, setup trusted publishing for the project in the
PyPI web portal.
