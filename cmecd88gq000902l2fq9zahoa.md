---
title: "Devopsified and Deployment of Java Micronaut Web Application into k3s kubernetes cluster with custom domain , ssl certificates and monitoring."
seoTitle: "deployment of Java Micronaut Web Application into k3s "
datePublished: Fri Aug 15 2025 05:06:16 GMT+0000 (Coordinated Universal Time)
cuid: cmecd88gq000902l2fq9zahoa
slug: devopsified-and-deployment-of-java-micronaut-web-application-into-k3s-kubernetes-cluster-with-custom-domain-ssl-certificates-and-monitoring
tags: deployment, kubernetes, devops, k3s, grafana

---

In this project, I focused on complete DevOps implementation of a three-tier web application built using Micronaut, React, and ScyllaDB. The application was containerized, deployed in a self-hosted lightweight K3s Kubernetes cluster, and exposed securely over a custom domain with Let's Encrypt SSL certificates and Traefik Ingress.

#### **i) Containerization and Image Management**

The frontend (React) and backend (Micronaut) components were:

* **Dockerized** separately with production-ready Dockerfiles.
    
* **Pushed to Docker Hub** for use in Kubernetes deployment.  
    This enabled version-controlled, reproducible deployments across environments.
    

#### **ii) Kubernetes Deployment on K3s**

A **K3s cluster** was manually provisioned and configured to run the application:

* Kubernetes **manifests** were written for deployments, services, and ingress rules.
    
* Each tier (frontend, backend, database) was independently deployed in its own pod.
    
* The backend services connected to a **ScyllaDB cluster** set up on two virtual machines.
    

#### **iii) Ingress & Domain Setup with Traefik and SSL**

**Traefik** was used as the Kubernetes ingress controller to handle traffic routing:

* **Ingress resources** were defined with custom routing rules for / and /api paths.
    
* The domain [notes.ksaurav.com.np](http://notes.ksaurav.com.np) was mapped to the Traefik LoadBalancer External IP.
    
* Use Cloud Flare as a DNS Manager.
    
* **Let’s Encrypt certificates** were issued automatically using cert-manager and a configured ClusterIssuer.
    

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfF1DBqIt0-d1TjRJjZIVUp3HV2iBzJ21_376SLKoITkKVH5dEOb7HoIJ9eZx6esl2IA6yBM0xekRH1GgB-EPvUoRIXS7QJSeuatViC2gQaPn5Pui1GoD357CxE4e6rkaGnkzd3zw?key=fTOKKJ83bvdGugPblu1ifA align="left")

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfx5pqfk86w74PCJuUDxYDvS9VJlm1gnTwvPEBpfi9NAkncCDzYBYBMn8enTKoW-dAgGwMQvTRhAmS0P39_0AlGc_gf5XFKa9-NFM4rHk9E3oLDBbcbAwZ-_zhDn1Lx51XY5xW-vw?key=fTOKKJ83bvdGugPblu1ifA align="left")

**Figure : Kubernetes manifest structure of the project**

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdAag_42WWWduEpYLcgCcOtAga-etQbvdcBPNg74SIIZSAqDTdy097kBNVqCjQk_JQ0T6syRMK0rBOk03oqaw3UwY11_4nFFtGEWttgZaXhQnJt9wo1fQXaJfOiG9sqihEdGml99w?key=fTOKKJ83bvdGugPblu1ifA align="left")

**Figure : Details of the nodes and running resources in k8s cluster**

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfZdVwSMwNH_6FKtGAL6wiD9D85E5ug-pbw01Mo5Dy3DeqMBnVpG_AbobUDrOEHCZYjfXn0wKS3FUkskVakaLejQ3eW9iUMOQRlDbUjijFUM_RxxAvOEqjs63s9jCBTwc8mBgqX?key=fTOKKJ83bvdGugPblu1ifA align="left")

**Figure : Final Deployed Three Tier notes maker app with SSL certificates and Custom Domain.**

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdeyYoFXriEY7yfuEb06U0Y4nxvChB5XzHhxAorl_1VstaFjg4Uzc4sBrMoGiLCqpVV2vaSpJYczM7xA8Qdf4lL5ugZZeOBvNuHI0K7a2QST62db25MeP-RBFW-wMLchujZHC75WA?key=fTOKKJ83bvdGugPblu1ifA align="left")

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcFL5lgQa6qYgcRhA6qm3F5FZsyG9S3jinhXMBhmUy0czQMShYTJSzLI8U03kPYOBQI6mNQzUjU54pJUuMHURz9CbcW-KvkbIxpQoWmriF42ArKent270kxOcCsuXfxefrGFSkBMg?key=fTOKKJ83bvdGugPblu1ifA align="left")

**Figure : Grafana Dashboard for application metrics monitoring**

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcYilbooKjos9nwDuvpXONqjuyLpMDQUPnO-LQZn-CJ8p6k2KH_d0JIbgiXCZhyTTg7rnzuieMwvy4fuGSZ_NxC37BquHTzhiaKMGiGdRGKBvCtj-mx7xlzNeugya5Y5MrUHhGj?key=fTOKKJ83bvdGugPblu1ifA align="left")

