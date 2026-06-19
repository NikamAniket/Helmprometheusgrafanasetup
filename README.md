**#Monitoring Setup on GKE using Prometheus & Grafana
Objective**


**Deploy Prometheus and Grafana on a Kubernetes cluster running on Google Kubernetes Engine (GKE) and access Grafana through a public LoadBalancer.**
Prerequisites
Google Cloud Account
GKE Cluster Created
kubectl Installed
Helm Installed
Access to Google Cloud Shell





**Step 1: Connect to GKE Cluster**

Open Google Cloud Shell and connect to your Kubernetes cluster.

Verify cluster connectivity:

kubectl get nodes

Expected Output:

NAME                                          STATUS   ROLES
gke-cluster-default-pool-xxxx                 Ready    <none>

This confirms that Cloud Shell is connected to the cluster.

**Step 2: Create Monitoring Namespace**

Create a dedicated namespace for monitoring components.

kubectl create namespace monitor

Verify:

kubectl get namespaces

Expected Output:

NAME              STATUS
default           Active
kube-system       Active
monitor           Active

Why?

Namespaces help logically separate monitoring resources from application workloads.

**Step 3: Add Prometheus Helm Repository**
Add the official Prometheus Community Helm repository.

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

Update Helm repositories:

helm repo update

Why?

Helm repositories contain ready-to-use Kubernetes application packages called Charts.

**Step 4: Install Prometheus & Grafana**
Deploy the kube-prometheus-stack chart.

helm install kube-prometheus-stack \
prometheus-community/kube-prometheus-stack \
--namespace monitor

This chart installs:

Prometheus
Grafana
AlertManager
Node Exporter
Kubernetes Monitoring Components

Check installation status:

kubectl get pods -n monitor

Expected Output:

NAME                                                     READY
kube-prometheus-stack-grafana                            3/3
kube-prometheus-stack-prometheus                         2/2
kube-prometheus-stack-alertmanager                       2/2

Wait until all pods show Running status.

**Step 5: Verify Resources**

Check Deployments:

kubectl get deployments -n monitor

Check Services:

kubectl get svc -n monitor

Check Pods:

kubectl get pods -n monitor

These commands help verify that all monitoring components were deployed successfully.

**Step 6: Expose Grafana Using LoadBalancer**

By default Grafana is accessible only inside the cluster.

To make it accessible from the internet:

Method 1 (Recommended)

Patch Grafana service:

kubectl patch svc kube-prometheus-stack-grafana \
-n monitor \
-p '{"spec":{"type":"LoadBalancer"}}'

Verify:

kubectl get svc -n monitor

Example Output:

NAME                             TYPE           EXTERNAL-IP
kube-prometheus-stack-grafana    LoadBalancer   34.xx.xx.xx

Wait 2-5 minutes for the external IP to be assigned.

Method 2 (Using GCP Console)
Open Google Kubernetes Engine
Click Clusters
Select Your Cluster
Click Workloads
Select Grafana Deployment
Click Actions
Choose Expose
Port: 3000
Service Type: LoadBalancer
Save

Google Cloud will create a public Load Balancer.

**Step 7: Retrieve Grafana Admin Password**

Get the default Grafana password.

kubectl --namespace monitor get secrets kube-prometheus-stack-grafana \
-o jsonpath="{.data.admin-password}" | base64 -d

Example Output:

admin123456

Default Username:

admin

Save the password securely.

**Step 8: Get Grafana External IP**

Check services:

kubectl get svc -n monitor

Example:

NAME                             TYPE           EXTERNAL-IP
kube-prometheus-stack-grafana    LoadBalancer   34.100.xx.xx

Copy the External IP.

**Step 9: Access Grafana**

Open Browser:

http://<EXTERNAL-IP>:3000

Example:

http://34.100.xx.xx:3000

Login Credentials:

Username:

admin

Password:

<Password Retrieved Earlier>
Step 10: Verify Prometheus Integration

After login:

Go to Connections → Data Sources
Select Prometheus
Verify Status = Healthy

Prometheus is automatically configured by kube-prometheus-stack.

**Useful Commands**

View Pods:

kubectl get pods -n monitor

View Services:

kubectl get svc -n monitor

View Deployments:

kubectl get deployments -n monitor

View Monitoring Namespace:

kubectl get all -n monitor

Delete Monitoring Stack:

helm uninstall kube-prometheus-stack -n monitor

Delete Namespace:

kubectl delete namespace monitor

**Architecture**

Internet User
|
|
LoadBalancer
|
|
Grafana
|
|
Prometheus
|
|
Kubernetes Cluster
|
|
Application Pods

Prometheus collects metrics from Kubernetes resources and Grafana visualizes those metrics through dashboards.
