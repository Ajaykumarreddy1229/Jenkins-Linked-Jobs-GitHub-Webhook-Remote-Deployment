# Jenkins-Linked-Jobs-GitHub-Webhook-Remote-Deployment
This document contains my hands-on learning notes on Jenkins, including:

Transferring files from Jenkins to a remote server
SSH configuration
Jenkins environment variables
Jenkins administration
Passwordless login
Build Executors
Poll SCM
GitHub Webhooks
Throttle Builds
Remote Triggering
Linked Jobs / Upstream and Downstream Jobs
**1. Transferring Files from Jenkins to Remote Server
Environment**

I used two servers:

Jenkins Server
Test Server – Amazon Linux 2023

The Jenkins server is configured to connect to the remote server using SSH.

SSH Configuration

Generate an SSH key:

ssh-keygen

Start the SSH agent:

eval $(ssh-agent -s)

Create the PEM key file:

vi MyKey.pem

Paste the private key content and save the file.

Set the correct permissions:

chmod 400 MyKey.pem

Add the key to the SSH agent:

ssh-add MyKey.pem

Go to the SSH directory:

cd /root/.ssh/

Copy the public key to the remote server:

ssh-copy-id -i id_rsa.pub ec2-user@REMOTE_SERVER_IP

Test the SSH connection:

ssh ec2-user@REMOTE_SERVER_IP

If the connection works without asking for a password, passwordless SSH is configured.

**2. Install Publish Over SSH Plugin**

In Jenkins:

Manage Jenkins
       ↓
Plugins
       ↓
Install "Publish Over SSH"

After installing the plugin:

Manage Jenkins
       ↓
System
       ↓
Publish Over SSH

Add the SSH server configuration.

SSH Server Configuration
Name       = TestServer
Hostname   = Remote Server Private IP
Username   = ec2-user

In the Key section, paste the private SSH key:

cat /root/.ssh/id_rsa

Then click:

Test Configuration

The connection should be successful.

**3. Create Jenkins Job to Transfer Files**

Create a new Jenkins job:

New Item
   ↓
Remotefiles
   ↓
Freestyle Project

Go to:

Build Steps
   ↓
Send files or execute commands over SSH

First create a file in the Jenkins workspace:

cd /var/lib/jenkins/workspace/remotefiles/

mkdir files

touch files/test.py
Source Files

To copy one file:

files/test.py

To copy multiple files:

