# Kind-Cluster-Setup
This guide explains how to set up Kind (Kubernetes in Docker) on an Ubuntu AWS EC2 instance, with steps for both single-node and multi-node Kubernetes clusters.

## Prerequisites
### Ubuntu EC2 Instance:

  - Instance type: t2.medium or larger (for multi-node clusters, use t2.large or above).
  - Docker and Kind require at least 4 GB RAM.
  - Ubuntu version: 20.04 or later.
### Access to the Instance:
  - SSH access to the AWS EC2 instance.

### Step 1: Update and Install Dependencies
  1) Update the system:

```bash
sudo apt update && sudo apt upgrade -y
```
  2) Install dependencies:

```bash

sudo apt install -y curl apt-transport-https ca-certificates software-properties-common
```

### Step 2: Install Required Tools
#### 2.1 Install Docker
1) Install Docker:

```bash

sudo apt update
sudo apt install docker.io -y
```
2) Start and enable Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```
3) Add your user to the Docker group:

```bash

sudo usermod -aG docker $USER
```
4) Verify Docker installation:

```bash
docker --version
```
#### 2.2 Install Kind
1) Download the Kind binary:

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.21.0/kind-linux-amd64
```
2) Make it executable and move it to your PATH:

```bash
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
3) Verify Kind installation:

```bash
kind --version
```
#### 2.3 Install kubectl
1) Download kubectl:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
2) Make it executable and move it to your PATH:

```bash

chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```
3) Verify kubectl installation:

```bash

kubectl version --client
```
### Step 3: Setup a Kind Cluster
#### Single-Node Cluster
1) Create the single-node cluster:

```bash
kind create cluster --name single-node-cluster
```
2) Verify the cluster:

  - Check cluster info:
```bash
kubectl cluster-info --context kind-single-node-cluster
```
  - List nodes:
```bash
kubectl get nodes
```

#### Multi-Node Cluster
1) Create a cluster configuration file: Save the following YAML as multi-node-config.yaml:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```
2) Create the multi-node cluster:

```bash
kind create cluster --name multi-node-cluster --config multi-node-config.yaml
```
3) Verify the cluster:

  - Check cluster info:
```bash
kubectl cluster-info --context kind-multi-node-cluster
```
  - List all nodes:
```bash

kubectl get nodes
```
### Step 4: Delete Clusters
To delete clusters when no longer needed:

  - Delete the single-node cluster:

```bash
kind delete cluster --name single-node-cluster
```
  - Delete the multi-node cluster:

```bash
kind delete cluster --name multi-node-cluster
```
### Step 5: Using the Kind Cluster
#### Deploy a sample application:

  - Create a simple Nginx Deployment:
```bash

kubectl create deployment nginx --image=nginx
```
  - Expose it as a Service:
```bash
kubectl expose deployment nginx --type=NodePort --port=80
```
  - Check the service:
```bash
kubectl get svc
```
#### Access the application:

  - Get the NodePort assigned to the Nginx service:
```bash
kubectl get svc nginx
```
  - Access the application using the public IP of your EC2 instance and the NodePort:
```php
http://<EC2-Public-IP>:<NodePort>
```

### Notes
  - Persistent Clusters: Kind clusters are ephemeral. Restarting the Docker service may cause cluster loss.
  - EC2 Security Groups: Ensure the necessary ports are open in your EC2 Security Group to access services via NodePort.

### Troubleshooting
1) Docker issues:

    - Restart Docker if you encounter errors:
```bash
sudo systemctl restart docker
```
2) Cluster creation issues:

    - Check Docker logs:
```bash
docker logs <container-id>
```
### References
  - Kind Documentation
  - Kubernetes Official Documentation
