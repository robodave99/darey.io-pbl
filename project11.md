# ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10
In Projects 7 to 10 you had to perform a lot of manual operations to seet up virtual servers, install and configure required software, deploy your web application.

This Project will make you appreciate DevOps tools even more by making most of the routine tasks automated with Ansible Configuration Management, at the same time you will become confident at writing code using declarative language such as YAML.

## INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE
1. Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
2. In your GitHub account create a new repository and name it ansible-config-mgt.

3.Instal Ansible
```javascript
sudo apt update

sudo apt install ansible
```
### Apache Installed
![Screenshot (115)](https://user-images.githubusercontent.com/52970510/169240503-89808013-1514-43dc-8d45-650d726ddef2.png)

Check your Ansible version by running ansible --version

4. Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.
> Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
### Screenshot
![Screenshot (116)](https://user-images.githubusercontent.com/52970510/169241080-f4205b3f-2efc-47ff-86ac-975d4809db2f.png)
> Configure Webhook in GitHub and set webhook to trigger ansible build.
### Screenshot
![Screenshot (117)](https://user-images.githubusercontent.com/52970510/169241445-3cc39f98-ce06-49a1-be0a-3ba7098bf188.png)
> Configure a Post-build job to save all (**) files, like you did it in Project 9.
### Screenshot
![Screenshot (118)](https://user-images.githubusercontent.com/52970510/169241915-c09c2f9b-06dc-4483-ab3a-32459704f0ef.png)

5. Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder
```javascript
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
### Testing setup
![Screenshot (119)](https://user-images.githubusercontent.com/52970510/169242746-e7a283d3-0801-4363-ab45-266394fd94f9.png)

![Screenshot (120)](https://user-images.githubusercontent.com/52970510/169243952-2074af6e-b5cf-4f0f-a5e5-233f882ccc0f.png)

Note: Trigger Jenkins project execution only for /main (master) branch.

## Prepare your development environment using Visual Studio Code
1. First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSC)
2. After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.
3. Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance
```javascript
git clone <ansible-config-mgt repo link>
```
### Cloned repo on Vscode
![Screenshot (121)](https://user-images.githubusercontent.com/52970510/169244133-3bc46f3b-3526-46ea-b722-7bc3453e465d.png)

## BEGIN ANSIBLE DEVELOPMENT
1. In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)
### Created new branch, prj-11 in the terminal
![Screenshot (122)](https://user-images.githubusercontent.com/52970510/169244807-9ef090f1-8bc2-4740-bcf0-093c6cd1a365.png)

2. Checkout the newly created feature branch to your local machine and start building your code and directory structure
3. Create a directory and name it playbooks – it will be used to store all your playbook files.
4. Create a directory and name it inventory – it will be used to keep your hosts organised.
5. Within the playbooks folder, create your first playbook, and name it common.yml
6. Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.
### Steps 2-6
![Screenshot (123)](https://user-images.githubusercontent.com/52970510/169245277-c6db416e-8073-445f-8cf4-9459150f2388.png)

### Set up an Ansible Inventory
An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

Save below inventory structure in the inventory/dev file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.
Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:
To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, please see this video:

For Windows users – ssh-agent on windows (https://youtu.be/OplGrY74qog)
For Linux users – ssh-agent on linux (https://youtu.be/OplGrY74qog)
```javascript
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```
Confirm the key has been added with the command below, you should see the name of your key
```javascript
ssh-add -l
```
Now, ssh into your Jenkins-Ansible server using ssh-agent
```javascript
ssh -A ubuntu@public-ip
```
Also notice, that your Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.

Update your inventory/dev.yml file with this snippet of code:
```javascript
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```
## CREATE A COMMON PLAYBOOK
It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.

In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your playbooks/common.yml file with following code:
```javascript
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```
### Screenshot
![Screenshot (124)](https://user-images.githubusercontent.com/52970510/169252657-5fd7efd8-c97b-4dcf-bb34-9f7b1fc18e41.png)

Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

Feel free to update this playbook with following tasks:

> Create a directory and a file inside it
> Change timezone on all servers
> Run some shell script
> …

For a better understanding of Ansible playbooks – watch this video from RedHat(https://youtu.be/ZAdJ7CdN7DY) and read this article(https://www.redhat.com/en/topics/automation/what-is-an-ansible-playbook).

## Update GIT with the latest code
Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.

In the real world, you will be working within a team of other DevOps engineers and developers. It is important to learn how to collaborate with help of GIT. In many organisations there is a development rule that do not allow to deploy any code before it has been reviewed by an extra pair of eyes – it is also called "Four eyes principle".

Now you have a separate branch, you will need to know how to raise a Pull Request (PR), get your branch peer reviewed and merged to the master branch.
Commit your code into GitHub:
1. use git commands to add, commit and push your branch to GitHub.
```javascript
git status

git add <selected files>

git commit -m "commit message"
```
### Screenshot
![Screenshot (125)](https://user-images.githubusercontent.com/52970510/169255892-44c28217-1509-4e56-adcd-7a67df0dedb9.png)

2. Create a Pull request (PR)
### Screenshot
![Screenshot (126)](https://user-images.githubusercontent.com/52970510/169256122-10fb33bf-3d3b-4daf-abff-56bccdbdb84d.png)

3. Wear a hat of another developer for a second, and act as a reviewer.
4. If the reviewer is happy with your new feature development, merge the code to the master branch.
5. Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.
### Screenshot
![Screenshot (128)](https://user-images.githubusercontent.com/52970510/169258211-5b579105-f71f-46f1-abfb-b289f3e9acc4.png)

Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.

## RUN FIRST ANSIBLE TEST
Now, it is time to execute ansible-playbook command and verify if your playbook actually works:
```javascript
cd ansible-config-mgt
```
```javascript
ansible-playbook -i inventory/dev.yml playbooks/common.yml
```
### Ansible Playbook running
![Screenshot (131)](https://user-images.githubusercontent.com/52970510/169258903-c0cdddef-2a49-4e02-8eb6-6dbfbda9e2b7.png)

You can go to each of the servers and check if wireshark has been installed by running which wireshark or wireshark --version
### Checking the Webserver2 for wireshark
![Screenshot (132)](https://user-images.githubusercontent.com/52970510/169259330-d61044a9-3285-47e5-be3f-f27e1a843b26.png)