files/*
Remote Directory

Specify the directory where the files should be copied on the remote server.

Example:

/home/ec2-user/files
Exec Command

Commands can also be executed on the remote server.

Example:

sh /home/ec2-user/files/bash.sh
Exclude Files

Specific files can be excluded when required:

files/test1.py
files/test2.py

Click:

Build Now

The files will be transferred to the remote server.

**4. Jenkins Variables**

Variables are used to store values that may change.

There are two major types:

User-defined variables
Jenkins environment variables
**4.1 User-Defined Variables**

User-defined variables are variables created by the user.

They can be:

Local variables
Global variables
Local Variable

A local variable works only within the job where it is defined.

Example:

course=DevOps

echo "This is $course Course"
echo "Here you are learning $course"
echo "$course is good"

If the variable is changed:

course=AWS

the output will change accordingly.

**4.2 Global Variables**

Global variables can be used by multiple Jenkins jobs.

Go to:

Manage Jenkins
   ↓
System
   ↓
Global properties
   ↓
Environment variables

Add:

Name  = course
Value = DevOps

Now Jenkins jobs can access:

$course
**5. Jenkins Environment Variables**

Jenkins provides predefined environment variables.

Example:

echo "Current Build Number is $BUILD_NUMBER"
echo "Job Name is $JOB_NAME"

To display all environment variables:

printenv
Common Jenkins Environment Variables
Variable	Description
BUILD_NUMBER	Current build number
BUILD_ID	Build ID
BUILD_URL	URL of the build
JOB_NAME	Name of the job
JOB_BASE_NAME	Short job name
WORKSPACE	Jenkins workspace directory
JENKINS_HOME	Jenkins home directory
JENKINS_URL	Jenkins URL
NODE_NAME	Jenkins node name
NODE_LABELS	Labels assigned to the node
EXECUTOR_NUMBER	Executor number
GIT_COMMIT	Git commit hash
GIT_BRANCH	Git branch
GIT_URL	Git repository URL
BUILD_TAG	Build tag
BRANCH_NAME	Branch name in Multibranch Pipeline
CHANGE_ID	Pull Request ID
CHANGE_TARGET	Target branch of Pull Request
**6. Jenkins Port Change**
To find the Jenkins service file:

find / -name jenkins.service

Edit the service file:

vi /usr/lib/systemd/system/jenkins.service

Change the Jenkins port as required.

After changing the port:

systemctl daemon-reload

systemctl restart jenkins.service

Make sure the new port is allowed in the server firewall/security group.

**7. Jenkins Without Password Login**

Jenkins security settings are stored in:

/var/lib/jenkins/config.xml

The security configuration contains:

<useSecurity>true</useSecurity>

Changing this setting disables Jenkins security.

Note: Disabling Jenkins authentication is not recommended for production systems because it allows unauthorized users to access Jenkins.

After changing the configuration:

systemctl restart jenkins.service
**8. Build Executors**

Jenkins uses executors to run builds.

If Jenkins has multiple executors, multiple builds can run at the same time.

Go to:

Manage Jenkins
   ↓
System
   ↓
Number of executors

For example:

Number of executors = 10

Jobs can also be configured to run concurrently:

Job
 ↓
Configure
 ↓
Execute concurrent builds if necessary
**9. Poll SCM**

Poll SCM means Jenkins periodically checks the Source Code Management system for changes.

Build Periodically vs Poll SCM

Build Periodically:

Runs the job according to a schedule.

It does not require a source-code change.

Poll SCM:

Checks the source repository according to a schedule and triggers a build when changes are detected.

Example cron schedule:

20 14 8 7 1

The Jenkins cron syntax is interpreted according to the Jenkins server's configured time zone.

**10. GitHub Webhook**

A GitHub Webhook allows GitHub to notify Jenkins when an event such as a push occurs.

This can trigger Jenkins without waiting for the next SCM polling interval.

Configure GitHub Webhook

In GitHub:

Repository
   ↓
Settings
   ↓
Webhooks
   ↓
Add Webhook

Configure the Jenkins webhook endpoint:

http://JENKINS_IP:8080/github-webhook/

Content Type:

application/json

In Jenkins:

Job
   ↓
Configure
   ↓
Build Triggers
   ↓
GitHub hook trigger for GITScm polling

Now when code is pushed to GitHub, GitHub sends the webhook notification to Jenkins.

**11. Maven Build**

For a Java project, Jenkins can execute Maven commands.

Example:

Build Steps
   ↓
Invoke top-level Maven targets

Maven target:

clean package

The general flow becomes:

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
**12. Throttle Builds**

Throttle Build is used to control how frequently builds can run.

Example configuration:

Maximum Builds = 3
Time Period   = 3 hours

This can be useful when we want to prevent excessive builds within a particular period.

**13. Remote Triggering**

Jenkins jobs can also be triggered remotely.

Go to:

Job
   ↓
Configure
   ↓
Build Triggers
   ↓
Trigger builds remotely

Configure an authentication token.

Example:

Token = mytoken

A remote request can then trigger the Jenkins job using the configured Jenkins URL and token.

Example format:

JENKINS_URL/job/JOB_NAME/build?token=TOKEN_NAME
**14. Linked Jobs**

Linked Jobs are used to connect multiple Jenkins jobs.

For example:

Job 1 → Job 2

Here:

Job 1 = Upstream Job
Job 2 = Downstream Job

When Job 1 completes successfully, Jenkins can automatically trigger Job 2.

Create Job 1

Create:

New Item
   ↓
job1
   ↓
Freestyle Project

Build Step:

echo "Welcome from Job 1"
Create Job 2

Create:

New Item
   ↓
job2
   ↓
Freestyle Project

Build Step:

echo "Welcome from Job 2"
Link Job 1 and Job 2

Go to:

job1
   ↓
Configure
   ↓
Post-build Actions
   ↓
Build other projects

Enter:

job2

Select:

Trigger only if build is stable

Save the configuration.

Now build Job 1.

The flow will be:

Job 1
  ↓
Build Successful
  ↓
Job 2
  ↓
Build Successful

If Job 1 fails:

Job 1
  ↓
FAILURE
  ↓
Job 2 is not triggered
**15. GitHub + Jenkins + Linked Jobs**

A practical CI/CD workflow can combine GitHub Webhooks, Jenkins jobs, linked jobs, and SSH deployment.

Developer
    ↓
GitHub Repository
    ↓
GitHub Webhook
    ↓
Jenkins
    ↓
Job 1
Build + Test
    ↓
SUCCESS
    ↓
Job 2
Deploy
    ↓
SSH
    ↓
Remote/Test Server
Complete Flow
Developer pushes code
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
      /     \
    YES      NO
     ↓        ↓
   Job 2     Stop
     ↓
   Deploy
     ↓
SSH
     ↓
Remote Server
**16. Key Concepts Learned**

Through this hands-on practice, I learned:

How Jenkins connects to remote servers using SSH
How to transfer files from Jenkins to a remote server
How to configure Publish Over SSH
How Jenkins variables work
Jenkins predefined environment variables
Jenkins administration basics
Jenkins port configuration
Build Executors
Poll SCM
GitHub Webhooks
Remote triggering
Throttling builds
Upstream and Downstream jobs
Linking multiple Jenkins jobs
Automating build, test, and deployment workflows
Conclusion

Jenkins can automate the complete CI/CD workflow by integrating with GitHub, building and testing applications, connecting jobs together, and deploying files or applications to remote servers.

A typical workflow is:

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

This demonstrates how Jenkins can be used to automate repetitive DevOps tasks and connect different stages of a CI/CD process.
