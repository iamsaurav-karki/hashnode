---
title: "🚀 DevOps Journey: Building and Deploying a Task Manager Web Application with AI-Assisted Development"
seoTitle: "Deploy Laravel App to AWS EC2 with GitLab CI/CD"
seoDescription: "Learn how to deploy a Laravel Task Manager app to AWS EC2 using GitLab CI/CD, Docker containers, and AI-assisted development. "
datePublished: Wed May 21 2025 18:15:00 GMT+0000 (Coordinated Universal Time)
cuid: cmelbtvpq000602laf46h4emu
slug: devops-journey-building-and-deploying-a-task-manager-web-application-with-ai-assisted-development
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1755776129385/46469cf2-f162-428f-9461-f4fc6f7a2482.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1755776065409/93855a8e-5d4e-4a2b-bfc7-426cd3b7a1f2.png
tags: ec2, laravel, aws, devops, cicd, gitlab-ci, ai-assisted-coding

---

# Introduction

As a DevOps Engineer, I have completed an exciting project that combines modern development practices with cutting-edge AI assistance. This article covers my journey of deploying a full-stack Task Manager application to AWS EC2 using GitLab CI/CD, while leveraging Claude 3.7 Sonnet for rapid application development.

## Project Overview

This project represents the perfect blend of development and operations - using AI to handle application code generation while focusing on mastering the DevOps pipeline for Laravel applications. The result is a production-ready Task Manager built with Laravel/Vue.js/MySQL stack, fully containerized and automatically deployed through a robust CI/CD pipeline.

### 🛠️ Tech Stack

* **Backend**: Laravel (PHP Framework)
    
* **Frontend**: Vue.js
    
* **Database**: MySQL
    
* **Containerization**: Docker & Docker Compose
    
* **CI/CD**: GitLab CI/CD with self-hosted runner
    
* **Cloud Platform**: AWS EC2
    
* **AI Assistant**: Claude 3.7 Sonnet for code generation
    

## Architecture and Infrastructure

### Application Architecture

The Task Manager application follows a modern three-tier architecture:

1. **Presentation Layer**: Vue.js frontend providing an intuitive user interface
    
2. **Business Logic Layer**: Laravel backend handling API endpoints and business rules
    
3. **Data Layer**: MySQL database for persistent storage
    

### Infrastructure Setup

* **EC2 Instance**: t2.medium instance running Ubuntu
    
* **Self-hosted GitLab Runner**: Configured for optimal cloud deployment
    
* **Docker Environment**: Containerized application stack for consistency across environments
    

## The DevOps Pipeline Journey

### 1\. AI-Assisted Development Phase

The first breakthrough in this project was leveraging Claude 3.7 Sonnet to rapidly generate the Task Manager application code. This approach allowed me to:

* Focus entirely on DevOps practices rather than spending time on application development
    
* Generate clean, production-ready Laravel and Vue.js code
    
* Implement best practices in the codebase from the start
    
* Accelerate the development timeline significantly
    

**Key Benefits of AI-Assisted Development:**

* Rapid prototyping and code generation
    
* Consistent code quality and structure
    
* More time to focus on infrastructure and deployment
    
* Learning opportunity to understand Laravel best practices
    

### 2\. Containerization Strategy

Docker containerization was crucial for ensuring consistency across development and production environments. The setup includes:

**Docker Compose Configuration:**

yaml

