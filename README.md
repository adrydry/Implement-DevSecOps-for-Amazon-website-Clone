# Implement-DevSecOps-for-Amazon-website-Clone
In this project, we will implement an End to End DevSecOps Project using many tools like Github, Jenkins, Sonarqube, Npm, Owasp, Docker, Kubernetes and Helm


**1. Connect to our Aws Account and Create our security Group**
For our website to be accessible, we need to enable the ports below:

<img width="1646" height="457" alt="image" src="https://github.com/user-attachments/assets/cfa35f4e-11de-496c-883c-d87b8d5419c7" />


The security group is created
<img width="1422" height="277" alt="image" src="https://github.com/user-attachments/assets/467fb0ce-f10e-45b9-b9bf-68094fb7b122" />

**2. Create our Ec2 instance**
<img width="1352" height="203" alt="image" src="https://github.com/user-attachments/assets/79f0153a-bd48-4270-ab5f-4e305c57aabc" />

We will install all the tools in this same server. That's why we decide to go with the t.2 large instance type so that the server will support the workloads.

**3. Update the system and install common packages**
- sudo apt update
- sudo apt upgrade -y
  Install Git:
- sudo apt update
- sudo apt install git -y
  
  <img width="672" height="207" alt="image" src="https://github.com/user-attachments/assets/c050ccfe-185e-4285-b2bc-d9532288fc89" />

  git version 2.43.0 is installed

**4. Install Java**
Since the system is already updated, let's install Java
- OpenJDK 17
sudo apt install -y openjdk-17-jdk

<img width="877" height="140" alt="image" src="https://github.com/user-attachments/assets/4ee723d7-2bfa-4647-be6b-17a5f81bb6af" />

**5. Install and Unlock Jenkins**
Go to https://www.jenkins.io/doc/book/installing/linux/#debianubuntu and copy the appropiate command

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

Enable and start Jenkins. Check Jenkins status
sudo systemctl enable --now jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

<img width="888" height="258" alt="image" src="https://github.com/user-attachments/assets/21b44b81-adbc-4b4e-8cbe-4fbbb3f74769" />

Jenkins is active and running and can be accessed through internet with ip adress:8080

<img width="1073" height="367" alt="image" src="https://github.com/user-attachments/assets/56df4595-2ad5-452d-8b0b-43ae40fa2849" />  

To unlock Jenkins, use: sudo cat /var/lib/jenkins/secrets/initialAdminPassword

<img width="815" height="152" alt="image" src="https://github.com/user-attachments/assets/d2bef3e7-53da-4031-a41a-6e1ea96bd4c0" />

**6. Install Docker and set up**

Go to https://docs.docker.com/engine/install/ubuntu/ and download the appropriate coomand:
a- Add Docker's official GPG key:
       sudo apt-get update
       sudo apt-get install ca-certificates curl
       sudo install -m 0755 -d /etc/apt/keyrings
       sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
       sudo chmod a+r /etc/apt/keyrings/docker.asc

 b- Add the repository to Apt sources:
       echo \
            "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
          $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
          sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
          sudo apt-get update

<img width="922" height="157" alt="image" src="https://github.com/user-attachments/assets/f5e51645-b434-4c70-b6e7-071b5cae0940" />

Docker is up and running

c- Add user to the Docker group
We need to allow to the user to access Docker. To add the user to the Docker Group, use the commands below: 

sudo usermod -aG docker $USER
newgrp docker (To refresh the docker group)

d- Check the status of the Docker: sudo systemctl status docker

<img width="871" height="202" alt="image" src="https://github.com/user-attachments/assets/87d4316e-5153-4919-a12e-0f53fd7146ca" />

**7- Install Trivy**

Docs: https://trivy.dev/v0.65/getting-started/installation/

sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy

trivy --version

<img width="572" height="112" alt="image" src="https://github.com/user-attachments/assets/c8d8b122-d0c1-4e2b-aa3f-aa49d6cb3055" />

Trivy is installed

**8- Prometheus**

Official downloads: https://prometheus.io/download/

Generic install steps:

a- Create a prometheus user
sudo useradd --system --no-create-home --shell /usr/sbin/nologin prometheus

