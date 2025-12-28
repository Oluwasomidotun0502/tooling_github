# Tooling Website Deployment Automation with Continuous Integration (CI) using Jenkins


## Introduction

In previous projects, we implemented horizontal scalability by deploying multiple web servers behind a load balancer for the Tooling Website. While manually configuring two or three servers is manageable, this approach does not scale efficiently when infrastructure grows to dozens or hundreds of servers.

DevOps focuses on agility, repeatability, and fast delivery of software. One of the key ways to achieve this is through automation of routine deployment tasks using Continuous Integration (CI) tools.

In this project, we introduce Jenkins, a free and open-source automation server, to automate the deployment of application source code from GitHub to a shared NFS storage, ensuring that every code change is automatically propagated to all web servers.

Jenkins was originally developed by Kohsuke Kawaguchi at Sun Microsystems and was initially named Hudson. Today, it is one of the most widely used CI/CD tools in the DevOps ecosystem.

According to CircleCI, Continuous Integration (CI) is a software development practice where developers frequently commit code to a shared repository, and each change is automatically built and tested to ensure code quality and faster delivery.

## Project Objective

The goal of this project is to enhance the existing Tooling Website architecture by:

- Introducing a Jenkins CI server

- Automatically pulling code changes from GitHub using webhooks

- Archiving build artifacts in Jenkins

- Deploying updated application files to a central NFS server

- Ensuring all web servers receive updates automatically via the shared NFS mount


## Updated Architecture Overview

### Components:

- Client (Browser)

- GitHub Repository (tooling)

- Jenkins Server (CI Engine)

- NFS Server (Shared Storage)

- Load Balancer (Apache)

- Web Server 1

- Web Server 2

- MySQL Database Server

### Traffic Flow:

- Client traffic → Load Balancer (TCP 80)

- Load Balancer → Web Servers (TCP 80)

- Web Servers → Database (TCP 3306)

- Jenkins → GitHub (Webhook)

- Jenkins → NFS Server (SSH / TCP 22)

- Web Servers → NFS Server (NFS TCP/UDP 2049 & 111)

![1-diagram.png](images/1-diagram.png)


## Step 1: Install and Configure Jenkins Server

### 1. Create Jenkins EC2 Instance

Launch an Ubuntu Server 20.04 LTS EC2 instance on AWS

Name the instance: Jenkins

![01-jenkins-instance](images/01-jenkins-instance.png)

### 2. Install Java (JDK)

Jenkins is a Java-based application.

```
sudo apt update
sudo apt install default-jdk-headless -y
```
![02-jenkins-install-update.png](images/02-jenkins-install-update.png)

![03-jenkins-install.png](images/03-jenkins-install.png)

