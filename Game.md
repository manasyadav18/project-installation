apt install docker.io
usermod -aG docker $USER # Replace with your username e.g ‘ubuntu’  usermod -aG docker ubuntu
newgrp docker

#We will now setup terraform in the machine 
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

#Now we will setup the AWS CL
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install

#Now we will setup Kubectl 
sudo apt install curl -y
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

#Every thing is setup and installed let’s make a IAM EC2 Role
mkdir supermario
cd supermario
git clone https://github.com/akshu20791/supermario-game
cd supermario-game
cd EKS-TF

#Go to aws -> S3 -> Create a s3 bucket with some unique name
edit the backend.tf file by → vim backend.tf
#Note →make sure to provide your bucket and region name in this file otherwise it doesn’t work and IAM role is also associated with your ec2 which helps ec2 to use other services such S3 bucket
terraform init
terraform validate
## go to AWS ACCOUNT -> GO TO VPC -> GO TO SUBNETS -> DELETE subnet in availability zone us-east-1e
terraform plan 
terraform apply --auto-approve
aws eks update-kubeconfig --name EKS_CLOUD --region us-east-1

#change the directory where deployment and service files are stored use the command → cd ..
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get all
kubectl describe service mario-service





monitoring :

1️⃣ Create Monitoring Folder (Deliverables requirement)

Inside your project:

mkdir -p k8s/monitoring
cd k8s/monitoring

2️⃣ Install Prometheus using Helm (Easiest way)

First install Helm if not installed.

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

Add repo:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

Install Prometheus Stack (includes Grafana + Node Exporter):

helm install monitoring prometheus-community/kube-prometheus-stack

This installs:

Prometheus

Grafana

Node Exporter

Alertmanager

All in one step ✅

3️⃣ Check Pods
kubectl get pods

You should see something like:

monitoring-grafana
monitoring-kube-prometheus-prometheus
monitoring-node-exporter

4️⃣ Expose Grafana (Required in question)

Check service:

kubectl get svc

Find:

monitoring-grafana

Edit service:

kubectl edit svc monitoring-grafana

Change

type: ClusterIP

to

type: NodePort

Save and exit.

5️⃣ Get Grafana NodePort

kubectl get svc monitoring-grafana

Example output:

NodePort: 3000:32000/TCP

So open in browser:

http://<EC2-PUBLIC-IP>:32000
6️⃣ Get Grafana Login Password
kubectl get secret monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

Login:

Username: admin
Password: (command output)
7️⃣ Add Prometheus Data Source

In Grafana:

Settings → Data Sources → Add Data Source → Prometheus

URL:

http://monitoring-kube-prometheus-prometheus:9090

Click:

Save & Test
8️⃣ Import Kubernetes Dashboard

In Grafana:

Dashboards → Import

Dashboard ID:

1860

This gives:

CPU Usage

Memory Usage

Disk Usage

Network I/O

Pod Metrics

Exactly what your acceptance criteria requires.

9️⃣ Verify Prometheus Metrics

Open:

http://<EC2-IP>:NodePort

or port forward:

kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090

Then open:

http://localhost:9090

Search:

node_cpu_seconds_total

If metrics appear ✅ Node Exporter working

🔟 Optional (Alert rule)

Example CPU alert:

groups:
- name: node-alerts
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: High CPU Usage










another method

Step 1 — Go to your Kubernetes server

SSH into the EC2 where Kubernetes is running.

ssh ubuntu@<EC2-PUBLIC-IP>
Step 2 — Install Helm (Kubernetes package manager)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

Check installation:

helm version
Step 3 — Add Prometheus Helm Repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

Update repo:

helm repo update
Step 4 — Install Prometheus + Grafana + Node Exporter

Run this command:

helm install monitoring prometheus-community/kube-prometheus-stack

This single command installs:

Prometheus

Grafana

Node Exporter

Alertmanager

Wait 1–2 minutes.

Step 5 — Check if pods are running
kubectl get pods

You should see pods like:

monitoring-grafana
monitoring-kube-prometheus-prometheus
monitoring-prometheus-node-exporter

If STATUS is Running, everything is correct.

Step 6 — Check Services
kubectl get svc

Find this service:

monitoring-grafana

It will show ClusterIP.

Step 7 — Expose Grafana outside Kubernetes

Edit the service:

kubectl edit svc monitoring-grafana

Find:

type: ClusterIP

Change to:

type: NodePort

Save and exit.

Step 8 — Get Grafana NodePort

Run:

kubectl get svc monitoring-grafana

Example output:

3000:32145/TCP

Here:

32145 = NodePort
Step 9 — Open Grafana in browser

Open:

http://<EC2-PUBLIC-IP>:32145

Example:

http://54.xx.xx.xx:32145

Grafana login page will appear.

Step 10 — Get Grafana Password

Run:

kubectl get secret monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

Login:

Username: admin
Password: (command output)
Step 11 — Open Dashboards

Inside Grafana:

Dashboards → Browse

You will already see dashboards like:

Kubernetes Cluster

Node Exporter

Pod metrics

These show:

CPU usage

Memory usage

Disk usage

Network traffic

Pod metrics

This satisfies your acceptance criteria.

Step 12 — Verify Prometheus

Port forward Prometheus:

kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090

Open browser:

http://localhost:9090

Search metric:

node_cpu_seconds_total

If results appear → Prometheus working correctly.
