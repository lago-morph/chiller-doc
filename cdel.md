# Continuous Delivery (CDel)
## Description
The CDel process is triggered by the approval of a pull request into a 
release branch (any branch named "release-*").  This generates a pull_request
event with type closed.

The CDel process reuses the containers built during CI.

## Design
During the CI process, we compute an identifier based on the name of the release
branch and the date/time (UTC time) that the CI process ran.
This is done in the compute-label job in the [build_and_test.yml](https://github.com/lago-morph/chiller/.github/workflows/build_and_test.yml) Action.
For instance, for the branch `release-0.1.1`, for a CI process that started on
August 5th, 2024 at 21:15:11 the identifier will be `v0.1.1-RC-20240805-211511`.

When the containers are built and stored in ghcr.io, they are tagged with
this identifier.

If the CI job completes successfully, the identifier is saved as a label on the pull request.
This is done in the set-label job in the [build_and_test.yml](https://github.com/lago-morph/chiller/.github/workflows/build_and_test.yml) Action.
A pull request can end up having multiple identifiers stored in labels if changes are made to the code in the pull request.

When a pull request is merged, the most recent identifier is retrieved from the pull request labels.
The merge commit is then tagged with this identifier.
This is done in the [tag_on_pr_close.yml](https://github.com/lago-morph/chiller/.github/workflows/tag_on_pr_close.yml) Action.

The continuous deployment (CDep) process will later use this tag to find the appropriate containers for deployment testing and eventually for automatic rollout to production.

## Status
This component is complete.
The most recent version is in the release-0.1.1 branch, and will be merged
into the main branch as part of this cleanup effort.
