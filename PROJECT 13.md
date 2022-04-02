# Ansible Dynamic Assignments (Include) and Community Roles

## Step 1 — Introducing Dynamic Assignment Into Our structure
I started a new branch in "ansible-config-mgt" repo in Github and called it "dynamic-assignments"
I created a new folder under this branch, named it dynamic-assignments.
Then inside this folder, I created a new file and named it env-vars.yml. (Note: We will instruct site.yml to include this playbook later).
My Github now has the below structure:
![image](https://user-images.githubusercontent.com/101482368/161305332-a4bc960d-d3cf-43f3-a742-6d95d5a84205.png)
We will require a way to set values to variables per specific environment. This is because, in configuring multiple environments, we have to consider that each of these environments will have certain unique attributes such as servername, ip-address etc
I created a folder env-vars to keep each environment’s variables file and created new files in it to use to set variables.
I pasted the instruction below into the env-vars.yml file inside dynamic assignments folder
![image](https://user-images.githubusercontent.com/101482368/161307024-f217037c-2ba9-40ca-b86c-80b7c9c7a00e.png)
Notes about script above;
It can be seen that include_vars syntax was used instead of include which as become deprecated. Variants of include_* like include_role, include_tasks, include_vars must be used.
Hence, variants of include_* must be used. They include:
* Similarly, for the same version of Ansible, variants of import such as import_role, import_tasks were also introduced.
* Special variables used in the script: {{ playbook_dir }} will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem.
* {{ inventory_file }} will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.
* The loop used with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.
* I found some other special variables on this site --> https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html

# Step 2 — Update site.yml with dynamic assignments
* I updated site.yml inside playbooks folder to make use of dynamic assignments. site.yml which is the entry point now looks like this 
![image](https://user-images.githubusercontent.com/101482368/161309305-02b97c9d-d0af-4d20-85cd-0dc2346e6e06.png)
* At this point, we need to create a role for MYSQL databse to install the MySQL package, create a database and configure users. 
* This will be achieved with the help of Ansible Galaxy (simply download a ready to use ansible role).
* The plan is to use MySQL role developed by geerlingguy
* I ssh'd into the JenkinsAnsible sever from my Visual Studio and then pulled the ansible-config-mgt repo
* I ran the following command
- git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
- git branch roles-feature
- git switch roles-feature
  ![image](https://user-images.githubusercontent.com/101482368/161314789-c46e0b33-e2a0-4125-b7ec-e63cd713af98.png)
  * inside roles directory, I created my new MySQL role with ansible-galaxy install geerlingguy.mysql and renamed the folder to mysql
  ![image](https://user-images.githubusercontent.com/101482368/161315578-17268056-0aee-4d5d-a718-127938af64db.png)
  ![image](https://user-images.githubusercontent.com/101482368/161315845-db864d94-00e1-4437-b8ea-cbb97cb566d0.png)
  I read the README.md file, and edited roles configuration to use correct credentials for MySQL required for the tooling website.
  ![image](https://user-images.githubusercontent.com/101482368/161318728-c5bcf741-22fb-46cf-b477-7612f9a0b51a.png)
* I uploaded the changes into my GitHub:
* I went through the codes again, after that I created a Pull Request and merged it to master branch on GitHub.
  
# LOAD BALANCER ROLES
* We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively
* I found some available roles from the community and downloaded them
![image](https://user-images.githubusercontent.com/101482368/161327864-38e11831-2f5d-4a3c-bd2a-583dc73670d9.png)
* my roles directory now looks like this
![image](https://user-images.githubusercontent.com/101482368/161328674-a6e9333f-0f82-4635-9e87-d56dac226c3e.png)
* I updated both static-assignment and site.yml files to refer the roles.
* I also added a condition to enable either apach or nginx since both of them cannot be used at the same time. Variables will be used for this.
 *I declared a variable in defaults/main.yml (enable_nginx_lb and enable_apache_lb )file inside the apache and bginx roles and thier values were set to false.
 * Ideclared another variable (lb_is_requuired) in both role with their values set to false
  ![image](https://user-images.githubusercontent.com/101482368/161335849-c2cc7728-d1df-4752-ad01-289282609446.png).
* I created loadbalncers.yml in the Static assignment folder and inserted the script below.
  ![image](https://user-images.githubusercontent.com/101482368/161336653-7295cca0-cf2d-48b0-a04c-b9bd860876cf.png)
* I updated site.yml to reference load balancer roles.
 ![image](https://user-images.githubusercontent.com/101482368/161336879-475caf3d-3245-4f2f-9baa-0294caf017f3.png)
* I made use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.
![image](https://user-images.githubusercontent.com/101482368/161337618-3946822a-54d1-4854-93ce-76e27dfa690f.png)
* When I need to use Apache, I can switch one to true and the other to false.
* I updated inventory for each enironment and ran the playbook against the environments
 

#######################################################
# At the end of this project i was able to learn about 
* Dynamic assigments by using include module. In previous project, I majorly worked with static assignments by using the import module/functionality.
* import = Static
  include = Dynamic
* import: All statements are pre-processed at the time playbooks are parsed. Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered.
* include: All statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.
