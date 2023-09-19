# CI-Jen-Artif-Ans-Sonarqube

CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP
___

SIMULATING A TYPICAL CI/CD PIPELINE FOR A PHP BASED APPLICATION

================

I'll create a CI/CD pipeline in continuation of the previous repo "ANSIBLE_MGT" . 

It will require a lot of servers to simulate all the different environments from dev/ci all the way to production. This will be quite a lot of servers altogether . For now , I'll focus on these environments first:

- Ci
- Dev
- Pentest

Both SIT – For System Integration Testing and UAT – User Acceptance Testing do not require a lot of extra installation or configuration. They are basically the webservers holding our applications. But Pentest – For Penetration testing is where we will conduct security related tests, so some other tools and specific configurations will be needed. In some cases, it will also be used for Performance and Load testing. Otherwise, that can also be a separate environment on its own. It all depends on decisions made by the manager.

What we want to achieve, is having Nginx to serve as a reverse proxy for our sites and tools. Each environment setup is represented in the below table and diagrams.

![CI-environment](/images/CI-Environment.png)


---

## Ansible Inventory should look like this


├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat

ci inventory file

[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_repository-Private-IP-Address>
dev Inventory file

[tooling]
<Tooling-Web-Server-Private-IP-Address>

[todo]
<Todo-Web-Server-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP-Address>
pentest inventory file

[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-for-Todo-Private-IP-Address>

[pentest-tooling]
<Pentest-for-Tooling-Private-IP-Address>

---

Observations:

You will notice that in the pentest inventory file, we have introduced a new concept pentest:children This is because, we want to have a group called pentest which covers Ansible execution against both pentest-todo and pentest-tooling simultaneously. But at the same time, we want the flexibility to run specific Ansible tasks against an individual group.

The db group has a slightly different configuration. It uses a RedHat/Centos Linux distro. Others are based on Ubuntu (in this case user is ubuntu). 

Therefore, the user required for connectivity and path to python interpreter are different. If all your environment is based on Ubuntu, you may not need this kind of set up. Totally up to you how you want to do this. Whatever works for you is absolutely fine in this scenario.

This makes us to introduce another Ansible concept called group_vars. With group vars, we can declare and set variables for each group of servers created in the inventory file.

For example, If there are variables we need to be common between both pentest-todo and pentest-tooling, rather than setting these variables in many places, we can simply use the group_vars for pentest. Since in the inventory file it has been created as pentest:children Ansible recognizes this and simply applies that variable to both children.

#### ANSIBLE ROLES FOR CI ENVIRONMENT

Create 2 roles

1. SonarQube - for continuous testing and inspection of code quality

2. Artifactory:  

##### Why do we need Artifactory?

Artifactory is a product by JFrog that serves as a binary repository manager. The binary repository is a natural extension to the source code repository, in that the outcome of your build process is stored. It can be used for certain other automation, but we will it strictly to manage our build artifacts.

Configuring Ansible For Jenkins Deployment
In previous projects, you have been launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.

To do this,

### Navigate to Jenkins URL

Install & Open Blue Ocean Jenkins Plugin

Create a new pipeline
![blue ocean](/images/blueocean-installed.png)

Create a jenkinsfile to build the Jenkinsfile gradually. This pipeline currently has just one stage called **Build** and the only thing we are doing is using the shell script module to echo Building Stage.

![build](/images/build.png)

![jenkinsfile](/images/jenkinsfile.png)

---
Now go back into the Ansible pipeline in Jenkins, and select configure

Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile

This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

![blueocean-cicd/test](/images/blueocean-cicd.png)

---

##### RUNNING ANSIBLE PLAYBOOK FROM JENKINS

Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

- Installing Ansible on Jenkins
- Installing Ansible plugin in Jenkins UI
- Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)

You can watch a 10 minutes video here to guide you through the entire setup

Note: Ensure that Ansible runs against the Dev environment successfully.

Possible errors to watch out for:

> Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of master (GitHub has discontinued the use of te word *Master*)

Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the .ansible.cfg file alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set.

---

Create a new git branch and name it `feature/jenkinspipeline-stages`
Currently we only have the Build stage. I'll add another stage called Test. 

update your jenkinsfile with a 2nd stage `code block` to create the next stage of the pipeline - *TEST* and push the changes to GitHub.


```
pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}

```
![scan-repo](/images/scan-repo.png)

![blueOcean-multiBranchPipeline](/images/blue-oceanmultibranch-test.png)

To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

Click on the "Administration" button and scan repository on the Jenkins project Dashboard


Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)
   1. Package 
   2. Deploy 
   3. Clean up
5. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch
6. Eventually, your main branch should have a successful pipeline like this in blue ocean
____________
---

##### RUNNING ANSIBLE PLAYBOOK FROM JENKINS

You probably now have a better overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

- Installing Ansible on Jenkins
- Installing Ansible plugin in Jenkins UI
- Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)

You can watch a 10 minutes video here to guide you through the entire setup

[Ansible-JenkinsVideo](https://www.youtube.com/watch?v=PRpEbFZi7nI)!


Create a new jenkinsfile or do it directly on Jenkins Dashboard.

Generate pipeline code snippets to checkout SCM for GITHUB and Ansible.

![git](/images/git-pipelineGenerator.png)
------
![ansible](/images/pipelineSyntax.png)

---
=======

Possible errors to watch out for:

Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of masters.

Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the .ansible.cfg file alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set.



================

Possible issues to watch out for when you implement this

Remember that `ansible.cfg` must be exported to environment variable so that Ansible knows where to find Roles. But because you will possibly run Jenkins from different git branches, the location of Ansible roles will change. Therefore, you must handle this dynamically. You can use Linux Stream Editor sed to update the section roles_path each time there is an execution. You may not have this issue if you run only from the main branch.

If you push new changes to Git so that Jenkins failure can be fixed. You might observe that your change may sometimes have no effect. Even though your change is the actual fix required. This can be because Jenkins did not download the latest code from GitHub. Ensure that you start the Jenkinsfile with a clean up step to always delete the previous workspace before running a new one. 

Sometimes you might need to login to the Jenkins Linux server to verify the files in the workspace to confirm that what you are actually expecting is there. Otherwise, you can spend hours trying to figure out why Jenkins is still failing, when you have pushed up possible changes to fix the error.

Another possible reason for Jenkins failure sometimes, is because you have indicated in the Jenkinsfile to check out the main git branch, and you are running a pipeline from another branch. So, always verify by logging onto the Jenkins box to check the workspace, and run git branch command to confirm that the branch you are expecting is there.

If everything goes well for you, it means, the Dev environment has an up-to-date configuration. But what if we need to deploy to other environments?

Are we going to manually update the Jenkinsfile to point inventory to those environments? such as sit, uat, pentest, etc.
Or do we need a dedicated git branch for each environment, and have the inventory part hard coded there.

Think about those for a minute and try to work out which one sounds more like a better solution.

Manually updating the Jenkinsfile is definitely not an option. And that should be obvious to you at this point. Because we try to automate things as much as possible.

Well, I wont be doing any of the highlighted options. What we will be doing is to parameterise the deployment. So that at the point of execution, the appropriate values are applied.









Moved to Reverse Proxy