# Continuous Deployment (CDep)
## Description
The continuous deployment component deals with rolling out a release to the production environment.
It consists of two parts
- Deployment tests
- Deployment to production

Deployment tests ensure that a clean install, an upgrade over the previous version, and a load test all run successfully in a test environment.
Deployment to production will perform a rollout into the production environment, monitor the application for a set amount of time, and if certain metrics are out of range (e.g., response time, 4xx and 5xx http response codes) it can perform an automated rollback to the last known good version.

## Design
The design is currently described as part of the CI/CD Overview.
See the sections [Pull request into main](https://github.com/lago-morph/chiller-doc/blob/main/cicd_overview.md#pull-request-into-main) and [The release is finalized](https://github.com/lago-morph/chiller-doc/blob/main/cicd_overview.md#the-release-is-finalized).
## Status
Conceptual design only, not yet prototyped.
