---
title: "Building a Production-Ready, No-Downtime Kubernetes Cluster with RKE2, MetalLB, Envoy Gateway and Cloudflare Integration"
seoTitle: "RKE2, MetalLB & Envoy for HA Kubernetes Cluster"
seoDescription: "build a high availability Kubernetes cluster using RKE2, MetalLB for bare-metal LoadBalancer, and Envoy Gateway for secure ingress traffic management."
datePublished: Mon Aug 11 2025 08:33:56 GMT+0000 (Coordinated Universal Time)
cuid: cme6uvwmr000a02jz3ijdfcjk
slug: no-downtime-kubernetes-setup-rke2-metallb-envoy-cloudflare
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1754900853464/5d779f5b-9bca-422b-af25-dd7e5254d113.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1754901039618/bc8573e8-ce7e-4351-9899-9441b43e54d0.png
tags: dns, cloudflare, kubernetes, load-balancing, production-ready, high-availability, kubernetes-cluster, metallb, envoy-proxy, rke2, bare-metal, envoy-gateway

---

In todays modern cloud-native infrastructure, reliability and zero downtime are crucial. Here, I will be guiding to setup **production-grade RKE2 Kubernetes cluster**, configuring **MetalLB for LoadBalancer support**, deploying the **Envoy Gateway** for traffic ingress and managing DNS records with **Cloudflare** to enable secure, highly available access.

# **Introduction**

We know, Kubernetes is a container orchestration platform for managing microservices and cloud-native applications. However, a cluster alone isn’t enough for high availability and production ready.

**For High availability** and **zero downtime we need:**

* Proper **networking and load balancing**
    
* Robust **ingress and API gateway solutions**
    
* Reliable **messaging systems** for decoupled services
    
* Automated \*\*DNS and SSL management
    

Here is the details steps to setup production ready Bare metal setup for zero downtime:

## **Step 1: Setting up RKE2 Kubernetes Cluster**

We use RKE2 for bare metal setup. In production , High Availability(HA) and fault tolerance plays a significant role. So to preserve high availability i am setting up 8 nodes cluster where 3 will be control plane and 5 will be Data plane.

### **Installing RKE2**

**Server Node (control plane) Installation. ( Run on three node)**

1\. Run the installer

```markdown
curl -sfL https://get.rke2.io | sh -
```

2\. Enable the rke2-server service

```markdown
systemctl enable rke2-server.service
```

3\. Start the service

```markdown
systemctl start rke2-server.service
```

4\. Follow the logs, if you like

```markdown
journalctl -u rke2-server -f
```

**After running this installation:**

The rke2-server service will be installed. The rke2-server service will be configured to automatically restart after node reboots or if the process crashes or is killed.

Additional utilities will be installed at /var/lib/rancher/rke2/bin/. They include: kubectl, crictl, and ctr. Note that these are not on your path by default.

A kubeconfig is located at /etc/rancher/rke2/rke2.yaml.

A token that can be used to register other server or agent nodes will be created at /var/lib/rancher/rke2/server/node-token

### **How RKE2 Join Tokens Work**

By default, RKE2 creates a **random token** stored at /var/lib/rancher/rke2/server/node-token on the server node. Agents use this token to authenticate and join the cluster.

You can replace this default token with **your own custom token**. This is especially useful for automation or managing multiple clusters.

### **Steps to Create and Use Your Own Custom Token**

1. **Create a token string**
    

Pick a secure token string — it can be any combination of characters, but keep it secret.

Example:

```markdown
MY_CUSTOM_TOKEN="mysecuretoken12345"
```

2. **Replace the token on the server node**
    

On your RKE2 server node, overwrite the node-token file with your custom token:

```markdown
echo "$MY_CUSTOM_TOKEN" | sudo tee /var/lib/rancher/rke2/server/node-token

sudo chmod 600 /var/lib/rancher/rke2/server/node-token
```

3. **Restart RKE2 server service**
    

```markdown
sudo systemctl restart rke2-server.service
```

**Worker Node (Data Plane) Installation**

**Run on 5 Worker nodes:**

1\. Run the installer

```markdown
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -
```

This will install the rke2-agent service and the rke2 binary onto your machine. Due to its nature, It will fail unless it runs as the root user or through sudo

2\. Enable the rke2-agent service

```markdown
systemctl enable rke2-agent.service
```

3\. Configure the rke2-agent service

```markdown
mkdir -p /etc/rancher/rke2/

vim /etc/rancher/rke2/config.yaml
```

Content for config.yaml:

```markdown
server: https://<server>:9345 ( first server node ip)

token: <token from server node>
```

The rke2 server process listens on port 9345 for new nodes to register. The Kubernetes API is still served on port 6443, as normal.

4\. Start the service

```markdown
systemctl start rke2-agent.service
```

5. Verify nodes joined
    

```markdown
sudo /var/lib/rancher/rke2/bin/kubectl get nodes
```

**To setup kubeconfig:**

```markdown
mkdir -p ~/.kube

sudo cp /etc/rancher/rke2/rke2.yaml ~/.kube/config

sudo chown $(id -u):$(id -g) ~/.kube/config
```

## **Step 2: Configuring MetalLB for LoadBalancer Support**

On bare-metal or VMs, Kubernetes LoadBalancer type services need a load balancer implementation. MetalLB provides this functionality by assigning external IPs from a pool.

### **Installing MetalLB**

Apply manifests:

```markdown
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.2/config/manifests/metallb-native.yaml
```

Create a ConfigMap with your IP address pool (use a range available in your network):

