## END TO END DEVOPS PROJECT ##

## A Spring Boot app that provides CRUD APIs for managing incidents. 
Step 1:  Features
- Create, Update, Delete incidents
- Track severity (LOW, MEDIUM, HIGH)
- Track status (OPEN, IN_PROGRESS, RESOLVED)
- REST API with JSON responses
- In-memory H2 database for testing
- Actuator endpoints for DevOps monitoring

## Run Locally and access the application. 
mvn spring-boot:run

Endpoints
- `POST /api/incidents` → Create incident
  URL: http://localhost:8080/api/incidents
  Body → raw → JSON:
{
  "title": "Database down",
  "severity": "HIGH",
  "status": "OPEN",
  "assignedTo": "DevOps Engineer"
}
 <img width="1433" height="929" alt="image" src="https://github.com/user-attachments/assets/6464c136-6bf1-41e9-b8e8-d68d3917323b" />

- `GET /api/incidents` → List all
   URL: http://localhost:8080/api/incidents
  <img width="1384" height="881" alt="image" src="https://github.com/user-attachments/assets/bf71dc40-88c0-4047-af6f-595271adca83" />

- `GET /api/incidents/{id}` → Get one
URL: http://localhost:8080/api/incidents/1
  <img width="1408" height="919" alt="image" src="https://github.com/user-attachments/assets/fba753f5-137c-4087-a967-daf9350d36d3" />

- `PUT /api/incidents/{id}` → Update
   URL: http://localhost:8080/api/incidents/1
   Body → JSON:
   {
  "title": "Database down",
  "severity": "HIGH",
  "status": "RESOLVED",
  "assignedTo": "DevOps Engineer"
}
  <img width="1424" height="945" alt="image" src="https://github.com/user-attachments/assets/7ec92c84-7cd5-4436-b51e-4d5fb9933d4c" />

- `DELETE /api/incidents/{id}` → Delete
   URL: http://localhost:8080/api/incidents/1
  <img width="1348" height="915" alt="image" src="https://github.com/user-attachments/assets/f4d28477-fb3a-4122-be34-662bd4746bbb" />

  
## Create AWS Infrastructure with Terraform
•Created a VPC with 2 public subnets (ap-south-1a & ap-south-1b).
•Attached an Internet Gateway (IGW).
•Created Route Table with route to IGW.
<img width="1897" height="820" alt="image" src="https://github.com/user-attachments/assets/33b12efb-2109-4ebe-ba67-dfb898e56620" />

•Launched an EC2 Instance (Ubuntu 22.04) in public subnet.

terraform init
terraform plan
terraform apply -auto-approve

2.Connect to EC2 Instance
ssh -i ec2linux.pem ubuntu@<EC2-Public-IP>

3.Install Docker on EC2
sudo apt update -y
sudo apt install docker.io -y sudo systemctl start docker sudo systemctl enable docker sudo usermod -aG docker ubuntu
# Logout & login again to apply docker group changes.

Note: Refer https://github.com/Tini-j-Mercy/cloudops-dashboard-terraform for tf scripts and infra provisioning using terraform. 

## Dockerize the created application 
Step 1: Create Dockerfile and build it 
In your project root, create a Dockerfile:
Run this inside your project directory:
docker build -t myapp:latest .
myapp:latest → local image name.
<img width="1878" height="366" alt="image" src="https://github.com/user-attachments/assets/8ca70dc0-8ce1-4030-9d26-03c369369ac2" />
docker images  -- to list the available images.

Step 2: Run the Container
docker run -d -p 8080:8080 --name myapp-container myapp:latest
-d → detached mode
-p 8080:8080 → maps container port 8080 to host port 8080
Check if running:
docker ps
<img width="1911" height="976" alt="image" src="https://github.com/user-attachments/assets/69daf146-77b2-4b3c-b58a-37a9c81f69f0" />

Step 3: Access in browser or Postman:
👉 http://localhost:8080
<img width="1384" height="881" alt="image" src="https://github.com/user-attachments/assets/bf71dc40-88c0-4047-af6f-595271adca83" />


## Login to Docker Hub
Step 1: docker login
Enter your Docker Hub username and password.

Step 2: Tag the Image for Docker Hub
docker tag myapp:latest yourdockerhubusername/myapp:latest

Step 3: Push the Image to Docker Hub
docker push yourdockerhubusername/myapp:latest
Now the image is available in Docker Hub.
<img width="1250" height="875" alt="image" src="https://github.com/user-attachments/assets/83297360-e49f-4f12-97db-2ad2ad78df9d" />


