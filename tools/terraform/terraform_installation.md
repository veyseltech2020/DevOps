This file shows how to install Terraform to our machine/instances

- Launch an EC2 instance using the Amazon Linux 2 AMI with security group allowing SSH connections.

- Connect to your instance with SSH.

ssh -i .ssh/<PEM.KEY> ec2-user@<IP-ADDRESS>.us-east-2.compute.amazonaws.com

- Update the installed packages and package cache on your instance.

sudo yum update -y

- Install yum-config-manager to manage your repositories.

sudo yum install -y yum-utils

- Use yum-config-manager to add the official HashiCorp Linux repository.

sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo

- Install Terraform.

sudo yum -y install terraform

- Verify that the installation

terraform version