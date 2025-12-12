# Jenkins CI/CD Pipeline for Deploying Web Content to Apache Web Server

This project demonstrates how to set up a complete CI/CD pipeline using **Jenkins**, **GitHub**, and an **Apache Web Server** hosted on Amazon Linux 2023 EC2 instances. Whenever developers push HTML files to GitHub, Jenkins automatically builds the project, runs tests, and deploys the updated files to the web server via SSH.

---

## 🚀 Project Overview

* Jenkins Server is set up on EC2 to automate deployment.
* Apache Web Server (Amazon Linux 2023) hosts the application.
* GitHub repository acts as the source of truth.
* Jenkins pulls code → runs tests → builds job → deploys HTML files to Apache server.
* Deployment is done using **Publish Over SSH Plugin** in Jenkins.

---

## 🏗️ Technologies Used

* **Amazon EC2**
* **Jenkins**
* **GitHub Webhooks**
* **Apache HTTP Server (httpd)**
* **Amazon Linux 2023**
* **Git**
* **Java 17 (Amazon Corretto)**

---

## 🔧 Setup Instructions

### 1️⃣ Install Jenkins on EC2

```bash
sudo hostnamectl set-hostname autoSrv
sudo yum update

sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

sudo yum upgrade
sudo dnf install java-17-amazon-corretto -y
sudo yum install jenkins -y

sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

sudo yum install git -y
```

Jenkins will run on:

```
http://<JENKINS_PUBLIC_IP>:8080
```

---

### 2️⃣ Install Apache on Web Server EC2

```bash
sudo yum update
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

After installation:

```bash
cd /var/www
sudo chmod 777 -R html
```

---

### 3️⃣ Configure Jenkins – Publish Over SSH

#### Install Plugin

Inside Jenkins:

> Manage Jenkins → Manage Plugins → Available
> Search: **Publish Over SSH** → Install

#### Add SSH Server

> Manage Jenkins → Configure System → Publish over SSH

* **Name:** srv1
* **Hostname:** srv1
* **Username:** ec2-user
* **SSH Private Key:** *Paste key of Web Server EC2*
* **Remote Directory:** `/var/www/html/`
* Click **Test Configuration**

---

### 4️⃣ Create Jenkins Freestyle Job

#### General

* Enable **GitHub Project**
* Paste project URL

#### Source Code Management (Git)

* Repository URL:
  `https://github.com/<your-username>/<repo>.git`
* Branch:
  `*/main   or   */master`

#### Build Triggers

Enable:

```
GitHub hook trigger for GITScm polling
```

#### Add Webhook in GitHub

GitHub → Repository → Settings → Webhooks → Add Webhook

* **Payload URL:**
  `http://<JENKINS_PUBLIC_IP>:8080/github-webhook/`
* **Event:** Just the push event
* **Active:** Yes

---

### 5️⃣ Deploy HTML Files via SSH

Inside the job:

> Build Environment → Send files or execute commands over SSH

Configuration:

* **Name:** srv1
* **Transfers:**

  * **Source files:**

    ```
    **/*.html
    ```
  * **Remote directory:**

    ```
    /
    ```

Save & Apply.

---

## 🔁 CI/CD Flow (Added)

This section describes the automated flow: when a developer commits code, how the pipeline fetches, tests, and runs the deployment automatically.

### Flow (high-level)

1. **Developer commits & pushes** HTML changes to GitHub (branch: `main` or `master`).
2. **GitHub Webhook** sends a POST payload to Jenkins at `/github-webhook/` on push.
3. **Jenkins receives webhook** and triggers the configured job (or pipeline).
4. **Jenkins clones/pulls** the repo and checks out the commit.
5. **Run automated steps** inside Jenkins job:

   * **Install dependencies** (if any). Example: `npm install` or other setup commands.
   * **Run tests** (unit / lint / integration). Example commands:

     ```bash
     # sample test commands (adjust as per your project)
     npm ci && npm test
     # or run a simple HTML validation / shell script
     ./ci/run_tests.sh
     ```
   * **Build** (if applicable): minify, bundle, or prepare assets.
6. **If tests pass**, Jenkins proceeds to deploy:

   * Use **Publish Over SSH** to copy `**/*.html`, css, js, assets to `/var/www/html/` on the web server.
   * Optionally run remote commands (e.g., `sudo systemctl restart httpd`) via SSH post-transfer.
7. **If tests fail**, Jenkins marks the build as failed and notifies developers (Email/Slack) — no deployment.
8. **Post-deploy verification** (optional): Jenkins can run a smoke test (curl the public site) to confirm successful deployment.

### Visual flow (ASCII)

```
Developer  --->  GitHub Repo  ---push--->  GitHub Webhook  --->  Jenkins
                                                      |
                                                      v
                                           Jenkins clones repo, runs tests
                                                      |
                                +---------------------+-------------------+
                                |                                         |
                          Tests pass                                  Tests fail
                                |                                         |
                  Deploy to Apache server via SSH                Mark build as failed
                                |                                         |
                       Smoke tests / Notify devs                 Notify devs / Fix code
```

---

## ✅ Sample Jenkins Pipeline (Declarative) - Optional

If you prefer a Jenkinsfile (pipeline-as-code) instead of Freestyle, add this `Jenkinsfile` to your repo:

```groovy
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Install') {
      steps {
        // adapt to your project: install tools/dependencies
        sh 'echo "No dependencies to install for static HTML"'
      }
    }
    stage('Test') {
      steps {
        // example: run lint or HTML validation script
        sh './ci/run_tests.sh || exit 1'
      }
    }
    stage('Build') {
      steps {
        sh 'echo "Build step (if any)"'
      }
    }
    stage('Deploy') {
      steps {
        // using publish over ssh configured as "srv1"
        publishOverSsh (publisherName: 'srv1') {
          transferSet (sourceFiles: '**/*.html', remoteDirectory: '/', removePrefix: '')
        }
        // optional remote command
        // sshCommand(remote: 'srv1', command: 'sudo systemctl restart httpd')
      }
    }
  }
  post {
    success { echo 'Deployment successful' }
    failure { echo 'Build or tests failed' }
  }
}
```

> Note: To use `publishOverSsh` in a Jenkins pipeline you may need the pipeline-compatible step or wrap with a shell scp/ssh command depending on plugin availability.

---

## 🎯 Outcome

Once configured, developers only need to push code. Jenkins will automatically fetch changes, run tests, and deploy to the Apache server if tests pass — creating a reliable automated CI/CD pipeline.

---

## ⭐ Suggested GitHub Repository Names

* `jenkins-apache-cicd-pipeline`
* `devops-cicd-automation`
* `jenkins-to-apache-deployment`
* `aws-jenkins-web-deployment`
* `github-jenkins-apache-cicd`

---

## 🖼️ Screenshots (Add Your Images Here)

### EC2 Servers
![EC2 Servers](screenshots/EC2_Servers.png)

### Jenkins Console Output 1
![Jenkins Console Output](screenshots/Jenkins_console_output.png)

### Jenkins Console Output 2
![Console Output 2](screenshots/JenkinsConsole_output2.png)

### Jenkins Project Page
![Jenkins Project](screenshots/Jenkins_project.png)

### Deployment Output 1
![Output 1](screenshots/Output1.png)

### Deployment Output 2
![Output 2](screenshots/Output2.png)


---

