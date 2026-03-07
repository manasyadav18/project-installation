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
