$$
\Huge \textcolor{purple}{Setup ~KUBERNETES ~on ~CentOS ~8/7 ~with ~Kubeadm ~and ~Vagrant}
$$

## Step 1: Start the vagrant box
Here is the Vagrantfile to spin up wer vagrant box.

Use the following image name for CentOS 7 and CentOS 8 -

### > CentOS 7 : - [centos/7](https://app.vagrantup.com/centos/boxes/7)
### > CentOS 8 : - [centos/8](https://app.vagrantup.com/centos/boxes/8)
Based on wer need we need to update the following vagrantfile.

We are going with two VMs here -

```sh
Master Node - 2 cpus, 2 GB Memory (Assinged IP - 100.0.0.1 )
Worker Node - 1 cpu, 1 GB Memory (Assinged IP - 100.0.0.2 )
Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box_download_insecure = true    
    master.vm.box = "centos/stream8"
    master.vm.network "private_network", ip: "100.0.0.1"
    master.vm.hostname = "master"
    master.vm.provider "virtualbox" do |v|
      v.name = "master"
      v.memory = 2048
      v.cpus = 2
    end
  end

  config.vm.define "worker" do |worker|
    worker.vm.box_download_insecure = true 
    worker.vm.box = "centos/stream8"
    worker.vm.network "private_network", ip: "100.0.0.2"
    worker.vm.hostname = "worker"
    worker.vm.provider "virtualbox" do |v|
      v.name = "worker"
      v.memory = 1024
      v.cpus = 1
    end
  end

end
```
> Copy the Worker Section multiple times to increase the desired Worker Node. \
Change the IP address of **"private_network"** accordingly. \
Add the additional IP address at host file **(sudo vi /etc/hosts)**


* VAGRANT COMMANDS
```sh
vagrant up
vagrant destroy -f
```
#
$$ \textcolor{red}{TROUBLESHOOT ~1} $$
```sh
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["hostonlyif", "create"]
```
**==Refer End of File==**
#

## Step 2: Update /etc/hosts on both nodes(master, worker)
> Error: (Vagrant::Util::Subprocess::LaunchError): USE GITBASH to resolve

* master node - SSH into the master node
```sh
vagrant ssh master

sudo vi /etc/hosts
100.0.0.1 master.ppd.com master
100.0.0.2 worker.ppd.com worker
```
* worker node- SSH into the worker node
```sh
vagrant ssh worker

vagrant@worker:~$sudo vi /etc/hosts
100.0.0.1 master.jhooq.com master
100.0.0.2 worker.jhooq.com worker
```
* Test the worker node by sending ping from master

ping worker

* Test the master node by sending ping from worker

ping master

## Step 3: Install Docker on both nodes (master, worker)
https://docs.docker.com/engine/install/centos/

* we need to install Docker on both the node. But before that we need to update the yum repos -
```sh
sudo yum install -y yum-utils
```
$$ \textcolor{red}{TROUBLESHOOT ~2} $$
```sh
[vagrant@master ~]$ sudo yum install -y yum-utils
CentOS Linux 8 - AppStream                                               30  B/s |  38  B     00:01
Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: No URLs in mirrorlist
```

**==Refer End of File==**
#


* Add the following docker repo to CentOS -

```sh
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo 
```

* Run the following docker installation command on both the nodes
```sh
sudo yum install docker-ce docker-ce-cli containerd.io
```

* Enable docker: on both master and worker node
```sh
sudo systemctl enable docker
```

* Start docker: on both master and worker node
```sh
sudo systemctl start  docker
```

* Check the docker service status
```sh
sudo systemctl status docker
```

Docker service should be up and running and we should get following output on the terminal

## Step 4: Disable SELinux on both nodes(master, worker)
* we need to disable the SELinux using following command
```sh
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

## Step 5: Disable CentOS firewall on both nodes(master, worker)

> Master Node
```sh
sudo systemctl disable firewalld
sudo systemctl stop firewalld
```
> Worker Node
```sh
sudo systemctl disable firewalld
sudo systemctl stop firewall
```

## Step 6: Disable swapping on both nodes(master, worker)

> Disable the swapping on master as well as a worker node. Because to install Kubernetes we need to disable the swapping on both the nodes

* Run following command on both master as well as worker node

```sh
free -m #To check SWAP
sudo swapoff -a
```

## Step 7: Enable the usage of "iptables" on both nodes(master, worker)
> Enable the usage of iptables which will prevent the routing errors happening. As the following runtime parameters:

```sh
sudo bash -c 'echo "net.bridge.bridge-nf-call-ip6tables = 1" > /etc/sysctl.d/k8s.conf'

sudo bash -c 'echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.d/k8s.conf'

sudo sysctl --system
```

## Step 8: Add the Kubernetes repo to rum.repos.d on both nodes(master, worker)
```sh
sudo vi /etc/yum.repos.d/kubernetes.repo

#Add following repo details -

