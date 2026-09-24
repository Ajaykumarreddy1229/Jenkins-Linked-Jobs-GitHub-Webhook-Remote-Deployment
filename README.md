# Jenkins Linked Jobs, GitHub Webhook & Remote Deployment

> **Topic:** Jenkins Administration, GitHub Webhooks, SSH Remote Deployment & Linked Jobs
> **Environment:** Jenkins, GitHub, Amazon Linux, SSH, Maven

---

# 1. Project Overview

This document contains my hands-on learning notes on Jenkins.

Topics covered:

```text
Remote File Transfer
SSH Configuration
Passwordless SSH
Publish Over SSH
Jenkins Variables
Jenkins Environment Variables
Jenkins Administration
Jenkins Port Configuration
Build Executors
Poll SCM
GitHub Webhooks
Throttle Builds
Remote Triggering
Upstream Jobs
Downstream Jobs
Linked Jobs
Remote Deployment
```

---

# 2. Remote File Transfer from Jenkins

## Environment

I used two servers:

```text
Jenkins Server
      |
      | SSH
      ↓
Test Server
Amazon Linux 2023
```

The Jenkins server connects to the remote server using SSH.

---

# 3. SSH Configuration

SSH allows Jenkins to securely communicate with the remote server.

---

## Generate SSH Key

Run:

```bash
ssh-keygen
```

This generates an SSH key pair.

Typical files:

```text
~/.ssh/
   |
   ├── id_rsa
   └── id_rsa.pub
```

```text
id_rsa
    ↓
Private Key

id_rsa.pub
    ↓
Public Key
```

---

# 4. Start SSH Agent

Start the SSH agent:

```bash
eval $(ssh-agent -s)
```

---

# 5. Create PEM Key File

Create the PEM file:

```bash
vi MyKey.pem
```

Paste the private key content and save the file.

Set the correct permissions:

```bash
chmod 400 MyKey.pem
```

---

# 6. Add SSH Key

Add the key to the SSH agent:

```bash
ssh-add MyKey.pem
```

---

# 7. SSH Directory

Go to:

```bash
cd /root/.ssh/
```

Check the files:

```bash
ls -la
```

---

# 8. Copy Public Key to Remote Server

Copy the public key:

```bash
ssh-copy-id -i id_rsa.pub ec2-user@REMOTE_SERVER_IP
```

Replace:

```text
REMOTE_SERVER_IP
```

with the private IP of your remote server.

---

# 9. Test SSH Connection

Test the connection:

```bash
ssh ec2-user@REMOTE_SERVER_IP
```

If the connection works without requesting a password, passwordless SSH authentication is configured.

---

# 10. Passwordless SSH Flow

```text
Jenkins Server
      |
      | SSH Key
      ↓
Remote Server
      |
      ↓
Passwordless Authentication
```

The public key is stored on the remote server, while the private key remains on the Jenkins server.

---

# 11. Publish Over SSH Plugin

Jenkins can use the **Publish Over SSH** plugin to transfer files and execute commands on remote servers.

Install the plugin from:

```text
Manage Jenkins
      ↓
Plugins
      ↓
Available Plugins
      ↓
Publish Over SSH
```

Install the plugin.

---

# 12. Configure Publish Over SSH

After installing the plugin:

```text
Manage Jenkins
      ↓
System
      ↓
Publish Over SSH
```

Add the remote SSH server.

Example:

```text
Name      : TestServer
Hostname  : Remote Server Private IP
Username  : ec2-user
```

---

# 13. SSH Private Key

The SSH private key can be configured in the plugin.

Example command to view the private key:

```bash
cat /root/.ssh/id_rsa
```

The private key should be handled securely.

> Never publish private SSH keys in GitHub or include them in public documentation.

---

# 14. Test SSH Configuration

After configuring the remote server:

```text
Test Configuration
```

A successful connection confirms that Jenkins can communicate with the remote server.

---

# 15. Create Jenkins Job for File Transfer

Create a new Jenkins job:

```text
New Item
    ↓
Remotefiles
    ↓
Freestyle Project
```

---

# 16. Jenkins Workspace

Example workspace:

```bash
cd /var/lib/jenkins/workspace/remotefiles/
```

Create a directory:

```bash
mkdir files
```

Create a test file:

```bash
touch files/test.py
```

Directory structure:

```text
remotefiles/
    |
    └── files/
          |
          └── test.py
```

---

# 17. Send Files over SSH

In the Jenkins job:

```text
Build Steps
      ↓
Send files or execute commands over SSH
```

---

# 18. Source Files

To copy one file:

```text
files/test.py
```

To copy multiple files:

```text
files/*
```

---

# 19. Remote Directory

Specify the destination directory on the remote server.

Example:

