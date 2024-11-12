# Setup

## This works but is somewhat outdated.  Use [AWS](aws.md) demo script instead.

## Minikube
Start a 1.28 version of k8s with minikube with some specific command-line arguments.
**This is destructive to the current minikube environment!**
```
minikube stop
minikube delete
minikube start --kubernetes-version=v1.28.12 --memory=5g --bootstrapper=kubeadm \
    --extra-config=kubelet.authentication-token-webhook=true \
    --extra-config=kubelet.authorization-mode=Webhook \
    --extra-config=scheduler.bind-address=0.0.0.0 \
    --extra-config=controller-manager.bind-address=0.0.0.0 \
    --feature-gates=SidecarContainers=true
```

## Prometheus Operator and Grafana
Clone the Kube-Prometheus git repository 
```
cd ~
git clone https://github.com/prometheus-operator/kube-prometheus/ kube-prometheus
```

Apply the manifests in a couple of steps to avoid race conditions during startup
```
cd ~/kube-prometheus
kubectl apply --server-side -f manifests/setup
kubectl wait \
    --for condition=Established \
    --all CustomResourceDefinition \
    --namespace=monitoring
kubectl apply -f manifests/
```

In two separate terminals, set up port forwarding for Grafana and Prometheus
```
cd ~/chiller/helm
make port-graf
```
This is a shortcut for `kubectl port-forward -n monitoring --address=192.168.88.130 svc/grafana 3000`.
```
cd ~/chiller/helm
make port-prom
```
This is a shortcut for `kubectl port-forward -n monitoring --address=192.168.88.130 svc/prometheus-k8s 9090`.

Note that the default username/password for Grafana is admin/admin.

## images for frontend, api, and load testing
There are two image sources
1. TTL images made using Make in the dev environment
2. ghcr.io images built by the CI GitHub Actions when a pull request is created into a release branch

To create the TTL images, do the following.
```
cd ~/chiller
make images-ttl
```
TTL images only persist for 24 hours.

ghcr.io images are built by the CI GitHub Actions whenever a pull request is created that targets a release branch.

## Install application with Helm
This should be done after installing the Prometheus Operator, as the Helm chart
includes CRDs for Prometheus.  
If done before, the CRDs will not be created when the application is installed.

Do **ONE** of the following, depending on if you want to use the TTL images or the ghcr.io images from the CI GitHub Actions.  If using the ghcr.io images, ensure that you have currently checked out a release branch or main in the dev environment.

TTL images
```
cd ~/chiller/helm
make ttl
```
**OR**

### ghcr images
```
cd ~/chiller/helm
make cicd
```
## Set up Grafana dashboard
Log into Grafana with admin/admin.  
Create a new dashboard, and copy and paste the JSON from [this URL](https://github.com/lago-morph/chiller/blob/release-0.1.1/monitoring/grafana/chiller_application).

# Show application
To set up port forwarding to access on host machine run this in a dedicated shell:
```
cd ~/chiller/helm
make port-app
```
This is a shortcut for `kubectl port-forward --address=192.168.88.130 svc/chiller-frontend 8080:80`.
Note: having this and locust running at the same time works, but there may be intermittent errors.

# Run load test to populate metrics
To set up port forwarding for locust run this in a dedicated shell:
```
cd ~/chiller/helm
make port-load
```
This is a shortcut for `kubectl port-forward --address=127.0.0.1 svc/chiller-frontend 8222:80`

Run locust load test
```
cd ~/chiller/load/locust
. ~/run/locust/.venv/bin/activate
make load-1
```
