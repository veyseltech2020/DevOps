1) to update and upgrade the system, first run the following commands

# sudo yum update -y

# sudo yum upgrade -y

2) to install the ansible you can run the following commands

# sudo yum -y install ansible  (for linux OS)

# sudo apt-get -y install ansible  (for ubuntu OS)

# sudo amazon-linux-extras install ansible2  (another way for installation)

3) to install Ansible with pip
    The other way to install Ansible is with the pip, the Python package manager. If pip isn’t already available on your system of Python, run the following commands to install it:
    
# `$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py

# $ python get-pip.py --user`

then run the command

# pip install --user ansible

4) to confirm the successful installation of Ansible, run the command below

# ansible --version

then check the output

# ansible 2.9.12
    config file = /etc/ansible/ansible.cfg
    configured module search path = [u'/home/ec2-user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
    ansible python module location = /usr/lib/python2.7/site-packages/ansible
    executable location = /usr/bin/ansible
    python version = 2.7.18 (default, Aug 27 2020, 21:22:52) [GCC 7.3.1 20180712 (Red Hat 7.3.1-9)]

explanation the output
    (Version Number of Ansible
    Path for the Ansible Config File
    Modules are searched in this order
    Ansible's Python Module path
    Ansible's executable file path
    Ansible's Python version with GNU Compiler Collection for Red Hat)