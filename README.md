# chiller-doc
Documentation for the chiller project.

## Overview
The Watch and Chill project (chiller for short) is a open source project to demonstrate how to set up CI/CD for a kubernetes-based application.  

The application itself is very simple, and is not the focus of this project.
There is just enough functionality to demonstrate a non-trivial 3-tier application with web front end, back end services, and a database.

The focus is on the activities supporting the development, testing and operation of the application.

The project is spread across three different GitHub repositories:
- The source code is in the [chiller](https://github.com/lago-morph/chiller) repository
- IaC scripts are in the [chiller-iac](https://github.com/lago-morph/chiller-iac) repository
- Documentation is in this repository

## Project components


The requirements and design for the application are part of the 
[application component](application.md).

The API which connects the front end to the back end services is defined in an OpenAPI specification.
Skeleton code for the back end services and a Python SDK to call the services 
are generated using [Swagger Codegen](https://github.com/swagger-api/swagger-codegen).
Documentation and implementation notes for the API definition and code generation are part of the [API definition component](api_definition.md).

Instructions for how to set up and use the development environment are part of the [development platform component](development_platform.md).

Instructions for how to manually run tests (unit, integration, browser), as well as for convenience Makefile scripts are part of the [developer testing component](developer_testing.md).

Instructions for how the git repository is intended to be used, including branching, pull requests, merging and how the CI/CD system integrates with GitHub are part of the [CI/CD Overview component](cicd_overview.md).

Documentation and GitHub Actions code for the continuous integration (CI) system, including what tests are run and what trigger them are part of the [continuous integration component](ci.md).

Documentation and GitHub Actions code for the continuous delivery (CDel) system, including how containers are built, how they are tagged, where they are stored, and how GitHub labels are used to tie the containers to the commit they are built from are part of the [continuous delivery component](cdel.md).

The design for running automated deployment tests and deploying into production are part of the [Continuous deployment component](cdep.md).

Terraform code for maintaining the production cloud environment is part of the [IaC environment setup component](iac.md).

The Helm chart for application installation is part of the 
[Application install component](install.md).

Documentation and install scripts for Prometheus Operator and Grafana are part of the [Observability - metrics component](obs_metrics.md).

Documentation and instructions for running load tests with Locust are part of the [Load testing component](load.md).

Planned, but not currently designed are the capability to ingest logs into a central data store, and to implement tracing with Jaeger.

## Current status

|Component|Status|
|---|---|---|
|[Application](application.md)|:green_circle: Complete|
|[API definition](api_definition.md)|:green_circle: Complete|
|[Development platform](development_platform.md)|:green_circle: Complete|
|[Developer testing](developer_testing.md)|:green_circle: Complete|
|[CI/CD Overview](cicd_overview.md)|:green_circle: Complete|
|[Continuous integration (CI)](ci.md)|:green_circle: Complete|
|[Continuous delivery (CDel)](cdel.md)|:yellow_circle: In process|
|[Continuous deployment (CDep)](cdep.md)|:orange_circle: Designed|
|[IaC - environment setup](iac.md)|:yellow_circle: In process|
|[Application install](install.md)|:yellow_circle: In process|
|[Observability - metrics](obs_metrics.md)|:yellow_circle: In process|
|[Load testing](load.md)|:yellow_circle: In process|
|Observability - tracing|:red_circle: Not started|
|Observability - logs|:red_circle: Not started|