**Figure : Lens Dashboard for Cluster Management**

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcUN23ploFKaNEVxPzIRVwl9AnyORa3i9T5HBMTVLo-4J8gjQWqPPQ3zMoxn4hV57-TbJ6a_pCbztI7daI2pML3QWu3OO_GCVReDCiSuFjKLkLwWZEOUW-vNz5FKqjpdpph59OX?key=fTOKKJ83bvdGugPblu1ifA align="left")

**Figure : New Relic Dashboard for K8s cluster and resource monitoring**

Complete manifests of the project:

Backend-deployment:

```markdown
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-app-backend
  namespace: notes-maker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notes-app-backend
  template:
    metadata:
      labels:
        app: notes-app-backend
    spec:
      containers:
      - name: notes-app-backend
        image: sauravkarki/backend:latest
        ports:
        - containerPort: 8080
        env:
        - name: MICRONAUT_SERVER_HOST
          value: "0.0.0.0"
        - name: MICRONAUT_SERVER_PORT
          value: "8080"
        - name: MICRONAUT_SERVER_CORS_ENABLED
          value: "true"
        - name: MICRONAUT_SERVER_CORS_CONFIGURATIONS_WEB_ALLOWED_ORIGINS
          value: "https://notes.ksaurav.com.np,https://notes.ksaurav.com.np,https://45.79.121.155:30000,https://localhost:3000,https://172.105.36.6:30000"
        - name: MICRONAUT_SERVER_CORS_CONFIGURATIONS_WEB_ALLOWED_METHODS
          value: "GET,POST,PUT,DELETE,OPTIONS"
        - name: MICRONAUT_SERVER_CORS_CONFIGURATIONS_WEB_ALLOWED_HEADERS
          value: "Content-Type,Authorization,X-Requested-With"
        - name: CASSANDRA_DEFAULT_CONTACT_POINTS
          value: "192.168.148.77,192.168.130.98"
        - name: CASSANDRA_DEFAULT_PORT
          value: "9042"
        - name: CASSANDRA_DEFAULT_DATACENTER
          value: "DC1"
        - name: CASSANDRA_DEFAULT_KEYSPACE
          value: "notes_maker"
        - name: LOG_LEVEL
          value: "DEBUG"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 20
          periodSeconds: 5
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
```

backend-service:

```markdown
apiVersion: v1
kind: Service
metadata:
  name: notes-backend-service
  namespace: notes-maker
spec:
  selector:
    app: notes-app-backend
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
  type: ClusterIP
```

frontend-deployment:

```markdown
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-frontend
  namespace: notes-maker
spec:
  replicas: 2
  selector:
    matchLabels:
      app: notes-frontend
  template:
    metadata:
      labels:
        app: notes-frontend
    spec:
      containers:
      - name: notes-frontend
        image: sauravkarki/frontend:latest
        ports:
        - containerPort: 3000
        env:
        - name: REACT_APP_API_URL
          value: "https://notes.ksaurav.com.np/api"
        - name: REACT_APP_ENVIRONMENT
          value: "production"
```

frontend-service

```markdown

apiVersion: v1
kind: Service
metadata:
  name: notes-frontend-service
  namespace: notes-maker
spec:
  selector:
    app: notes-frontend
  ports:
  - port: 3000
    targetPort: 3000
    protocol: TCP
  type: ClusterIP
```

lets-encrypt:

```markdown
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: notes-maker
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: saurav@gmail.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: traefik
```

notes-ingress

```markdown
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: notes-app-ingress
  namespace: notes-maker
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web,websecure
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - notes.ksaurav.com.np
    secretName: notes-tls-secret
  rules:
  - host: notes.ksaurav.com.np
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: notes-frontend-service
            port:
              number: 3000
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: notes-backend-service
            port:
              number: 8080
```

For containerization: docker-compose:

```markdown
version: '3.8'
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - MICRONAUT_SERVER_HOST=0.0.0.0
      - MICRONAUT_SERVER_PORT=8080
      - CASSANDRA_DEFAULT_CONTACT_POINTS=scylladb
      - CASSANDRA_DEFAULT_PORT=9042
      - CASSANDRA_DEFAULT_DATACENTER=datacenter1
      - CASSANDRA_DEFAULT_KEYSPACE=notes_maker
      - LOG_LEVEL=DEBUG
    depends_on:
      scylladb:
        condition: service_healthy
    networks:
      - notes_maker_network
    restart: on-failure

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://172.64.36.6:8080/api
      - REACT_APP_ENVIRONMENT=development
    depends_on:
      - backend
    networks:
      - notes_maker_network

  scylladb:
    image: scylladb/scylla:5.4
    ports:
      - "9042:9042"
    volumes:
      - scylla_data:/var/lib/scylla
    networks:
      - notes_maker_network
    healthcheck:
      test: ["CMD-SHELL", "cqlsh -e 'SELECT now() FROM system.local;' || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 10
      start_period: 60s
    environment:
      - SCYLLA_CLUSTER_NAME=notes_maker_cluster       
volumes:
  scylla_data:

networks:
  notes_maker_network:
    driver: bridge
```