# Dockerfile and GitHub action for publishing JSII packages

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/udondan/jsii-publish)][releases]
[![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/udondan/jsii-publish)][hub]
[![Docker Cloud Automated build](https://img.shields.io/docker/cloud/automated/udondan/jsii-publish)][hub-builds]
[![Docker Pulls](https://img.shields.io/docker/pulls/udondan/jsii-publish)][hub]
[![GitHub](https://img.shields.io/github/license/udondan/jsii-publish)][MITlicense]

Package building and publishing to npm, PyPI, NuGet and Maven (GitHub).

## Examples

### Usage in a GitHub workflow

```yml
---
name: Publish packages

on:
  push:
    branches:
      - refs/tags/v*

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Get the version
        id: get_version
        run: echo ::set-output name=VERSION::${GITHUB_REF/refs\/tags\/v/}

      - name: Checkout code
        uses: actions/checkout@v1
        with:
          fetch-depth: 1

      - name: Publish packages
        uses: udondan/jsii-publish@v0.6.1
        with:
          VERSION: ${{ steps.get_version.outputs.VERSION }}
          BUILD_SOURCE: true
          BUILD_PACKAGES: true
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
          PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}
          NUGET_TOKEN: ${{ secrets.NUGET_TOKEN }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITHUB_REPOSITORY: ${{ GITHUB_REPOSITORY }}
```

In above example all actions are executed in the same step. You as well can split the actions into separate steps like so:

```yml
---
name: Publish packages

on:
  push:
    branches:
      - refs/tags/v*

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Get the version
        id: get_version
        run: echo ::set-output name=VERSION::${GITHUB_REF/refs\/tags\/v/}

      - name: Checkout code
        uses: actions/checkout@v1
        with:
          fetch-depth: 1

      - name: Build source
        uses: udondan/jsii-publish@v0.6.1
        with:
          VERSION: ${{ steps.get_version.outputs.VERSION }}
          BUILD_SOURCE: true

      - name: Build packages
        uses: udondan/jsii-publish@v0.6.1
        with:
          BUILD_PACKAGES: true

      - name: Publish to npm
        uses: udondan/jsii-publish@v0.6.1
        with:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

      - name: Publish to PyPI
        uses: udondan/jsii-publish@v0.6.1
        with:
          PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}

      - name: Publish to NuGet
        uses: udondan/jsii-publish@v0.6.1
        with:
          NUGET_TOKEN: ${{ secrets.NUGET_TOKEN }}

      - name: Publish to Maven GitHub
        uses: udondan/jsii-publish@v0.6.1
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITHUB_REPOSITORY: ${{ GITHUB_REPOSITORY }}
```

### Usage for running the Docker image locally

```bash
docker run -it \
    --workdir /workdir \
    --volume $(pwd):/workdir \
    --env VERSION=0.3.0 \
    --env BUILD_SOURCE=true \
    --env BUILD_PACKAGES=true \
    --env NPM_TOKEN \
    --env PYPI_TOKEN \
    --env NUGET_TOKEN \
    --env GITHUB_TOKEN \
    --env GITHUB_REPOSITORY="${OWNER}/${REPOSITORY}"
    udondan/jsii-publish:0.6.1
```

The package code can be mounted to any location in the container. Just make sure you set the workdir to the same value. In the example above I use `/workdir`.

Parameters passed per env:

- **VERSION**: If set, the version in `package.json` will be updated with this value
- **BUILD_SOURCE**: If `true`, the source will be compiled from TS to JS
- **BUILD_PACKAGES**: If `true`, all configured JSII packages will be built
- **NPM_TOKEN**: Your publish token for npm. If passed, package will be published to npm
- **PYPI_TOKEN**: Your publish token for PyPI. If passed, package will be published to PyPI
- **NUGET_TOKEN**: Your publish token for NuGet. If passed, package will be published to NuGet
- **GITHUB_TOKEN**: The token to interact with GitHub. If passed, the Maven package will be published to GitHub packages.
  If you run the GitHub action, the token will be automatically be generated byu GitHub and is available as `${{ GITHUB_TOKEN }}`. If you run the Docker image yourself, you need to pass in a [personal access token](https://github.com/settings/tokens) with `read:packages` and `write:packages` capabilities.
- **GITHUB_REPOSITORY**: The url slug of your repository, which is `${OWNER}/${REPOSITORY}`. In a github action you can just pass `${{ GITHUB_REPOSITORY }}`.

## License

The project is licensed under [MIT][MITlicense].

   [hub]: https://hub.docker.com/r/udondan/jsii-publish
   [hub-builds]: https://hub.docker.com/r/udondan/jsii-publish/builds
   [releases]: https://github.com/udondan/jsii-publish/releases
   [MITlicense]: https://github.com/udondan/jsii-publish/blob/master/LICENSE
