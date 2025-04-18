# React-.NET-Application-Deployment
Deploy a React frontend and .NET backend application on AWS (or any cloud platform) using Terraform (or CloudFormation), Kubernetes, and a fully automated CI/CD pipeline.

## Step 1: Infrastructure Setup with Terraform
Location: terraform/

Commands:
Open the Terraform Directory:
```bash
cd terraform
```
A)Initializes a terraform working directory:
```bash
terraform init
```
B)Validate configuration files syntax:
```bash
terraform --validate
```
C)Check for resource creation
```bash
terraform plan 
```

D)Create:
```bash
terraform apply
```
This will provision EKS cluster, IAM roles, and VPC.

## Step 2 :GitHub Webhook Setup to Trigger Jenkins Pipeline

Configure Jenkins Job:

1.Go to the Jenkins pipeline job.

2.Check "GitHub project" and enter the GitHub repo URL.

3.Under "Build Triggers", select: "GitHub hook trigger for GITScm polling"

4.Add Webhook in GitHub Repo
Go to the GitHub repo → Settings → Webhooks → Add webhook
Set: Payload URL:http://jenkins-url>/github-webhook/

5.Content type: application/json and Click Add webhook


## Step 3: Build and Test Backend, Containerize, Push to ECR


Jenkins pipeline ensures:

1.Run .NET backend tests using dotnet test

2.Build Docker images for frontend and backend

3.Scan images with Trivy

4.Push to ECR


Deploy with:
```bash
kubectl apply -f k8s/deployments/backend-deployment.yaml
kubectl apply -f k8s/deployments/frontend-deployment.yaml
kubectl apply -f k8s/services/backend-service.yaml
kubectl apply -f k8s/deployments/frontend-service.yaml
```


Apply HPA:

Check for metrics-server is installed :
```bash
kubectl get deployment metrics-server -n kube-system
```
If not installed:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Now apply your HPA:
```bash
kubectl apply -f k8s/hpa.yaml
```

## Steps 4: Monitoring and Logging Setup

To monitor the application and cluster:

- Prometheus for metrics scraping.
- Grafana for visualization.
- Fluent Bit(Fluentd lightweight version) for log forwarding to AWS CloudWatch.
- Monitored parameters include: CPU usage, memory usage, and pod failures.

---

1. Install Prometheus using Helm:

Prometheus is installed using Helm. Add the Helm repo and install Prometheus:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus \
  --namespace monitoring --monitoring
```

Check if Prometheus is running:
```bash
kubectl get pods -n monitoring
```
Access Prometheus dashboard:
```bash
kubectl port-forward svc/prometheus-server -n monitoring 9090:80
```
Open in browser: http://localhost:9090

2. Install Grafana using Helm

Add and Install Grafana via Helm in the same monitoring namespace:
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install grafana grafana/grafana \
  --namespace monitoring --monitoring \
  --set adminPassword='admin' \
  --set service.type=ClusterIP
```
Access Grafana UI:
```bash
kubectl port-forward svc/grafana -n monitoring 3000:80
```
Open in browser: http://localhost:3000

Username: admin
Password: kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 -d

After logging in:

Add Prometheus as a data source: http://prometheus-server.monitoring.svc.cluster.local

Import dashboards for Kubernetes, nodes, and pods.

3. Install Fluent Bit to forward logs to CloudWatch

Add Fluent Bit Helm repo:
```bash
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update
```
2.Install Fluent Bit using the above config:
```
helm install fluent-bit fluent/fluent-bit \
  --namespace logging --create-namespace \
  -f k8s/logging/fluent-bit/values.yaml
```
3.Confirm logs are visible in your CloudWatch Log Group.
```bash
kubectl logs <fluent-bit-pod> -n logging
```



## 5. Network Policies

To secure communication between pods, we have applied a basic Kubernetes NetworkPolicy.
Network policy setup requires a CNI plugin like Calico.

To check your plugin:

```bash
kubectl get pods -n kube-system
```
If Not Installed:
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

:Policy Objective

- Block all incoming traffic to `backend` pods by default.
- Only allow traffic from pods with label `app: frontend`.
