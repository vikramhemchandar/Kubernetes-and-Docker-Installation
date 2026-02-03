# Kubernetes and Docker Installation

## What is Kubernetes?
Kubernetes is an open-source system developed by Google, now overseen by the CNCF. It simplifies container orchestration. Thus, developers can build apps without dealing with infrastructure complexities. This system provides container management and scheduling. Users can dictate application behaviors within a cluster. Kubernetes abstracts the underlying infrastructure, streamlining container tasks and application deployment.

## Benefits of Kubernetes
- Kubernetes is good at managing containers across diverse nodes
- Kubernetes is compatible with many public and private cloud infrastructures
- Kubernetes allows you to automatically scale your application when it needs more resources to run successfully
- Kubernetes self-healing feature automatically restarts failing containers and replaces unhealthy containers

## Pre-requisites
To install Kubernetes on a Ubuntu 22.04 Long-Term-Service (LTS) machine, make sure it meets the following requirements:
- 2 CPUs
- At least 2GB of RAM
- At least 8 GB of Disk Space (recommended) 
- A reliable internet connection
If the machine meets the above requirements, you are ready to follow this tutorial. Let’s start with the step-by-step process on how to install Kubernetes on Ubuntu.

> [!NOTE]
> It is recommended to install Kubernetes on Ubuntu 22.04 LTS (Jammy Jellyfish). Using the official Kubernetes documentation for installing and managing Kubernetes on Ubuntu 22.04 provides significant advantages in terms of reliability, security, and accuracy, particularly by leveraging kubeadm for cluster bootstrapping. It ensures that the installation follows best practices, such as disabling swap, using appropriate container runtimes (like containerd), and setting up necessary CNI plugins.

## Installing Kubernetes on Ubuntu
Installing Kubernetes is not a straightforward task. To create a flexible and high-performing cluster, it will need to follow several steps other than installing Kubernetes components. Also, we have to configure machines in a way they can communicate with each other

Overall, installing Kubernetes on Ubuntu involves steps such as:
- [1. Update & Upgrade][step1]
- [2. Disabling swap][step2]
- [3. Setting up hostnames][step3]
- [4. Setting up the IPV4 bridge on all nodes][step4]
- [5. Installing Kubernetes components -kubelet, kubeadm & kubectl on all nodes][step5]
- [6. Installing Docker or a suitable containerization tool][step6]
- [7. Initializing the Kubernetes cluster][step7]
- [8. Configuring Kubectl and Calico][step8]
- [9. Adding worker nodes][step9]
- [10. Verfiy and test][step10]

[step1]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-0---update--upgrade
[step2]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-1---disable-swap
[step3]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-3---update-the-etchosts-file-for-hostname-resolution
[step4]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-4---set-up-the-ipv4-bridge-on-all-nodes
[step5]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-5---install-kubelet-kubeadm-and-kubectl-on-each-node
[step6]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-6---install-docker
[step7]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-7---initialize-the-kubernetes-cluster-on-the-master-node
[step8]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-8---configure-kubectl-and-calico
[step9]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-9---add-worker-nodes-to-the-cluster
[step10]:https://github.com/vikramhemchandar/Kubernetes-and-Docker-Installation?tab=readme-ov-file#step-10---verify-the-cluster-and-test

> [!NOTE]
> Create Two EC2 instances. One for **Master Node** and another for **Worker Node**

### Step 0 - Update & Upgrade
On any Linux-based system, including EC2 instances, the first step is to update and upgrade the system package index
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

```
sudo nano /etc/hosts
```
At the bottom of the file, add the master and worker nodes IP addresses (as shown below)

Master node <br>
<img width="468" height="373" alt="image" src="https://github.com/user-attachments/assets/469d4378-2a6e-49c8-9e07-310d7378669b" />

Worker node<br>
<img width="468" height="405" alt="image" src="https://github.com/user-attachments/assets/5b86d00f-25b9-468c-be4e-626cef20c6f0" />

Now ping test between Master and Worker nodes from the both the nodes respectively
```
ping -c 4 k8s-master
ping -c 4 k8s-worker
```
Master node <br>
<img width="468" height="283" alt="image" src="https://github.com/user-attachments/assets/52867fa6-c610-4d9a-a7af-9faeb72cf875" />

