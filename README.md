# Kubernetes  

## Project Overview  

This project sets up a lightweight Kubernetes cluster using Vagrant and VirtualBox. It provisions:  

- **One master node** (`k8s-master`) that manages the cluster  
- **Two worker nodes** (`k8s-worker-1`, `k8s-worker-2`) that run containerized workloads  
- **An embedded ETCD cluster** running on the master node for storing Kubernetes cluster data  

All virtual machines have the necessary dependencies installed, including:  
- Docker container runtime  
- Kubernetes components (`kubeadm`, `kubectl`, `kubelet`)  
- Network configurations and optimizations  
- ETCD, which is configured using the following configuration file:  
  ```sh
  /etc/kubernetes/pki/etcd/ca.crt
  /etc/kubernetes/pki/etcd/server.crt
  /etc/kubernetes/pki/etcd/server.key
  ```

The master node is initialized with `kubeadm`, and Flannel is deployed as the pod network. Worker nodes are automatically configured to join the cluster.  

## Getting Started  

To set up this Kubernetes project locally, first, clone the repository:  
```sh
git clone https://github.com/DanielDimitrov1/k8s-vm-environment-lab.git
```  

Navigate to the project directory and start the virtual machines using Vagrant:  
```sh
cd k8s-vm-environment-lab
vagrant up
```  

Once the setup is complete, follow the steps below to configure the cluster.  

## Setup Instructions  

### Joining Worker Nodes to the Cluster  

1. **Access the Master Node**  
   Connect to the master node using SSH.  
   ```sh
   vagrant ssh master
   ```  

2. **Generate the Join Command**  
   Create a token that worker nodes will use to join the cluster.  
   ```sh
   sudo kubeadm token create --print-join-command
   ```  
   Copy the output.  

3. **Join Worker Nodes**  
   Use the generated token to connect each worker node to the cluster.  
   ```sh
   vagrant ssh worker-1
   sudo <paste_command_from_master_node>
   ```  

4. **Verify Kubelet Status on the Master Node**  
   Ensure the Kubelet service is running on the master node.  
   ```sh
   sudo systemctl status kubelet
   ```  
   If it's inactive, start it:  
   ```sh
   sudo systemctl start kubelet
   ```  

### Configure kubectl on the Master Node  

5. **Check if the `.kube` Directory Exists**  
   Verify if the kubeconfig file is available for `kubectl` to work.  
   ```sh
   ls $HOME/.kube/config
   ```  
   If the directory is missing, create it:  
   ```sh
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```  

6. **Verify Node Status**  
   Check if all nodes have successfully joined the cluster.  
   ```sh
   kubectl get nodes
   ```  

### Label Worker Nodes  

7. **Assign Worker Roles to Nodes**  
   Label worker nodes to indicate their role in the cluster.  
   ```sh
   kubectl label node k8s-worker-1 node-role.kubernetes.io/worker=worker
   kubectl label node k8s-worker-2 node-role.kubernetes.io/worker=worker
   ```  

### ETCD Health Check  

8. **Check the ETCD Cluster Status**  
   Verify that the ETCD cluster is healthy and operational.  
   ```sh
   kubectl exec -it -n kube-system etcd-k8s-master -- \
   etcdctl --cacert /etc/kubernetes/pki/etcd/ca.crt \
           --cert /etc/kubernetes/pki/etcd/server.crt \
           --key /etc/kubernetes/pki/etcd/server.key endpoint health
   ```  

9. **List ETCD Cluster Members**  
   Display the list of all ETCD members in the cluster.  
   ```sh
   kubectl exec -it -n kube-system etcd-k8s-master -- \
   etcdctl --cacert /etc/kubernetes/pki/etcd/ca.crt \
           --cert /etc/kubernetes/pki/etcd/server.crt \
           --key /etc/kubernetes/pki/etcd/server.key member list
   ```  
