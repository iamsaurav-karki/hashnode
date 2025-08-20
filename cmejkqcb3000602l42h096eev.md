---
title: "Creating a Kubernetes User with RBAC and Custom Kubeconfig: Step-by-Step Guide"
datePublished: Tue Aug 12 2025 12:47:03 GMT+0000 (Coordinated Universal Time)
cuid: cmejkqcb3000602l42h096eev
slug: creating-a-kubernetes-user-with-rbac-and-custom-kubeconfig-step-by-step-guide-2eb3653c0fe9
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1755670239573/2176cbd1-df85-42ec-a479-46bb9818413a.png

---

### Introduction

Kubernetes offers fine-grained access control through **RBAC (Role-Based Access Control)**. By creating dedicated users with specific permissions, we can ensure secure and controlled cluster access.

In this guide, I’ll walk through **creating a Kubernetes user**, generating and approving a certificate signing request (CSR), and configuring a **custom kubeconfig** file all with RBAC in place.

### Why Use RBAC in Kubernetes?

RBAC enables you to:

* Assign permissions per **user** or **service account**
    
* Restrict actions to specific **namespaces** or **resources**
    
* Enforce the **principle of least privilege**
    
* Maintain an **audit trail** for compliance and security
    

### Step 1: Generate Certificates for the User

Kubernetes client authentication can use certificates. Let’s generate a private key for our user **saurav**:

```markdown
openssl genrsa -out saurav.pem 2048
```

First Install openssl if not installed.

Create a **Certificate Signing Request (CSR)**:

```markdown
openssl req -new -key saurav.pem -out saurav.csr -subj "/CN=saurav"
```

Here, **CN** (Common Name) represents the username recognized by Kubernetes.

### Step 2: Create a Kubernetes CertificateSigningRequest Resource

Kubernetes requires the CSR in **base64** format:

```markdown
cat saurav.csr | base64 | tr -d "\n" && echo
```

Use the output to create a **CSR manifest**:

sudo vim saurav\_CSR.yaml

```markdown
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
name: saurav
spec:
request: <base64_encoded_csr>
signerName: kubernetes.io/kube-apiserver-client
expirationSeconds: 86400
usages:
- digital signature
- key encipherment
- client auth
```

Apply it:

kubectl apply -f saurav\_CSR.yaml

### Step 3: Approve the CSR

List pending CSRs:

```markdown
kubectl get csr
```

Approve the request:

```markdown
kubectl certificate approve saurav
```

Extract the issued certificate:

```markdown
kubectl get csr saurav -o jsonpath='{.status.certificate}' | base64 --decode > saurav.crt
```

### Step 4: Create a Custom Kubeconfig for the User

We now embed:

* client-certificate-data (base64 of saurav.crt)
    
* client-key-data (base64 of saurav.pem)
    

```markdown
cat saurav.crt | base64 | tr -d "\n" && echo # output of this command goes to 
cat saurav.pem | base64 | tr -d "\n" && echo # output of this command goes to
```

Add these into a kubeconfig template that points to your cluster’s **CA data** and **API server URL**.

Sample kubeconfig.yaml

```markdown
apiVersion: v1
clusters:
- cluster:
certificate-authority-data: LS0tLS1CRUdJTiBDRV45Q0FURS0tLS0tCk1JSUJlVENDQVIrZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWtNU0l3SUFZRFZRUUREQmx5YTJVeUxYTmwKY25abGNpMWpZVUF4TnpJek1qa3hOelE0TUI0WERUSTBNRGd4TURFeU1Ea3dPRm9YRFRNME1EZ3dPREV5TURrdwpPRm93SkRFaU1DQUdBMVVFQXd3WmNtdGxNaTF6WlhKMlpYSXRZMkZBTVRjeU16STVNVGMwT0RCWk1CTUdCeXFHClNNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJJRlUxZjY3NjRiNEMwMW5NcEd2d2V1enJ3dW5SY0tTdXNnOExjWEYKa2Y5bHEydm95bXRJYndHQ3lnRzZqamF6bkhjMUgwUE5DZEx5M2xGVkhaWWV0N09qUWpCQU1BNEdBMVVkRHdFQgovd1FFQXdJQ3BEQVBCZ05WSFJNQkFmsfsrwr432dBMVVkRGdRV0JCUjJTRW01czJmZ1V5enNsOEF0CjBYbkVrcExrQVRBS0JnZ3Foa2pPUFFRREFnTklBREJGQWlFQXpTalFIZWYwNzRkZWZ6TG5WVExpL0xadkNiancKOTlsN09qQisvUHRhQ09zQ0lHM2trQ2ZvdFFJRmZoS2NLaThxTmpqcVFGaVo0UzV1Zjg3VVlhbm9vZDRzCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
server: https://:6443
name: default
contexts:
- context:
cluster: default
user: default
name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
user:
client-certificate-data: <replace with output of this cat saurav.crt | base64 | tr -d "\n" && echo>
client-key-data: <output of these here: cat saurav.pem | base64 | tr -d "\n" && echo>
```

### Step 5: Create RBAC Role and RoleBinding

Define a role for namespace default:

```markdown
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
namespace: recharge-v2
name: recharge-prod-role
rules:
- apiGroups: [""]
resources: ["pods","pods/log"]
verbs: ["get", "watch", "list", "delete"]
```

Bind the role to the user:

```markdown
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: recharge-prod-rolebinding
namespace: default
subjects:
- kind: User
name: saurav
apiGroup: rbac.authorization.k8s.io
roleRef:
kind: Role
name: recharge-prod-role
apiGroup: rbac.authorization.k8s.io
```

Apply the manifests o role and rolebinding.

### Step 6: Test the User Access

Use the new kubeconfig:

```markdown
kubectl - kubeconfig=kubeconfig.yaml get pods -n dev
```

The user should only be able to **list**, **watch**, **get**, and **delete pods** in the dev namespace.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755670238027/047e5248-f852-4256-9a24-cc343d798520.jpeg align="left")

### Conclusion

By combining RBAC, certificates, and a custom kubeconfig, we created a secure, limited-access Kubernetes user. This approach:

* Protects sensitive resources
    
* Limits user permissions
    
* Enables compliance and audit readiness
    

This process is ideal for **granting developers, testers, or external partners** controlled access to your cluster.