**sudo vim metallb-ip\_assignment.yaml**

```markdown
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 132.48.33.23-132.48.33.23
  - 132.40.34.27-132.40.34.27
```

Note: Please replace the ip with your public ip ranges , this Ip will be assigned to Envoy gateway service of type load balancers and all the services , pods can be accessible through this IP

**sudo vim L2advertisement.yaml**

```markdown
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
  - first-pool
```

kubectl apply -f metallb-ip\_assignment.yaml

kubectl apply -f L2advertisement.yaml

## **Step 3: Deploying Envoy Gateway**

Envoy Gateway is a modern ingress gateway built on the Envoy proxy, which acts as an edge router for Kubernetes clusters.

### **Installing Envoy Gateway with Helm**

Install the Gateway API CRDs and Envoy Gateway:

```markdown
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v0.0.0-latest -n envoy-gateway-system --create-namespace
```

Wait for Envoy Gateway to become available:

```markdown
kubectl wait --timeout=5m -n envoy-gateway-system deployment/envoy-gateway --for=condition=Available
```

Install the GatewayClass, Gateway, HTTPRoute and example app:

```markdown
kubectl apply -f https://github.com/envoyproxy/gateway/releases/download/latest/quickstart.yaml -n default
```

Note: quickstart.yaml defines that Envoy Gateway will listen for traffic on port 80 on its globally-routable IP address, to make it easy to use browsers to test Envoy Gateway.

**<mark>Note:</mark>** Customize quickstart.yaml as per your setup and applications..

The quickstart.yaml is a ready-to-use example manifest designed to help you quickly deploy:

The Gateway API resources (GatewayClass, Gateway, HTTPRoute)

A simple backend application (echo server)

Necessary Kubernetes resources (ServiceAccount, Service, Deployment)

What the Quickstart Does:

**GatewayClass**

```markdown
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: eg
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```

`Defines a GatewayClass named eg.`

`This tells Kubernetes which controller (here Envoy Gateway) will manage Gateways of this class.`

`Mandatory: Yes, you need a GatewayClass so Envoy Gateway knows to control the Gateway resource.`

**Gateway**

```markdown
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: eg
spec:
  gatewayClassName: eg
  listeners:
    - name: http
      protocol: HTTP
      port: 80
```

`Creates a Gateway resource named eg using the GatewayClass eg.`

`Listens on port 80 for HTTP traffic.`

`Acts as the “edge router” that accepts incoming requests.`

`Envoy Gateway watches for Gateways and programs Envoy proxy accordingly.`

`Mandatory: Yes, you must have at least one Gateway to handle incoming traffic.`

**Backend ServiceAccount**

```markdown
apiVersion: v1
kind: ServiceAccount
metadata:
  name: backend
```

`ServiceAccount for the backend pod to run with.`

`Mandatory: No, but recommended for RBAC and security best practices.`

**Backend Service**

```markdown
apiVersion: v1
kind: Service
metadata:
  name: backend
  labels:
    app: backend
spec:
  ports:
    - name: http
      port: 3000
      targetPort: 3000
  selector:
    app: backend
```

`Defines a Service exposing port 3000 for pods labeled app=backend.`

`Allows the Gateway to route traffic to the backend pods.`

`Mandatory: Yes, you need services for routing backend traffic.`

**Backend Deployment**

```markdown
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
      version: v1

  template:
    metadata:
      labels:
        app: backend
        version: v1

    spec:
      serviceAccountName: backend
      containers:
        - image: gcr.io/k8s-staging-gateway-api/echo-basic:v20231214-v1.0.0-140-gf544a46e
          ports:
            - containerPort: 3000
```

`Runs one replica of the echo server backend.`

`This pod serves as the target for HTTPRoute traffic.`

`Mandatory: No — you can replace this with your own apps, but you need some backend pods to route traffic to.`

**HTTPRoute**

```markdown
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: backend
spec:
  parentRefs:
    - name: eg
  hostnames:
    - "www.example.com"
  rules:
    - backendRefs:
        - kind: Service
          name: backend
          port: 3000
          weight: 1
      matches:
        - path:
            type: PathPrefix
            value: /
```

`Routes incoming HTTP requests matching hostname www.example.com and path prefix / to the backend service on port 3000.`

`parentRefs ties this route to the Gateway named eg.`

`Mandatory: Yes, HTTPRoutes (or other route types) tell the Gateway how to route incoming requests to backend services.`

Note: For more customization you can explore envoy gateway and apply the configurations such as:

* Client Traffic Policy
    
* Backend Traffic Policy
    
* Customize EnvoyProxy
    
* Add Grpc routes and so on.
    

## **Step 4: Adding DNS Records to Cloudflare**

Cloudflare manages your DNS and adds features like CDN, security, and SSL termination.

Adding LoadBalancer IP to Cloudflare DNS

* Log in to your Cloudflare dashboard.
    
* Select your domain.
    
* Go to DNS section.
    
* Add an A record:
    
* Name: e.g., app.example.com
    
* IPv4 address: Use the MetalLB LoadBalancer IP assigned to Envoy Gateway
    
* TTL: Auto
    
* Proxy status: Enable Proxy (orange cloud) for Cloudflare protection
    

Save the record.

## **Conclusion**

With these steps, you now have a fully functioning production-ready Kubernetes environment with:

RKE2 cluster with high availability, MetalLB for LoadBalancer IP assignment on bare-metal, Envoy Gateway for ingress traffic routing, Cloudflare DNS for external access and security

This setup ensures no downtime, easy scalability, and secure ideal for modern microservices and cloud-native apps.