# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)
In this project you will continue working with ansible-config-mgt repository and make some improvements of your code. Now you need to refactor your Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – it allows you to organize your tasks and reuse them when needed.

Code Refactoring

Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

In your case, you will move things around a little bit in the code, but the overal state of the infrastructure remains the same.

## Step 1 – Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require Copy Artifact plugin.
1. Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.
```javascript
sudo mkdir /home/ubuntu/ansible-config-artifact
```
2. Change permissions to this directory, so Jenkins could save files there –
```javascript
chmod -R 0777 /home/ubuntu/ansible-config-artifact
```
3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins
### Installing the copy artifact Plugin
![Screenshot (134)](https://user-images.githubusercontent.com/52970510/169768747-b5a8e531-af35-4b78-941f-2c923da666f6.png)

4. Create a new Freestyle project (you have done it in Project 9) and name it save_artifacts.
5. This project will be triggered by completion of your existing ansible project. Configure it accordingly:
### Configuring save_artifacts project
![Screenshot (135)](https://user-images.githubusercontent.com/52970510/169769333-9d18766e-bb9f-4634-bc7f-e078422b0c3f.png)

Note: You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

6. The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.
### Configuring the build step of the save_artifacts project
![Screenshot (136)](https://user-images.githubusercontent.com/52970510/169769484-1720eaeb-4e5e-46e3-bdc7-0ab5cb8e6051.png)

7. Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master branch).

If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.

Now your Jenkins pipeline is more neat and clean.

# REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML
## Step 2 – Refactor Ansible code by importing other playbooks into site.yml

Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it refactor.

DevOps philosophy implies constant iterative improvement for better efficiency – refactoring is one of the techniques that can be used, but you always have an answer to question "why?". Why do we need to change something if it works well?

In Project 11 you wrote all tasks in a single playbook common.yml, now it is pretty simple set of instructions for only 2 types of OS, but imagine you have many more tasks and you need to apply this playbook to other servers with different requirements. In this case, you will have to read through the whole playbook to check if all tasks written there are applicable and is there anything that you need to add for certain server/OS families. Very fast it will become a tedious exercise and your playbook will become messy with many commented parts. Your DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use your playbook.

Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Let see code re-use in action by importing other playbooks.

1. Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. Dont worry, you will understand more what this means shortly.
2. Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.
3. Move common.yml file into the newly created static-assignments folder.
4. Inside site.yml file, import common.yml playbook.
### Screenshot showing correct file structure and common.yml playbook imported into site.yml file
![Screenshot (138)](https://user-images.githubusercontent.com/52970510/169771101-13899a74-66b7-47b5-bd55-327405870e50.png)

5. Run ansible-playbook command against the dev environment
Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it common-del.yml. In this playbook, configure deletion of wireshark utility.
### Updating the common-del.yml file with deletion of wireshark utility included
![Screenshot (139)](https://user-images.githubusercontent.com/52970510/169771889-59f7fe8b-c7b7-4039-a923-108ba7946e2f.png)

update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers:
### Screenshot
![Screenshot (140)](https://user-images.githubusercontent.com/52970510/169772222-7d5262fc-73ec-4100-b92d-ea31d9d1ae4e.png)

```javascript
cd /home/ubuntu/ansible-config-mgt/

ansible-playbook -i inventory/dev.yml playbooks/site.yaml
```
### Ansible playbook running
![Screenshot (141)](https://user-images.githubusercontent.com/52970510/169772840-8ecb63eb-a342-4339-9247-b790541c7df4.png)

Make sure that wireshark is deleted on all the servers by running wireshark --version

Now you have learned how to use import_playbooks module and you have a ready solution to install/delete packages on multiple servers with just one command.

# CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’
## Step 3 – Configure UAT Webservers with a role ‘Webserver’
We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.

1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.

Tip: Do not forget to stop EC2 instances that you are not using at the moment to avoid paying extra. For now, you only need 2 new RHEL 8 servers as Web Servers and 1 existing Jenkins-Ansible server up and running.

2. To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.

There are two ways how you can create this folder structure:

>Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)
```javascript
mkdir roles
cd roles
ansible-galaxy init webserver
```
>Create the directory/files structure manually

Note: You can choose either way, but since you store all your codes in GitHub, it is recommended to create folders and files there rather than locally on Jenkins-Ansible server.

The entire folder structure should look like below, but if you create it manually – you can skip creating tests, files, and vars or remove them if you used ansible-galaxy

the roles structure should look like this;
```javascript
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
```

3. Update your inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of your 2 UAT Web servers
### Addding UAT webservers to uat.yml file
![Screenshot (143)](https://user-images.githubusercontent.com/52970510/169774017-4664d966-32c3-4ce6-b98b-6d88c702ce22.png)

4. In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path    = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles.
5. It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

> Install and configure Apache (httpd service)
> Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
> Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
> Make sure httpd service is started

  Your main.yml may consist of following tasks:
  ```javascript
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
### Adding some tasks to the webserver role
![Screenshot (145)](https://user-images.githubusercontent.com/52970510/169776306-eaea1ab4-1e1c-41b5-93c0-a19ddc8a5216.png)

# REFERENCE WEBSERVER ROLE
## Step 4 – Reference ‘Webserver’ role
Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role.
### Referencing webserver role in uat-webservers.yml file
![Screenshot (146)](https://user-images.githubusercontent.com/52970510/169777115-38c20f07-ec18-4dce-84e3-2c241c0f395c.png)

Remember that the entry point to our ansible configuration is the site.yml file. Therefore, you need to refer your uat-webservers.yml role inside site.yml.

So, we should have this in site.yml
  
### Referencing uat-webservers.yml role inside site.yml
![Screenshot (147)](https://user-images.githubusercontent.com/52970510/169777675-0e99329f-f175-4a28-9a56-0c27c17ae76f.png)

## Step 5 – Commit & Test

Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.
  
### Screenshots
![Screenshot (148)](https://user-images.githubusercontent.com/52970510/169777919-ac1e77a7-fb1f-47f4-8051-5b7a2d5fe297.png)

![Screenshot (149)](https://user-images.githubusercontent.com/52970510/169778037-ce5060ed-289a-444c-94c1-d607217c8e5a.png)

![Screenshot (150)](https://user-images.githubusercontent.com/52970510/169778140-33d889ca-07c4-405c-b52e-b1387ff589a7.png)

Now run the playbook against your uat inventory and see what happens:
```javascript
sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml
```
### Ansible playbook running
![Screenshot (151)](https://user-images.githubusercontent.com/52970510/169818149-9e7b57da-3eac-41b0-bcb2-c9b9626b5438.png)

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:
```javascript
http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php

or

http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
```
    
### Screenshot
![Screenshot (152)](https://user-images.githubusercontent.com/52970510/169818394-41ab929e-8c30-484b-8275-204ed023fce9.png)
