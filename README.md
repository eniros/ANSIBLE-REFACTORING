# ANSIBLE-REFACTORING
Jenkins CI/CD on a 3-tier application && Ansible Configuration Management Dev and UAT servers using Static Assignments

Ansible Refactoring and Static Assignments (IMPORTS AND ROLES)

In the previous CI-CD Ansibile Jenkins project, I implemented CI/CD and Configuration Managment solution on the Development Servers using Ansible 

In this project, I will be extending the functionality of this architecture and introducing configurations for UAT environment.

![jenkinsansiblearc](https://user-images.githubusercontent.com/61475969/205504382-e09a5aa0-be28-477a-8075-b0d438678a05.png)

STEP 1 - Jenkins Job Enhancement/ Refactoring

Download Copy Artifacts plugin on jenkins. This will help us copy all our artifacts to a specific directory in our Jenkins server
Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

Make a directory inside the root directory.

<img width="1398" alt="Screenshot 2022-12-04 at 17 05 14" src="https://user-images.githubusercontent.com/61475969/205504921-cb2b4a33-dc1a-4190-8d3a-86f3ddacf88a.png">

<img width="1398" alt="Screenshot 2022-12-04 at 17 07 38" src="https://user-images.githubusercontent.com/61475969/205504968-2055e81d-494a-4ecc-b184-aa184cd433cb.png">

On the Jenkins-Ansible server, create a new directory called ````ansible-config-artifact```` by running the command below;

```sudo mkdir /home/ubuntu/ansible-config-artifact```

Change permission of the directory

```sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact```

<img width="569" alt="Screenshot 2022-12-04 at 17 15 05" src="https://user-images.githubusercontent.com/61475969/205505350-f91c2658-423c-43c3-bb62-915c7dae074f.png">

Create a new Freestyle project and name it save_artifacts.

This project will be triggered by completion of your existing ansible project. Configure it accordingly:

   - Configure the directory where you want to copy your artifacts.
   - create a Build step and choose Copy artifacts from other project
   - Specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

<img width="719" alt="Screenshot 2022-12-04 at 17 30 19" src="https://user-images.githubusercontent.com/61475969/205506127-3109f46b-68c8-4bb0-88b2-59dcbb87cd9f.png">

<img width="719" alt="Screenshot 2022-12-04 at 17 30 44" src="https://user-images.githubusercontent.com/61475969/205506142-3506742b-b96b-4880-8aa4-4e59ef49ce5f.png">

We configured the number of build to 2. This is useful because whenever the jenkins pipeline runs, it creates a directory for the artifacts and it takes alot of space. By specifying the number of build, we can choose to keep only 2 of the latest builds and discard the rest.

Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master/main branch).

If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.

Now your Jenkins pipeline is more neat and clean.

<img width="1058" alt="Screenshot 2022-12-04 at 18 07 56" src="https://user-images.githubusercontent.com/61475969/205507821-69d56d3b-0785-4fc4-abd1-f18a40257d53.png">

Step 2 – Refactor Ansible code by importing other playbooks

Importing Playbooks is a great way to refactor your Ansible code. It is a great way to reuse code. We can import other playbooks into a particular playbook. Breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration.

Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. 

<img width="762" alt="Screenshot 2022-12-04 at 18 13 45" src="https://user-images.githubusercontent.com/61475969/205508103-494bd140-d495-4791-a441-3326ed93843f.png">

Move common.yml file into the newly created static-assignments folder.

Inside site.yml file, import common.yml playbook by adding the following code to site.yml;

```---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
<img width="769" alt="Screenshot 2022-12-04 at 18 20 15" src="https://user-images.githubusercontent.com/61475969/205508397-0c425aa8-d192-4854-82bd-ededcb618f45.png">

We can also create a new yml file called common-del.yml under static-assignments. This will delete wireshark installation we made in the last project. 

Add the following script to common-del.yml:

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```

<img width="771" alt="Screenshot 2022-12-04 at 18 25 10" src="https://user-images.githubusercontent.com/61475969/205508607-407969ca-6234-427b-90bf-1292a5c4ec2c.png">

We update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers


Step 3 – Configure UAT Webservers with a role ‘Webserver’

We will create and configure our UAT Servers 
We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.

Launch 2 EC2 RHEL instance. (web1-uat, web2-uat)

<img width="954" alt="Screenshot 2022-12-04 at 19 05 14" src="https://user-images.githubusercontent.com/61475969/205510240-ceae648c-16ff-4193-b84a-bd4107c12f2a.png">


To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory. I created my roles folder in my ansible-config-mgt project.


Creating folder structure can be done in two ways:

Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront) - I used this option

Creating folder structure can be done in two ways:

Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront) - I used this option

```
mkdir roles
cd roles
ansible-galaxy init webserver
```

Create the directory/files structure manually

<img width="233" alt="Screenshot 2022-12-04 at 18 30 06" src="https://user-images.githubusercontent.com/61475969/205508800-3ffbe6dd-abf0-480e-9dc8-b0a8db733725.png">

Update inventory/uat.yml file:

```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
```

In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path= /home/ubuntu/ansible-config-artifact/roles, so Ansible could know where to find configured roles.

Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

```

---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

Install and configure Apache (httpd service)

Make sure that wireshark is deleted on all the servers by running wireshark --version
<img width="816" alt="Screenshot 2022-12-04 at 19 02 00" src="https://user-images.githubusercontent.com/61475969/205510080-eb77b8df-ea7e-45d4-9266-71769969c2f2.png">

Step 4 – Reference ‘Webserver’ role

Create a new assignment for uat-webservers static-assignments/uat-webservers.yml. and refrence the webserver role inside the file as foollows:
---
- hosts: uat-webservers
  roles:
    - webserver
Update site.yml to include the following:
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml

Commit the repo and merge to master. Then run the following code below after the successful build of save_artifacts project on Jenkins:
ansible-playbook -i /home/ubuntu/ansible-config-artifact/inventory/uat.yml /home/ubuntu/ansible-config-artifact/playbooks/site.yml --forks 1

The webservers should be displaying webcontent now if the whole setup is successful. http://Web1-UAT-Server-Public-IP-or-Public-DNS-Name/index.php

<img width="999" alt="Screenshot 2022-12-04 at 19 06 29" src="https://user-images.githubusercontent.com/61475969/205510276-5d736d30-05c4-4de8-94e0-1e9b6c58ecec.png">





