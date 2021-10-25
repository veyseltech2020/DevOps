
ANSIBLE PROJECT

Provisioning a Web Server and a Database Server with a Dynamic Website using Ansible.

This project shows how to provision a Web Server and a Database Server with a Dynamic Website using the Ansible tool. It contains the necessary files which handle the content to run the project.

PROJECT'S OUTLINE

Part 1 - Build the Infrastructure (3 EC2 Instances with Red Hat Enterprise Linux 8 AMI)
Part 2 - Install Ansible on the Controller Node
Part 3 - Pinging the Target Nodes
Part 4 - Install, Start and Enable MariaDB
Part 5 - Configure User Credentials and Database Schema
Part 6 - Install, Start and Enable Apache Web Server and Other Dependencies
Part 7 - Pull the Code and Make Necessary Changes


LET'S GET STARTED...

PART 1: BUILDING THE INFRASTRUCTURE
- Get to the AWS Console and spin-up 3 EC2 Instances with Red Hat Enterprise Linux 8 AMI.

- Configure the security groups as shown below:
  Controller Node ----> Port 22 SSH
  Target Node1 -------> Port 22 SSH, Port 3306 MYSQL/Aurora
  Target Node2 -------> Port 22 SSH, Port 80 HTTP


PART 2: INSTALL ANSIBLE ON THE CONTROLLER NODE
- Connect to your Controller Node.

- Run the commands below to install Python3 and Ansible.
# sudo yum install -y python3 
# pip3 install --user ansible

- Check Ansible's installation with the command below.
# ansible --version


PART 3: PINGING THE TARGET NODES
- Run the command below to transfer your pem key to your Ansible Controller Node.
# scp -i <PATH-TO-PEM-FILE> <PATH-TO-PEM-FILE> ec2-user@<CONTROLLER-NODE-IP>:/home/ec2-user

- Make a directory named <ansible_project> under the home directory and cd into it.
# mkdir <ansible_project>
# cd <ansible_project>

- Create a file named <inventory.txt> with the command below.
# vim <inventory.txt> (you can use vi or nano to create a file)

- Paste the content below into the inventory.txt file.
[servers]
db_server   ansible_host=<YOUR-DB-SERVER-IP>   ansible_user=ec2-user  ansible_ssh_private_key_file=~/<YOUR-PEM-FILE>
web_server  ansible_host=<YOUR-WEB-SERVER-IP>  ansible_user=ec2-user  ansible_ssh_private_key_file=~/<YOUR-PEM-FILE>

- Create a file named <ping-playbook.yml> and paste the content below.
# touch <ping-playbook.yml>

- Add the following content to ping the hosts
  - name: ping them all
    hosts: all
    tasks:
      - name: pinging
        ping:

- Run the command below for pinging the servers.
# ansible-playbook <ping-playbook.yml> -i inventory.txt

Create another file named <ansible.cfg> in the project directory.
# vim <ansible.cfg>

- Add the content below
  [defaults]
  host_key_checking = False

- Run the command below again.
# ansible-playbook <ping-playbook.yml> -i inventory.txt

- Append the content below into <ansible.cfg> file in order to avoid using -i inventory.txt option in ansible-playbook commands.
  inventory=inventory.txt


PART 4: INSTALL, START and ENABLE MARIADB

- Create another file named <project.yml> under the <ansible_project> directory.
# vim <project.yml>

- Write the content below into the <project.yml> file.
  - name: db configuration
    hosts: db_server
    tasks:
      - name: install mariadb and PyMySQL
        become: yes
        yum:
          name: 
              - mariadb-server
              - python3-PyMySQL
          state: latest

      - name: start mariadb
        become: yes  
        command: systemctl start mariadb

      - name: enable mariadb
        become: yes
        systemd: 
          name: mariadb
          enabled: true

- Run the playbook
# ansible-playbook <project.yml>

- Open up a new Terminal or Window and connect to the db_server instance and check if MariaDB is installed, started, and enabled.
# mysql --version

- Create a file named <db_load_script.sql> under /home/ec2-user folder
# vim <db_load_script.sql>

- Write the content below in it
  USE ecomdb;
  CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

  INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");

- Append the content below into <project.yml> file for transferring the sql script into database server.
  - name: copy the sql script
    copy:
      src: ~/db_load_script.sql
      dest: ~/

- Run the command below.
# ansible-playbook <project.yml>

- Check if the file has been sent to the database server.

