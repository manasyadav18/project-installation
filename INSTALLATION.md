INSTALL JENKINS:
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
cat /var/lib/jenkins/secrets/initialAdminPassword

NOW INSTALL TOMCAT:
cd /opt
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.115/bin/apache-tomcat-9.0.115.zip
apt install unzip -y
unzip apache-tomcat-9.0.115.zip
mv apache-tomcat-9.0.115 tomcat
sudo chown -R ubuntu:ubuntu /opt/tomcat
cd /opt/tomcat/bin
chmod +x *.sh
chmod +x startup.sh
./startup.sh

CHANGE THE PORT:
cd /opt/tomcat/conf
nano server.xml
<Connector port="8081" protocol="org.apache.coyote.http11.Http11NioProtocol"

INSTALL SONARQUBE:9000
sudo su
sudo apt install -y openjdk-17-jdk
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" /etc/apt/sources.list.d/pgdg.list'
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add –
sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable postgresql
sudo systemctl start postgresql
sudo systemctl status postgresql
sudo -i -u postgres
createuser ddsonar
psql
ALTER USER ddsonar WITH ENCRYPTED password 'ddsonar';
CREATE DATABASE ddsonarqube OWNER ddsonar;
GRANT ALL PRIVILEGES ON DATABASE ddsonarqube to ddsonar;
\q
exit
sudo apt install zip -y
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.0.0.68432.zip
sudo unzip sonarqube-10.0.0.68432.zip
sudo mv sonarqube-10.0.0.68432 sonarqube
sudo mv sonarqube /opt/
sudo groupadd ddsonar
sudo useradd -d /opt/sonarqube -g ddsonar ddsonar
sudo chown ddsonar:ddsonar /opt/sonarqube -R
sudo nano /opt/sonarqube/conf/sonar.properties

EDIT THIS LINE IN THE NANO FILE:
sonar.jdbc.username=ddsonar
sonar.jdbc.password=ddsonar
Below these two lines, add the following line of code.
sonar.jdbc.url=jdbc:postgresql://localhost:5432/ddsonarqube

sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
ADD THE LINE INSIDE NANO FILE:
RUN_AS_USER=ddsonar

sudo nano /etc/systemd/system/sonar.service

ADD THE LINE INSIDE NANO FILE:
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
sudo systemctl status sonar
sudo nano /etc/sysctl.conf

ADD THE FOLLOWING LINES:
vm.max_map_count=262144
fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
sudo reboot

pipeline:
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
                    sh 'mv /var/lib/jenkins/workspace/Project/EcommerceApp/target/EcommerceApp.war /home/ubuntu/apache-tomcat-9.0.115/webapps/'
                }
            }
        }
    }
}