```text
/home/ec2-user/files
```

The flow becomes:

```text
Jenkins Workspace
      |
      ↓
files/test.py
      |
      | SSH
      ↓
Remote Server
      |
      ↓
/home/ec2-user/files/
```

---

# 20. Execute Remote Commands

Commands can also be executed on the remote server.

Example:

```bash
sh /home/ec2-user/files/bash.sh
```

Flow:

```text
Jenkins
   |
   | SSH
   ↓
Remote Server
   |
   ↓
Execute Command
```

---

# 21. Exclude Files

Specific files can be excluded when required.

Example:

```text
files/test1.py
files/test2.py
```

This allows unnecessary files to be excluded from the transfer.

---

# 22. Build the Job

Click:

```text
Build Now
```

Jenkins will:

```text
Find Files
    ↓
Connect to Remote Server
    ↓
Transfer Files
    ↓
Execute Commands
```

---

# 23. Jenkins Variables

Variables are used to store values that may change.

Two important types are:

```text
1. User-Defined Variables
2. Jenkins Environment Variables
```

---

# 24. User-Defined Variables

User-defined variables are variables created by the user.

They can be:

```text
Local Variables
Global Variables
```

---

# 25. Local Variables

A local variable can be used within the job or script where it is defined.

Example:

```bash
course=DevOps

echo "This is $course Course"
echo "Here you are learning $course"
echo "$course is good"
```

Output uses the current value of:

```text
course
```

If changed:

```bash
course=AWS
```

the output changes accordingly.

---

# 26. Global Variables

Global environment variables can be configured in Jenkins.

Go to:

```text
Manage Jenkins
      ↓
System
      ↓
Global properties
      ↓
Environment variables
```

Example:

```text
Name  = course
Value = DevOps
```

Jobs can then access the variable:

```bash
$course
```

---

# 27. Jenkins Environment Variables

Jenkins provides predefined environment variables.

Example:

```bash
echo "Current Build Number is $BUILD_NUMBER"
echo "Job Name is $JOB_NAME"
```

To display environment variables:

```bash
printenv
```

---

# 28. Common Jenkins Environment Variables

| Variable          | Description                         |
| ----------------- | ----------------------------------- |
| `BUILD_NUMBER`    | Current build number                |
| `BUILD_ID`        | Build ID                            |
| `BUILD_URL`       | URL of the build                    |
| `JOB_NAME`        | Name of the job                     |
| `JOB_BASE_NAME`   | Short job name                      |
| `WORKSPACE`       | Jenkins workspace directory         |
| `JENKINS_HOME`    | Jenkins home directory              |
| `JENKINS_URL`     | Jenkins URL                         |
| `NODE_NAME`       | Jenkins node name                   |
| `NODE_LABELS`     | Labels assigned to the node         |
| `EXECUTOR_NUMBER` | Executor number                     |
| `GIT_COMMIT`      | Git commit hash                     |
| `GIT_BRANCH`      | Git branch                          |
| `GIT_URL`         | Git repository URL                  |
| `BUILD_TAG`       | Build tag                           |
| `BRANCH_NAME`     | Branch name in Multibranch Pipeline |
| `CHANGE_ID`       | Pull Request ID                     |
| `CHANGE_TARGET`   | Target branch of Pull Request       |

---

# 29. Jenkins Port Configuration

Jenkins runs on a configured HTTP port.

To locate the Jenkins service file:

```bash
find / -name jenkins.service
```

Example service file:

```text
/usr/lib/systemd/system/jenkins.service
```

Open the file:

```bash
vi /usr/lib/systemd/system/jenkins.service
```

Change the Jenkins port as required by your environment.

---

# 30. Reload Jenkins Service

After changing the service configuration:

```bash
systemctl daemon-reload
```

Restart Jenkins:

```bash
systemctl restart jenkins.service
```

Check Jenkins:

```bash
systemctl status jenkins.service
```

Make sure the new port is allowed in the EC2 security group and server firewall.

---

# 31. Jenkins Security

Jenkins security configuration is stored in:

```text
/var/lib/jenkins/config.xml
```

One important setting is:

```xml
<useSecurity>true</useSecurity>
```

Changing security configuration can affect Jenkins authentication and authorization.

> Disabling Jenkins security is **not recommended for production systems**, because it can allow unauthorized access.

After configuration changes, Jenkins may need to be restarted:

```bash
systemctl restart jenkins.service
```

---

# 32. Build Executors

Jenkins uses **executors** to run builds.

An executor is a slot that can execute a Jenkins build.

Example:

```text
Executors = 3
```

Conceptually:

```text
Jenkins
   |
   ├── Executor 1 → Build A
   ├── Executor 2 → Build B
   └── Executor 3 → Build C
```

---

# 33. Configure Executors