```yaml
# docker-compose.yml
version: "3"
services:
    # PHP service
    app:
        build:
            context: .
            dockerfile: Dockerfile
        container_name: todo_app
        restart: unless-stopped
        tty: true
        environment:
            SERVICE_NAME: app
            SERVICE_TAGS: dev
        working_dir: /var/www/html
        volumes:
            - ./:/var/www/html
            - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini
        networks:
            - app-network
        depends_on:
            - db

    # Nginx service
    webserver:
        image: nginx:alpine
        container_name: todo_webserver
        restart: unless-stopped
        tty: true
        ports:
            - "8080:80"
        volumes:
            - ./:/var/www/html
            - ./nginx/conf.d/:/etc/nginx/conf.d/
        networks:
            - app-network
        depends_on:
            - app

    # MySQL service
    db:
        image: mysql:8.0
        container_name: todo_db
        restart: unless-stopped
        tty: true
        ports:
            - "3306:3306"
        environment:
            MYSQL_DATABASE: ${DB_DATABASE}
            MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
            MYSQL_PASSWORD: ${DB_PASSWORD}
            MYSQL_USER: ${DB_USERNAME}
            SERVICE_TAGS: dev
            SERVICE_NAME: mysql
        volumes:
            - dbdata:/var/lib/mysql
        networks:
            - app-network

# Docker volumes
volumes:
    dbdata:
        driver: local

# Docker networks
networks:
    app-network:
        driver: bridge
```

**Benefits of Containerization:**

* Environment consistency between development and production
    
* Simplified deployment process
    
* Easy scaling and management
    
* Isolation of application dependencies
    

### 3\. GitLab CI/CD Pipeline Implementation

The heart of this DevOps project lies in the sophisticated CI/CD pipeline configured in GitLab. The pipeline consists of three main stages:

#### Build Stage

* Compiles and builds the Laravel application
    
* Runs composer install for PHP dependencies
    
* Builds Docker images
    
* Caches dependencies for faster subsequent builds
    

#### Push Stage

* Pushes built Docker images to the container registry
    
* Tags images appropriately for version control
    
* Ensures image availability for deployment
    

#### Deploy Stage

* Deploys the application to EC2 instance
    
* Performs health checks
    
* Manages container lifecycle
    
* Handles database migrations and configurations
    

**.gitlab-ci.yaml**

```yaml
stages:
  - build
  - push
  - deploy

cache:
  paths:
    - .docker/cache
variables:
  DOCKER_HOST: "tcp://docker:2375"
  DOCKER_TLS_CERTDIR: ""

build:
  stage: build
  tags:
    - saurav
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
    
  before_script:
    # Docker setup
    - docker info
    - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

  script:
    - docker-compose down -v
    - docker-compose -f docker-compose.yml build
    - docker-compose -f docker-compose.yml up -d
    - sleep 10
    - docker-compose -f docker-compose.yml ps
    # Get container IDs
    - APP_CONTAINER_ID=$(docker-compose -f docker-compose.yml ps -q app)
    - WEBSERVER_CONTAINER_ID=$(docker-compose -f docker-compose.yml ps -q webserver)
    
    # Commit containers
    - docker commit $APP_CONTAINER_ID $DOCKER_USERNAME/todo_app:latest
    - docker commit $WEBSERVER_CONTAINER_ID $DOCKER_USERNAME/todo_webserver:latest
    
    # Cache images
    - mkdir -p .docker/cache
    - docker save $DOCKER_USERNAME/todo_app:latest $DOCKER_USERNAME/todo_webserver:latest | gzip > .docker/cache/images.tar.gz

push:
  stage: push
  tags:
    - saurav
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  before_script:
    - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
    - mkdir -p .docker/cache
    - docker load < .docker/cache/images.tar.gz || true
  script:
    - docker push $DOCKER_USERNAME/todo_app:latest
    - docker push $DOCKER_USERNAME/todo_webserver:latest
  only:
    - main
deploy:
  stage: deploy
  tags:
    - saurav
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  before_script:
    - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
    - docker pull $DOCKER_USERNAME/todo_app:latest
    - docker pull $DOCKER_USERNAME/todo_webserver:latest
  script:
    - echo "Deploying application on EC2..."
    - docker-compose -f docker-compose.yml down --remove-orphans
    - docker-compose -f docker-compose.yml up -d
    - sleep 15

    # Install dependencies
    - docker-compose -f docker-compose.yml exec -T app bash -c "composer install --optimize-autoloader --no-dev"

    # Set permissions
    - docker-compose -f docker-compose.yml exec -T app bash -c "chown -R www-data:www-data /var/www/html/storage"
    - docker-compose -f docker-compose.yml exec -T app bash -c "chmod -R 775 /var/www/html/storage"
    - docker-compose -f docker-compose.yml exec -T app bash -c "chown -R www-data:www-data /var/www/html/bootstrap/cache"
    - docker-compose -f docker-compose.yml exec -T app bash -c "chmod -R 775 /var/www/html/bootstrap/cache"
    
    # Run Laravel commands
    - docker-compose -f docker-compose.yml exec -T app php artisan migrate --force
    - docker-compose -f docker-compose.yml exec -T app php artisan config:cache
    - docker-compose -f docker-compose.yml exec -T app php artisan route:cache
  only:
    - main
```

