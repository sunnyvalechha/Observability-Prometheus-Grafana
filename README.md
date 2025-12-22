Monitoring with Prometheus & Grafana

Aws: 
* Ubuntu t2.medium

Download: https://prometheus.io/download/
* wget https://github.com/prometheus/prometheus/releases/download/v3.8.1/prometheus-3.8.1.linux-amd64.tar.gz
* mv prometheus-3.8.1.linux-amd64 prometheus	# rename

# Run prometheus as background process
* nohup ./prometheus > prometheus.log 2>&1 &	# run prometheus
* ps -ef | grep prometheus
* http://35.154.92.2:9090/

# Prometheus exporters
* A Prometheus exporter is a dedicated agent or service that collects metrics from external systems (like databases, servers, or applications).

# Prometheus node exporters # download from same page

* wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
* tar -xvf /root/download/node_exporter-1.10.2.linux-amd64.tar.gz
* mv node_exporter-1.10.2.linux-amd64 node_exporter
* cd /root/node-exporter/node_exporter
* nohup ./node_exporter > node_porter.log 2>&1 &
* Add below lines in "vi /root/prometheus-server/prometheus/prometheus.yml"

- job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
	  
* kill the prometheus process and run prometheus again, refresh the UI & search for "node".
* test any query related to "node_" like (cpu, memory)


# Add targets (hosts) in prometheus to monitor
* Add any small instance
* setup node exporter as did earlier.
* vi /root/prometheus-server/prometheus/prometheus.yml	# add new node ip with port number
* kill the prometheus process and run again, go to targets to validate.

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100","13.201.20.17:9100"]
	  
* Run query you can there are 2 targets --> node_cpu_seconds_total

# Reload service

- if we run this in a UI browser ==> "http://13.126.203.246:9090/-/reload"		# Only POST or PUT requests allowed
- curl -s -XPOST localhost:9090/-/reload
- ./prometheus -h		# look for -> enable shutdown & reload via http request
- nohup ./prometheus --web.enable-lifecycle > prometheus.txt 2>&1 &
- Check "Last successful configuration reload" in Status

# Mysql exporter

* apt install mysql-server -y
* systemctl status mysql
* download a different exporter for mysql on a different node.
* wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.18.0/mysqld_exporter-0.18.0.linux-amd64.tar.gz
* mkdir mysqld; cd mysqld
* tar xvf /root/downloads/mysqld_exporter-0.18.0.linux-amd64.tar.gz
* mv mysqld_exporter-0.18.0.linux-amd64 mysqld
* mv mysqld/mysqld_exporter /usr/local/bin/
* chmod +x /usr/local/bin/mysqld_exporter
* mysql	# login

- Create Prometheus Exporter Database User to Access the Database

CREATE USER 'mysqld_exporter'@'13.235.18.123:9000' IDENTIFIED BY 'StrongPass-123';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysqld_exporter'@'13.235.18.123:9000'; 
FLUSH PRIVILEGES; 
EXIT

# Add Prometheus system user and group

sudo groupadd --system prometheus 
sudo useradd -s /sbin/nologin --system -g prometheus prometheus


Configure the Database Credentials

sudo vi /etc/.mysqld_exporter.cnf
 
# Add correct username and password for user create 
[client] 
user=mysqld_exporter 
password=StrongPass-123
 
# Set ownership permissions: 
sudo chown root:prometheus /etc/.mysqld_exporter.cnf


Create systemd Unit File

sudo vim /etc/systemd/system/mysql_exporter.service
 
#Add the following content
[Unit]
	Description=Prometheus MySQL Exporter
	After=network.target
	User=prometheus
	Group=prometheus
 
	[Service]
	Type=simple
	Restart=always
	ExecStart=/usr/local/bin/mysqld_exporter \
	--config.my-cnf /etc/.mysqld_exporter.cnf \
	--collect.global_status \
	--collect.info_schema.innodb_metrics \
	--collect.auto_increment.columns \
	--collect.info_schema.processlist \
	--collect.binlog_size \
	--collect.info_schema.tablestats \
	--collect.global_variables \
	--collect.info_schema.query_response_time \
	--collect.info_schema.userstats \
	--collect.info_schema.tables \
	--collect.perf_schema.tablelocks \
	--collect.perf_schema.file_events \
	--collect.perf_schema.eventswaits \
	--collect.perf_schema.indexiowaits \
	--collect.perf_schema.tableiowaits \
	--collect.slave_status \
	--web.listen-address=0.0.0.0:9104
	
	[Install]
	WantedBy=multi-user.target


Reload systemd and start mysql_exporter service

$sudo systemctl daemon-reload
$sudo systemctl enable mysql_exporter
$sudo systemctl start mysql_exporter


# Configure MySQL Endpoint to be Scraped by Prometheus

vi prometheus.yml

scrape_configs: 
- job_name: mysql_server1
    static_configs:
    - targets: ['3.110.28.134:9104']

curl -s -XPOST localhost:9090/-/reload









==================================================

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