wget -O prometheus.tar.gz "https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-amd64.tar.gz"
tar -xvf prometheus.tar.gz
cd prometheus-*/

sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml

sudo chown -R prometheus:prometheus /etc/prometheus /data

<img width="698" height="162" alt="image" src="https://github.com/user-attachments/assets/e18157b4-a605-45a9-bf9b-0ef378028905" />

Prometheus is installed

b- Create Prometheus Service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090

[Install]
WantedBy=multi-user.target

c- Enable and Start the service

sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus

<img width="708" height="138" alt="image" src="https://github.com/user-attachments/assets/cd75bc60-0e63-4269-a3e0-5e58d6981093" />

Let's veify if we can access Prometheurs through internet with the port 9090

<img width="702" height="267" alt="image" src="https://github.com/user-attachments/assets/7daaa244-413c-4535-ac93-59924fed68d8" />

Prometheus is up and running as we can see on the Target Health

<img width="1805" height="175" alt="image" src="https://github.com/user-attachments/assets/35e403f3-c8d5-4fd8-93c0-9b575c53ac70" />

**9- Node Exporter**

Docs: https://prometheus.io/docs/guides/node-exporter/

sudo useradd --system --no-create-home --shell /usr/sbin/nologin node_exporter

wget -O node_exporter.tar.gz "https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz"
tar -xvf node_exporter.tar.gz
sudo mv node_exporter-*/node_exporter /usr/local/bin/
rm -rf node_exporter*

<img width="672" height="207" alt="image" src="https://github.com/user-attachments/assets/205f64db-f5d1-4b45-8c76-6d226e1cc48a" />

Create Systemd service: (/etc/systemd/system/node_exporter.service)

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target

Enable & start:
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter

<img width="892" height="155" alt="image" src="https://github.com/user-attachments/assets/fa7c46c7-6774-4b50-9b3a-494835c1778a" />



**10- Add Node_Exporter and Jenkins to the Prometheus Yaml file**

Prometheus scrape config:

Add to /etc/prometheus/prometheus.yml:

  - job_name: "node_exporter"
    static_configs:
      - targets: ["<ip-address>:9100"]

  - job_name: "jenkins"
    metrics_path: /prometheus
    static_configs:
      - targets: ["<jenkins-ip>:8080"]

<img width="582" height="241" alt="image" src="https://github.com/user-attachments/assets/8c2bc3f3-9f85-41eb-9f87-6b20d1002c54" />

Now, we have to make sure that our configuration is correctValidate config by using : promtool check config /etc/prometheus/prometheus.yml

<img width="820" height="100" alt="image" src="https://github.com/user-attachments/assets/2a47794e-a3d5-487d-9749-a160b95ef27f" />

Now let's restart Prometheus with : sudo systemctl restart prometheus. After refreshing our Prometheus dashboard, we noticed that Node Exporter is a new metric

<img width="1815" height="170" alt="image" src="https://github.com/user-attachments/assets/3ace15e5-c3c2-4364-99d7-11c1a9acc682" />


**11- Grafana set up**
Docs: https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/

sudo apt-get install -y apt-transport-https software-properties-common wget

sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt-get update
sudo apt-get install -y grafana

sudo systemctl daemon-reload
sudo systemctl enable --now grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server

Grafana is up and running
<img width="940" height="138" alt="image" src="https://github.com/user-attachments/assets/ad956157-7696-433d-bb2a-a95dfc6a9b17" />

Let's see if we can access Grafana from internet with the port 3000

<img width="623" height="372" alt="image" src="https://github.com/user-attachments/assets/963a8d15-e714-47e0-b2b5-20936e7d45a3" />

**12 - Grafana sign in and connection to the data**

When you log in the first time, Grafana uses the defaults:Username: admin, Password: admin

<img width="827" height="272" alt="image" src="https://github.com/user-attachments/assets/104a3154-fd08-43ed-9f04-5864c1cde3f0" />

Now we need to add data. Click on data Sources and select Prometheus, insert the prometheus ip adress and save

<img width="597" height="340" alt="image" src="https://github.com/user-attachments/assets/2ba1b974-b197-4d78-9294-687f447253df" />

