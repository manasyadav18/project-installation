SONARQUBE INSTALLATION:
sudo apt update
sudo apt upgrade -y
sudo apt install -y openjdk-21-jdk
java -version
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" /etc/apt/sources.list.d/pgdg.list'
sudo wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable postgresql
sudo systemctl start postgresql
sudo systemctl status postgresql
psql --version

sudo -i -u postgres
createuser ddsonar
psql
ALTER USER ddsonar WITH ENCRYPTED password 'ddsonar';
CREATE DATABASE ddsonarqube OWNER ddsonar;
GRANT ALL PRIVILEGES ON DATABASE ddsonarqube to ddsonar;
\l
\du
\q
exit
sudo apt install zip -y
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-26.2.0.119303.zip
sudo unzip sonarqube-26.2.0.119303.zip 
sudo mv sonarqube-26.2.0.119303 sonarqube
sudo mv sonarqube /opt/
sudo groupadd ddsonar
sudo useradd -d /opt/sonarqube -g ddsonar ddsonar
sudo chown ddsonar:ddsonar /opt/sonarqube -R
sudo nano /opt/sonarqube/conf/sonar.properties
	#sonar.jdbc.username=ddsonar //remove #
	#sonar.jdbc.password=Admin@123 //remove#
	sonar.jdbc.url=jdbc:postgresql://localhost:5432/ddsonarqube //add this line
sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
	RUN_AS_USER=ddsonar // add in first line
sudo 	nano /etc/systemd/system/sonar.service
	[Unit]
	Description=SonarQube service
	After=syslog.target network.target
	[Service]
	Type=forking
	ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
	ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
	User=ddsonar
	Group=ddsonar
	Restart=always
	LimitNOFILE=65536
	LimitNPROC=4096
	[Install]
	WantedBy=multi-user.target
sudo systemctl enable sonar
sudo systemctl start sonar
sudo systemctl status sonar //Esc+:wq to exit
sudo nano /etc/sysctl.conf
	vm.max_map_count=262144
	fs.file-max=65536
	ulimit -n 65536
	ulimit -u 4096
sudo reboot

sonar-token
squ_167a0e15f0696c19b329307e1b78a5949d6fd2db

JENKINS INSTALLATION:
sudo apt update -y
#sudo apt install openjdk-17-jdk -y
sudo apt install fontconfig openjdk-21-jre
sudo apt update -y
sudo apt install maven -y
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

TOMCAT INSTALLATION:
sudo apt update
sudo apt install zip
sudo wget link of tomcat
sudo chmod 755 /home
sudo chmod 755 /home/ubuntu
sudo chown -R jenkins:jenkins /home/ubuntu/apache-tomcat-9.0.115/webapps/
cd apache-tomcat-9.0.115
cd bin
chmod 700 *.sh
./startup.sh

AWSCLI INSTALLATION:
sudo apt update
sudo apt install -y curl unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

DOCKER INSTALLAION:


K8s INSTALLATION EKS:
sudo apt update
sudo apt install -y curl unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws configure
create a IAM user and paste the access key and secret key here
aws eks --region example_region update-kubeconfig --name cluster_name
sudo apt update
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
sudo curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | sudo tar xz -C /usr/local/bin
eksctl version
kubectl get pods

If you get the error: 
Go to eks cluster -> compute -> click on node -> click on instance → security -> click on securty group -> edit inbound rule -> delete existing rule and allow all traffic 

vi pod1.yml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: c00
      image: ubuntu
      command: ["/bin/bash", "-c","while true; do echo hello world; sleep 10; done"]


ANSIBLE INSTALLATION:


TERRAFORM INSTALLATION:

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/Msocial123/EcommerceApp.git'
            }
        }
        stage('Check Project Structure') {
            steps {
                sh 'ls -la'
            }
        }
        stage('Compile') {
            steps {
                dir('EcommerceApp') {  
                    sh 'mvn clean compile'
                }
            }
        } 
        stage('Test') {
            steps {
                dir('EcommerceApp') {
                    sh 'mvn test'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir('EcommerceApp') {
                    withSonarQubeEnv('sonar-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage('Package') {
            steps {
                dir('EcommerceApp') {
                    sh 'mvn package'
                }
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                dir('EcommerceApp') {
                    sh 'mv /var/lib/jenkins/workspace/project/EcommerceApp/target/EcommerceApp.war /home/ubuntu/apache-tomcat-9.0.115/webapps/'
                }
            }
        }
    }
	post {
        success {
            emailext(
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
<h2>Build Successful ✅</h2>
<p><b>Job Name:</b> ${env.JOB_NAME}</p>
<p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
<p><b>Build URL:</b> <a href="${env.BUILD_URL}">
${env.BUILD_URL}</a></p>
""",
                to: "manasyadav940@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
<h2>Build Failed ❌</h2>
<p><b>Job Name:</b> ${env.JOB_NAME}</p>
<p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
<p>Check Console Output:</p>
<p><a href="${env.BUILD_URL}">
${env.BUILD_URL}</a></p>
""",
                to: "manasyadav940@gmail.com"
            )
        }
    }
}