## Deploy Dockerized App on EC2 
Step 1: Connect to EC2
From your local machine:
ssh -i terraform-key2.pem ec2-user@<ec2-public-ip>

Step 2: Login to Docker Hub (inside EC2)
docker login
Enter Docker Hub username & password (or access token if enabled).

Step 3: Pull Image from Docker Hub
docker pull yourdockerhubusername/myapp:latest

Step 4: Run Container on EC2
docker run -d -p 8080:8080 --name myapp-container yourdockerhubusername/myapp:latest

Step 6: Allow Traffic via Security Group
When you created EC2 with Terraform, check its Security Group:
Add inbound rule:
Port 8080 → Source = 0.0.0.0/0 (or your IP for security).

Step 7: Access Application
Now you can hit the application in browser/Postman:
http://publicip:8080


##  CREATE ALB and access the application using dns  

Step 1: Security Group Setup

Go to EC2 → Security Groups.

Create 2 SGs:

ALB-SG → allow inbound HTTP (80) from 0.0.0.0/0 (internet).

EC2-SG → allow inbound 8080 only from ALB-SG (not the internet).

Attach EC2-SG to your VM instance.
<img width="1886" height="869" alt="image" src="https://github.com/user-attachments/assets/d037a342-c68d-49d6-8ff1-17127d71ac98" />

Step 2 : Create Target Group
Type: Instance (since you’re directly pointing to EC2).
Protocol: HTTP
Port: 8080
Health check path: /actuator/health
Register Target :   Add your EC2 instance to the Target Group.
<img width="1529" height="478" alt="image" src="https://github.com/user-attachments/assets/9dee819f-578a-4c3e-8cdb-267312c87914" />

Step 3:  Create Application Load Balancer (ALB)
Type: Internet-facing
Listeners:
HTTP :80 → Forward to Target Group
Choose subnets (public subnets across at least 2 AZs).
Attach SG that allows port 80.
<img width="1884" height="807" alt="image" src="https://github.com/user-attachments/assets/6c4150bf-25ed-4fff-82d9-14f1bd3e37e3" />

Note: Terraform script has been created for alb coniguration.
<img width="1908" height="965" alt="image" src="https://github.com/user-attachments/assets/c20b567e-521f-40bf-99b9-44f824005bbf" />

Step 4: Access Application
Now you can hit the application in browser/Postman:
http://<dns-name>/api/incidents
<img width="1403" height="786" alt="image" src="https://github.com/user-attachments/assets/565ed4f6-8518-4413-a72b-dad5c64994a1" />
<img width="1434" height="767" alt="image" src="https://github.com/user-attachments/assets/0e842e0c-65f3-4959-a6f3-d313e98459c7" />

## Create EKS Cluster using Terraform
Step 1: create the EKS cluster 
Terraform scripts were used to create the EKS cluster along with required IAM roles, node groups, and VPC/subnets.
Run the following commands:
terraform init
terraform plan
terraform apply -auto-approve
<img width="1450" height="406" alt="image" src="https://github.com/user-attachments/assets/e2190bed-0f5e-4866-914f-5f3a0ed96381" />

Step 2: Configure kubectl
Install kubectl locally or on EC2 to interact with the EKS cluster.
Update kubeconfig for the cluster:
aws eks --region <region> update-kubeconfig --name <cluster_name>
<img width="1317" height="70" alt="image" src="https://github.com/user-attachments/assets/b7ea6047-d046-4a4d-a333-9f5f0e3bcb0c" />

Step 3: Verify cluster access:
kubectl get nodes
<img width="1093" height="116" alt="image" src="https://github.com/user-attachments/assets/bb4a5801-9d9b-4e00-9583-f68d47f7eb36" />

Step 4: Create Kubernetes Deployment & Service
deployment.yaml → Defines Spring Boot application deployment and container image.
service.yaml → Type LoadBalancer to expose application externally.
Step 5: Apply Deployment & Service
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
Step 6: Check pods and service:
kubectl get pods
kubectl get svc
<img width="1460" height="273" alt="image" src="https://github.com/user-attachments/assets/694dbb88-ff37-4983-969f-acf608a7ff92" />

Step 5: Access the Application
Access the app using the EXTERNAL-IP of the LoadBalancer service:
http://<EXTERNAL-IP>/api/incidents
<img width="1663" height="278" alt="image" src="https://github.com/user-attachments/assets/1384b027-e91c-48f3-950d-ec54441abf4b" />

## CI/CD Pipeline with Jenkins
Step 1: Jenkins Installation
sudo apt update
sudo apt install openjdk-11-jdk -y
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins

