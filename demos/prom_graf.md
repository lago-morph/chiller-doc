# Setup

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

In two separate terminals, set up port forwarding
```
kubectl port-forward -n monitoring --address=192.168.88.130 svc/grafana 3000
```
```
kubectl port-forward -n monitoring --address=192.168.88.130 svc/prometheus-k8s 9090
```
Note that the default username/password for Grafana is admin/admin.

## TTL images for frontend, api, and load testing
```
cd ~/chiller
make images-ttl
```

## Install application with Helm
This must be done after installing the Prometheus Operator, as the Helm chart
includes CRDs for Prometheus.
```
cd ~/chiller/helm
helm install chiller ./chiller --post-renderer ./kust.sh
```
## Set up Grafana dashboard
Log into Grafana with admin/admin.  
Create a new dashboard, and copy and paste the JSON from [this URL](https://github.com/lago-morph/chiller/blob/release-0.1.1/monitoring/grafana/chiller_application).

# Show application
Set up port forwarding to access on host machine
```
kubectl port-forward --address=192.168.88.130 svc/chiller-frontend 8080:80
```
Note: having this and locust running at the same time works, but there will be intermittent errors.

# Run load test to populate metrics
Set up port forwarding for locust
```
kubectl port-forward --address=127.0.0.1 svc/chiller-frontend 8222:80
```
Run locust load test
```
cd ~/chiller/load/locust
. ~/run/locust/.venv/bin/activate
make load-1
```

