# jenkins-first-project: Building Jenkins Pipelines on Amazon Linux 2 AWS EC2 Instance
In this Project, I am going to exhibit following hands-on experiences
- [ ]  configure Jenkins Server on Docker on Amazon Linux 2 EC2 instance using Cloudformation Service,
- [ ]  build simple Jenkins pipelines,
- [ ]  build Jenkins pipelines with Jenkinsfile,
- [ ]  integrate Jenkins pipelines with GitHub using Webhook
- [ ]  configure Jenkins Pipeline to build Python code

## Outline

- Part 1 - Installing Jenkins on Docker using Cloudformation Template

- Part 2 - Integrating Jenkins with GitHub using Webhook

- Part 3 - Configuring Jenkins Pipeline with GitHub Webhook to Build the Java Code

## Part 1 - Install Jenkins on Docker using Cloudformation Template

Launch a pre-configured `Clarusway Jenkins Server` from the AMI of Clarusway (ami-0644c22f90412d908) running on Amazon Linux 2, allowing SSH (port 22) and HTTP (ports 80, 8080) connections using the [Clarusway Jenkins Server Cloudformation Template](./jenkins-on-docker-cfn-template.yml). Clarusway Jenkins Server is configured with admin user `call-jenkins` and password `Call-jenkins1234`.

- Or launch and configure a Jenkins Server on Amazon Linux 2 AMI with security group allowing SSH (port 22) and HTTP (ports 80, 8080) connections using the [Cloudformation Template for Jenkins on Docker Installation](./jenkins-on-docker-cfn-template.yml).

- Connect to your instance with SSH.

    ```bash
    ssh -i .ssh/<'your pem'.pem> <ec2-user@'Public IPv4 DNS of your EC2 Instance' >
    ```

  - Get the administrator password from `var/jenkins_home/secrets/initialAdminPassword` file.

    ```bash
    docker exec -it jenkins bash
    cat /var/jenkins_home/secrets/initialAdminPassword
    exit
    ```

  - The administrator password can also be taken from Docker logs.

    ```bash
    docker logs jenkins
    ```

  - Enter the temporary password to unlock the Jenkins.

  - Install suggested plugins.

  - Create first admin user (`call-jenkins:Call-jenkins1234`).

  - Check the URL, then save and finish the installation.

  - Open your Jenkins dashboard and navigate to `Manage Jenkins` >> `Manage Plugins` >> `Available` tab

  - Search and select `GitHub Integration` plugin, then click to `Install without restart`. Note: No need to install the other `Git plugin` which is already installed can be seen under `Installed` tab.

## Part 2 - Integrating Jenkins with GitHub using Webhook

- Create a public project repository `jenkins-first-project` on your GitHub account.

- Clone the `jenkins-first-project` repository on local computer.

- Write a simple python code which prints `Hello World` and save it as `hello-word.py`.

```python
print('Hello World')
```

- Commit and push the local changes to update the remote repo on GitHub.

```bash
git add .
git commit -m 'added hello world'
git push
```

- Go back to Jenkins dashboard and click on `New Item` to create a new job item.

- Enter `first-job-triggered` then select `free style project` and click `OK`.

- Enter `My first job triggered from GitHub` in the description field.

- Put a checkmark on `Git` under `Source Code Management` section, enter URL of the project repository, and let others be default.

```text
https://github.com/gulec2000/jenkins-first-project
```

- Put a checkmark on `GitHub hook trigger for GITScm polling` under `Build Triggers` section, and click `apply` and `save`.

- Go to the `jenkins-first-project` repository page and click on `Settings`.

- Click on the `Webhooks` on the left hand menu, and then click on `Add webhook`.

- Copy the Jenkins URL from the AWS Management Console, paste it into `Payload URL` field, add `/github-webhook/` at the end of URL, and click on `Add webhook`.

```text
http://'Public IPv4 DNS of your instance':8080/github-webhook/
```

- Change the python code to print `Hello World for Jenkins Job` and save.

```python
print('Hello World for Jenkins Job')
```

- Commit and push the local changes to update the remote repo on GitHub.

```bash
git add .
git commit -m 'updated hello world'
git push
```

- Observe the new built under `Build History` on the Jenkins project page.

## Part 3 - Configuring Jenkins Pipeline with GitHub Webhook to Build the Java Code

- Go to the Jenkins dashboard and click on `New Item` to create a pipeline.

- Enter `pipeline-with-jenkinsfile-and-webhook` then select `Pipeline` and click `OK`.

- Enter `Simple pipeline configured with Jenkinsfile and GitHub Webhook to build Java code` in the description field.

- To build the `java` code with Jenkins pipeline using the `Jenkinsfile` and `GitHub Webhook`, we need;

  - a java code to build

  - a java environment to run the build stages on the java code

  - a Jenkinsfile configured for an automated build on our repo

- Create a java file on the `jenkins-first-project` GitHub repository, name it as `Hello.java`, add coding to print `Hello from Java` and save.

```java
public class Hello {

    public static void main(String[] args) {
        System.out.println("Hello from Java");
    }
}
```

- Since the Jenkins Server is running on Java platform, we can leverage from the already available java environment.

- Update the `Jenkinsfile` with the following pipeline script, and explain the changes.

- `yum install java-devel` to install java dependencies.

```groovy
pipeline {
    agent { label 'master' }
    stages {
        stage('build') {
            steps {
                echo 'Compiling the java source code'
                sh 'javac Hello.java'
            }
        }
        stage('run') {
            steps {
                echo 'Running the compiled java code.'
                sh 'java Hello'
            }
        }
    }
}
```

- Commit and push the changes to the remote repo on GitHub.

```bash
git add .
git commit -m 'updated jenkinsfile and added Hello.java'
git push
```

- Observe the new built triggered with `git push` command on the Jenkins project page.