Step 2:  Access Jenkins in browser: http://<ec2-public-ip>:8080
Unlock Jenkins using:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Install suggested plugins and create admin user.
<img width="1910" height="961" alt="image" src="https://github.com/user-attachments/assets/98104890-9c80-408b-95ee-877b2377dd55" />

Step 3: Install Required Jenkins Plugins
Go to Manage Jenkins → Manage Plugins → Available
Docker Pipeline → Build and push Docker images
Kubernetes CLI Plugin → Deploy to Kubernetes/EKS
Git Plugin → Pull code from GitHub
Pipeline → For Jenkinsfile pipeline execution
Credentials Binding Plugin → Securely manage passwords & tokens

Step 4: Configure Jenkins Credentials
Go to Manage Jenkins → Credentials → Global
GitHub Credentials → Username & Personal Access Token
Docker Hub Credentials → Username & Password (or access token)
AWS Credentials → Access Key & Secret Key (for EKS access)
These credentials will be used in the pipeline securely.

Step 5: Create Jenkins Pipeline Job
Go to New Item → Pipeline
Enter Job Name: Springboot-EKS-Pipeline
Select Pipeline → OK

Step 6: Connect GitHub Repository
In pipeline config → choose Pipeline script from SCM
SCM: Git
Repo URL: https://github.com/<your-username>/<repo>.git
Branch: main

Step 7: Jenkinsfile https://github.com/Tini-j-Mercy/Incident-management-service/blob/main/Jenkins-pipeline/sample
This controls your CI/CD: 

Step 8: Run Pipeline
Click Build Now
Watch logs → Code → Docker Build → Push → EKS Deployment update.
<img width="1906" height="969" alt="image" src="https://github.com/user-attachments/assets/8742969b-0003-4e13-8bb5-d8de2eb12944" />

Step 9: Verify Deployment
kubectl get deployments
kubectl get pods
<img width="804" height="161" alt="image" src="https://github.com/user-attachments/assets/2f8ec2a2-0c22-4a0b-9762-65dc574a6bbd" />
kubectl describe deployment springboot-app
<img width="1875" height="476" alt="image" src="https://github.com/user-attachments/assets/8bf1a72e-0a85-4c4d-ac5a-d32238933648" />

## 🚀 Prometheus & Grafana Setup on Kubernetes
Step 1: Add Helm Repositories
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

Step 2: Create monitoring Namespace
kubectl create namespace monitoring

Step 3: Install Prometheus using Helm
helm install prometheus prometheus-community/prometheus -n monitoring

✅ This installs:
*Alertmanager
*Prometheus server
*Node Exporter
*Kube State Metrics
*Pushgateway

Step 4:  Verify Prometheus Deployment
Check the Helm release:
helm list -n monitoring
<img width="1697" height="101" alt="image" src="https://github.com/user-attachments/assets/a77e3dfc-e1c6-4783-8e72-fd74e0aa5148" />
Check if pods are running:
kubectl get pods -n monitoring
<img width="1092" height="214" alt="image" src="https://github.com/user-attachments/assets/43e26564-4f5b-4ef2-a13b-ff4a6ba65f93" />

Step 5: Accessing Prometheus UI
Once Prometheus is deployed in the monitoring namespace, you can access the Prometheus UI through the AWS LoadBalancer service.
Verify Prometheus Service
kubectl get svc -n monitoring
<img width="1891" height="338" alt="image" src="https://github.com/user-attachments/assets/dcf1b3ef-3699-47d4-9b8b-60db7daef1b7" />

Step 6: Open Prometheus UI in Browser
Navigate to:
http://<loadbalancer-dns>
<img width="1887" height="916" alt="image" src="https://github.com/user-attachments/assets/bc08de15-6c17-4d73-952f-79c73b742069" />
<img width="1913" height="967" alt="image" src="https://github.com/user-attachments/assets/64369bba-3353-464e-89f3-f8ddcf5c97e6" />

Step 7: 📊 Accessing Grafana UI
Install Grafana
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana \
  --namespace monitoring \
  --create-namespace \
  --set service.type=LoadBalancer \
  --set adminUser=admin \
  --set adminPassword=admin123 \
  --set persistence.enabled=true \
  --set persistence.size=2Gi
  <img width="1510" height="873" alt="image" src="https://github.com/user-attachments/assets/67f883b5-0528-4955-b4fc-24c94b133553" />

Step 8: Verify Service
kubectl get svc -n monitoring
<img width="1897" height="414" alt="image" src="https://github.com/user-attachments/assets/2eba00e5-d9bf-4a97-8df9-321caffa3453" />

