# ansible-mgt
Continuous Integration of code to the pipeline with  ansible. & MORE

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

![Jenkins-Copy plugin](/jenkins-copyartifacts.png)

screenshoT **'

Create a new Freestyle project (you have done it in Project 9) and name it save_artifacts.

This project will be triggered by completion of your existing ansible project. Configure it accordingly: 


screenshots 
![jenkins-save](/Screenshot%202023-08-25%20at%2023.08.03.png)

Note: You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

The main idea of save_artifacts project is to save artifacts into `/home/ubuntu/ansible-config-artifact` directory. 

To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

![Build-Steps](/build-step.png)

---

Test the set up by making some change in README.MD file inside your ansible-mgt repository (right inside master branch).
If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.

Now your Jenkins pipeline is neater

---

