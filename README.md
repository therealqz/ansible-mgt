# ansible-mgt
Continuous Integration /CD to the pipeline with  ansible. & MORE

#####
ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

---
I'll refactor the  Ansible code, create assignments, and use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – 
The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

***

#### STEP 1
 1. Jenkins job enhancement
 ___

Make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require Copy Artifact plugin.

In the  Jenkins-Ansible server , I'll create a new directory called ansible-config-artifact – We'll store all artifacts there after each build.

`sudo mkdir /home/ubuntu/ansible-mgt-artifact`

Change permissions to this directory, so Jenkins could save files there – `chmod -R 0777 /home/ubuntu/ansible-mgt-artifact`

Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

![Jenkins-Copy plugin](images/jenkins-copyartifacts.png)

screenshoT **'

Create a new Freestyle project (you have done it in Project 9) and name it save_artifacts.

This project will be triggered by completion of your existing ansible project. Configure it accordingly: 


screenshots 
![jenkins-save](images/Screenshot%202023-08-25%20at%2023.08.03.png)

Note: You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

The main idea of save_artifacts project is to save artifacts into `/home/ubuntu/ansible-config-artifact` directory. 

To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

![Build-Steps](images/build-step.png)

---

Test the set up by making some change in README.MD file inside your ansible-mgt repository (right inside master branch).
If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.

Now your Jenkins pipeline is neater


Verify the presence of the artifacts on the server .

![verify-artifacts](images/verify-artifacts.png)
---

#### 
REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML
_____________
Step 2 – 

Refactor Ansible code by importing other playbooks into site.yml




Earlier we created a single playbook common.yml-a simple set of instructions for only 2 types of OS, but when you have many more tasks and you need to apply this playbook to other servers with different requirements. In this case, you will have to read through the whole playbook to check if all tasks written there are applicable . and is there anything that you need to add for certain server/OS families. Very fast it will become a tedious exercise and your playbook will become messy with many commented parts. Your DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use your playbook.

Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Let see code re-use in action by importing other playbooks.

1. Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. 

2. Create a new folder in root of the repository and name it `static-assignments`

 The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. 

3. Move common.yml file into the newly created static-assignments folder.

4. Inside site.yml file, import common.yml playbook.

![ImportCommon](images/importCommon.png)

Your folder structure should now loook like this . The `tree` command helps.

![Tree](images/tree.png)


5. Run ansible-playbook command against the dev environment

Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it `common-del.yml`. 

 In this playbook, configure deletion of wireshark utility.


 
 
 `- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
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
      autoclean: yes`

update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers:

`cd /home/ubuntu/ansible-mgt/`

`ansible-playbook -i inventory/dev.yml playbooks/site.yaml`

> Make sure that wireshark is deleted on all the servers by running wireshark --version

Now you have learned how to use import_playbooks module and you have a ready solution to install/delete packages on multiple servers with just one command

Before running your playbook, you can pass a `--check` flag to your command to do a validation first.

![Delete Wireshark](images/delete-wireshark.png)

---
________________

CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’

**Step 3** – Configure UAT Webservers with a role ‘Webserver’
---





We a clean `dev` environment, so let us put it aside and configure 2 new Web Servers as UAT. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.

Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.

Tip: Do not forget to stop EC2 instances that you are not using at the moment to avoid paying extra. For now, you only need 2 new RHEL 8 servers as Web Servers and 1 existing Jenkins-Ansible server up and running.

To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.
There are two ways how you can create this folder structure:

Use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory (you need to create roles directory upfront)

`mkdir roles`
`cd roles`
`ansible-galaxy init webserver`

Create the directory/files structure manually
Note: You can choose either way, but since you store all your codes in GitHub, it is recommended to create folders and files there rather than locally on Jenkins-Ansible server.

The entire folder structure should look like below, but if you create it manually – you can skip creating tests, files, and vars or remove them if you used ansible-galaxy

![Tree](images/roles-tree.png)

---

#### 

REFERENCE THE WEBSERVER ROLE


Within the **static-assignments** folder, create a new assignment for uat-webservers `uat-webservers.yml` This is where you will reference the role.

![Reference-Role](images/reference-Roles.png)

Remember that the entry point to our ansible configuration is the `site.yml` file. Therefore, you need to refer your `uat-webservers.yml` role inside `site.yml`

![]()

#### Step 5 – Commit & Test

Commit your changes, create a Pull Request and merge them to master branch, **_make sure webhook triggered two consequent Jenkins jobs_**,f they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.



Now run the playbook against your uat inventory and see what happens:

`sudo ansible-playbook -i /home/ubuntu/ansible-config-artifact/inventory/uat.yml /  home/ubuntu/ansible-config-artifact/playbooks/site.yml`

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:


tests and screenshots.

![play](/images/Playbook-works.png)

![uat-webserver](/images/website-dir.png)

![website](/images/website.png)

AWS dashboard showing the servers

![aws-dashboard](/images/aws-console.png)

Day I completed this.
![date](/images/project-date.png)

Now the achitecture looks like this .

[Architecture](/images/architecture.png)