Worker node <br>
<img width="468" height="294" alt="image" src="https://github.com/user-attachments/assets/6667ac7b-199f-4d4f-9533-b493acf28ae4" />

### Step 4 - Set up the IPv4 bridge on all nodes
In this step we're going to configure ipv4 bridge on all the nodes and this involves loading the kernel modules. The below command creates a configuration file for Kubernetes and adds two entries – overlay and the br_netfilter
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

Create another configuration file (the critical kernel parameters) to add specific network entries and persists these changes across reboots by running
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```
Master node <br>
<img width="1634" height="932" alt="image" src="https://github.com/user-attachments/assets/36eeee10-35ad-4e41-9186-ea2e47123a6d" /><br>
<img width="1636" height="1756" alt="image" src="https://github.com/user-attachments/assets/3e6645c0-8e25-4d76-9eaf-059936e80904" /><br>

Worker node <br>
<img width="1630" height="812" alt="image" src="https://github.com/user-attachments/assets/acf2c3cc-26a9-42d1-8336-8262df65aa49" /><br>
<img width="1664" height="1756" alt="image" src="https://github.com/user-attachments/assets/8322bac2-979d-4f42-b79f-d8749f2953da" /><br>

### Step 5 - Install kubelet, kubeadm, and kubectl on each node
This step is to install kubelet, kubeadm, and kubectl on each node to create a Kubernetes cluster. These k8s packages play an important role in managing a Kubernetes cluster

_Kubelet_ is the node agent that runs on every node and is responsible for ensuring containers are running in a Pod as specified by the Pod's specifications. (Pods are the smallest deployable units in a Kubernetes cluster).

_Kubeadm_ is used to bootstrap a Kubernetes cluster, including setting up the master node and helping worker nodes join the cluster.

_Kubectl_ is a CLI tool for Kubernetes to run commands to perform various actions such as deploying applications, inspecting resources, and managing cluster operations directly from the terminal.

Before installing these, we must update the package index
```
sudo apt update
```
To install the requisite packages (on both Master and worker nodes)
```
sudo apt install -y ca-certificates curl gpg
```
Fetch the public key from Google and store it in the folder we created in the previous step. This key is important to verify that the Kubernetes packages we download are genuine and haven't been tampered with
```
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
Add the gpg key and create the Kubernetes repository and add it to the system
```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list	
Now again update the package indes

sudo apt update
```
_Note:_ after update, we should see Kubernetes repositories in the packages

<img width="2648" height="2692" alt="image" src="https://github.com/user-attachments/assets/93e80a8e-5132-48b6-8bc7-276481522589" /><br>

Now we are ready to install kubelet, kubeadm, and kubectl (with version and without version)
```
sudo apt install -y kubelet=1.26.5-00 kubeadm=1.26.5-00 kubectl=1.26.5-00 
                          or
sudo apt install -y kubelet kubeadm kubectl
```
<img width="2796" height="2100" alt="image" src="https://github.com/user-attachments/assets/947c7f25-09d2-4ffd-b2bf-91f33d2d65f3" /><br>

Lock the version
```
sudo apt-mark hold kubelet kubeadm kubectl
```
_apt-mark hold_ is used to prevent the automatic installation, upgrade or removal of specific packages by package management tools like apt or apt-get. This is particularly useful when you need to maintain a specific version of a package or prevent accidental system changes
<img width="468" height="67" alt="image" src="https://github.com/user-attachments/assets/5a7b914c-7ff2-4ce3-87af-b5cc1e05f5d8" /> <br>

Worker node <br>
<img width="2976" height="1566" alt="image" src="https://github.com/user-attachments/assets/bd1412bb-0923-47d9-bbe3-c02e330da1b6" /><br>
<img width="2660" height="2102" alt="image" src="https://github.com/user-attachments/assets/fced6a6e-452e-49e5-b331-e913f95d73ae" /><br>
<img width="409" height="63" alt="image" src="https://github.com/user-attachments/assets/4845773b-27ce-4fd7-b393-b1f40ec7a84b" /><br>

