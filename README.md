# cd-demo-jd
A scale and continuous delivery demo using Jenkins on DC/OS.
- Now updated for adding the Docker build and publish 
- Works with the latest DC/OS (1.12), Mesosphere Github repo access and Docker credentials

## What We Are Showing
### Exercise 1. Manual Setup of Jenkins Configuration for an Automated CI/CD Workflow 
- Git push, then automated Jenkins Git build,  pull to Dockerhub and deployed app into DCOS.
- Shows the DCOS open source Jenkins is functional and that DCOS can manage the automated resource handling in literally showing Continuous Integration and Continuous Deployment.

![DCOS Deployed Jenkins to CICD](https://github.com/jdyver/cd-demo-jd/blob/master/img/CD-Intro.png)

### Exercise 2. Scaled Jenkins Deployment - Deploy 50 Jobs
- Show that this is a scalable Jenkins solution by executing 50 jobs with dynamically created task executors based on the available resources from DCOS.

![DCOS - Jenkins Service Jobs](https://github.com/jdyver/cd-demo-jd/blob/master/img/DCOS-ServiceJenkinsOverview.png)

## Prerequisites
### Exercise 1 Prerequisites (First run only)

#### Item 1. Clone cd-demo to your unit and create/sync it to your Github repo
- Command: `git clone https://github.com/jdyver/cd-demo-jd.git`

### Exercise 2 Prerequisites

#### Item 1. Setup Python3 and Pip3 (First run only needed)

 - painful 30 minute installation - Just google away to get it right # I'll probably try to change this to bash later

 a. OSX example: https://docs.python-guide.org/starting/install3/osx/

 b. Centos example: https://www.rosehosting.com/blog/how-to-install-python-3-6-4-on-centos-7/

#### Item 2. Pip3: Install requirements

```
pip3.6 install -r requirements.txt
```

 a. CentOS (not logged in as root): Add sudo

#### Item 3. Github Create repo/branch 
- Everytime - check cd-demo folder with 'git remote -v' or 'git branch'

 a. Manually create new repo within your Github account

 b. git clone https://github.com/jdyver/cd-demo-jd.git

 c. cd cd-demo

 d. git remote set-url upstream https://github.com/<Your Username\>/\<New Repo Name\>
- OR just git remote set-url --push origin https://github.com/<Your Username\>/New_Repo_Name

 e. git push -u upstream master

 f. Skip to Step 4

 - If using from locally forked repository

      a. git checkout -b testing

      b. git push origin testing    

      c. (if push fails)
       - Setup Git SSH Key (https://help.github.com/articles/generating-an-ssh-key/)    
       - (ssh -T git@github.com success, but push still fails)  
            `git remote set-url origin git@github.com:jdyver/cd-demo.git` (your repo)

#### Item 4. Dockerhub:             

- Ensure that you have a dockerhub account and you know the username and password


## Exercise 1. Manual Setup of Jenkins Configuration for an Automated CI/CD Workflow
- Manual Setup of Jenkins to Automate  Git build, Docker pull and Jenkins deploy

### Step 0. Follow Prerequisite Section 1

### Step 1. Prepare DCOS
    
 a. Single cluster configured for CLI
 
 - If doing the 50 job (Section 2), go ahead and clear the DCOS profiles

    `dcos cluster remove --all`

 - Connect to the cluster
    
    `dcos cluster add https://<your url>`

 b. Install Marathon-LB
 
 - Get MLB IP 

 - (Option: Edge-LB; Thanks AF; To be tested)

Edge-LB-JenkinsOPT.json
```
{
    "apiVersion": "V2",
    "name": "edgelb-proxy-jenkins",
    "count": 1,
    "autoCertificate": true,
    "haproxy": {
        "frontends": [
          {
                "bindPort": 11080,
                "protocol": "HTTP",
                "linkBackend": {
                    "defaultBackend": "cicd"
                }
            }
        ],
        "backends": [
            {
                "name": "cicd",
                "protocol": "HTTP",
                "services": [
                    {
                        "marathon": {
                            "serviceID": "/jenkins-deployed-app"
                        },
                        "endpoint": {
                            "portName": "jenkins-deployed-app"
                        }
                    }
                ]
            }
        ],
        "stats": {
            "bindPort": 6090
        }
    }
}
```

 c. Install Jenkins
 - Demo preference: Version 3.5.2-2.107.2 or older
 ```
jamess-mbp:cd-demo jd$ dcos package install jenkins --package-version=3.5.2-2.107.2 --yes
By Deploying, you agree to the Terms and Conditions https://mesosphere.com/catalog-terms-conditions/#certified-services
WARNING: If you didn't provide a value for `storage.host-volume` (either using the CLI or via the Advanced Install dialog),
YOUR DATA WILL NOT BE SAVED IN ANY WAY.

Installing Marathon app for package [jenkins] version [3.5.2-2.107.2]
Jenkins has been installed.
 ```

 - Latest: Version 3.5.4-2.150.1 or later
 ```
 jamess-mbp:cd-demo jd$ dcos package install jenkins --yes
 ```

### Step 2. Setup cd-demo's JSON:
 - Edit file: conf/cd-demo-app.json - line 21
     - "HAPROXY_0_VHOST": "\<Public Agent IP\>",

![Edit HAPROXY](https://github.com/jdyver/cd-demo-jd/blob/master/img/Jenkins-Deployed-App-HAPROXY2.png)

### Step 3. Prepare Jenkins:
- Open the Jenkins UI from DC/OS
- Credentials > System > Global Credentials > Add Credentials: Add Dockerhub account (Input Description)

![Jenkins - Credentials](https://github.com/jdyver/cd-demo-jd/blob/master/img/Jenkins-Credentials.png)

- UPDATE! IF Version 3.5.2-2.107.2 or older; Skip this step (IF you have no idea, then do this step)
    - Add Pipeline: Step API plugin within Jenkins
    - Manage Jenkins > [Scroll Down] Manage Plugins > [Select] Pipeline: Step API > Select [Download now and install after restart] > Select [Restart after update]
    - Detailed steps: https://github.com/jdyver/cd-demo-jd/tree/master/notes

- Setup New Job:
    - Select New Item (New Job), Freestyle project, give it a name and select OK at the bottom

- Setup Github 
    - Select Source Code Management > Git >
    - Repo URL: `https://github.com/jdyver/cd-demo` (Your Github URL for the cd-demo repo)

![Jenkins - Source Code](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-1.png)

- Setup Auto Build (polls every 1 minute for build changes)
    - Project Name > Configure > Build Triggers > Poll SCM >
    - Input: `* * * * *`

![Jenkins - Polling](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-2.png)

- Setup Dockerhub
    - Select Build > [Add Build Step] > Docker Build and Publish > 
    - Repo Name: `jdyver/cd-demo` (Your Docker repo username + '/cd-demo')
    - Tag: `$GIT_COMMIT`
    - Registry credentials: `Dockerhub`

![Jenkins - Build](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-3.png)

-  Setup Marathon
    - Select Post-build Actions > [Add post-build action] > Marathon Deployment > [Advanced] >
    - Marathon URL: `http://leader.mesos:8080`
    - Definition File: `conf/cd-demo-app.json` (Option: null; Update: marathon.json)
    - Docker Image: `jdyver/cd-demo:$GIT_COMMIT`

![Jenkins - Post-build](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-5.png)

- Hit save

### Exercise 1 Demo is now setup; To show the demo....

### Step 4. Github Repo: edit site/_posts/2017-12-25-welcome-to-cd-demo.markdown
 - edit some descriptive text

```
jamess-mbp:cd-demo jd$ cat site/_posts/2017-12-25-welcome-to-cd-demo.markdown << EOF
---
layout: post
title:  "Welcome to James' Continuous Delivery Demonstration!"
categories: demo
---

This is an example post that you can make using Markdown to demonstrate a website being statically generated and deployed!  ...and a belated Merry Christmas!
EOF
```

### Step 5. Jenkins deploys the Jenkins_Deployed_App
- At a high level, once the minute poll from Jenkins completes... (Can view from 'Console Output' within the cd-demo project)
    - Jenkins checks Github if there are any commits.
    - Jenkins builds the container image in the DCOS sandbox and tags as latest.
    - Jenkins pushes container image to Dockerhub.
    - Jenkins tells Marathon (DCOS) to update/deploy the 'jenkins_deployed_app' container from Dockerhub to a DCOS node.

- In the background, DCOS:
    - Gives resources to a Jenkins executor to run the job.
    - Gives resources to the newly deployed jenkins-deployed-app.
    - Marathon-LB automatically discovers the app and manages web requests via port 80.

![CD Webpage Output](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-4.png)

### Step 6. Open a tab and go to the Marathon-LB's \<Public_IP\>
- Here you can see that the jenkins_deployed_app is deployed
    - Marathon-LB automatically picked up the service port to serve the site at port 80
        - Note: If it doesn't open either the jenkins_deployed_app isn't completely up or the port (HAPROXY) wasn't updated so go to MLB to pull what port it is running on (...or go back and follow step 2, then commit).

![CD Webpage Output](https://github.com/jdyver/cd-demo-jd/blob/master/img/CD-Demo-Output.png)

## Exercise 2. Scaled Jenkins Deployment - Deploy 50 Jobs
### Step 0. Prerequisites 0.1 and 0.2

 - 0.2 Has to be done/checked every time

    `git branch`

### Step 1. Single cluster configured for CLI
 - Clear out the old/other cluster profiles (This requirement to be removed)

    `dcos cluster remove --all`

- Add the active cluster (Works with https also)

    `dcos cluster add http://<your url>`

### Step 2. Install Jenkins
 a. Through CLI or UI

```
cd-demo jd$ dcos package install --yes jenkins
```

 b. DO NOT DO - JUST FOR NOTES; OPTION (Do not do this if you do not do 2a) 

 — <b>slash at the end of the URL!</b>

```
cd-demo jd$ python3 bin/demo.py install --latest $(dcos config show core.dcos_url)/
```

Actual Input:
```
JD # echo "python3 bin/demo.py dynamic-agents $(dcos config show core.dcos_url)/"

python3 bin/demo.py dynamic-agents https://jdyckowsk-elasticl-y6v3kcwpkpoz-1132299876.us-west-2.elb.amazonaws.com/
```

 — <b>slash at the end of the URL!</b>

### Step 3. Run the cd-demo 50 job script
 — <b>slash at the end of the URL!</b>

```
cd-demo jd$ python3 bin/demo.py dynamic-agents $(dcos config show core.dcos_url)/
```

 — <b>slash at the end of the URL!</b>

![Python Dynamic-Agents Output](https://github.com/jdyver/cd-demo-jd/blob/master/img/cd-demo_dynamic-agents.png)

### Exercise 2 Demo is now setup...

### Step 4. Go to the Jenkins UI

 - What you see are 50 jobs that have been created through automation and are randomly timed to fail/succeed within 2 minutes.
     - Rerun for a more mixed view of jobs that have succeeded (Sunny icon), failed/succeeded (Cloudy icon)  or just always failed (Lightning Cloud icon).

![Jenkins - 50 Jobs](https://github.com/jdyver/cd-demo-jd/blob/master/img/Jenkins-50-Finale.png)

### Step 5. Show Jenkins Executor Scaling on DCOS
 - In the DCOS UI, you can select the Jenkins Service and see the Jenkins executors dynamically scale out and then back in to manage these 50 jobs based on the available resources of the DCOS cluster.  In the example outputs below, there are limited resources (Example 1) or more unlimited resources (Example 2) and shown by Jenkins deploying more executors automatically within DCOS based on the demand.

 - Example 1: Below shows Jenkins running on a single node of resources so automatically deploying 4 executors within DCOS

![Jenkins with Single Node Resources](https://github.com/jdyver/cd-demo-jd/blob/master/img/DCOS-ServiceJenkins1.png)

 - Example 2: Below shows Jenkins running on more resources so automatically deploying 10 executors within DCOS

![Jenkins with Multiple Node Resources](https://github.com/jdyver/cd-demo-jd/blob/master/img/DCOS-ServiceJenkins2.png)

 - Once all of the jobs complete; Jenkins kills the executors and DCOS clears the resources for more / other jobs

![Jenkins Killed Executors](https://github.com/jdyver/cd-demo-jd/blob/master/img/DCOS-ServiceKilledJenkins3.png)

## TBD (In order of priority) ------------------------------------
- demo.py: Change / remove DC/OS CLI profile check
- demo.py: Change to bash (basically a rewrite)
## -------------------------------------------------------------
