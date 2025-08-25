---
title: "Building an End-to-End CI/CD Pipeline with GitLab, Docker, and AWS EKS"
seoTitle: "Building an End-to-End CI/CD Pipeline with GitLab."
seoDescription: "Learn how to build a complete CI/CD pipeline using GitLab CI/CD, Docker, Maven, and SonarQube, and deploy applications seamlessly on AWS. "
datePublished: Thu Aug 21 2025 18:15:00 GMT+0000 (Coordinated Universal Time)
cuid: cmeqya7m5003s02laglti9ib9
slug: building-an-end-to-end-cicd-pipeline-with-gitlab-docker-and-aws-eks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1756116247229/6ee236cd-4900-4cad-9364-ac3a22f7e22b.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1756116234349/84b6ef02-87ab-4d28-badb-810f2f476746.png
tags: docker, yaml, gitlab-ci, gitlab-runner, pipeline, eks, gitlab-cicd

---

# Introduction

In the world of software development, speed and reliability are everything. Companies need to deliver new features quickly, without compromising on stability or security. This is where **CI/CD (Continuous Integration and Continuous Delivery)** pipelines come in.

As part of my learning journey in DevOps and Cloud Computing, I designed and implemented a **complete CI/CD pipeline** for a Java-based application. This project integrates tools like **GitLab CI/CD, Maven, SonarQube, Docker, and AWS EKS (Kubernetes)** to automate the entire software delivery process—from writing code to running it in the cloud.

**In this article, I’ll share:**

* The problem I wanted to solve
    
* The solution architecture
    
* How I built the pipeline step by step
    
* The challenges I faced and how I solved them
    
* My key learnings
    

---

## Problem Statement

When working on software projects, teams usually face these challenges:

* **Manual deployments** take a lot of time and are error-prone.
    
* **Inconsistent environments** make it hard to reproduce bugs ("works on my machine" issue).
    
* **No quality checks** before merging code increases risks of bugs and vulnerabilities.
    
* **Scaling issues** when the application gets more traffic than expected.
    

I wanted to build a solution where:

* Every code change is automatically tested and verified.
    
* Code quality is checked before deployment.
    
* Applications are packaged in containers for consistency.
    
* Deployments are automated and scalable using cloud infrastructure.
    

---

## Solution Overview

To solve the above problems, I built an **end-to-end CI/CD pipeline** with the following tools:

1. **Maven** – to build and test the Java application.
    
2. **SonarQube** – to check code quality and security issues.
    
3. **Docker** – to containerize the application.
    
4. **GitLab Package & Container Registry** – to store build artifacts and Docker images.
    
5. **AWS EKS (Elastic Kubernetes Service)** – to deploy and scale the application.
    

---

## CI/CD Pipeline Design

The pipeline was created in **GitLab CI/CD** and divided into clear stages:

1. **Build & Test** – Run unit tests using Maven.
    
2. **Deploy Artifact** – Package the application and store it in GitLab Package Registry.
    
3. **Code Quality Check** – Use SonarQube to scan the code.
    
4. **Docker Build** – Build the Docker image of the application.
    
5. **Docker Push** – Push the image to GitLab Container Registry.
    
6. **AWS Configure** – Authenticate the GitLab runner with AWS.
    
7. **Kubernetes Deploy** – Deploy the application to AWS EKS using Kubernetes manifests.
    

**.gitlab-ci.yml:**

