# Kubernetes Cluster Setup with kubeadm

## What is kubeadm?
<p align="center">
  <img width="150" height="150" src="img/kubeadm.png">
</p>

[Official Documentation](https://kubernetes.io/docs/reference/setup-tools/kubeadm/#:~:text=Kubeadm%20is%20a%20tool%20built,bootstrapping%2C%20not%20about%20provisioning%20machines.)

Kubeadm is a tool built to provide kubeadm init and kubeadm join as best-practice “fast paths” for creating Kubernetes clusters. kubeadm performs the actions necessary to get a minimum viable cluster up and running.

## Prerequisites:

* SSH key pair on your local machine. 

* Ansible installed on your local machine.

* 3 Ubuntu 18.04 machines, 1 master, 2 workers. 

* Make sure your local machine can ssh to all Ubuntu machines. 

* Knowledge of running ansible playbooks.

* Knowledge of running containers. 

* Modify [hosts](hosts) file. 

## Setup 

### Step 1: Create Workspace

* Clone the repository, and change directory to 'kubernetes-cluster'

### Step 2: Change 'hosts' file

* Open [hosts](hosts) file. Change the following:

master ansible_host= ENTER IP address of your master-node

worker1	ansible_host= ENTER IP address of your worker1

worker2 ansible_host= ENTER IP address of your worker2

- save and quit

### Step 3: Run 'initial.yml' playbook

* Run [initial.yml](initial.yml) playbook
```
ansible-playbook -i hosts ~/kubernetes-cluster/initial.yml 
```

#### The above playbook: 
1. Creates a user 'ubuntu'.
2. Configures the sudoers file to allow the ubuntu user to run sudo commands without a password prompt.
3. Adds the public key of your local machine to the remote ubuntu user's authorized key list. This will allow you to SSH into each server as the ubuntu user.

### Step 4: Run 'kube-dependencies.yml' playbook

* Run [kube-dependenices.yml](kube-dependenices.yml) playbook
```
ansible-playbook -i hosts ~/kubernetes-cluster/kube-dependencies.yml
```
#### The above playbook:
1. Installs Docker on master and worker nodes.
2. Installs apt-transport-https, allowing you to add external HTTPS sources to your APT sources list.
3. Adds the Kubernetes APT repository's apt-key for key verification.
4. Adds the Kubernetes APT repository to your remote servers' APT sources list.
5. Installs kubelet and kubeadm.
6. Installs kubectl on master node.

### Step 5: Setting up master node

* Run [master.yml](master.yml) playbook
```
ansible-playbook -i hosts ~/kubernetes-cluster/master.yml
```
#### The above playbook:
1. Initializes a cluster by running 'kubeadm init'.
2. Creates a '.kube' directory under /home/ubuntu. 
3. Copies the /etc/kubernetes/admin.conf file to 'ubuntu' user's home directory. 
4. Runs 'kubectl apply' to install Flannel. 

### Step 6: Check the status of 'master' node 
```
ssh to your master node and run: kubectl get nodes 
```
you should see: 

NAME=master , STATUS=ready, ROLES=master

### Step 7: Setting up worker nodes

* Switch back to your local host (workspace)
* Run 'workers.ym' playbook
```
ansible-playbook -i hosts ~/kubernetes-cluster/workers.yml
```
#### The above playbook: 
1. On master node, gets the join command that needs to be run on worker nodes. 
2. Runs the join command on all worker nodes. 

### Step 8: Check the status of 'worker' nodes

1. SSH to your 'master' node and run: 
```
kubectl get nodes
``` 
This time you should see your 'master' node, and the 'worker' nodes

_**<p style='color:green'>If all your nodes have the value 'Ready' for STATUS, it means they are part of the cluster and raed to run workloads.</p>**_

