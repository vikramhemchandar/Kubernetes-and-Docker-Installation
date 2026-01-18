# Kubernetes and Docker Installation

## What is Kubernetes?
Kubernetes is an open-source system developed by Google, now overseen by the CNCF. It simplifies container orchestration. Thus, developers can build apps without dealing with infrastructure complexities. This system provides container management and scheduling. Users can dictate application behaviors within a cluster. Kubernetes abstracts the underlying infrastructure, streamlining container tasks and application deployment.

## Benefits of Kubernetes
- Kubernetes is good at managing containers across diverse nodes
- Kubernetes is compatible with many public and private cloud infrastructures
- Kubernetes allows you to automatically scale your application when it needs more resources to run successfully
- Kubernetes self-healing feature automatically restarts failing containers and replaces unhealthy containers

## Pre-requisites
To install Kubernetes on a Ubuntu machine, make sure it meets the following requirements:
- 2 CPUs
- At least 2GB of RAM
- At least 8 GB of Disk Space (recommended) 
- A reliable internet connection
If the machine meets the above requirements, you are ready to follow this tutorial. Let’s start with the step-by-step process on how to install Kubernetes on Ubuntu.

> [!NOTE]
> It is recommended to install Kubernetes on Ubuntu 22.04 (Jammy Jellyfish) Long-Term-Service (LTS). Using the official Kubernetes documentation for installing and managing Kubernetes on Ubuntu 22.04 provides significant advantages in terms of reliability, security, and accuracy, particularly by leveraging kubeadm for cluster bootstrapping. It ensures that the installation follows best practices, such as disabling swap, using appropriate container runtimes (like containerd), and setting up necessary CNI plugins.

## Installing Kubernetes on Ubuntu
Installing Kubernetes is not a straightforward task. To create a flexible and high-performing cluster, it will need to follow several steps other than installing Kubernetes components. Also, we have to configure machines in a way they can communicate with each other

Overall, installing Kubernetes on Ubuntu involves steps such as:

- Disabling swap
- Setting up hostnames
- Setting up the IPV4 bridge on all nodes
- Installing Kubernetes components on all nodes
- Installing Docker or a suitable containerization tool
- Initializing the Kubernetes cluster
- Configuring Kubectl and Calico
- Adding worker nodes

> [!NOTE]
> Create Two EC2 instances. One for **Master Node** and another for **Worker Node**

### Step 0 - Update & Upgrade
Any EC2 instance, for that matter any linux based system, first step is to update and upgrade the package index
```
sudo apt update && apt upgrade -y
```
### Step 1 - Disable Swap
We know about swap space on hard drives, which OS systems try to use as if it were RAM. Operating systems try to move less frequently accessed data to the swap space to free up RAM for more immediate tasks. However, accessing data in swap is much slower than accessing data in RAM because hard drives are slower than RAM.

Kubernetes schedules work based on the understanding of available resources. If workloads start using swap, it can become difficult for Kubernetes to make accurate scheduling decisions. Therefore, it’s recommended to disable swap before installing Kubernetes.

We can do it with the following commands:  
The _sudo swapoff -a_ command temporarily disables swap on your system. Then, the _sudo sed -i '/ swap / s/^/#/' /etc/fstab_ command modifies a configuration file to keep the swap remains off even after a system reboot.
```
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
sudo swapon –show
```
Master Node <br>
<img width="468" height="84" alt="image" src="https://github.com/user-attachments/assets/0a3527fc-da20-496e-a779-2df90574e4e7" />

Worker Node <br>
<img width="468" height="99" alt="image" src="https://github.com/user-attachments/assets/bba183c6-52e0-4be7-8fbd-7f883d8db640" />

### Step 2 - Set up hostnames
We are going to configure the host names on both nodes. When working with Kubernetes, it's always recommended to assign host names for nodes so that Kubernetes can use these host names to identify the nodes within a cluster.
```
sudo hostnamectl set-hostname ‘k8s-master’
exec bash
hostname
```
Master node <br>
<img width="468" height="92" alt="image" src="https://github.com/user-attachments/assets/53334827-e9c6-471c-bb82-98082159132f" />

Worker node <br>
<img width="468" height="79" alt="image" src="https://github.com/user-attachments/assets/6d69e8f7-82c5-4f18-8a2d-cb8814b07878" />

### Step 3 - Update the /etc/hosts file for hostname resolution
This step is to update the /etc/hosts file to enable host name resolution. Once we've set up the host names we need to go further and map the host names to their respective IP addresses as well for host name resolution.

_Note:_ Make sure to enable All IMCP – IPv4 in Security groups for both the EC2 instances
<img width="468" height="138" alt="image" src="https://github.com/user-attachments/assets/468e8410-d697-4291-b04d-8a93c939d150" />