<img width="606" height="116" alt="image" src="https://github.com/user-attachments/assets/bd5e2805-72f9-40bc-85b5-c4ae4f0c1b78" />

Go to Grafana dashboard, click on new import and look for the id.
*Dashboard id*
Node_Exporter 1860 Docs: https://grafana.com/grafana/dashboards/1860-node-exporter-full/
jenkins 9964 Docs: https://grafana.com/grafana/dashboards/9964-jenkins-performance-and-health-overview/
kubernetes 18283 Docs: https://grafana.com/grafana/dashboards/18283-kubernetes-dashboard/

<img width="781" height="130" alt="image" src="https://github.com/user-attachments/assets/abef91a5-f6c0-4b0b-bf17-c406e0f51b0b" />

Choose Prometheus and click on import
<img width="781" height="130" alt="image" src="https://github.com/user-attachments/assets/9eedcaca-00cc-4629-a2b9-dc459bb1c955" />


<img width="1462" height="248" alt="image" src="https://github.com/user-attachments/assets/9cf44831-c533-49fe-8ce8-6c671a769199" />


Since, we don't have any job niw in our Jenkins Server, the metrics are not yet applicable. Also the Prometheus metrics plugins are not yet installed

 <img width="1422" height="307" alt="image" src="https://github.com/user-attachments/assets/d28a71c1-65a0-42ea-b176-ba4a24432dbf" />

**13- Jenkins Plugins to install**
For this project, we need to install all the plugins below:

Eclipse Temurin installer Plugin
NodeJS
Email Extension Plugin
OWASP Dependency-Check Plugin
Pipeline: Stage View Plugin
SonarQube Scanner for Jenkins
Prometheus metrics plugin
Docker API Plugin
Docker Commons Plugin
Docker Pipeline
Docker plugin
docker-build-step

After installing all those plugins, select restart Jenkins. Jenkins will restart automatically and there is a change in Grafana Dashboard

<img width="1211" height="271" alt="image" src="https://github.com/user-attachments/assets/d5eddc8b-414c-491e-8dec-e1cb5dc99d63" />


**14- SonarQube Docker Container Run for Analysis**
docker run -d --name sonarqube \
  -p 9000:9000 \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_logs:/opt/sonarqube/logs \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  sonarqube:lts-community

Docker images is created

<img width="802" height="123" alt="image" src="https://github.com/user-attachments/assets/51db56d4-9d8d-4cb3-82e2-f9c0a5828483" />

Let's access Sonarqube through internet with port 9000

<img width="525" height="207" alt="image" src="https://github.com/user-attachments/assets/e8d0196b-e42b-4e04-87e0-4d3343346378" />


**15- Jenkins Credentials to Store**

Purpose     	    ID	               Type                                	Notes

Email     	   mail-cred	          Username/app password	

SonarQube	     sonar-token	        Secret text	                           From SonarQube application

Docker Hub	   docker-cred	        Secret text	                           From your Docker Hub profile

Webhook example:
http://<jenkins-ip>:8080/sonarqube-webhook/

For mail cred, Go to gmail account, click on app password and create password for specific applications

<img width="630" height="182" alt="image" src="https://github.com/user-attachments/assets/6c5422ed-d3e3-4ccc-933e-7e4730d8b76d" />

<img width="752" height="175" alt="image" src="https://github.com/user-attachments/assets/08d55f5b-208d-4894-a807-0860435ff799" />

For Sonar token, go to sonarqube, click on administration, security and users

<img width="1087" height="161" alt="image" src="https://github.com/user-attachments/assets/73147359-fdf1-429e-94bc-8ccea000838a" />

Let's create our webhook:

On Sonarqube, click on configuration and webhook

<img width="297" height="262" alt="image" src="https://github.com/user-attachments/assets/00084b7e-68a4-4883-9b67-7a43a2a58581" />
 The Sonarqube Jenkins Webhook is created
<img width="1081" height="128" alt="image" src="https://github.com/user-attachments/assets/b893ab27-e7f9-4477-b1ed-effab16f0099" />

For the secret text of Docker Hub: Go to dockerhub.com
