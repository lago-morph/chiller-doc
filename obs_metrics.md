# Observability - Metrics
## Description
This component consists of the Prometheus operator and Grafana, plus the various configurations and CRDs that allow me to customize it for my environment.

Some (outdated) notes are in wiki page [Prometheus and Grafana](https://github.com/lago-morph/chiller/wiki/Prometheus-and-Grafana)

## Design
The basis of the observability portion is the [Kube Prometheus Stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) distribution.
This consists of a number of components, but for our purposes it is the Prometheus Operator and Grafana.
In addition, we need to add some additional options to our pods when they are installed so that they can provide metrics in the appropriate format.

### Prometheus Operator
[Prometheus](https://prometheus.io/) is a metrics collection and storage system.

The [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator) (PO going forward) is a [Kubernetes Operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) that manages individual installations of Prometheus.
What this means is that the PO itself does not have any monitoring functionality (although the default Helm chart will also deploy one instance of Prometheus).
Through the use of Kubernetes Custom Resource Definitions (CRDs), the Kubernetes administrator can deploy and configure multiple copies of Prometheus running on a single cluster.

For the purposes of this project, we will utilize two CRDs from the PO:
- prometheuses.monitoring.coreos.com - defines an installation of Prometheus, with top-level rules for what parts of the cluster are to be monitored
- servicemonitors.monitoring.coreos.com - defines a Kubernetes Service to be monitored by a Prometheus installation

There are ton of additional CRDs that can be used to configure AlertManager and various other types of Prometheus probes.  We don't use those in this project.

In general, when we install a copy of the chiller application it will have its own unique namespace.
We then deploy the prometheuses CRD and set up attributes to restrict it to our namespace and a specific set of tags.  That is done through the serviceMonitorNameSpaceSelector and the serviceMonitorSelector fields in the prometheuses CRD.
We then deploy servicemonitor CRDs for each service we want to monitor.
These CRDs need to be in the right namespace and have a label that matches the prometheuses CRD serviceMonitorSelector field.
This means the metrics for each install are independent from each other, allowing us to run "production" and multiple deployment tests simultaneously without them stepping on each other's toes.

### Grafana
Grafana is used more or less in the default configuration.
One feature of this distribution of Grafana is it can be configured to continuously scan for a ConfigMap with a specific label, and if found, will import the contents of that ConfigMap as a custom dashboard.
The configuration details for that are in the Grafana chart [documentation](https://github.com/grafana/helm-charts/tree/main/charts/grafana#sidecar-for-dashboards).
The dashboard used for this application tracks the response time through both the front-end and the back-end for each request, and tracks the fetch rate and insert rate in the database.

### Application metrics
All of this infrastructure isn't much use if we can't get the data we want.
In some cases you can find infrastructure that provides metrics to Prometheus with no modifications.
In our case, we weren't so lucky, and had to do some extra cofiguration in order to export the appropriate metrics.
One of the design decisions was not to make ANY application-level code changes to support metrics.

#### Chiller application
The application consists of two components, chiller-frontend and chiller-api.
Both of these are written in Python using the [Flask framework](https://flask.palletsprojects.com/en/stable/).
Flask was chosen for the simple reason that the swagger codegen program used to generate the skeleton of the api (see [the API documentation](api_definition.md)) generated code for use with Flask.
For simplicity, I chose to use Flask also for the frontend, which was based loosely on the application used for one of the Flask tutorials.
Flask is a WSGI Application.  It can run in development from the command line, but when deployed to production it should use a WSGI server.
The server chosen is [Gunicorn](https://gunicorn.org/).
Gunicorn collects and exports metrics in [StatsD](https://github.com/statsd/statsd) format.
StatsD and Prometheus are fundamentally incompatible - StatsD expects metrics to be "pushed" to a central monitoring server, and Prometheus expects metrics to be made available at a URL so that Prometheus can "pull" the stats periodically.
This turned out to be an excellent learning opportunity to understand how to integrate various infrastructure components into a Prometheus system without any application source code changes.
In each application pod running either the chiller-frontend or chiller-api component, a [sidecar](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) container is inserted when the pod is deployed to Kubernetes.
The sidecar used is the [statsd_exporter](https://github.com/prometheus/statsd_exporter), part of the Prometheus ecosystem.
Stats_exporter runs as a process next to the application container, sharing the same network namespace.  
It listens for statsd-style push metrics, stores the metrics temporarily, and exposes a metrics endpoint that Prometheus can use to pull the metrics periodically.
Gunicorn is then configured to talk to statsd_exporter as if it was a statsd daemon, in effect translating the metrics from one format to another.
Statsd_exporter is quite configurable in terms of mapping metrics between the statsd input and the prometheus-style output.  
See the [documentation](https://github.com/prometheus/statsd_exporter?tab=readme-ov-file#metric-mapping-and-configuration) for more details.
In this application we make light use of this remapping, see statsd-cm.yaml in the Helm chart.

### Postgres database metrics
Postgres also does not speak native Prometheus.
Luckily there is another application similar to statsd_exporter, but for Postgres - [postgres-exporter](https://github.com/prometheus-community/postgres_exporter).
Postgres-exporter runs as its own pod/service, and periodically queries Postgres for metrics, then makes them available in Prometheus format.
This one is a little less interesting because it runs as its own independent pod/service/deployment, and required much less invasive work to the Helm chart to get working.

## Status
This works as designed for a single version of Chiller in the cluster deployed to the default namespace.

The reason this is not yet marked as complete is that I need to add the functionality that supports multiple independent installations of the chiller application in different namespaces on the same cluster.
Concretely, that means:
- Modify application installation process (Helm chart) to support independent installs by namespace
- Change the default behavior of the monitoring chart to not install a prometheuses CRD for the default namespace
- Modify the Helm chart to deploy the appropriate prometheuses and servicemonitor CRDs for a namespace-specific deployment
- Modify the Helm chart to deploy a ConfigMap for the Grafana custom dashboard that is specific to the application namespace
