# Kubernetes - A Beginner's Guide  

## Table of Contents  
1. [What is Kubernetes?](#what-is-kubernetes)  
2. [Why Kubernetes?](#why-kubernetes)  
3. [How Kubernetes Works?](#how-kubernetes-works)  
4. [Kubernetes Architecture](#kubernetes-architecture)  
5. [Getting Started with Kubernetes](#getting-started-with-kubernetes)  
6. [Step-by-Step Deployment Example](#step-by-step-deployment-example)  
    - [Step 1: Create a Kubernetes Cluster on Linode](#step-1-create-a-kubernetes-cluster-on-linode)  
    - [Step 2: Configure Kubernetes CLI (`kubectl`)](#step-2-configure-kubernetes-cli-kubectl)  
    - [Step 3: Deploy the Sentiment Analysis App](#step-3-deploy-the-sentiment-analysis-app)  
    - [Step 4: Expose the Application with a Service](#step-4-expose-the-application-with-a-service)  
7. [Additional Concepts](#additional-concepts)  
8. [Cleaning Up](#cleaning-up)  

---

## What is Kubernetes?

Kubernetes (often abbreviated as **K8s**) is an **open-source tool** that helps manage **containers** (lightweight software packages). Instead of running applications on a single server, Kubernetes enables you to:  
1. Run applications across multiple machines (known as **nodes**).  
2. Scale applications automatically based on demand.  
3. Handle failures (e.g., restart crashed applications).  

Think of Kubernetes as an **orchestra conductor** that ensures all parts of your application (the containers) work together seamlessly, regardless of where they’re running.

---

## Why Kubernetes?  

1. **Simplified Deployment**: Deploy applications consistently, even across different environments.  
2. **Automatic Scaling**: Easily add more resources to handle traffic spikes.  
3. **Fault Tolerance**: Automatically replaces failed application instances.  
4. **Resource Efficiency**: Optimizes the use of hardware and resources.  
5. **Portability**: Run applications on any cloud provider or your own machines.  

---

## How Kubernetes Works?  

At its core, Kubernetes runs containers inside **Pods**. These Pods are deployed across a cluster of machines. Kubernetes ensures that:  
- Applications inside Pods are always running.  
- Traffic is routed to the right Pod using **Services**.  
- Resources like storage and configurations are dynamically managed.

---

## Kubernetes Architecture  

1. **Control Plane** (Master): Manages the cluster.  
    - **API Server**: Handles communication with `kubectl` commands.  
    - **etcd**: Stores the cluster's state.  
    - **Scheduler**: Decides which Pod runs on which machine.  
    - **Controller Manager**: Ensures the cluster matches the desired configuration.

2. **Worker Nodes**: Where applications (Pods) are actually run.  
    - **kubelet**: Makes sure the Pods are healthy and running.  
    - **kube-proxy**: Routes network traffic to the correct Pod.  

3. **Add-Ons**: Includes tools for logging, monitoring, and networking.  

---

## Getting Started with Kubernetes  

### Prerequisites  
1. **Basic understanding of containers** (e.g., Docker).  
2. A Linux/macOS/Windows machine with the following installed:  
   - **kubectl** (Kubernetes CLI tool).  
   - A terminal to execute commands.  

---

## Step-by-Step Deployment Example  

Let’s deploy a **Sentiment Analysis Application** to Kubernetes using Linode’s **Kubernetes Engine (LKE)**.  

---

### Step 1: Create a Kubernetes Cluster on Linode  

1. **Log in to the Linode Dashboard**.  
2. Go to the **Kubernetes Section**.  
3. Click on **Create Cluster**.  
4. Select:  
   - **Region**: Choose a location (e.g., Dallas, Singapore).  
   - **Plan**: Pick a pricing tier (based on CPU/RAM needs).  
5. Customize additional settings as needed.  
6. Once the cluster is running, download the **Kubeconfig file**.  

Save this file as [`kube.yml`](./kube.yml) locally on your computer.  

---

### Step 2: Configure Kubernetes CLI (`kubectl`)  

1. Export the `kube.yml` file for Kubernetes to use:
   ```bash
   export KUBECONFIG=kube.yml
   ```

2. Install `kubectl` (if not already installed):  
   **Linux/Mac**:
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   ```

   **Windows**: [Download the binary](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/).

3. Verify `kubectl` is connected to your cluster:
   ```bash
   kubectl get nodes
   ```
   This will show the list of nodes in your Linode cluster.

---

### Step 3: Deploy the Sentiment Analysis App  

#### [Create a Deployment File (`deployment.yml`)](./deployment.yml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
  labels:
    app: analysis
spec:
  replicas: 10
  selector:
    matchLabels:
      app: analysis
  template:
    metadata:
      labels:
        app: analysis
    spec:
      containers:
      - name: analysis
        image: mementomori11723/sentimental-analysis
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
```

- **Replicas**: Ensures 10 copies of the app are running for high availability.  
- **Image**: The Docker image for sentiment analysis.  
- **Port**: Exposes the app on `8080`.  

Deploy it:
```bash
kubectl apply -f deployment.yml
```

Verify the Pods:
```bash
kubectl get pods
```

---

### Step 4: Expose the Application with a Service  

#### [Create a Service File (`service.yml`)](./service.yml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-service
  annotations:
    service.beta.kubernetes.io/linode-loadbalancer-throttle: "4"
  labels:
    app: analysis-test
spec:
  type: LoadBalancer
  selector:
    app: analysis
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
  sessionAffinity: None
```

- **Type**: `LoadBalancer` creates an external IP for your app.  
- **Selector**: Targets Pods with the `app: analysis` label.  
- **Ports**: Maps external traffic (port `80`) to the container's port (`8080`).  

Deploy it:
```bash
kubectl apply -f service.yml
```

Get the external IP:
```bash
kubectl get services
```

Use the `EXTERNAL-IP` in your browser or with `curl` to access the app.

---

## Additional Concepts  

### What is a Pod?  
A Pod is the smallest deployable unit in Kubernetes. It wraps one or more containers (e.g., Docker).  

### What is a Service?  
A Service exposes your application to the outside world and balances traffic across Pods.  

### What is a Deployment?  
A Deployment manages the number of Pods running and ensures they stay healthy.  

---

## Cleaning Up  

To delete the application:  
```bash
kubectl delete -f deployment.yml
kubectl delete -f service.yml
```

To delete the cluster:  
1. Log in to the Linode Dashboard.  
2. Delete the Kubernetes cluster from the **Kubernetes Section**.

---

## Final Notes  

This guide walks you through creating and deploying a basic application with Kubernetes. For more advanced setups like **monitoring**, **logging**, or **CI/CD integration**, explore tools like **Prometheus**, **Grafana**, and **ArgoCD**.  

If you have any questions or need further clarification, feel free to ask!
