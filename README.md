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

**5. Install Jenkins**
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

Jenkins is active and running