### Step 6 - Install Docker
The Docker platform allows you to create, distribute, and run applications within containers. These containers offer a lightweight and portable environment, ensuring consistent performance across various setups. Docker serves as the container runtime, playing a vital role in Kubernetes by facilitating the efficient management and deployment of containerized applications. Install docker with the command:
```
sudo apt install docker.io
```
_Note –_ docker.io installs Full Docker Engine that includes Docker CLI, containerd, daemon, networking and image management
<img width="2940" height="3146" alt="image" src="https://github.com/user-attachments/assets/5217945b-ad35-4c7b-9d11-9c327a49b42a" /><br>

Next, configure containerd on all nodes to ensure its compatibility with Kubernetes. First, create a folder for the configuration file with the sudo mkdir /etc/containerd command.
```
sudo mkdir /etc/containerd
```

Then, create a default configuration file for containerd and save it as config.toml using the sudo sh -c "containerd config default > /etc/containerd/config.toml" command.
```
sudo sh -c "containerd config default > /etc/containerd/config.toml"
```

After running these commands, we need to modify the config.toml file to locate the entry that sets "SystemdCgroup" to false and changes its value to true. This is important because Kubernetes requires all its components, and the container runtime uses systemd for cgroups.
```
sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml
```

Next, restart containerd and kubelet services to apply the changes you made on all nodes.
```
sudo systemctl restart containerd.service
sudo systemctl restart kubelet.service
```

And enable kubelet service 
```
sudo systemctl enable kubelet.service
```
**_Repeat the same commands in worker node_**

### Step 7 - Initialize the Kubernetes cluster on the master node
When we initialize a Kubernetes control plane using kubeadm, several components are deployed to manage and orchestrate the cluster. Some examples of these components are kube-apiserver, kube-controller-manager, kube-scheduler, etcd, kube-proxy. We need to download the images of these components by running the following command.
```
sudo kubeadm config images pull
```
Initialize Kubernetes Cluster with Kubeadm (master node) With all the prerequisites in place, initialize the Kubernetes cluster on the master node using the following Kubeadm command. 
```
sudo kubeadm init
```
Note: Make sure to copy the kubeadm join after kubeadm init (as shown in the screenshot), this will require to add to the worker node (Step 9)
<img width="2274" height="204" alt="image" src="https://github.com/user-attachments/assets/5cdc2260-dd96-4332-bd8b-a5209faca8ff" /><br>

To manage the cluster, we should configure kubectl on the master node. Create the .kube directory in the home folder and copy the cluster's admin configuration to the personal .kube directory. Next, change the ownership of the copied configuration file to give the user the permission to use the configuration file to interact with the cluster. Run the following commands on the master node:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Next, use kubectl commands to check the cluster and node status:
```
kubectl get nodes
```
Master node<br>
<img width="2974" height="2974" alt="image" src="https://github.com/user-attachments/assets/af98479d-9f58-4a08-a8e7-4bf184d91b0a" /><br>

### Step 8 - Configure kubectl and Calico
Install Kubernetes Network Plugin (master node) To enable communication between pods in the cluster, you need a network plugin. Install the Calico network plugin with the following command from the master node:
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
Only on Master node<br>
<img width="2492" height="1188" alt="image" src="https://github.com/user-attachments/assets/c5a7bc63-4df2-429c-ab94-27119f8eddea" /><br>

### Step 9 - Add Worker nodes to the cluster
The above command gives IP address and a tocketn, copy the the command (as shown below screenshot) and add it in the worker nodes.<br>
<img width="2274" height="204" alt="image" src="https://github.com/user-attachments/assets/e9bb4ba5-f114-42ac-8f8a-488f5bad738b" /><br>
```
kubeadm join 172.31.37.105:6443 --token i2upnd.fp6njjdc1nk2zxm5 \ --discovery-token-ca-cert-hash sha256:4db80163a7040ad51bb144a8088c271428931a9be2e69edf10cb2db62b5c7c97
```
On worker node response for kubeadm join<br>
<img width="2982" height="292" alt="image" src="https://github.com/user-attachments/assets/6d68873c-7cb0-4717-bdae-de457860d302" />

### Step 10 - Verify the cluster and test
Verify the cluster and test (master node) Finally, we want to verify whether our cluster is successfully created:
```
kubectl get pods -n kube-system
kubectl get nodes
```
<img width="1826" height="672" alt="image" src="https://github.com/user-attachments/assets/b19a2c4c-e74e-4b73-bd96-763c73550556" />










