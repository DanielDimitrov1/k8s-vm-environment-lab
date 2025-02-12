# -*- mode: ruby -*-
# vi: set ft=ruby :

NUM_WORKER_NODES = 2
IP_NW = "192.168.56."
IP_START = 10

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.box_check_update = false

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
    apt-get update
    apt-get install -y containerd.io
    mkdir -p /etc/containerd
    containerd config default | tee /etc/containerd/config.toml
    systemctl restart containerd
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    swapoff -a
    sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
    modprobe overlay
    modprobe br_netfilter
    cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
    sysctl --system
    chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  SHELL

  config.vm.define "master" do |master|
    master.vm.hostname = "k8s-master"
    master.vm.network "private_network", ip: IP_NW + "#{IP_START}"
    master.vm.provision "shell", inline: <<-SHELL
      kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=#{IP_NW}#{IP_START}
      mkdir -p $HOME/.kube
      cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      chown $(id -u):$(id -g) $HOME/.kube/config
      kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
    SHELL
  end

  (1..NUM_WORKER_NODES).each do |i|
    config.vm.define "worker-#{i}" do |worker|
      worker.vm.hostname = "k8s-worker-#{i}"
      worker.vm.network "private_network", ip: IP_NW + "#{IP_START + i}"
    end
  end
end