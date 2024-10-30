# Continuous Deployment (CDep)
## Description
The continuous deployment component deals with rolling out a release to the production environment.
It consists of two parts
- Deployment tests
- Automatic deployment to production

Deployment tests ensure that a clean install, an upgrade over the previous version, and a load test all run successfully in a test environment.
Deployment to production will perform a rollout into the production environment, monitor the application for a set amount of time, and if certain metrics are out of range (e.g., response time, 4xx and 5xx http response codes) the application will be rolled back to the last known good version.

## Design
Parts of the CDep process are triggered in three different ways
- Individual deployment tests can be triggered on demand by the developer to run against the most recent images in a release branch
- All deployment tests are run automatically when a pull request is created from a release branch into main.  They must complete successfully before the pull request can be merged
- Deployment into production is triggered when a pull request is merged successfully into the main branch

The design is partially described as part of the CI/CD Overview.
See the sections [Pull request into main](https://github.com/lago-morph/chiller-doc/blob/main/cicd_overview.md#pull-request-into-main) and [The release is finalized](https://github.com/lago-morph/chiller-doc/blob/main/cicd_overview.md#the-release-is-finalized).

### Deployment tests

Three types of deployment tests are run.
- Initial
- Upgrade
- Load

They can be initiated on-demand or as part of an automatic CD run when a pull request is created from a release branch into main.  
The test initiator is the user who requested the test (for on-demand tests) or the user who created the pull request (for triggered tests).
On-demand tests are triggered by invoking the appropriate workflow using the gh CLI or the web interface.

In the following description the **production** version is the set of images and install scripts associated with the most recent release on the main branch of the repository.  
The **test** version is the set of images and install scripts associated with a specified release branch of the repository.

#### Initial
The objective of the initial test is to verify that
- All components are properly installed in a new environment
- Simulated users experience the desired level of service after a clean install (errors and response times)

Test steps:
1. The test version of the application is installed in a newly-created namespace.
2. Checks are made to make sure the specified number of k8s components are running in the test namespace.
3. A small number of simulated users are deployed to access the application
4. After a short period metrics are examined to ensure that the application is within bounds for error rate and response time.
5. Create a GitHub issue related to the test run and attach test report to issue.
6. Remove simulated users.
7. If test was successful, close issue, uninstall application, and delete namespace.
8. If test was not successful notify test initiator of failure.

#### Upgrade
The objective of the upgrade test is to verify that
- During the upgrade the application does not behave in an undesired way
- Components of the previous version are properly removed as part of an upgrade
- Components from the new version are properly installed as part of an upgrade
- Simulated users experience the desired level of service after an upgrade (errors and response times)

Test steps:
1. The production version of the application is installed in a newly-created namespace.
2. A small number of simulated users are deployed to access the application during the upgrade.
3. The test version of the application is then installed using a helm upgrade.  During the upgrade process, metrics are collected against the simulated users.
4. Performance metrics are checked to verify that there was minimal service disruption during the upgrade.
5. Checks are run to ensure that the correct number and version of k8s components are running in the test namespace.
6. The application runs for a specified period after the upgrade completes with simulated users.
7. User metrics are checked to ensure the application is performing within acceptable limits.
8. Create a GitHub issue related to the test run and attach test report to issue.
9. Remove simulated users.
10. If test was successful, close issue, uninstall application, and delete namespace.
11. If test was not successful notify test initiator of failure (leave issue open).
12. If the associated issue is closed (implies test was unsuccessful and testing environment no longer needed) uninstall application and delete namespace.

#### Load
The objective of the load test is to verify that
- The application performs within desired limits when heavily loaded
The load test is run against an upgrade, as that is the environment that is assumed in production.

Test steps:
1. The production version of the application is installed in a newly-created namespace.
2. The test version of the application is then installed using a helm upgrade.
3. A large number of simulated users are deployed to access the application
4. User metrics are checked to ensure the application is performing within acceptable limits.
5. Create a GitHub issue related to the test run and attach test report to issue.
6. Remove simulated users.
7. If test was successful, close issue, uninstall application, and delete namespace.
8. If test was not successful notify test initiator of failure (leave issue open).
9. If the associated issue is closed (implies test was unsuccessful and testing environment no longer needed) uninstall application and delete namespace.

### Deployment to production
When a pull request is merged into main, this triggers an automatic deployment into production.

Note that for this project, a set of simulated users will be manually deployed to mimic real users of the production environment.  The deployment process itself does not deploy simulated users.

Deployment steps:
1. The test version of the application is installed using a helm upgrade.
2. During the upgrade and for a set period after it is complete user metrics are monitored to ensure that they do not deviate from acceptable limits.  If a problem is detected the deployment is immediately aborted.
3. When the upgrade is complete checks are run to ensure that the correct number and version of k8s components are running in the production environment.  If not, the deployment is aborted
4. If at any time the deployment is aborted the upgrade is rolled back to the previous version
5. Create a GitHub issue related to the deployment and attach the test report to the issue
7. If the deployment was successful close the issue, tag the merge commit with the release version, and tag the container images with the release version in ghcr.
8. Notify the user that merged the pull request (and therefore triggered the deployment) of the deployment status

Unlike the deployment tests, no automated action is taken when the GitHub issue associated with an unsuccessful deployment is later closed.
If a deployment is aborted, the production environment is rolled back to the application version running before the deployment was attempted.  
In this case the commit at the HEAD of the main branch will not represent the code running in production.
Manual steps should be taken to reset the HEAD pointer for main in the repository to the appropriate pre-merge commit, effectively undoing the merge that led to an aborted deployment.

### Environments
There are two environments that can be targeted, dev and prod.
Both have a single installation of Prometheus and Grafana.
In the Dev environment, this installation is shared by multiple instances
of the application, organized by namespace.
Dev is used for both on-demand and CD-triggered release tests.

Both environments are on AWS EKS clusters maintained with Terraform.

### Finding the correct images
The CDep process depends on tags created during the CDel process.
In Git, tags are just objects in the refs/tags directory that point to a commit.
There is some logic required to find out what the latest tag is that is visible from the head of a given branch.
With an up-to-date cloned git repository, the command to do this is
`git describe --tags --abbrev=0 <releasebranchname>`.
This does not appear to be possible using the `gh` CLI.
This is possible, but more complex using the github REST API.
With the REST API I would have to backtrack through all the commits on the branch, checking for each commit if there is an object in refs/tags that points to that commit.
## Status
Design only, not yet prototyped.