- Append the content below into <project.yml> file in order to login to db_server and set a root password.
  - name: Create password for the root user
        mysql_user:
          login_password: ''
          login_user: root
          name: root
          password: "vysl2020"

- Create a file named "<.my.cnf>" under home directory on controller node.
# vim <.my.cnf>

- Paste the content below.
  [client]
  user=root
  password=vysl2020

  [mysqld]
  wait_timeout=30000
  interactive_timeout=30000
  bind-address=0.0.0.0

- Append the content below into <project.yml> file.
  - name: copy the .my.cnf file
        copy:
          src: ~/.my.cnf
          dest: ~/

- Run the command below and check if everything is ok.
# ansible-playbook <project.yml>


PART 5: CONFIGURE USER CREDENTIALS and DATABASE SCHEMA

- Append the content below into <project.yml> file in order to create a remote database user.
    - name: Create db user with name 'remoteUser' and password 'vysl2020' with all database privileges
      mysql_user:
        name: remoteUser
        password: "vysl2020"
        login_user: "root"
        login_password: "vysl2020"
        priv: '*.*:ALL,GRANT'
        state: present
        host: "{{ hostvars['web_server'].ansible_host }}"

- Run the command below to check if everything is ok.
# ansible-playbook <project.yml>

- Append the content below into <project.yml> file in order to create a database schema for the products table.
    - name: Create database schema
      mysql_db:
        name: prodb
        login_user: root
        login_password: "vysl2020"
        state: present

- Append the content below into <project.yml> file in order to check if the products table is already imported.

    - name: check if the database has the table
      shell: |
        echo "USE prodb; show tables like 'products'; " | mysql
      register: resultOfShowTables

    - name: DEBUG
      debug:
        var: resultOfShowTables

- Append the content below into <project.yml> file in order to import the products table.

    - name: Import database table
      mysql_db:
        name: prodb   # This is the database schema name.
        state: import  # This module is not idempotent when the state property value is import.
        target: ~/db_load_script.sql # This script creates the products table.
      when: resultOfShowTables.stdout == "" # This line checks if the table is already imported. If so this task doesn't run.

- Run the command below to check if there is an error.
# ansible-playbook <project.yml>

- Connect to the database instance to check if there is a table named products.

- Run the command below.
# echo "USE prodb; show tables like 'products'; " | mysql

- Append the content below in order to restart the MariaDB service.
    - name: restart mariadb
      become: yes
      service: 
        name: mariadb
        state: restarted


PART 6: INSTALL, START and ENABLE APACHE WEB SERVER and OTHER DEPENDENCIES

- Append the content below into the <project.yml> file
  - name: web server configuration
    hosts: web_server
    become: yes
    tasks: 
      - name: install the latest version of Git, Apache, Php, Php-Mysqlnd
        package:
          name: 
            - git
            - httpd
            - php
            - php-mysqlnd
          state: latest

- Append the content below into the <project.yml> file.
    - name: start the server and enable it
      service:
        name: httpd
        state: started
        enabled: yes


PART 7: PULL THE CODE and MAKE NECESSARY CHANGES

- Append the content below into the <project.yml> file.
    - name: clone the repo of the website
      shell: |
        if [ -z "$(ls -al /var/www/html | grep .git)" ]; then
          git clone https://github.com/kodekloudhub/learning-app-ecommerce.git /var/www/html/
          echo "ok"
        else
          echo "already cloned..."
        fi
      register: result

    - name: DEBUG
      debug:
        var: result

    - name: Replace a default entry with our own
      lineinfile:
        path: /var/www/html/index.php
        regexp: '172\.20\.1\.101'
        line: "$link = mysqli_connect('{{ hostvars['db_server'].ansible_host }}', 'remoteUser', 'vysl2020', 'prodb');"
      when: not result.stdout == "already cloned..."

The first line checks if the result of the ls -al /var/www/html | grep .git command is null.

The second line clones the project content into /var/www/html directory.

For the third task, explain that the lineinfile module finds the file specified in the path value, searches for the regular expression specified in the regexp property, and replaces the matching line with the value of the line property. But this task is run only if the standard output of the first task is not "already cloned...".

- Append the content below into playbook.yml file.
    - selinux:
        state: disabled    

    - name: Restart service httpd
      service:
        name: httpd
        state: restarted

- Run the command below.
# ansible-playbook <project.yml>

- Check if you can see the website on your browser.