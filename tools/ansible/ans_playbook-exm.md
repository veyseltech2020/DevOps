This file contains some example playbooks yaml files.

1)  to make sure all our hosts are up and running.
- create a yaml file named "<playbookname.yml>" and write the following content in the file.

---
- name: Test Connectivity
  hosts: all
  tasks:
   - name: Ping test
     ping:

- run the yaml file
# ansible-playbook <playbookname.yml>

2)  Create a text file named "<filename1>" and write "Hello Clarusway" with using vim. Then create a yaml file name "<playbookname.yml>" and send the "<filename1>" to the hosts.

---
- name: Copy for linux
  hosts: webservers
  tasks:
   - name: Copy your file to the webservers
     copy:
       src: /home/ec2-user/<filename1>
       dest: /home/ec2-user/<filename1>

- name: Copy for ubuntu
  hosts: ubuntuservers
  tasks:
   - name: Copy your file to the ubuntuservers
     copy:
       src: /home/ec2-user/<filename1>
       dest: /home/ubuntu/<filename1>

- run the yaml file
# ansible-playbook <playbookname.yml>

- Connect the nodes with SSH and check if the text files are copied or not.

3)  Create a yaml file named "<playbookname.yml>" as below.

# vim <playbookname.yml>

---
- name: Copy for linux
  hosts: webservers
  tasks:
   - name: Copy your file to the webservers
     copy:
       src: /home/ec2-user/<filename1>
       dest: /home/ec2-user/<filename1>
       mode: u+rw,g-wx,o-rwx

- name: Copy for ubuntu
  hosts: ubuntuservers
  tasks:
   - name: Copy your file to the ubuntuservers
     copy:
       src: /home/ec2-user/<filename1>
       dest: /home/ubuntu/<filename1>
       mode: u+rw,g-wx,o-rwx

- name: Copy for node1
  hosts: node1
  tasks:
   - name: Copy using inline content
     copy:
       content: '# This file was moved to /etc/ansible/<filename1>'
       dest: /home/ec2-user/<filename2>

   - name: Create a new text file
     shell: "echo Hello World > /home/ec2-user/<filename3>"

- run the yaml file
# ansible-playbook <playbookname.yml>

- Connect the node1 with SSH and check if the text files are there.

4)  Install Apache server with "<playbookname.yml>". After the installation, check if the Apache server is reachable from the browser.

# ansible-doc yum
# ansible-doc apt

# vim <playbookname.yml>

---
- name: Apache installation for webservers
  hosts: webservers
  tasks:
   - name: install the latest version of Apache
     yum:
       name: httpd
       state: latest

   - name: start Apache
     shell: "service httpd start"

- name: Apache installation for ubuntuservers
  hosts: ubuntuservers
  tasks:
   - name: install the latest version of Apache
     apt:
       name: apache2
       state: latest

- run the yaml file
# ansible-playbook <playbookname.yml>

5)  Create "<playbookname.yml>" and remove the Apache server from the hosts.
# vim <playbookname.yml>

---
- name: Remove Apache from webservers
  hosts: webservers
  tasks:
   - name: Remove Apache
     yum:
       name: httpd
       state: absent

- name: Remove Apache from ubuntuservers
  hosts: ubuntuservers
  tasks:
   - name: Remove Apache
     apt:
       name: apache2
       state: absent
   - name: Remove unwanted Apache2 packages from the system
     apt:
       autoremove: yes
       purge: yes

- run the yaml file
# ansible-playbook <playbookname.yml>

6)  This time, install Apache and wget together with playbook6.yml. After the installation, enter the IP-address of node2 to the browser and show the Apache server. Then, connect node1 with SSH and check if "wget and apache server" are running.

# vim <playbookname.yml>

---
- name: play 4
  hosts: ubuntuservers
  tasks:
   - name: installing apache
     apt:
       name: apache2
       state: latest

   - name: index.html
     copy:
       content: "<h1>Hello Clarusway</h1>"
       dest: /var/www/html/index.html

   - name: restart apache2
     service:
       name: apache2
       state: restarted
       enabled: yes

- name: play 5
  hosts: webservers
  tasks:
    - name: installing httpd and wget
      yum:
        pkg: "{{ item }}"
        state: present
      with_items:
        - httpd
        - wget

- run the yaml file
# ansible-playbook <playbookname.yml>

7)  Remove Apache and wget from the hosts with "<playbookname.yml>".

# vim <playbookname.yml>

---
- name: play 6
  hosts: ubuntuservers
  tasks:
   - name: Uninstalling Apache
     apt:
       name: apache2
       state: absent
       update_cache: yes
   - name: Remove unwanted Apache2 packages
     apt:
       autoremove: yes
       purge: yes

- name: play 7
  hosts: webservers
  tasks:
   - name: removing apache and wget
     yum:
       pkg: "{{ item }}"
       state: absent
     with_items:
       - httpd
       - wget

- run the yaml file
# ansible-playbook <playbookname.yml>

8)  Using ansible loop and conditional, create users with "<playbookname.yml>".

# vim <playbookname.yml>
---
- name: Create users
  hosts: "*"
  tasks:
    - user:
        name: "{{ item }}"
        state: present
      loop:
        - person1
        - person2
        - person3
        - person4
      when: ansible_os_family == "RedHat"

    - user:
        name: "{{ item }}"
        state: present
      loop:
        - person5
        - person6
      when: ansible_os_family == "SUSE"

    - user:
        name: "{{ item }}"
        state: present
      loop:
        - person7
        - person8
      when: ansible_os_family == "Debian" or ansible_os_family == "20.04"

- run the yaml file
# ansible-playbook <playbookname.yml>

9)  Gathering facts

- create a playbook named "<facts.yml>".

# vim <facts.yml>

- name: show facts
  hosts: all
  tasks:
    - name: print facts
      debug:
        var: ansible_facts

- run the command
# ansible-playbook <facts.yml>

- create a playbook named "<ipaddress.yml>".

# vim <ipaddress.yml>

- hosts: all
  tasks:
  - name: show IP address
    debug:
      msg: >
       This host uses IP address {{ ansible_facts.default_ipv4.address }}

- run the command
# ansible-playbook <ipaddress.yml> -i inventory.aws_ec2.yml