[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```


## Step 9: Install Kubernetes on both nodes(master, worker)
```sh
sudo yum install -y kubeadm kubelet kubectl
```

## Step 10: Enable and Start Kubelet on both nodes(master, worker)
* Run the following command both on master and worker nodes.

  * Enable the kubelet
```sh
sudo systemctl enable kubelet
```

* Start the kubelet
```sh
sudo systemctl start kubelet
```

* Status the kubelet
```sh
sudo systemctl status kubelet
```

$$ \textcolor{red}{TROUBLESHOOT ~3} $$
```sh

[vagrant@master yum.repos.d]$ sudo kubeadm init --apiserver-advertise-address=100.0.0.1 --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.25.0
[preflight] Running pre-flight checks
        [WARNING FileExisting-tc]: tc not found in system path
error execution phase preflight: [preflight] Some fatal errors occurred:
```

**==Refer End of File==**
#
$$ \textcolor{red}{TROUBLESHOOT ~4} $$
```sh
[vagrant@master yum.repos.d]$ sudo kubeadm init --apiserver-advertise-address=100.0.0.1 --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.25.0
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR CRI]: container runtime is not running: output: E0829 11:11:55.450810   37104 remote_runtime.go:925] "Status from runtime service failed" 
```
**==Refer End of File==**
#


## Step 11: Initialize Kubernetes cluster only on master node

> Initialize the Kubernetes cluster (-apiserver-advertise-address=100.0.0.1 this is the IP address we have assigned in the /etc/hosts)
```sh
sudo kubeadm init --apiserver-advertise-address=100.0.0.1 --pod-network-cidr=10.244.0.0/16

#Note down the kubeadm join command
kubeadm join 100.0.0.1:6443 --token cfvd1x.8h8kzx0u9vcn4trf \
    --discovery-token-ca-cert-hash sha256:cc9687b47f3a9bfa5b880dcf409eeaef05d25505f4c099732b65376b0e14458c
```

## Step 12: Move kube config file to current user (only run on master)
> To interact with the Kubernetes cluster and to use kubectl command, we need to have the Kube config file with us.

* Use the following command to get the kube config file and put it under the working directory.
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Step 13: Apply CNI from kube-flannel.yml(only run on master)
* After the master of the cluster is ready to handle jobs and the services are running, for the purpose of making containers accessible to each other through networking, we need to set up the network for container communication.

> Get the CNI(container network interface) configuration from [flannel](https://github.com/coreos/flannel#flannel). \
But before downloading the [flannel](https://github.com/coreos/flannel#flannel) we should make sure that we have installed wget

```sh
sudo yum install wget
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

>>Note – But since we are working on the VMs so we need to check our Ethernet interfaces first. \
Look out for the Ethernet i.e. eth1 which has a ip address 100.0.0.1(this is the ip address which we used in vagrant file)
```sh
 ip a s
1: lo: <LOOPBACK,UP,LOWER_UP>
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:bb:14:75 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fb:48:77 brd ff:ff:ff:ff:ff:ff
    inet 100.0.0.1
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP>
Now we need to add the extra args for eth1 in kube-flannel.yml
```
```sh
vi kube-flannel.yml
Searche for – “/flanneld”

In the args section add in the last line: – –iface=eth1

- --iface=eth1
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth1         #Only this line gets added
```

* Apply the flannel configuration
```sh
kubectl apply -f kube-flannel.yml
```

## Step 14: Join master node run only on worker node
In the Step 11 we generated the token and kubeadm join command.

Now we need to use that join command from our worker node
```sh
sudo kubeadm join 100.0.0.1:6443 --token cfvd1x.8h8kzx0u9vcn4trf --discovery-token-ca-cert-hash sha256:cc9687b47f3a9bfa5b880dcf409eeaef05d25505f4c099732b65376b0e14458c
```
## Step 15: Checking
* Create Deployment Yaml File
```sh
vi deployment.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

* Create Deployment
```sh
kubectl apply -f deployment.yaml
kubectl get deploy -A
```

# OR

* Create deployment from CLI
```sh
kubectl create deployment nginx --image=nginx
```
#

```sh
kubectl get deployments
kubectl describe deployment nginx
kubectl create service nodeport nginx --tcp=80:80
kubectl get svc

# NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
# kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        31m
# nginx        NodePort    10.108.188.250   <none>        80:30753/TCP   13s

kubectl get nodes

# NAME     STATUS   ROLES           AGE   VERSION
# master   Ready    control-plane   31m   v1.25.0
# worker   Ready    <none>          13m   v1.25.0
```

* To check if it is accessed by both Worker & Node
```sh
curl master:30753   #refer kubectl get svc
curl worker:30753   #refer kubectl get svc
```
*The above will display as:*
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
```


#

* To Delete Deployment
```sh
kubectl get deploy -A
kubectl delete deploy nginx-deployment
```
#
# TROUBLESHOOT 1

Right click on "This PC" / "My Computer" on windows desktop
1. Select "Properties"
2. Go to "Advanced" tab
3. Click "Environment Variables..." at the bottom
4. Under System Variables click "New..."
5. Set "Variable name" to "VBOX_INSTALL_PATH"
6. Set "Variable value" to "C:\Program Files\Oracle\VirtualBox\"
7. Select "OK" and close all the other settings windows
> source: https://github.com/mitchellh/vagrant/issues/3852

# TROUBLESHOOT 2
```sh
cd /etc/yum.repos.d/

sudo sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sudo sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
sudo yum update -y
```
# TROUBLESHOOT 3
```sh
sudo dnf install -y iproute-tc
```
# TROUBLESHOOT 4

> Source : https://github.com/containerd/containerd/issues/4581
```sh
sudo rm /etc/containerd/config.toml
sudo systemctl restart containerd
sudo kubeadm init
```
