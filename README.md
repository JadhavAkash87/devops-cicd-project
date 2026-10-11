# AWS EC2 + Docker + Jenkins CI/CD Website Deployment

A beginner-friendly DevOps project that deploys a simple static website
using **GitHub, Jenkins, Docker, and AWS EC2**.

> **Project status:** The website was built into a Docker image and run
> as a container. The container returned `HTTP 200 OK` when tested
> locally with `curl http://localhost`. Public access depends on the EC2
> public IP and network settings. Jenkins automation can be extended
> using the pipeline example below.

## Table of Contents

-   [Project Overview](#project-overview)
-   [Architecture](#architecture)
-   [Tools Used](#tools-used)
-   [Project Files](#project-files)
-   [Prerequisites](#prerequisites)
-   [Step 1: Create the GitHub
    Repository](#step-1-create-the-github-repository)
-   [Step 2: Create the Website](#step-2-create-the-website)
-   [Step 3: Create the Dockerfile](#step-3-create-the-dockerfile)
-   [Step 4: Prepare the EC2 Instance](#step-4-prepare-the-ec2-instance)
-   [Step 5: Clone the Repository on
    EC2](#step-5-clone-the-repository-on-ec2)
-   [Step 6: Build the Docker Image](#step-6-build-the-docker-image)
-   [Step 7: Run the Container](#step-7-run-the-container)
-   [Step 8: Test the Website](#step-8-test-the-website)
-   [Step 9: Configure the AWS Security
    Group](#step-9-configure-the-aws-security-group)
-   [Step 10: Jenkins Pipeline](#step-10-jenkins-pipeline)
-   [Troubleshooting](#troubleshooting)
-   [Security and Cost Notes](#security-and-cost-notes)
-   [What I Learned](#what-i-learned)

## Project Overview

This project demonstrates the basic workflow of packaging a static
website in a Docker image and running it on an AWS EC2 Ubuntu server.
Nginx serves the website on port 80.

### Main workflow

1.  Store `Dockerfile` and `index.html` in GitHub.
2.  Use Jenkins to check out the source code.
3.  Build a Docker image from the Dockerfile.
4.  Run the image as a container on EC2.
5.  Test the website locally and, after network configuration, access it
    using the EC2 public IPv4 address.

## Architecture

``` text
Developer
   |
   v
GitHub repository
   |
   v
Jenkins Pipeline (checkout / build / deploy)
   |
   v
Docker image: devops-web:v1
   |
   v
Docker container: devops-web
   |
   v
Nginx listens on container port 80
   |
   v
AWS EC2 public IP : 80
   |
   v
Website in browser
```

**Important:** The Jenkins deployment stage below assumes Jenkins can
run Docker commands on the same EC2 host that serves the website. If
Jenkins and the website are on different EC2 instances, the deployment
design must be adjusted.

## Tools Used

-   **AWS EC2:** Virtual server for hosting the website.
-   **Amazon VPC / Security Group:** Network access control.
-   **Ubuntu Linux:** Operating system on EC2.
-   **Git and GitHub:** Source-code version control.
-   **Jenkins:** CI/CD automation.
-   **Docker:** Container image build and container runtime.
-   **Nginx:** Web server for the static HTML page.

## Project Files

``` text
devops-cicd-project/
├── Dockerfile
├── index.html
└── README.md
```

### `index.html`

``` html
<!DOCTYPE html>
<html>
<head>
    <title>My DevOps Project</title>
</head>
<body>
    <h1>My First DevOps Deployment!</h1>
    <p>Deployed using Jenkins and Docker.</p>
</body>
</html>
```

### `Dockerfile`

``` dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

Explanation:

-   `FROM nginx:alpine` uses a lightweight Nginx base image.
-   `COPY` copies the website HTML file into Nginx's default web
    directory.
-   `EXPOSE 80` documents the port used by Nginx inside the container.
    Docker port publishing is configured when the container is started.

## Prerequisites

Before starting, you need:

-   An AWS account and an Ubuntu EC2 instance.
-   A GitHub account and a repository.
-   Git installed on EC2.
-   Docker installed and running on the EC2 instance where you build/run
    the container.
-   Jenkins installed and configured if you want to use the pipeline.
-   An EC2 security group configured for HTTP access when public website
    access is required.

Use your own GitHub repository URL in all commands. Do not copy another
person's repository URL.

## Step 1: Create the GitHub Repository

Create a GitHub repository named `devops-cicd-project`.

Add these files to the repository:

-   `index.html`
-   `Dockerfile`
-   `README.md`

Commit and push the files to the `main` branch.

If using Git locally:

``` bash
git init
git add Dockerfile index.html README.md
git commit -m "Add website and Docker configuration"
git branch -M main
git remote add origin https://github.com/YOUR-GITHUB-USERNAME/devops-cicd-project.git
git push -u origin main
```

If the repository already exists and files are already committed on
GitHub, do not run `git init` again unnecessarily.

## Step 2: Create the Website

Create `index.html` in the project root and add the HTML shown above.
Commit the file to GitHub.

## Step 3: Create the Dockerfile

Create `Dockerfile` in the same directory as `index.html`. It must be
named exactly `Dockerfile` (capital D, no `.txt` extension).

Commit it to GitHub.

## Step 4: Prepare the EC2 Instance

Connect to the Ubuntu EC2 instance using SSH or the AWS browser-based
connection.

Check Ubuntu and Git:

``` bash
lsb_release -a
git --version
```

Install Docker if the `docker` command is missing:

``` bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo docker --version
sudo systemctl status docker
```

If Docker is not installed, `docker: command not found` may appear.
Install Docker on the **specific instance where you run the Docker
commands**. Installing it on one EC2 instance does not install it on
another.

## Step 5: Clone the Repository on EC2

Move to a suitable directory, then clone your repository:

``` bash
cd ~
git clone https://github.com/YOUR-GITHUB-USERNAME/devops-cicd-project.git
cd devops-cicd-project
ls -la
```

Confirm that `Dockerfile` and `index.html` are both present.

If you have already cloned the repository, enter the existing repository
directory instead of cloning it again. If you need the latest committed
code:

``` bash
git pull origin main
```

## Step 6: Build the Docker Image

Run this command from the directory containing both `Dockerfile` and
`index.html`:

``` bash
sudo docker build -t devops-web:v1 .
```

Verify that the image exists:

``` bash
sudo docker images
```

You should see an image named `devops-web` with tag `v1`.

## Step 7: Run the Container

Start the container and publish host port 80 to container port 80:

``` bash
sudo docker run -d --name devops-web -p 80:80 devops-web:v1
```

Check the running container and port mapping:

``` bash
sudo docker ps
```

Expected port mapping includes:

``` text
0.0.0.0:80->80/tcp
```

If a container named `devops-web` already exists and you intentionally
want to replace it, remove it first:

``` bash
sudo docker rm -f devops-web
```

Then run the `docker run` command again. Removing a container stops that
container, so make sure this is the correct application before doing so.

## Step 8: Test the Website

Test from the EC2 terminal:

``` bash
curl -I http://localhost
curl http://localhost
```

A response such as `HTTP/1.1 200 OK` indicates the local web server is
responding. The HTML output should include
`My First DevOps Deployment!`.

To inspect the HTML file actually being served by the container:

``` bash
sudo docker exec devops-web cat /usr/share/nginx/html/index.html
```

To inspect container logs:

``` bash
sudo docker logs devops-web
```

## Step 9: Configure the AWS Security Group

For public HTTP access, open the AWS Console:

1.  Go to **EC2 → Instances**.
2.  Select the correct EC2 instance.
3.  Open **Security → Security groups**.
4.  Select the attached security group.
5.  Open **Inbound rules → Edit inbound rules**.
6.  Add or confirm this rule:

  Type   Protocol     Port Source
  ------ ---------- ------ -------------
  HTTP   TCP            80 `0.0.0.0/0`

Save the rules. This makes HTTP reachable from the public internet, so
use it only when public access is intended.

Then open the website in a browser:

``` text
http://YOUR-EC2-PUBLIC-IP
```

Use `http://` for this project unless HTTPS has been configured
separately.

If the website does not open even though `curl http://localhost` returns
`200 OK`, check:

-   The EC2 instance has a public IPv4 address.
-   The security group attached to that exact instance allows inbound
    TCP port 80.
-   The subnet route table has a route to an Internet Gateway for public
    IPv4 traffic.
-   Network ACL rules allow the required inbound and return traffic.
-   The browser is using the correct current public IP and `http://`.

## Step 10: Jenkins Pipeline

In Jenkins, create a Pipeline job and configure it to use a Pipeline
script. The following example checks out the repository, builds the
image, and replaces the running container.

**Use this example only if the Jenkins agent has Git and Docker
installed and is authorized to run Docker commands, and if deployment
should happen on that same EC2 host.** A Jenkins user often cannot run
`sudo` non-interactively by default. Configure permissions appropriately
rather than adding broad, unsafe sudo permissions.

Replace `YOUR-GITHUB-USERNAME` with your actual GitHub username.

``` groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/YOUR-GITHUB-USERNAME/devops-cicd-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t devops-web:v1 .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker rm -f devops-web || true'
                sh 'docker run -d --name devops-web -p 80:80 devops-web:v1'
            }
        }
    }
}
```

### Run the Jenkins pipeline

1.  Save the Pipeline script in the Jenkins job.
2.  Select **Build Now**.
3.  Open **Console Output**.
4.  Confirm the checkout, build, and deploy stages complete
    successfully.
5.  Test `curl -I http://localhost` on the deployment EC2 instance and
    then test the public IP in a browser.

If Jenkins runs on a separate EC2 instance, this pipeline will build and
deploy on the Jenkins agent, not automatically on the website server.
Use a proper deployment method (for example, a securely configured SSH
deployment or an image registry plus a deployment agent) for separate
hosts.

## Troubleshooting

### `docker: command not found`

Install Docker on the exact EC2 instance where the command is being run:

``` bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo docker --version
```

### `permission denied` when running Docker

Try `sudo docker ...` for a manual terminal test. For Jenkins, configure
Docker access for the Jenkins agent user using an appropriate
security-reviewed setup, then restart/reload the relevant service if
required. Membership in the `docker` group effectively grants root-level
control of the host.

### `Bind for 0.0.0.0:80 failed: port is already allocated`

Another process or container is already using port 80. Check:

``` bash
sudo docker ps
sudo ss -ltnp | grep ':80'
```

Stop or reconfigure the conflicting service only after confirming what
it is.

### `curl localhost` works but the public website does not open

Check the public IPv4 address, the security group attached to the
instance, subnet route table/Internet Gateway, and network ACL. A local
HTTP 200 response alone does not prove public connectivity.

### The Nginx welcome page appears

Check whether the correct `index.html` was copied into the image:

``` bash
sudo docker exec devops-web cat /usr/share/nginx/html/index.html
```

Rebuild the image from the directory containing the latest `Dockerfile`
and `index.html`, then replace the container.

### GitHub changes do not appear on the website

Pull the latest committed code before building, or let Jenkins check out
the latest commit. Build a new image and replace the running container.
A GitHub commit alone does not update an already-running Docker
container.

## Security and Cost Notes

-   Do not commit AWS access keys, passwords, private keys, or Jenkins
    secrets to GitHub.
-   Use least-privilege IAM permissions.
-   Public HTTP (`0.0.0.0/0` on port 80) allows anyone to reach the
    website.
-   Stop or terminate unused EC2 resources when you no longer need them.
    EBS volumes, Elastic IPs, and other resources may continue to incur
    charges depending on your configuration.
-   For a production deployment, configure HTTPS, logging, monitoring,
    secure credentials, and a controlled deployment strategy.

## What I Learned

-   Creating and maintaining a GitHub repository.
-   Writing a Dockerfile for an Nginx static website.
-   Building Docker images and running containers.
-   Mapping container port 80 to EC2 port 80.
-   Testing the website with `curl`.
-   Configuring AWS Security Group rules for HTTP.
-   Understanding Jenkins Pipeline stages for checkout, build, and
    deployment.
-   Troubleshooting Docker installation, port mapping, and public
    network access.

------------------------------------------------------------------------

**Author:** Akash Tryambak Jadhav\
**Project:** AWS + Jenkins + Docker CI/CD Website Deployment