### 4\. Self-Hosted GitLab Runner Configuration

Setting up a self-hosted GitLab Runner on EC2 provided several advantages:

**Configuration Benefits:**

* Direct access to AWS resources
    
* Reduced network latency
    
* Custom environment setup
    
* Cost-effective for frequent deployments
    

**Runner Setup Process:**

1. EC2 instance preparation with Docker and GitLab Runner
    
2. Runner registration with the GitLab project
    
3. Custom executor configuration for optimal performance
    
4. Security group configurations for proper access
    

## Key DevOps Achievements

### 🔄 Continuous Integration/Continuous Deployment

* Automated deployment pipeline
    
* Zero-downtime deployment strategy
    
* Rollback capabilities for failed deployments
    

### 🐳 Container Orchestration

* Multi-container application management
    
* Environment-specific configurations
    
* Scalable architecture design
    

## Challenges and Solutions

### Challenge 1: Laravel-Specific Deployment Requirements

**Problem**: Laravel applications require specific configurations for production deployment. **Solution**: Created custom Docker configurations with proper PHP extensions, Composer optimizations, and Laravel-specific environment setup.

### Challenge 2: Database Migration Management

**Problem**: Ensuring database schema consistency across deployments. **Solution**: Integrated Laravel migrations into the CI/CD pipeline with proper rollback strategies.

### Challenge 3: Environment Variables Management

**Problem**: Securing sensitive configuration data. **Solution**: Utilized GitLab CI/CD variables and Docker secrets for secure credential management.

## Learning Outcomes and Skills Developed

This project significantly enhanced my DevOps skill set:

### Technical Skills

* **Container Technologies**: Advanced Docker and Docker Compose usage
    
* **CI/CD Pipelines**: GitLab CI/CD best practices and optimization
    
* **Cloud Infrastructure**: AWS EC2 configuration and management
    
* **Laravel Deployment**: Production-ready PHP application deployment
    

### Process Skills

* **Problem Solving**: Debugging pipeline issues and deployment challenges
    
* **Documentation**: Comprehensive project documentation and knowledge sharing
    

## Appendix: Screenshots

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755775703694/c3ad978c-b63f-427c-b803-022335b7fc24.jpeg align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755775723915/7e9de794-b34c-4252-8b41-33f518069918.jpeg align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755775740805/8ebe07c8-f62a-488f-b8b4-7affc3f64b69.jpeg align="center")

## Future Enhancements

The project provides a solid foundation for further improvements:

### Planned Enhancements

* **Kubernetes Migration**: Container orchestration at scale
    
* **Infrastructure as Code**: Terraform implementation for infrastructure management
    
* **Advanced Monitoring**: Prometheus and Grafana integration
    
* **Multi-Environment Setup**: Staging and production environment separation
    
* **Automated Testing**: Enhanced test coverage and automated quality gates
    

### Scalability Considerations

* Load balancer integration
    
* Database clustering and replication
    
* CDN implementation for static assets
    
* Microservices architecture exploration
    

## Conclusion

This DevOps journey demonstrates the powerful combination of AI-assisted development and modern deployment practices. By leveraging Claude 3.7 Sonnet for rapid application development, I was able to focus entirely on mastering the DevOps pipeline for Laravel applications.

The project showcases:

* **Modern Development Practices**: AI-assisted coding combined with traditional DevOps
    
* **Production-Ready Infrastructure**: Scalable and maintainable deployment architecture
    
* **Continuous Learning**: Practical application of DevOps principles and tools
    
* **Innovation**: Embracing new technologies while maintaining reliability