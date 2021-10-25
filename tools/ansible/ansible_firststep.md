This steps tell us how to INSTALL ANSIBLE to the machine and take the necessary steps to use the ansible on our machine.

1)  Create a Control Node

2)  Connect to the Control Node with SSH

3)  Run the following commands

- to update the system;
# sudo yum update -y

- to install the ansible
# sudo amazon-linux-extras install ansible2

- to confirm the installation
# ansible --version

- Connect to the control node and for this basic inventory, edit /etc/ansible/hosts, and add a few remote systems (manage nodes) to the end of the file. For this example, use the IP addresses of the servers.
# sudo su
# cd /etc/ansible
# ls
# vim hosts
[webservers]
node1 ansible_host=<node1_ip> ansible_user=ec2-user
node2 ansible_host=<node1_ip> ansible_user=ec2-user
[ubuntuservers]
node2 ansible_host=<node2_ip> ansible_user=ubuntu
[all:vars]
ansible_ssh_private_key_file=/home/ec2-user/<pem file>

- Edit /etc/ansible/ansible.cfg as adding below.
# vim ansible.cfg
[defaults]
interpreter_python=auto_silent

- Copy your pem file to the /etc/ansible/ directory. First go to your pem file directory on your local computer and run the following command.
# scp -i <pem file> <pem file> ec2-user@<public DNS name of the control node>:/home/ec2-user