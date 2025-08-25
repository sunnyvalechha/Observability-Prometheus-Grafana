Git repo URL: https://github.com/iam-veeramalla/observability-zero-to-hero/

What is Observability?
- While monitoring focuses on predefined metrics, observability allows you to ask any question about your system's behavior and get answers based on the data it generates.

Example:
* What is the disk utilization of a EKS node from last 24 hours?
* What is the CPU utilization of a particular node of a Kubernetes cluster?
* Out of 1000 http request, how many failed along with the reason of failure and how many succeed?
* Memory leak of the application?

- Obervability helps us to fix all the above issues with the help of **3 pillars of observability**

1. Metrics - Give insights about the state of the system. Deal with historical data of events like cpu, memory, disk, https requests.
2. Logging - Give insights why is the system in this particular state.
3. Tracing - Give insights how to fix this particuar state.

Metrics Example:
<img width="1307" height="753" alt="image" src="https://github.com/user-attachments/assets/8513e216-5526-4736-8647-61965b1e4960" />

# Metrics & Monitoring

* Architecture of Prometheus
* Component of Prometheus

<img width="831" height="1080" alt="image" src="https://github.com/user-attachments/assets/6fa1f9dc-56a2-44e8-8c7d-db4d4b7e9f32" />




- Prometheus scrapes (pull) information (cpu, memory) in multiple ways few of them are listed below:
1. Node exporter
2. Kube state metrics
3. Custom metrics - these are related to our application such as total time which http has took to process the request or In 24hours how many users have created account in application.
4. mysql exporter - it will continuesly talk to the mysql and get the information through IP address.

* Prometheus scrape these metrics and store in "time series database" (TSDB)
* As a user we run prometheus query language "PromQL" query and import the information from proemetheus TSDB.
* A user is talking to a component of prom. server which is a http server.

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

* Clone a repository

        git clone https://github.com/iam-veeramalla/observability-zero-to-hero.git
        cd observability-zero-to-hero/day-2/

* Deploy chart into namespace

        helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring -f ./custom_kube_prometheus_stack.yml

<img width="1649" height="380" alt="image" src="https://github.com/user-attachments/assets/4555f9da-43ae-4357-abf5-8bdc6a73c54c" />

                kubectl --namespace monitoring get pods -l "release=monitoring"
                kubectl --namespace monitoring get secrets monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo                # Get grafana user id & password (prom-operator)
                
                
                kubectl port-forward service/prometheus-operated -n monitoring 9090:9090
                kubectl port-forward service/monitoring-grafana -n monitoring 8080:80
                kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093