```yaml
stages:
  - maven-test
  - maven-deploy
  - sonarqube-stage
  - docker-build
  - docker-push
  - docker-run
  - aws-configure
  - kubernetes-stage

maven_test_job: 
  stage: maven-test
  tags:
    - saurav
  script:
    - mvn clean package
    - mvn test
  artifacts:
    name: capestone-project
    paths:
      - target/my-webapp.war
    reports:
      junit:
        - target/surefire-reports/TEST-com.example.MyServletTest-junit.xml
        - target/surefire-reports/TEST-com.example.AppTest-junit.xml

    untracked: false
    when: on_success
    access: all
    expire_in: 30 days

maven_deploy_job:
  stage: maven-deploy
  tags:
    - saurav
  script:
    - mvn clean package
    - ls -l target/my-webapp.war  # Corrected to check for the WAR file
    - mvn deploy -s settings.xml
  artifacts:
    paths:
      - target/my-webapp.war  # Pass the WAR file to next stages


sonarqube-check:
  stage: sonarqube-stage
  tags:
    - saurav
  image: maven:3.6.3-jdk-11
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
    GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script: 
    - mvn verify sonar:sonar -Dsonar.projectKey=udemy2631411_capstone-project-cicd_AZWLlQvd4T47fbOoKwXE
  allow_failure: true
  only:
    - main

docker_build_job:
  stage: docker-build
  tags:
    - saurav
  script:
    - docker build -t registry.gitlab.com/udemy2631411/capstone-project-cicd:$CI_PIPELINE_IID .
  after_script:
    - docker images


docker_push_job:
  stage: docker-push
  tags:
    - saurav
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD registry.gitlab.com
    - docker push registry.gitlab.com/udemy2631411/capstone-project-cicd:$CI_PIPELINE_IID

.docker_run_job:
  stage: docker-run
  tags:
    - saurav
  script:
    - docker run -d -p 83:8080 registry.gitlab.com/udemy2631411/capstone-project-cicd:$CI_PIPELINE_IID

aws_configuration_job:
  stage: aws-configure
  tags:
    - saurav
  before_script:
    - aws configure set aws_access_key_id "$AWS_ACCESS_KEY"
    - aws configure set aws_secret_access_key "$AWS_SECRET_ACCESS_KEY"
    - aws configure set default_region "$AWS_DEFAULT_REGION"
  script:
    - aws s3 ls 

kubernetes_job:
  stage: kubernetes-stage
  tags:
    - saurav
  script:
    - aws eks update-kubeconfig --region us-east-1 --name capstone
    # Update the image tag in the manifest
    - 'sed -i "s|image: registry.gitlab.com/devopsagents/java-project:latest|image: registry.gitlab.com/udemy2631411/capstone-project-cicd:$CI_PIPELINE_IID|g" Application.yml'
    # Apply the updated manifest
    - kubectl apply -f Application.yml

```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756115426202/b396bf22-3dd1-4529-9a7c-29209c9dbc1b.jpeg align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756115556192/17af8eba-11c0-4af3-ab60-ee03e4adc025.jpeg align="center")

## Infrastructure on AWS

For this project, I used AWS services to host and deploy the application:

* **EC2 instances** – to run GitLab runners and supporting services.
    
* **Amazon EKS** – to manage Kubernetes clusters for deployment.
    
* **IAM Roles** – to securely give GitLab access to AWS resources.
    
* **VPC & Security Groups** – to control networking and security.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756115356997/914aefaa-0d82-4d57-85d4-d3f2907c836b.jpeg align="center")

---

## Challenges and How I Solved Them

Like any real-world project, I faced a few challenges:

1. **SonarQube Authentication**
    
    * *Problem*: GitLab couldn’t connect to SonarQube.
        
    * *Solution*: Configured an authentication token in GitLab CI/CD variables and updated the Maven SonarQube plugin.
        
2. **Docker Push to Registry**
    
    * *Problem*: Pipeline failed when pushing Docker images.
        
    * *Solution*: Used GitLab’s built-in variables (`CI_JOB_TOKEN`, `CI_REGISTRY_USER`) for secure login.
        
3. **Deploying to AWS EKS**
    
    * *Problem*: The GitLab runner couldn’t authenticate with EKS.
        
    * *Solution*: Installed AWS CLI and `kubectl` inside the runner and configured it with IAM credentials.
        

---

## Key Learnings

Working on this project gave me hands-on experience with:

* **Automation** – How to completely automate builds, tests, and deployments.
    
* **Code Quality** – Using SonarQube to enforce clean and secure code.
    
* **Containerization** – Packaging applications with Docker for consistency.
    
* **Scalability** – Deploying applications on Kubernetes to handle dynamic traffic.
    
* **CI/CD Best Practices** – Structuring pipelines into modular jobs for easier debugging and maintenance.
    

---

## Conclusion

This project showed me how powerful DevOps practices can be when applied properly. By integrating **GitLab CI/CD, Docker, SonarQube, and AWS EKS**, I built a secure, automated, and scalable pipeline that takes code from commit to deployment in the cloud.

This experience not only improved my technical skills but also gave me confidence to tackle more advanced topics such as monitoring, logging, and advanced deployment strategies like blue-green or canary deployments.

I’m excited to continue learning and applying these practices to real-world systems!

## Appendix: Screenshots

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756115616548/40fdcc93-12f4-41aa-bf88-dd34b2ec5778.jpeg align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756115664992/32604092-31ea-4dcd-8ff7-dd01eadd4724.jpeg align="center")

##