Step 9 : Open Grafana in Browser
http://<grafana-loadbalancer-dns>
<img width="1902" height="964" alt="image" src="https://github.com/user-attachments/assets/0d05becb-fd06-4dd3-bb7d-d394eb37c607" />

Login with:
Username: admin
Password: admin123
<img width="1900" height="967" alt="image" src="https://github.com/user-attachments/assets/442c0762-e255-4bce-b415-9f6d1545d999" />

Step 10: Add Prometheus as Data Source
Go to Connections → Data Sources → Add Data Source.
Select Prometheus.
Set URL = http://prometheus-server.monitoring.svc.cluster.local.
Save & Test.
Import Dashboards
Go to Dashboards → Import.
Example Dashboard ID: 1860 (Kubernetes Cluster Monitoring).
Connect it to your Prometheus data source.
<img width="1890" height="961" alt="image" src="https://github.com/user-attachments/assets/f98f1834-1755-499c-9ba2-8db2eb307553" />

Step 7: sanity check
In Grafana, go to Explore → pick Prometheus at top-left → run the query up.
You should see a series per target (node-exporter, kube-state-metrics, etc.).
<img width="1544" height="889" alt="image" src="https://github.com/user-attachments/assets/da5ddc07-4b0c-4c41-ab98-81f0d8de2543" />

Step 8: Import Dashboards
In Grafana → left menu Dashboards → Import.
Paste a dashboard ID from Grafana.com, e.g.:
1860 → Node Exporter Full (great host/node overview)
3662 → Prometheus 2.0 Stats
Click Load.
Choose Prometheus as the data source (the one you added).
Import → dashboard appears with live data.
<img width="1892" height="971" alt="image" src="https://github.com/user-attachments/assets/9b78e272-fcc2-4af6-9259-6553ff2ba611" />
 
 ##  Application-level Monitoring for Spring Boot
The next step is application-level monitoring for your Spring Boot app inside Kubernetes. That means exposing custom app metrics (from your Spring Boot incident-service) and scraping them in Prometheus, then building Grafana dashboards for them.

Here’s what you should do next:

✅ 1. Enable Metrics in Spring Boot (Micrometer + Actuator)

Spring Boot integrates seamlessly with Prometheus using Micrometer.

In your pom.xml, add:
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>


Then, in application.properties:

management.endpoints.web.exposure.include=health,info,prometheus
management.endpoint.prometheus.enabled=true
Now your app exposes metrics at:
👉 http://localhost:8080/actuator/prometheus
<img width="1121" height="961" alt="image" src="https://github.com/user-attachments/assets/28bf01c6-c6a7-4486-9b07-c4d8f48eafc9" />

Step 2: Deploy the updated image (already automated via Jenkins pipeline).

Configure Prometheus to Scrape Spring Boot

Update the Prometheus ConfigMap with a new scrape job:
scrape_configs:
  - job_name: 'springboot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['incident-service.default.svc.cluster.local:8080']
Replace incident-service.default.svc.cluster.local:8080 with your actual service name + namespace or LoadBalancer DNS.

Apply the changes and restart Prometheus:

kubectl apply -f prometheus-config.yaml -n monitoring
kubectl delete pod -l app=prometheus-server -n monitoring

<img width="1240" height="970" alt="image" src="https://github.com/user-attachments/assets/9b0fda94-558b-48d0-be9d-3ecb3b6aaa72" />

Step 3 — Verify Metrics in Prometheus
Forward Prometheus UI 
 → go to Status → Targets.
You should see springboot-app with UP status.
<img width="1412" height="579" alt="image" src="https://github.com/user-attachments/assets/317df1ee-3fb9-4fdf-9801-9ef6d5de3c5f" />
Run a sample query:
http_server_requests_seconds_count
<img width="1903" height="759" alt="image" src="https://github.com/user-attachments/assets/07cf5f58-1c78-4ce8-a915-14c206a61109" />
This confirms Prometheus is scraping your Spring Boot app.

Step 4: Create Custom Dashboard 
Go to Dashboards → New Dashboard → Add Panel
Choose a metric like:
http_server_requests_seconds_count → API requests count
http_server_requests_seconds_sum → Response time
process_cpu_usage → CPU usage
Apply visualization (Graph, Gauge, Table, etc.)
5. Save
Click Save Dashboard
✅ Now you’ll have real-time Spring Boot app metrics visualized in Grafana.
<img width="1739" height="892" alt="image" src="https://github.com/user-attachments/assets/658e7e53-c9f1-4d37-b713-add78c7cc2d7" />
<img width="1909" height="707" alt="image" src="https://github.com/user-attachments/assets/faa0d7a8-2f74-48dc-8232-42074729d32c" />


