# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES
IMPORTANT NOTICE: Ansible is an actively developing software project, so you are encouraged to visit Ansible Documentation for the latest updates on modules and their usage.

Last 2 projects have already equipped you with some knowledge and skills on Ansible, so you can perform configurations using playbooks, roles and imports. Now you will continue configuring your UAT servers learning and practicing new Ansible concepts and modules.

In this project we will introduce dynamic assignments by using include module.

Now you may be wondering, what is the difference between static and dynamic assignments?

Well, from Project 12, you can already tell that static assignments use import Ansible module. The module that enables dynamic assignments is include.

Hence,
```javascript
import = Static
include = Dynamic
```
When the import module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.

Take note that in most cases it is recommended to use static assignments for playbooks, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature. However, you can use dynamic assignments for environment specific variables as we will be introducing in this project.

## INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE
In your https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.

Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml. We will instruct site.yml to include this playbook later. For now, let us keep building up the structure.
Note: Depending on what method you used in the previous project you may have or not have roles folder in your GitHub repository ??? if you used ansible-galaxy, then roles directory was only created on your Jenkins-Ansible server locally. It is recommended to have all the codes managed and tracked in GitHub, so you might want to recreate this structure manually in this case ??? it is up to you.

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment???s variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

Your layout should now look like this:
  
![Screenshot (155)](https://user-images.githubusercontent.com/52970510/170165196-a1f7a790-54c6-4fb0-af13-bf44a2dce17e.png)

Now paste the instruction below into the env-vars.yml file:
  
![Screenshot (156)](https://user-images.githubusercontent.com/52970510/170165333-2283712f-f03d-411c-9dac-f868bb7a7f57.png)

Notice 3 things to notice here:

1. We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:
> include_role
> include_tasks
> include_vars
  
In the same version, variants of import were also introduces, such as:
> import_role
> import_tasks

2. We made use of a special variables { playbook_dir } and { inventory_file }. { playbook_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. { inventory_file } on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

# UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS
Update site.yml file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats)

site.yml should now look like this:
  
### Screenshot
  
![Screenshot (157)](https://user-images.githubusercontent.com/52970510/170166113-e2b42856-5670-4905-a2e7-ef8a4d822644.png)

Community Roles
Now it is time to create a role for MySQL database ??? it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.
  
### Download Mysql Ansible Role
You can browse available community roles at (https://galaxy.ansible.com/home)
We will be using a MySQL role developed by geerlingguy.

Hint: To preserve your your GitHub in actual state after you install a new role ??? make a commit and push to master your ???ansible-config-mgt??? directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it ??? we will be using Jenkins later for a better purpose.

On Jenkins-Ansible server make sure that git is installed with git --version, then go to ???ansible-config-mgt??? directory and run
```javascript
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```
Inside roles directory create your new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql
```javascript
mv geerlingguy.mysql/ mysql
```
### Creating mysql role and renaming it to mysql
  
![Screenshot (159)](https://user-images.githubusercontent.com/52970510/170166625-99921270-54d2-4086-8774-ea3af18000df.png)

Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.
  
### Screenshot
  
![Screenshot (164)](https://user-images.githubusercontent.com/52970510/170166837-4ed05c10-ce4c-4823-b2ea-ed649c7e3e53.png)

Now it is time to upload the changes into your GitHub:
```javascript
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```
Now, if you are satisfied with your codes, you can create a Pull Request and merge it to main branch on GitHub.
  
## LOAD BALANCER ROLES
We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:
1. NGINX
2. APACHE

With your experience on Ansible so far you can:
> Decide if you want to develop your own roles, or find available ones from the community
  
> Update both static-assignment and site.yml files to refer the roles

Important Hints:
> Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one ??? this is where you can make use of variables.

> Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

> Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.
  
> Declare another variable in both roles load_balancer_is_required and set its value to false as well
  
### Screenshot
  
![Screenshot (165)](https://user-images.githubusercontent.com/52970510/170167443-d9e2cd32-abc1-4717-abbc-f9541d5e9a37.png)

> Update both assignment and site.yml files respectively
  
### Updating the loadbalancer.yml file
  
![Screenshot (166)](https://user-images.githubusercontent.com/52970510/170167935-91c20f38-9e7e-40e1-ae86-6d84f178ffd9.png)

### Updating the Site.yml file
  
![Screenshot (167)](https://user-images.githubusercontent.com/52970510/170168085-5d6feb06-60a1-4549-85e2-c134353408ea.png)

Now you can make use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

You will activate load balancer, and enable nginx by setting these in the respective environment???s env-vars file.
```javascript
enable_nginx_lb: true
load_balancer_is_required: true
```
The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

To test this, you can update inventory for each environment and run Ansible against each environment.
  
### Running Ansible Playbook using;
  
```javascript
ansible-playbook -i inventory/uat.yml playbooks/site.yml
```
![Screenshot (170)](https://user-images.githubusercontent.com/52970510/170168566-c1ef6562-c8dc-4728-8a7a-e7197c91e264.png)


  
  
  
  
