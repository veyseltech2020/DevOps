Ansible Ad-hoc commands

1)  To confirm that all our hosts are located by Ansible, we will run the commands below.
# ansible all --list-hosts
# ansible webservers --list-hosts

2)  To make sure that all our hosts are reachable, we will run various ad-hoc commands that use the ping module.
# ansible all -m ping
# ansible <server_name> -m ping
# ansible <node_name> -m ping

3)  "ansible-doc <module_name>" command is used for seeing the explanation and examples of a specific module.
# ansible-doc ping  (successful ping returns the pong response)

4)  How it is possible to ping without opening the ICMP port? Here is the answer;
# ansible all -m ping -o

5)  Emphasize the warning about the deprication of the usage of Python2. Go on with the following operations to get rid of the warning.
# vim ansible.cfg

- Add the following lines to /etc/ansible/ansible.cfg file.
    [defaults]
    interpreter_python=auto_silent

- Run the command below;
# ansible --help

6)  How much the system is up and what is load avarage.
#   ansible webservers -a "uptime"
#   web_server1 | CHANGED | rc=0 >>
#   13:00:59 up 42 min,  1 user,  load average: 0.08, 0.02, 0.01

7)  Some other ansible commands
# ansible webservers -m shell -a "systemctl status sshd"
# ansible webservers -m command -a 'df -h'
# ansible webservers -a 'df -h'

8)  Run the commands below for explaining how to transfer a file.
- create a file named 'testfile'
# vi testfile

- add the content below to the file
# "This is a test file"

- run the commands below;
# ansible webservers -m copy -a "src=/etc/ansible/testfile dest=/home/name_of_the_file/testfile"
# ansible node1 -m shell -a "echo Hello World! This is Veysel ORS > /home/name_of_the_user/testfile2 ; cat testfile2"

9)  Using the Shell Module
- Run the command
# ansible webservers -b -m shell -a "yum -y update ; yum -y install nginx ; service nginx start; systemctl enable nginx"
if you get error, try the following command;
# ansible webservers -b -m shell -a "amazon-linux-extras install -y nginx1 ; systemctl start nginx ; systemctl enable nginx" 

- For Ubuntu Servers, run the command below;
# ansible node3 -b -m shell -a "apt update -y ; apt-get install -y nginx ; systemctl start nginx; systemctl enable nginx"

- Run the command below to remove the nginx package.
# ansible webservers -b -m shell -a "yum -y remove nginx"

10) Using the Yum and Package Module
- run the command below
# ansible-doc yum

- other commands
# ansible webservers -b -m yum -a "name=nginx state=present"
# ansible -b -m package -a "name=nginx state=present" all

# ansible all -m setup
# ansible-playbook <name_of_the_playbook_yaml_file>
# ansible-galaxy search nginx
# ansible-galaxy search nginx --platform EL (EL is for centos)
# ansible-galaxy search nginx --platform EL | grep geerl
# ansible-galaxy install geerlingguy.nginx
# ansible-galaxy list
# ansible-config dump | grep ROLE

11) Using your own inventory file
- create a file named inventory.
# vim inventory

- edit the file as shown below.
    [webservers]
    node1 ansible_host=<node1_ip> ansible_user=ec2-user

    [webservers:vars]
    ansible_ssh_private_key_file=/home/ec2-user/<YOUR-PEM-FILE-NAME>.pem

- install Apache Server to node1
# ansible -i inventory -b -m yum -a "name=httpd state=present" node1

- uninstall Apache Server to node1
# ansible -i inventory -b -m yum -a "name=httpd state=absent" node1


12) Working with sensitive data

A1)
- Create encypted variables using "ansible-vault" command
# ansible-vault create <secret.yml>

- Write the <new vault password>: xxxxxx and Confirm the <new vault password>: xxxxxx
    username: person
    password: xxxxxx

- Look at the content
# cat <secret.yml>

B1)
How to use it?

- create a file named "<create-user.yml>".
# vim <create-user.yml>

- add the content

- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret.yml
  tasks:
    - name: creating user
      user:
        name: "{{ username }}"
        password: "{{ password }}"

- run the playbook named "<create-user.yml>"
# ansible-playbook <create-user.yml>
(You will get ERROR! Attempting to decrypt but no vault secrets found)

- run the following command;
# ansible-playbook --ask-vault-pass <create-user.yml>

- write your password
# Vault password: xxxxxx

- to verify it
# ansible all -b -m command -a "grep person /etc/shadow"

-----------------------------------------------------------------------------------------------------

A2)
- Create another encypted variables using "ansible-vault" command but this time use SHA (Secure Hash Algorithm) for your password:
# ansible-vault create <secret-1.yml>

- Write the <new vault password>: xxxxxx and Confirm the <new vault password>: xxxxxx
    username: person1
    pwhash: xxxxxx

- Look at the content
# cat <secret-1.yml>

B2)
How to use it?

- create a file named "<create-user-1.yml>".
# vim <create-user-1.yml>

- add the content below

- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret-1.yml
  tasks:
    - name: creating user
      user:
        name: "{{ username }}"
        password: "{{ pwhash | password_hash ('sha512') }}"

- run the playbook named "<create-user-1.yml>"
# ansible-playbook --ask-vault-pass <create-user-1.yml>

- write your password
# Vault password: xxxxxx

- to verify it
# ansible all -b -m command -a "grep person1 /etc/shadow"