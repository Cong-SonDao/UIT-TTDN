#  Project 01: Kubernetes Fundamentals & On-Prem Deployment

##  Learning Objectives

- Understand the core concepts and architecture of Kubernetes.
- Learn how to set up a Kubernetes cluster in an on-premises environment.
- Gain hands-on experience with cluster initialization, node joining, and network setup.
- Practice basic cluster management using `kubectl`.

---

##  Prerequisites

- Basic knowledge of Linux command-line (Ubuntu recommended).
- Understanding of virtualization (used Proxmox for VM setup).
- 3 VMs with Ubuntu 24.04+ installed on Proxmox (1 master-node, 2 worker-node)
- Internet connection for downloading packages.

---

##  1. Kubernetes Fundamentals

###  What is Kubernetes?
Kubernetes is an open-source container orchestration platform designed to automate the deployment, scaling, and management of containerized applications.

###  Kubernetes Architecture
![Image](https://github.com/user-attachments/assets/97fb6121-b047-445c-acc2-6225ca57068d)

###  Core Components
- **Pod**:  Pod is the smallest and most basic deployable unit. It represents a single instance of a running process in your cluster.
- **Node**: A Node is a physical or virtual machine that runs your application workloads.
- **Cluster**: A Cluster is the entire system that runs and manages your containerized applications.
- **Deployment**: Describes the desired state for pods and manages updates.
- **Service**: Exposes pods to internal or external traffic.

###  Basic kubectl Commands
```bash
# Kubernetes Cluster command 
kubectl get nodes -o wide                 # List all nodes in the Cluster and show IPS

# Kubernetes Pod command
kubectl get pods                          # List all pod
kubectl get pods -o wide                  # Show detailed information about pod 
kubectl get pods -l <label>=<value>       # Show pods who have a specific label
kubectl describe pod                      # Show a pod's detail
kubectl logs <name_pod>                   # View logs of pod
kubectl delete pod <name_pod>             # Delete a pod

# Kubernetes Deployment commands
kubectl create deployment <name> --image=<image>           # Create a deployment.
kubectl get deployments                                    # List all deployments.
kubectl describe deployment <name>                         # Show deployment details of one deployment.
kubectl scale deployment <name> --replicas=<number>        # Increase or decrease a deployment.
kubectl rollout restart deployment/<name>                  # Restart a deployment.
kubectl rollout status deployment/<name>                   # View the status of a deployment.
kubectl create deployment <name> --image=<image> -o yaml   # Create a deployment without applying it and print the YAML file.

# Kubernetes Service commands
kubectl get services                       # List all services.
kubectl describe service <name>            # Show service details.
kubectl expose pod <name>                  # Expose a pod to a service.
kubectl delete service <name>              # Delete a service.
kubectl port-forward pod <local-port>:<remote-port>        # Forward a local port to a pod.

# Kubernetes ConfigMap & Secret commands
kubectl create configmap <name> --from-literal=<key>=<value>   # Create a ConfigMap.
kubectl create secret generic <name> --from-literal=<key>=<value> # Create a Secret.
kubectl get configmaps                     # List all ConfigMaps.
kubectl get secrets                        # List all Secrets.
kubectl describe configmap <name>          # Show ConfigMap details.

# Kubernetes Namespace commands
kubectl get namespaces                     # List all namespaces.
kubectl create namespace <name>            # Create a namespace.
kubectl delete namespace <name>            # Delete a namespace.
kubectl config set-context --current --namespace=<name>      # Switch to a different namespace.

# Kubernetes Resource commands
kubectl apply -f <file>                     # Apply a resource config file.
kubectl edit <type> <name>                  # Edit a resource file inside your terminal.
kubectl delete -f <file>                    # Delete a resource from a config file.
kubectl get <type>                          # List all resources of a specific type.
kubectl describe <type> <name>              # Show details about a specific resource.

# Kubernetes Statistics & Event commands
kubectl top nodes                           # Display node resource usage.
kubectl top pods                            # Display pod resource usage.
kubectl get events                          # Show all cluster events.

# Kubernetes Permissions
kubectl get roles                           # List all roles.
```

##  2. On-Prem Kubernetes Deployment

### 2.1. VM Preparation
- Minimum of 3 instances of Ubuntu 24.04. 
- Minimum of 2 GB RAM and 2 vCPUs.
- Minimum 20GB of free hard drive space.

### 2.2. Kubernetes Lab Setup
| Node | Name | IP Address |
|------|------|------------|
| Node 1 | master-node1 | `10.0.1.21` | 
| Node 2 | worker-node1 | `10.0.1.22` |
| Node 3 | worker-node2 | `10.0.1.23` |

### 2.3. Install Kubernetes On Ubuntu 24.04 Step-by-step

#### Step 1: Configure hostnames (all nodes)
```bash
# Start by updating the /etc/hosts files and specifying the IP addresses and hostnames for each node in the cluster. This will allow seamless communication between the master and worker nodes.

# On master-node
sudo nano /etc/hosts
# Add the following lines:
10.0.1.21 master-node1
10.0.1.22 worker-node1
10.0.1.23 worker-node2          # Save and Exit 

sudo nano /etc/hostname
# Add the following lines:
master-node1                    # Save and Exit

# On worker-node1
sudo nano /etc/hosts
# Add the following lines:
10.0.1.21 master-node1
10.0.1.22 worker-node1
10.0.1.23 worker-node2          # Save and Exit 

sudo nano /etc/hostname
# Add the following lines:
worker-node1                    # Save and Exit

# On worker-node2
sudo nano /etc/hosts
# Add the following lines:
10.0.1.21 master-node1
10.0.1.22 worker-node1
10.0.1.23 worker-node2          # Save and Exit 

sudo nano /etc/hostname
# Add the following lines:
worker-node2                    # Save and Exit

# To verify the hostname in each node, run the command:
hostname
# To check network on Cluster
ping  -c 3 worker-node1
ping  -c 3 master-node1
ping  -c 3 worker-node2
```

#### Step 2: Disable swap space (all nodes)
```bash
# To disable swap on the nodes, run the command:
sudo swapoff -a
# To check whether the swap space has been disabled, run the command:
swapon --show
```

#### Step 3: Load Containerd modules (all nodes)
```bash
# Run the following commands:
sudo modprobe overlay
sudo modprobe br_netfilter
# Create a configuration file as shown and specify the modules to load them permanently.
sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF
```

#### Step 4: Configure Kubernetes IPv4 networking (all nodes)
```bash
# Create a Kubernetes configuration file in the /etc/sysctl.d/ directory:
sudo nano /etc/sysctl.d/k8s.conf

# Add the following lines:
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward   = 1

# Save and exit. Then apply the settings by running the following command:
sudo sysctl --system
```

#### Step 5: Install Docker (all nodes)
```bash
# To install it, run the following commands on all nodes
sudo apt update
sudo apt install docker.io -y
sudo systemctl status docker            # To verify if Docker is running, run:
sudo systemctl enable docker

# The installation of Docker also comes with containerd, a lightweight container runtime that streamlines the running and management of containers.
sudo mkdir /etc/containerd
# Next, create a default configuration file for containerd.
sudo sh -c "containerd config default > /etc/containerd/config.toml"
# Update the SystemdCgroup directive by setting it to true as shown.
sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml
# Next, restart the containerd service to apply the changes made.
sudo systemctl restart containerd.service
sudo systemctl status containerd.service        
```

#### Step 6: Install Kubernetes components (all nodes)
```bash
# The next step is to install Kubernetes. Currently, Kubernetes v1.31 is the current version. Be sure to install the prerequisite packages on all nodes.
sudo apt-get install curl ca-certificates apt-transport-https  -y
# Add the Kubernetes GPG signing key.
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# Thereafter, add the official Kubernetes repository to your system.
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
# Once added, update the sources list for the system to recognize the newly added repository.
sudo apt update
# To install these salient Kubernetes components, run the following command:
$ sudo apt install kubelet kubeadm kubectl -y
```

#### Step 7: Initialize Kubernetes cluster (on master node)
```bash
# Run following Kubeadm command from the master node only to initialize the Kubernetes Cluster
sudo kubeadm init --control-plane-endpoint=master-node1 # Replace the hostname with your own.
# Run following command
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Step 8: Join Worker nodes to the cluster (on worker nodes)
```bash
# To do this, run the kubeadm join command generated in Step 7 when initializing the cluster.
sudo kubeadm join master-node1:6443 --token zowdxy.xb0tdph2s822rupq --discovery-token-ca-cert-hash sha256:4239c5dcb961240a572262064b93af227045e9edd180218a51f6125023380a1f

# If you missed the token before you can recreate it by following command:
kubeadm token create --print-join-command  # Use this on master-node

# Now check the nodes in the cluster once again on master node
kubectl get nodes
```

#### Step 9: Install Calico network add-on plugin (on master node)
```bash
# To install calico network add-on plugin, run following command below
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/calico.yaml

# After the successfull installation of calico, nodes status will change to ready in few minutes
kubectl get pods -n kube-system
```

#### Step 10: Testing Kubernetes Cluster
```bash
kubectl create namespace demo-namespace
```

### References
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Documentation](https://docs.docker.com/)