### 3. Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
```
![04-jenkins-installlation.png](images/04-jenkins-installation.png)

![05-jenkins-key-error.png](images/05-jenkins-key-error.png)

![06-jenkins-key-issue-fix.png](images/06-jenkins-key-issue-fix.png)

![07-jenkins-update.png](images/07-jenkins-update.png)

![08-jenkins-successful-install.png](images/08-jenkins-successful-install.png)


### 4. Start and Verify Jenkins Service

```
sudo systemctl status jenkins
```
![09-jenkins-status-error.png](images/09-jenkins-status-error.png)

![10-jenkins-error-log-older=version.png](images/10-jenkins-error-log-older=version.png)

![11-jenkins-version17-install.png](images/11-jenkins-version17-install.png)

![12-java-version-version-confirm.png](images/12-java-version-confirm.png)

![13-jenkisp-running.png](images/13-jenkinsp-running.png)


### 5. Open Jenkins Port

Open TCP port 8080 in the Jenkins EC2 Security Group inbound rules

![001-port.png](images/001-port.png)


### 6. Initial Jenkins Setup

Access Jenkins from your browser:

http://<JENKINS_PUBLIC_IP>:8080

![14-jenkins-web-test.png](images/14-jenkins-web-test.png)

Retrieve the initial admin password:

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
![15-initialAdminPassword.png](images/15-initialAdminPassword.png)

![16-password-login-page.png](images/16-password-login-page.png)


Install Suggested Plugins

![17-plugins-pages.png](images/17-plugins-page.png)


Create an Admin User

![18-create-user-login.png](images/18-create-user-login.png)

![19-ready-jenkins.png](images/19-ready-jenkins.png)

Complete setup

![20-homepage.png](images/20-homepage.png)


## Step 2: Configure Jenkins to Retrieve Code from GitHub Using Webhooks

### 1. Enable GitHub Webhook

In your GitHub repository:

Go to Settings → Webhooks

Add Jenkins webhook URL:

http://<JENKINS_PUBLIC_IP>:8080/github-webhook/

![21-githun-webhook-config.png](images/21-githun-webhook-config.png)

![38-web-hook-get.png](images/38-web-hook-get.png)

### 2. Create Jenkins Freestyle Project

Jenkins Dashboard → New Item

Project Name: tooling_github

Project Type: Freestyle Project

![22-jenkins-config.png](images/22-jenkins-config.png)

![23-jenkins-new-item-setup.png](images/23-jenkins-new-iten-setup.png)


### 3. Configure Source Code Management

Select Git

Repository URL:

https://github.com/<your-github-username>/tooling.git

![24-jenkins-source-config.png](images/24-jenkins-source-config.png)

Branch Specifier:

*/master

Add GitHub credentials if required

Save the configuration.


### 4. Run Initial Build

Click Build Now

![25-build-now.png](images/25-build-now.png)

If successful, Jenkins will:

Pull source code from GitHub

![26-build-status.png](images/26-build-status.png)

Create a build

Store workspace files locally

Verify build success via Console Output.


### 5. Enable Webhook Trigger & Artifact Archiving

Edit project configuration:

Build Triggers

✔ GitHub hook trigger for GITScm polling

Post-build Actions

✔ Archive the artifacts

Files to archive: **


Save configuration.

Now, any push to GitHub automatically triggers a Jenkins build.


### Step 3: Deploy Artifacts to NFS Server via SSH

### 1. Install “Publish Over SSH” Plugin

Jenkins Dashboard → Manage Jenkins

Manage Plugins → Available

Search: Publish Over SSH

Install (without restart)

![27-jenkins-config-to-copy-nfs-files.png](images/27-jenkins-config-to-copy-nfs-files.png)

![28-publish-over-ssh-config.png](images/28-publish-over-ssh-config.png)

### 2. Configure Publish Over SSH

Go to:
Manage Jenkins → Configure System

Under Publish over SSH:

SSH Server Name: NFS-Server

Hostname: <NFS_PRIVATE_IP>

Username: ec2-user

Remote Directory: /mnt/apps

Paste private key (PEM file contents)

Test connection → Success

![29-test-config-successful.png](images/29-test-config-successful.png)

Save configuration.


### 3. Configure Job to Deploy Files to NFS

Edit Jenkins job → Post-build Actions

Add:

✔ Send build artifacts over SSH

Transfer Set Configuration:

Source files: **

Remote directory: /mnt/apps

![30-configure-jenkins-job.png](images/30-configure-jenkins-job.png)

![31-config.png](images/31-config.png)

![32-config.png](images/32-config.png)

![33-config.png](images/33-config.png)

Save configuration.


## Validation & Testing

Make a change to README.md in GitHub

Push changes to master

GitHub webhook triggers Jenkins build

Jenkins:

Pulls updated code

Archives artifacts

Copies files to NFS /mnt/apps

![34-png](images/34.png)

Verify on NFS server:

cat /mnt/apps/README.md

![39-trigger-check.png](images/39-trigger-check.png)

![37-output.png](images/37-output.png)

![40-github-update.png](images/40-github-update.png)

If changes are reflected, the CI pipeline is working correctly.


### Challenges Faced and Resolutions

During the implementation of this CI automation project, several real-world issues were encountered. Resolving these issues provided deeper insight into Jenkins, Linux systems, and CI troubleshooting.

### 1. Jenkins Failed to Start Due to Java Version Mismatch

### Problem:

After installing Jenkins, the service failed to start. Checking the service status and logs revealed that Jenkins was incompatible with the installed Java version.

### Evidence:

Jenkins service status showed startup failure

Error logs indicated an unsupported or older Java version

Command used to verify Jenkins status:
```
sudo systemctl status jenkins
```

Command used to inspect logs:

```
sudo journalctl -u jenkins --no-pager
```
The logs showed that Jenkins required Java 17 or higher, but an older Java version was installed.


## Solution:

Step 1: Check Current Java Version

```
java -version
```
Step 2: Install Java 17

```
sudo apt update
sudo apt install openjdk-17-jdk -y
```

Step 3: Set Java 17 as the Default Version

```
sudo update-alternatives --config java
```

Select Java 17 from the list.

Confirm the change:

```
java -version
```

Step 4: Restart Jenkins Service

```
sudo systemctl restart jenkins
sudo systemctl status jenkins
```

### Outcome

Jenkins started successfully

Jenkins became accessible via the browser on port 8080

CI server was ready for further configuration



### 2. Jenkins Repository GPG Key Error During Installation

### Problem:

During Jenkins installation, the system failed to update packages because the Jenkins repository GPG key was missing or invalid.

### Evidence

Running:
```
sudo apt update
```
returned GPG key verification errors, and Jenkins packages could not be authenticated.

### Solution

Step 1: Remove Any Faulty Jenkins Key (if present)

```
sudo rm -f /usr/share/keyrings/jenkins-keyring.asc
```

Step 2: Download and Add the Official Jenkins GPG Key

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

Step 3: Add Jenkins Repository Correctly

```
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
```

Step 4: Update Package List and Install Jenkins

```
sudo apt update
sudo apt install jenkins -y
```

### Outcome

Package update completed successfully

Jenkins installed without GPG or authentication errors

Jenkins service was ready to be started


## Key Takeaway

These issues highlight the importance of:

- Matching CI tools with the correct runtime versions

- Carefully validating repository GPG keys during installation

- Using system logs and service status checks for effective troubleshooting

Resolving these challenges strengthened practical CI troubleshooting and Linux system administration skills.
Overall, this project reflects real-world CI challenges and demonstrates the ability to debug, document, and automate production-like systems with confidence.


## Conclusion

In this project, we successfully implemented a Continuous Integration (CI) solution using Jenkins to automate deployment of the Tooling Website.

### Key achievements:

Automated code retrieval using GitHub webhooks

Centralized artifact storage in Jenkins

Automated deployment to shared NFS storage

Eliminated manual updates on multiple web servers

Improved scalability, reliability, and deployment speed

This project demonstrates real-world DevOps CI practices and forms a solid foundation for advanced CI/CD pipelines in future projects.




# Author

Oluwasomidotun Adepitan

DevOps & Cloud Engineer

LinkedIn https://www.linkedin.com/in/oluwasomidotun-adepitan-62ab47246?utm_source=share&utm_campaign=share_via&utm_content=profile&utm_medium=ios_app