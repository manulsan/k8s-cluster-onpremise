# Getting Started with k8s-cluster-onpremise
This is to install k8s on Ubuntu 20.04

# Target Cluster

![k8s-cluster](https://user-images.githubusercontent.com/31757599/148323039-ec54f14e-2bc0-43d1-affc-24b68fec8976.png)
  
# Information source 
    - https://phoenixnap.com/kb/install-kubernetes-on-ubuntu#ftoc-heading-1
  
# Install Steps
  ## Step 0: Generic commands
    lsb_release -a
    sudo cat /etc/hosts     // todo : make set hostnames. master and worker node
    free -m                 // checks: check swap status
    sudo swapoff -a         // set swap off
    free -m                 // checks : check swap status
    sudo vi fstat           // todo : make "swap line" comment with adding #
    ping google.com         // todo : do "ping command" at each node 
  ## Step 1: Install docker 
	  sudo apt-get update  
	  sudo apt-get install docker.io  
	  docker ––version
    // Docker version 20.10.7, build 20.10.7-0ubuntu5~20.04.2  
	  // Repeat the process on each server that will act as a node.
  ## Step 2: Start and Enable Docker
	  sudo systemctl enable docker
	  sudo systemctl status docker
	  sudo systemctl start docker
	  // Repeat on all the other nodes.
  ## Step 3: Install Kubernetes
    sudo apt-get install curl
	  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add	  
	  // repeat the previous command to install the signing keys.
	  //Add Software Repositories Kubernetes is not included in the default repositories. To add them, enter the following:
	  sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
	  // Repeat on each server node.
  ## Step 4: Kubernetes Installation Tools
	Kubeadm (Kubernetes Admin) is a tool that helps initialize a cluster. It fast-tracks setup 
  by using community-sourced best practices. Kubelet is the work package, which runs on every node 
  and starts containers. The tool gives you command-line access to clusters.

	  sudo apt-get install kubeadm kubelet kubectl
	  sudo apt-mark hold kubeadm kubelet kubectl
	  kubeadm version
	  //  Repeat for each server node.
  ## Step 5: Kubernetes Deployment
	  sudo swapoff –a   // swap off, todo: make /etc/fstat file's swap field	  
	  sudo hostnamectl set-hostname master-node  // Assign Hostname for Each Server Node 
	  or
	  vim /etc/hosts        // modify file named below for hostnames
	  sudo hostnamectl set-hostname worker01
  ## Step 5: Initialize Kubernetes on Master Node
	  sudo kubeadm init --pod-network-cidr=10.244.0.0/16
	  mkdir -p $HOME/.kube
	  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ### Error on some of versions,  apply belows
    info source: https://www.inflearn.com/questions/298449
    쿠버네티스의 cgroup과 도커의 cgroup 이름이 일치하지 않아서 발생하는 문제로 
    다음 두 명령어를 사용해 도커 cgroup을 "cgroupfs" 에서 systemd로 변경이 가능합니다. 
    반드시 daemon.json 파일 생성후에 도커 데몬을 재시작해주시기 바랍니다.   
    $ cat <<EOF | sudo tee /etc/docker/daemon.json
    {
      "exec-opts": ["native.cgroupdriver=systemd"]
    }
    EOF
    $ service docker restart
    $ kubeadm init
    $ sudo docker info | grep -i cgroup      // for double check for cgroup
  ## Step 6: Deploy Pod Network to Cluster
	  sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

  ## Step 7: Verify that everything is running and communicating:
	  kubectl get pods --all-namespaces

  ## Step 8: Join Worker Node to Cluster
    -Info source : m.blog.naver.com/zippy/222073632636
    //------------- master node
	  -To join master node as worker node, it need to be joined.
     "kubeadm join"  needs parameters( token , hash value)
    // get YOUR_TOKEN
    $ kubeadm token list

    // create YOUR_TOKEN if not exists
    $ kubeadm token create

    // get HASH_VALUE
    $ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
    
    // get INERNAL_IP
    $kubectl get nodes -o wide               // to get INTERNAL_IP

    //--------- worker node
    //ex> $ kubeadm join 192.168.123.232:6443 --discovery-token abcdef.1234567890abcdef --discovery-token-ca-cert-hash sha256:1234..cdef 1.2.3.4:6443
    
    $ kubeadm join INERNAL_IP:6443 --discovery-token YOUR_TOKEN --discovery-token-ca-cert-hash sha256:HASH_VALUE

  ## Step 10: checking
	  - kubectl get nodes 
    
# Other stuffers
  ## Uninstall docker
	  dpkg -l | grep -i docker
    sudo apt-get purge -y docker-engine docker docker.io docker-ce docker-ce-cli
    sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce  
    sudo rm -rf /var/lib/docker /etc/docker
    sudo rm /etc/apparmor.d/docker
    sudo groupdel docker
    sudo rm -rf /var/run/docker.sock

    or

    sudo apt-get purge -y docker-engine docker docker.io docker-ce  
    sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce  
    sudo umount /var/lib/docker/
    sudo rm -rf /var/lib/docker /etc/docker
    sudo rm -rf /var/lib/containerd
    sudo rm /etc/apparmor.d/docker
    sudo groupdel docker
    sudo rm -rf /var/run/docker.sock
    sudo rm -rf /usr/bin/docker-compose

  ## Uninstall K8S
    sudo kubeadm reset
    sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube* 
    sudo apt-get autoremove
    sudo rm -rf ~/.kube  && sudo rm -fr /etc/kubernetes  && sudo rm -fr /var/lib/etcd


    # Git login and save informations  to prevent repeated userinfo requests
  ## Git stuffs for keeping login
    - https://pinedance.github.io/blog/2019/05/29/Git-Credential
    - git config credential.helper 'cache --timeout=600'
    - git config --global credential.helper store




    