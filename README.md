# Green and Blue Deployment using EKS and Jenkins

This guide outlines the steps to create a green-blue deployment in Kubernetes using EKS and Jenkins. The setup includes roles, RBAC, and service accounts for secure interaction between Jenkins and the Kubernetes cluster.

---

## Prerequisites
1. **EKS Cluster**:
   - Launch an EKS cluster on a separate instance.
   - Ensure `kubectl` is configured to access the cluster.
2. **Jenkins Server**:
   - Install Docker and `kubectl` on the Jenkins server.
   - Install Trivy for vulnerability scanning.
3. **SonarQube Server**:
   - Launch a server for SonarQube and install Docker.
4. **Namespace**:
   - Create a namespace in the EKS cluster for Jenkins operations.

---

## Step-by-Step Setup

### 1. Create a Namespace
```bash
kubectl create namespace webapps
```

---

### 2. Create a ServiceAccount
Create a file `sa.yaml`:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```

Apply the ServiceAccount:
```bash
kubectl apply -f sa.yaml
```

---

### 3. Create a Role
Create a file `role.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - secrets
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

Apply the Role:
```bash
kubectl apply -f role.yaml
```

---

### 4. Create a RoleBinding
Create a file `rolebinding.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role 
subjects:
- namespace: webapps 
  kind: ServiceAccount
  name: jenkins 
```

Apply the RoleBinding:
```bash
kubectl apply -f rolebinding.yaml
```

---

### 5. Create a Token for the ServiceAccount
Create a file `token.yaml`:

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: jenkins
```

Apply the token:
```bash
kubectl apply -f token.yaml
```

Retrieve the token:
```bash
kubectl get secret mysecretname -n webapps
```
Save the token for later use in Jenkins.

---

## Example Token
Here is a placeholder example of the token generated:

```plaintext
eyJhbGciOiJSUzI1NiIsImtpZCI6IkQ5Snl2a2VUSUtxUk10NE10ajlXYmZ5dlJjcm1EQXF3bUY4R2k4ZmdpZXMifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ3ZWJhcHBzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im15c2VjcmV0bmFtZSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqZW5raW5zIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYmQ4NTYxY2YtZTkzNS00YjdkLWEwZWUtYTljZmIzZTZkYjIyIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OndlYmFwcHM6amVua2lucyJ9.BvvItJcQXu9nHJ8lSptnXDyWtFKE8I7RiMA6omCd_hiqnx-GHznuIuE1AxtB4DSuNpZyUaa7_lMu7D_vO9srUqqAQsqMBz2d3Vzbuf-YAKztjkrAb_wQWEFfE36PruHSttBz_SxHuryFZO437JqTnDcVDcuvuJ-aHWHiwb8bLyKuQ_GKEyT4EQ3PiGDFEP_ifc2JJrh90JjyINifCHvDy3ybHvNVO3Nf1p5R04fOyXcxwo5b9SB5OLc1uJoifbMm6iJ07MyD1ZSzL7cLMsV0fxSeM4ryZNXZnPZ4ur4vDpXkcuOmL5SMQtma26MOJ_8h319vRbczoNBmgx--Cfk6Ew
```

---

## Images
### Kubernetes Namespace Creation
![Kubernetes Namespace Creation](https://tse1.mm.bing.net/th?id=OIP.VXnkCHS76XEKhCSR9nX87QHaD2&pid=Api)

### Kubernetes Service Account Setup
![Kubernetes Service Account Setup](https://tse1.mm.bing.net/th?id=OIP.xNpwMEmqm94R3wByXI3qHAAAAA&pid=Api)

### Kubernetes RBAC Role and RoleBinding
![Kubernetes RBAC Role and RoleBinding](https://tse1.mm.bing.net/th?id=OIP.PaEz8SASsqO043XWecKr6wHaEb&pid=Api)

---

This concludes the setup for the green-blue deployment with Jenkins integration. Deployments and rollbacks can now be automated via Jenkins pipelines using the created ServiceAccount and token.
