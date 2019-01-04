# cd-demo-jd
A scale and continuous delivery demo using Jenkins on DC/OS.
- Props to the original creator, but now this is updated for the latest DC/OS (1.12), Mesosphere Github repo access and Docker credentials

## Current Possibilities
1. Manual Deployment
- Git build, Docker pull and automatic Jenkins deploy
- Show that this is a functional Jenkins

![Show Output](URL)

2. Scaled Deployment
- Show that this is a scalable Jenkins solution by executing 50 jobs

![Show Output](URL)

### Exercise 0. Prerequisites
- I just wish this wasn't so bad
#### Step 0.1 - Only needed for Manual Deployment (First run only)
1. Clone cd-demo to your unit and create/sync it to your Github repo
- Command: `git clone https://github.com/jdyver/cd-demo.git`

#### Step 0.2 - Additional Steps Needed for Scaled Deployment
Item 1. Setup Python3 and Pip3 (First run only needed)

    - painful 30 minute installation - Just google away to get it right # I'll probably try to change this to bash later

    a. OSX example: https://docs.python-guide.org/starting/install3/osx/

    b. Centos example: https://www.rosehosting.com/blog/how-to-install-python-3-6-4-on-centos-7/

    c. Pip3 Setup (CentOS): sudo ln pip3.6 pip3
    
Item 2. Pip3: Install requirements

    `pip3 install -r requirements.txt`
    
    a. CentOS (not logged in as root): Add sudo

Item 3. Github Create repo/branch 
- Everytime - check cd-demo folder with 'git remote -v' or 'git branch'

    a. Manually create new repo within your Github account

    b. git clone https://github.com/jdyver/cd-demo-jd.git

    c. cd cd-demo

    d. git remote set-url upstream https://github.com/<Your Username>/<New Repo Name>

    e. git push -u upstream master

    f. Skip to Step 4

    - If using from locally forked repository

        a. git checkout -b testing

        b. git push origin testing    

        c. (if push fails)
    
        - Setup Git SSH Key (https://help.github.com/articles/generating-an-ssh-key/)    
        - (ssh -T git@github.com success, but push still fails)  
            `git remote set-url origin git@github.com:jdyver/cd-demo.git` (your repo)

4. Dockerhub:             

    - Ensure that you have a dockerhub account and you know the username and password


### Exercise 1. Edit - Manual Git build, Docker pull and Jenkins deploy
0. Prerequisite 0.1

1. Setup Application
    
    a. Single cluster configured for CLI
    - If doing the 50 job (Section 2), go ahead and clear the DCOS profiles

    `dcos cluster remove --all`

    - Connect to the cluster
    `dcos cluster add https://<your url>`

    b. Install Marathon-LB
    - Get MLB IP 

    c. Install Jenkins

1. Within the Github repo:
    - Edit file: conf/cd-demo-app.json - line 21
        - "HAPROXY_0_VHOST": "\<Public Agent IP\>",

![Edit HAPROXY](https://github.com/jdyver/cd-demo-jd/blob/master/img/Jenkins-Deployed-App-HAPROXY.png)



2. Within Jenkins:
- Open the Jenkins UI from DC/OS
- Credentials > System > Global Credentials > Add Credentials (Input Description): Add Github and Dockerhub accounts

! Decscription at end

![Jenkins - Credentials](URL)

- Select New Job:
    - Select Freestyle and give it a name
    - Source Code Mgmt Section Git:
        1. Repo URL: `https://github.com/jdyver/cd-demo` (Your Git URL)
        2. Credentials: `Github`

![Jenkins - Source Code](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-1.png)

- Auto Build (1 minute)
    - Project Name > Configure > Build Triggers > Poll SCM
    - Input: `* * * * *`

![Jenkins - Polling](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-2.png)

- Build > [Add Build Step] > Docker Build and Publish > 
    - Repo Name: `jdyver/cd-demo` (Your Docker repo username + '/cd-demo')
    - Tag: `$GIT_COMMIT`
    - Registry credentials: `Dockerhub`

![Jenkins - Build](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-3.png)

-  Post-build Actions > [Add post-build action] > Marathon Deployment > [Advanced] >
    - Marathon URL: `http://leader.mesos:8080`
    - Definition File: `conf/cd-demo-app.json`
    - Docker Image: `jdyver/cd-demo:$GIT_COMMIT`

![Jenkins - Post-build](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-5.png)

- Hit save

3. Github Repo: edit site/_posts/2017-12-25-welcome-to-cd-demo.markdown
    - edit some text

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

4. Once the minute poll from Jenkins completes it will see the commit and (re)deploy the Jenkins_Deployed_App

![CD Webpage Output](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-4.png)

5. Open a tab and go to the Marathon-LB's \<Master_IP\>

- If it doesn't open either the jenkins_deployed_app isn't completely up or the port (HAPROXY) wasn't updated so go to MLB to pull what port it is running on

![CD Webpage Output](https://github.com/jdyver/cd-demo-jd/blob/master/img/CD-Demo-Output.png)

### Exercise 2. Deploy 50 Jobs
0. Prerequisites 0.1 and 0.2

- 0.2 Has to be done/checked every time

    `git branch`

1. Single cluster configured for CLI

     `dcos cluster remove --all`

     `dcos cluster add http://<your url>`

2. Install Jenkins

     a. Through CLI or UI

     `cd-demo jd$ dcos package install --yes jenkins`

     b. DO NOT DO - JUST FOR NOTES; OPTION (Do not do this if you do not do 2a) 

     — <b>slash at the end of the URL!</b>

     `cd-demo jd$ python3 bin/demo.py install --latest http://jdyckowsk-elasticl-108ld3uvv6r15-1683503373.us-west-2.elb.amazonaws.com/`

     — <b>slash at the end of the URL!</b>

3. Run script

— <b>slash at the end of the URL!</b>

`cd-demo jd$ python3 bin/demo.py dynamic-agents http://jdyckowsk-elasticl-108ld3uvv6r15-1683503373.us-west-2.elb.amazonaws.com/`

— <b>slash at the end of the URL!</b>

![Python Dynamic-Agents Output](https://github.com/jdyver/cd-demo-jd/blob/master/img/Dynamic-Agents-Running.png)

4. Now in action, go back to the Jenkins UI

![Jenkins - 50 Jobs](https://github.com/jdyver/cd-demo-jd/blob/master/img/Jenkins-50-Finale.png)

What you see are 50 jobs that have been created through automation and are randomly timed to fail/succeed within 2 minutes.
- Rerun for a more mixed view of jobs that have succeeded, failed/succeeded or just always failed.

5. In the DC/OS UI, you can select the Jenkins Service and see the Jenkins executors scale to manage these 50 jobs.



### TBD (In order of priority)
- demo.py: Change / remove DC/OS CLI profile check
- demo.py: Change to bash (basically a rewrite)

# --------------------------------------------------------------------
