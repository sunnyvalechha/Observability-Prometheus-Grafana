Reference repo: git clone https://github.com/iam-veeramalla/observability-zero-to-hero.git

What is Observability?
- While monitoring focuses on predefined metrics, observability allows you to ask any question about your system's behavior and get answers based on the data it generates.

Example:
* disk utilization of an EKS node over the last 24 hours?
* CPU utilization of a particular node of a Kubernetes cluster?
* Out of 1000 HTTP requests, how many failed, along with the reason for failure, and how many succeeded?
* Memory leak of the application?

- Observability helps us to fix all the above issues with the help of **3 pillars of observability**

1. Metrics - Give insights about the state of the system. Deal with historical data of events like cpu, memory, disk, https requests.
2. Logging - Give insights into why the system is in this particular state.
3. Tracing - Give insights on how to fix this particular state.

# Metrics & Monitoring

* Architecture of Prometheus
* Component of Prometheus

- Prometheus scrapes (pulls) information (cpu, memory) in multiple ways few of which are listed below:
1. Node exporter
2. Kube state metrics
3. Custom metrics - these are related to our application, such as total time which http has taken to process the request, or In 24hour,s how many users have created an account in the application.
4. mysql exporter - it will continuously talk to the mysql and get the information through IP address.

* Prometheus scrapes these metrics and store in "time series database" (TSDB)
* As a user, we run a Prometheus query language "PromQL" query and import the information from Prometheus TSDB.
* A user is talking to a component of Prom. A server that is a http server.

Installation:
* Setup a EKS cluster (any k8 will work)
* Associate OIDC provider same did in EKS setup
* Install helm

        curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
        chmod 700 get_helm.sh
        ./get_helm.sh

        helm repo update

* Create a seperate namespace

        kubectl create ns monitoring

* Deploy chart into namespace

        helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring -f ./custom_kube_prometheus_stack.yml
        kubectl --namespace monitoring get pods -l "release=monitoring"

  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
kubectl get svc | grep prometheus
kubectl get svc prometheus-kube-prometheus-prometheus --output yaml > prometheus.yml
kubectl get svc prometheus-kube-prometheus-alertmanager --output yaml > alertmanager.yml

Note: Edit both yaml files and change type from ClusterIP to LoadBalancer

kubectl get svc | grep prometheus

Note: Access with EXTERNAL-IP URL of both LoadBalancer add respective ports 9090 & 9093 

* Prometheus collect any node information from Node exporters.
* Prometheus also scrape metric from "kube state metric" this ksm is already running on the k8s cluster and communicate with API server.
* Prometheus also scrape metric through custom metrics which is written through developers.
