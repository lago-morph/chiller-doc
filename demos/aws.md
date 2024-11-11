# Setup

## AWS Cloud Playground
Create a new playground on [A Cloud Guru](https://learn.acloud.guru/cloud-playground/cloud-sandboxes).

Create cluster using AWS CLI and Terraform on local Linux machine.

On local Linux machine:
```
aws configure
```
And type in key info from playground launch page.

## Create AWS EKS cluster

Create the cluster, install LBC, install monitoring **(this takes 12-17 minutes)**:
```
cd ~/chiller-iac/exp/aws
terraform init
terraform apply --auto-approve
```

After this is done, get the credentials for kubectl
```
cd ~/chiller-iac/exp/aws
make kubectl
```

# Install chiller application
Make sure you have updated the kubectl credentials in the previous step.

```
cd ~/chiller/helm
make cicd
```

Get the address of the chiller application load balancer:
```
cd ~/chiller/helm
make get-ingress
```
Copy and paste this link into local browser.  It will **take about 5 minutes** before it is available.

# Set up simulated user load

Set up port forwarding for locust, run this in a dedicated shell:
```
cd ~/chiller/helm
make port-load
```
This is a shortcut for `kubectl port-forward --address=127.0.0.1 svc/chiller-frontend 8222:80`

Run locust load test
```
cd ~/chiller/load/locust
. ~/run/locust/.venv/bin/activate
make load-10
```

# View Grafana dashboard

Set up port forwarding for Grafana
```
cd ~/chiller/helm
make port-graf
```
This is a shortcut for `kubectl port-forward -n monitoring --address=192.168.88.130 svc/kube-prometheus-stack-grafana 3000:80`.

In a browser window open:
```
http://192.168.88.130:3000
```
Username/password is `admin`/`admin`.

Select the "Chiller application" dashboard.

# Prometheus
Can also view the web interface for Prometheus (good for debugging).
Do this in a dedicated shell:
```
cd ~/chiller/helm
make port-prom
```
This is a shortcut for `kubectl port-forward -n monitoring --address=192.168.88.130 svc/prometheus-operated 9090`.
```

Then can open prometheus in a local browser at the following address:
```
http://192.168.88.130:9090
```
