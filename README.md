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

### Multi-Platform Builds

For projects with platform-specific wheels (x64, ARM64, Python 3.10+),
use the `artefact_pattern` input to download all artefacts:

<!-- markdownlint-disable MD046 -->

```yaml
steps:
  - name: 'Publish to PyPI'
    uses: lfreleng-actions/pypi-publish-action@main
    with:
      environment: 'production'
      tag: 'v1.0.0'
      artefact_pattern: 'python-package-*'
      attestations: true
```

<!-- markdownlint-enable MD046 -->

This downloads all artefacts matching the pattern and merges them into the
dist directory for publishing.

## Inputs

<!-- markdownlint-disable MD013 -->

| Name                     | Required | Description                                                |
| ------------------------ | -------- | ---------------------------------------------------------- |
| path_prefix              | False    | Directory location containing project code (default: '.')  |
| environment              | False    | Mandatory environment, e.g. development, production        |
| tag                      | True     | Tag for this build/release                                 |
| artefact_path            | False    | Path/location of build artefacts                           |
| artefact_pattern         | False    | Glob pattern to download and merge build artefacts         |
| one_password_item        | False    | 1Password vault credential for PyPI publishing             |
| op_service_account_token | False    | 1Password service account credential to access vault       |
| pypi_credential          | False    | PyPI API credential from GitHub secrets                    |
| trusted_publishing       | False    | Controls trusted publishing behaviour (auto/true/false)    |
| publish_disable          | False    | Disables the final publishing step that uploads packages   |
| attestations             | False    | Enables GitHub support for artefact attestations           |
| no_checkout              | False    | Do not checkout local repository; used for testing         |

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

### Trusted Publishing Input

When `trusted_publishing` is `auto` or `true`, the calling workflow job
**must** grant `id-token: write` permission for the OIDC token exchange to
succeed, and `contents: read` so the default `actions/checkout` step can run:

<!-- markdownlint-disable MD046 -->

```yaml
jobs:
  publish:
    runs-on: 'ubuntu-latest'
    permissions:
      contents: read
      id-token: write  # IMPORTANT: mandatory for trusted publishing
    steps:
      - name: 'Publish to PyPI'
        uses: lfreleng-actions/pypi-publish-action@main
        with:
          environment: 'production'
          tag: 'v1.0.0'
          trusted_publishing: 'true'
```

<!-- markdownlint-enable MD046 -->

The `trusted_publishing` input controls how the action selects its
authentication method:

<!-- markdownlint-disable MD013 -->

| Value   | Behaviour                                                                                                                                        |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `auto`  | Default. The action selects trusted publishing when the package already exists in the index. Falls back to credential-based methods otherwise.   |
| `true`  | Always use trusted publishing, even for first-time publishes. Use this when you pre-configure trusted publishing in the PyPI web portal.         |
| `false` | Never use trusted publishing. Always use credential-based methods (1Password or GitHub secret).                                                  |

<!-- markdownlint-enable MD013 -->

### Pre-Configured Trusted Publishing

<!-- markdownlint-disable MD013 -->

PyPI now supports configuring trusted publishers **before** a package has
ever had a release. When you pre-configure trusted publishing for a
new project on PyPI (or Test PyPI), set `trusted_publishing: 'true'` to
allow the first publish to use OIDC authentication directly, without
requiring a static API key.

<!-- markdownlint-enable MD013 -->

```yaml
- name: 'Publish to PyPI'
  uses: lfreleng-actions/pypi-publish-action@main
  with:
    environment: 'production'
    tag: 'v1.0.0'
    trusted_publishing: 'true'
```