Go to:

```text
Manage Jenkins
      ↓
System
      ↓
Number of executors
```

Example:

```text
Number of executors = 10
```

The appropriate number depends on the available system resources and workload.

---

# 34. Concurrent Builds

A job can also be configured to run multiple builds concurrently when appropriate.

Go to:

```text
Job
   ↓
Configure
   ↓
Execute concurrent builds if necessary
```

This allows multiple executions of the job to run at the same time.

---

# 35. Poll SCM

**Poll SCM** means Jenkins periodically checks the source-code repository for changes.

If a relevant change is detected, Jenkins can trigger a build.

Flow:

```text
Jenkins
   |
   ↓
Check Git Repository
   |
   ↓
Changes?
  / \
YES  NO
 |    |
 ↓    ↓
Build Wait
```

---

# 36. Build Periodically vs Poll SCM

## Build Periodically

Runs the job according to a schedule.

It does **not** require a source-code change.

```text
Schedule
   ↓
Build
```

## Poll SCM

Checks the source repository according to a schedule.

If changes are detected:

```text
Schedule
   ↓
Check Repository
   ↓
Changes Found
   ↓
Build
```

---

# 37. Jenkins Cron Syntax

Example:

```text
20 14 8 7 1
```

Jenkins cron syntax follows the Jenkins scheduler format.

The actual execution time depends on the Jenkins server's configured time zone.

---

# 38. GitHub Webhook

A GitHub Webhook allows GitHub to notify Jenkins when an event occurs.

For example:

```text
Developer
    ↓
Git Push
    ↓
GitHub
    ↓
Webhook
    ↓
Jenkins
    ↓
Build
```

This avoids waiting for the next SCM polling interval.

---

# 39. Configure GitHub Webhook

In GitHub:

```text
Repository
    ↓
Settings
    ↓
Webhooks
    ↓
Add Webhook
```

Configure the Jenkins webhook endpoint:

```text
http://JENKINS_IP:8080/github-webhook/
```

Use your actual Jenkins URL and port.

---

# 40. Webhook Content Type

Set:

```text
application/json
```

Then configure the required webhook event, such as push events.

---

# 41. Jenkins Webhook Trigger

In Jenkins:

```text
Job
   ↓
Configure
   ↓
Build Triggers
   ↓
GitHub hook trigger for GITScm polling
```

Now the flow becomes:

```text
Developer
    ↓
Push Code
    ↓
GitHub
    ↓
Webhook
    ↓
Jenkins
    ↓
Build
```

---

# 42. Maven Build

For Java projects, Jenkins can execute Maven commands.

Example:

```text
Build Steps
      ↓
Invoke top-level Maven targets
```

Maven target:

```bash
clean package
```

---

# 43. GitHub + Jenkins + Maven Flow

```text
Developer
    ↓
GitHub
    ↓
Webhook
    ↓
Jenkins
    ↓
Checkout Code
    ↓
Maven Build
    ↓
Test
    ↓
Artifact
```

---

# 44. Throttle Builds

Build throttling can be used to limit how frequently builds run or how many builds can execute within configured limits.

Example practice configuration:

```text
Maximum Builds = 3
Time Period    = 3 hours
```

This can help prevent excessive build activity.

---

# 45. Remote Triggering

Jenkins jobs can also be triggered remotely.

Go to:

```text
Job
   ↓
Configure
   ↓
Build Triggers
   ↓
Trigger builds remotely
```

Configure an authentication token.

Example:

```text
Token = mytoken
```

---

# 46. Remote Trigger Flow

```text
External System
      |
      | HTTP Request
      ↓
Jenkins
      |
      ↓
Jenkins Job
      |
      ↓
Build
```

Example URL format:

```text
JENKINS_URL/job/JOB_NAME/build?token=TOKEN_NAME
```

Use the authentication and authorization mechanism appropriate for your Jenkins security configuration.

---

# 47. Linked Jobs

Linked Jobs allow multiple Jenkins jobs to be connected.

Example:

```text
Job 1
  ↓
Job 2
```

Here:

```text
Job 1 = Upstream Job
Job 2 = Downstream Job
```

---

# 48. Upstream Job

The **upstream job** is the job that triggers another job.

Example:

```text
Job 1
  ↓
Job 2
```

Job 1 is the upstream job.

---

# 49. Downstream Job

The **downstream job** is triggered by another Jenkins job.

Example:

```text
Job 1
  ↓
Job 2
```

Job 2 is the downstream job.

---

# 50. Create Job 1

Create:

```text
New Item
   ↓
job1
   ↓
Freestyle Project
```

Build step:

```bash
echo "Welcome from Job 1"
```

---

# 51. Create Job 2

Create:

```text
New Item
   ↓
job2
   ↓
Freestyle Project
```

