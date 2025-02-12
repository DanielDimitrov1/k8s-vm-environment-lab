# Kubernetes  

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
