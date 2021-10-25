Here are some commands to configure ansible
1)  Connect to the control node for building a basic inventory.
    Edit /etc/ansible/hosts file by appending the connection information of the remote systems to be managed.
    Along with the hands-on, public or private IPs can be used.

- become root
# $ sudo su

- go to the file
# $ cd /etc/ansible

- create a file named hosts
# $ vim hosts

- then copy the following content to the file you create as named hosts

[webservers]
node1 ansible_host=<node1_ip> ansible_user=<name_of_the_user>
node2 ansible_host=<node2_ip> ansible_user=<name_of_the_user>

[all:vars]
ansible_ssh_private_key_file=/home/name_of_the_user/<pem file>

2)  Copy your pem file to the /etc/ansible/ directory.
- First, go to your pem file directory on your local PC
- run the following command.

# $ scp -i <pem file> <pem file> name_of_the_user@<public DNS name of Control Node>:/home/name_of_the_user

