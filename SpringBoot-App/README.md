# Documentation: CI/CD Setup with Jenkins, SonarQube, Docker, MicroK8s, and ArgoCD , And Monitoring with Prometheus/Grafana
## 1. IAM User Configuration - Maybe you can implement
### Provide IAM User access to EC2 with Visual Studio Code.

## 2. Jenkins CI/CD Configuration
### Prerequisites:
- Create an EC2 instance of type t2.medium.
- Allow the following ports: 8080, 8081, 9000, 8000, 80, 443, 22, 9090,3000
{you can custom with your ip instead 0.0.0.0/0}

## 2.1. Jenkins Installation:
### Update the system and install Java:
```
sudo yum update -y
sudo yum install openjdk-17-jre -y
java -version

```
### Install Jenkins:
```
# Add commands to install Git here
    sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    sudo yum upgrade -y
    
	# Add required dependencies for the jenkins package
    sudo yum install -y fontconfig java-17-openjdk
    sudo yum install -y jenkins
    sudo systemctl daemon-reload

```
## 3. Enable and start Jenkins:
```
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

```

## 4. Jenkins Plugin Installation:

- Install the following plugins:
	+ gitlab
	+ gitlab api
	+ sonar scanner
	+ sonar quality gates
	+ quality gates
	+ sonar gerrit
	+ sonarQube Generic Coverage
	+ docker-pipeline

## 5. Jenkins System Configuration:

- Connect Jenkins with GitLab using a token.
- Add SCM configuration.
- Setup webhook for automatic build triggering.

## 6. Maven Installation:
```
sudo yum install maven -y
wget https://dlcdn.apache.org/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz
tar -xvf apache-maven-3.9.6-bin.tar.gz
mv apache-maven-3.9.6-bin.tar.gz maven

rm -rf /usr/share/maven
mv maven /usr/share/maven
```

## 2.2. SonarQube Installation:
Prerequisites and Installation:
```
# 2.2.1. Install OpenJDK 11
$ sudo apt-get install openjdk-11-jdk -y

# 2.2.2. Install and Configure PostgreSQL
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
$ wget -q https://www.postgresql.org/media/keys... -O - | sudo apt-key add -
$ sudo apt install postgresql postgresql-contrib -y
$ sudo systemctl enable postgresql
$ sudo systemctl start postgresql
$ sudo passwd postgres
$ su - postgres
$ createuser sonar
$ psql
ALTER USER sonar WITH ENCRYPTED password 'my_strong_password';
CREATE DATABASE sonarqube OWNER sonar;
GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;
\q
$ exit

# 2.2.3. Download and Install SonarQube
$ sudo apt-get install zip -y
$ sudo wget https://binaries.sonarsource.com/Dist...
$ sudo unzip sonarqube-9.6.1.59531.zip
$ sudo mv sonarqube-9.6.1.59531 sonarqube
$ sudo mv sonarqube /opt/

# 2.2.4. Add SonarQube Group and User
$ sudo groupadd sonar
$ sudo useradd -d /opt/sonarqube -g sonar sonar
$ sudo chown sonar:sonar /opt/sonarqube -R

# 2.2.5. Configure SonarQube
$ sudo vi /opt/sonarqube/conf/sonar.properties
# Update sonar.jdbc.username, sonar.jdbc.password, sonar.jdbc.url
$ sudo vi /opt/sonarqube/bin/linux-x86-64/sonar.sh
# Update RUN_AS_USER=sonar

# 2.2.6. Setup Systemd service
$ sudo vi /etc/systemd/system/sonar.service
# Paste service configuration
$ sudo systemctl enable sonar
$ sudo systemctl start sonar
$ sudo systemctl status sonar

# 2.2.7. Modify Kernel System Limits
$ sudo vi /etc/sysctl.conf
# Add vm.max_map_count=262144 and fs.file-max=65536
$ sudo reboot

# 2.2.8. Access SonarQube Web Interface
Access SonarQube in a web browser at your server's IP address on port 9000.
```
2.3. Docker Installation:
Install Docker:
```
# Removing Previously installed docker Dependancies
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine

# Adding Docker Repo
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Installing Docker Engine
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
- Docker Post Installation:
```
sudo su -
usermod -aG docker admin
<!-- usermod -aG docker ubuntu -->
systemctl restart docker

sudo chmod 777 /var/run/docker.sock
```
- Add your application to GitLab.

## 2.4. Jenkins CI/CD Pipeline:
- SonarQube Integration:

- On SonarQube, access via ip:9000, and create a Token.
- In Jenkins, configure SonarQube servers.
- Create a webhook on SonarQube.
- Jenkins Job Configuration:

- Set discard policy to retain only the 3 latest builds and builds younger than 3 days.
- Specify the path to your Jenkinsfile.
- Test the Build:
```
cd java-maven-sonar-argocd-helm-k8s/spring-boot-app
docker build -t hoangguruu/complete-ci-cd:1 .
docker run -d -p 8000:8080 -t hoangguruu/complete-ci-cd:1
```
## 2.5. Jenkinsfile Configuration:
The Jenkinsfile provided in the documentation outlines various stages of the CI/CD pipeline, including checking out the code, building and testing, static code analysis using SonarQube, and more.
## 3. MicroK8s Installation:

```
sudo snap install microk8s --classic
sudo ufw allow in on cni0 && sudo ufw allow out on cni0
sudo ufw default allow routed
sudo usermod -a -G microk8s ubuntu
newgrp microk8s
```
## 3.1. MicroK8s Add-ons:

```
microk8s enable dns 
microk8s enable dashboard
microk8s enable storage
microk8s kubectl get all --all-namespaces
```
## 4. ArgoCD Installation:
Detailed steps are provided for the installation of ArgoCD and its integration with MicroK8s.

- Install ArgoCD by MicroK8s

```
microk8s.kubectl create namespace argocd
microk8s.kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

microk8s.kubectl get all -n argocd

microk8s.kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
microk8s.kubectl get all -n argocd
```
	+ Check with machine_ip:{port}
	+ Login : 
```
microk8s.kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
- Connect Repo
	+ creat Token
	+ Create Application
	
	+ Remember Path
	
	+ And check
```
microk8s.kubectl get po,svc
microk8s.kubectl get pods
```

## 5. Monitoring with Prometheus and Grafana:
The document concludes with steps to install Prometheus and Grafana for monitoring purposes.

## 5.1 Prometheus
```
microk8s enable helm3

```
```
microk8s.helm3 repo add prometheus-community https://prometheus-community.github.io/helm-charts
microk8s.helm3 repo update

microk8s.helm3 install prometheus prometheus-community/prometheus
microk8s.kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext

microk8s.kubectl get pods
microk8s.kubectl get svc

```
- Check mymachineip:{port}
## 5.2 Grafana
```
microk8s.helm3 repo add grafana https://grafana.github.io/helm-charts
microk8s.helm3 repo update
microk8s.helm3 install grafana grafana/grafana

microk8s.kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo


microk8s.kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext

microk8s.kubectl get svc
```
- Check mymachineip:{port}


` Need Elastic Ip`