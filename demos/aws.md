# Setup

## AWS Cloud Playground
Create a new playground on [A Cloud Guru](https://learn.acloud.guru/cloud-playground/cloud-sandboxes).

Create cluster using AWS CLI and Terraform on local Linux machine.

On local Linux machine:
```
aws configure
```
When prompted, paste in the Access Key Id and Secret Access Key from the playground launch page.  Make sure to use `us-east-1` as the region (or change the region used by Terraform in the `variables.tf` file).

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
This is a shortcut for `aws eks update-kubeconfig --region $(terraform output -raw region) --name $(terraform output -raw eks_cluster_name)`

# Install chiller application
**Make sure you have updated the kubectl credentials in the previous step.**

```
cd ~/chiller/helm
make cicd
```
This uses Helm to install the chiller application using the most recent containers that went through the CI/CD process in the git branch currently checked out (right now `release-0.1.1`).

Get the address of the chiller application load balancer:
```
cd ~/chiller/helm
make get-ingress
```
This is a shortcut for `kubectl get ingress ingress-chiller -o json | jq .status.loadBalancer.ingress[0].hostname` followed by some sed edits to make it easier to copy/paste (`sed -e "s/\"//g" | sed -e "s/^/http:\/\//"`).

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
This simulates 10 users constantly creating a new user, adding 4 movies, then logging out.

# View Grafana dashboard

Set up port forwarding for Grafana in a dedicated shell:
```
cd ~/chiller/helm
make port-graf
```
This is a shortcut for `kubectl port-forward -n monitoring --address=192.168.88.130 svc/kube-prometheus-stack-grafana 3000:80`.

In a browser window open:

http://192.168.88.130:3000

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

Then can open prometheus in a local browser at the following address:

http://192.168.88.130:9090