Build step:

```bash
echo "Welcome from Job 2"
```

---

# 52. Link Job 1 and Job 2

Go to:

```text
job1
   ↓
Configure
   ↓
Post-build Actions
   ↓
Build other projects
```

Enter:

```text
job2
```

Select:

```text
Trigger only if build is stable
```

Save the configuration.

---

# 53. Linked Job Flow

When Job 1 succeeds:

```text
Job 1
  ↓
SUCCESS
  ↓
Job 2
  ↓
SUCCESS
```

If Job 1 fails:

```text
Job 1
  ↓
FAILURE
  ↓
Job 2
  ↓
Not Triggered
```

---

# 54. GitHub + Jenkins + Linked Jobs

A practical workflow can combine:

```text
GitHub
GitHub Webhook
Jenkins
Linked Jobs
Maven
SSH
Remote Server
```

Architecture:

```text
Developer
    ↓
GitHub Repository
    ↓
GitHub Webhook
    ↓
Jenkins
    ↓
Job 1
    ↓
Build + Test
    ↓
SUCCESS
    ↓
Job 2
    ↓
Deploy
    ↓
SSH
    ↓
Remote/Test Server
```

---

# 55. Complete CI/CD Flow

```text
Developer Pushes Code
          ↓
    GitHub Repository
          ↓
     GitHub Webhook
          ↓
        Jenkins
          ↓
        Job 1
          ↓
        Build
          ↓
        Test
          ↓
      Successful?
        /       \
      YES        NO
       ↓          ↓
     Job 2       STOP
       ↓
    Deploy
       ↓
      SSH
       ↓
 Remote Server
```

---

# 56. Complete Architecture

```text
                         Developer
                             |
                             ↓
                          GitHub
                             |
                         Webhook
                             |
                             ↓
                    Jenkins Controller
                             |
                             ↓
                          Job 1
                       Build + Test
                             |
                       ┌─────┴─────┐
                       ↓           ↓
                    SUCCESS      FAILURE
                       |             |
                       ↓             ↓
                     Job 2          STOP
                    Deploy
                       |
                       ↓
                      SSH
                       |
                       ↓
                Remote/Test Server
```

---

# 57. Remote Deployment Flow

```text
Jenkins
   |
   | SSH
   ↓
Remote Server
   |
   ├── Transfer Files
   |
   └── Execute Commands
```

This allows Jenkins to automate deployment activities on remote machines.

---

# 58. Important Jenkins Administration Concepts

During this practice, I also learned basic Jenkins administration concepts:

```text
Jenkins Port
Jenkins Service
Jenkins Home
Jenkins Executors
Environment Variables
Job Configuration
Build Triggers
Security Configuration
Plugins
Credentials
```

---

# 59. Key Concepts Learned

Through this hands-on practice, I learned:

```text
✓ SSH Remote Connections
✓ Passwordless SSH
✓ Remote File Transfer
✓ Publish Over SSH
✓ Jenkins Variables
✓ Jenkins Environment Variables
✓ Jenkins Administration
✓ Jenkins Port Configuration
✓ Build Executors
✓ Concurrent Builds
✓ Poll SCM
✓ GitHub Webhooks
✓ Maven Builds
✓ Build Throttling
✓ Remote Triggering
✓ Upstream Jobs
✓ Downstream Jobs
✓ Linked Jobs
✓ Remote Deployment
✓ CI/CD Automation
```

---

# 60. Final CI/CD Architecture

```text
                         GitHub
                            |
                            ↓
                     GitHub Webhook
                            |
                            ↓
                         Jenkins
                            |
                     ┌──────┴──────┐
                     ↓             ↓
                   Job 1       Jenkins Admin
                     |
                Build + Test
                     |
                     ↓
                  SUCCESS
                     |
                     ↓
                   Job 2
                     |
                  Deploy
                     |
                     ↓
                    SSH
                     |
                     ↓
              Remote Server
                     |
                     ↓
              Application
```

---

# 61. Key Takeaway

The main workflow I practiced is:

```text
GitHub
   ↓
Webhook
   ↓
Jenkins
   ↓
Build
   ↓
Test
   ↓
Job 1
   ↓
Job 2
   ↓
SSH Deployment
   ↓
Remote Server
```

This hands-on practice helped me understand how Jenkins can connect different stages of a CI/CD process and automate repetitive DevOps tasks.

---

# 62. DevOps Learning Flow

```text
GitHub
   ↓
Webhook
   ↓
Jenkins
   ↓
Maven
   ↓
Build
   ↓
Test
   ↓
Linked Jobs
   ↓
SSH
   ↓
Remote Deployment
```

**Learn → Practice → Troubleshoot → Automate → Improve 🚀**
