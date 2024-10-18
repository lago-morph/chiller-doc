# Continuous Integration (CI)
## Description
The CI process is triggered by a pull request created against a release
branch (any branch named "release-*").
All unit, integration, and browser tests are run on the code during CI.
If any of the tests fail, the pull request cannot be approved until the issues
are fixed.

CI and CDel are intertwined, because we create docker images during
the CI process and keep track of the image labels.  
If the pull request is merged, the continuous delivery process will reuse the
images built during CI.
## Design
CI uses GitHub actions.
As detailed in the CI/CD overview section, we use access controls on the
release (any branch named "release-*") and main branches to force all merges
to go through pull requests.

The Action that controls the CI process is [build_and_test.yml](https://github.com/lago-morph/chiller/blob/main/.github/workflows/build_and_test.yml).

The steps for CI (and a little bit of CD, described in more detail in cdel.md) are:
- Build the python packages for the frontend, backend api, and the sdk (api calling wrappers) in [build-packages.yml](https://github.com/lago-morph/chiller/blob/main/.github/workflows/build-packages.yml)
- Run unit tests on api ([unit-test-chiller-frontend.yml](https://github.com/lago-morph/chiller/blob/main/.github/workflows/unit-test-chiller-frontend.yml)) and frontend ([unit-test-chiller-frontend.yml](https://github.com/lago-morph/chiller/blob/main/.github/workflows/unit-test-chiller-frontend.yml))
- Build images for backend api service ([build-chiller-api-image.yml](https://github.com/lago-morph/chiller/blob/main/.github/workflows/build-chiller-api-image.yml)) and frontend ([build-chiller-frontend-image.yml](https://github.com/lago-morph/chiller/blob/main/.github/workflows/build-chiller-frontend-image.yml)) and store them in ghcr.io
- Run integration test on backend api ([integration-test-chiller.yml](https://github.com/lago-morph/chiller/blob/main/.github/workflows/integration-test-chiller.yml))
- Run end-to-end browser test ([browser-test-chiller.yml](https://github.com/lago-morph/chiller/blob/main/.github/workflows/browser-test-chiller.yml)
- Some extra steps to keep track of the images being built to use later for CDel

As much as possible the process is parallelized to speed up the process.

If any of the steps fail, the pull request cannot be approved.
If the issues are fixed and pushed back into the feature branch that was the source of the pull request, the CI process automatically runs again since GitHub will generate a new pull request event.

## Status
This component is complete.
The most recent version is in the release-0.1.1 branch, and will be merged
into the main branch as part of this cleanup effort.
