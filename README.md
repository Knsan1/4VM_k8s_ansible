# Kubernetes Cluster Setup with Vagrant and Ansible

## Create Directories for Syncing with Home Directory of Each VM

$ mkdir ansible master node01 node02 node03

## Generate SSH Key

$ mkdir .ssh
$ cd .ssh
$ ssh-keygen -t rsa -b 2048 -f /home/kms/4VM/.ssh/node_rsa -q -N ""
$ cd ../

## Ensure `Vagrantfile` Exists and Bring Up VMs

$ vi Vagrantfile

$ vagrant up
$ vagrant status

## Test Access to VMs

$ vagrant ssh ansible
$ vagrant ssh master-node
$ vagrant ssh worker-node01
$ vagrant ssh worker-node02
$ vagrant ssh worker-node03

## Copy SSH Key from Ansible VM to Other VMs

$ vagrant ssh ansible
$ ssh-keygen -t rsa -b 2048 -f /home/vagrant/.ssh/id_rsa -q -N ""
$ ssh-copy-id master-node
$ ssh-copy-id worker-node01
$ ssh-copy-id worker-node02
$ ssh-copy-id worker-node03

## Verify Ansible Installation

$ ansible --version

## Create Directory and Put Necessary Playbooks and Config Files

$ mkdir playbook
$ cd playbook
$ touch ansible.cfg hostlist install_packages.yaml k8s_master_setup.yaml worker_node_join.yaml kubectl_install.sh

## Test Ansible Connection

$ ansible -i hostlist -m ping all
$ ansible -i hostlist -m shell -a "hostname;uptime" all

## Install Required Packages on All Nodes

$ ansible-playbook -i hostlist install_packages.yaml -C
$ ansible-playbook -i hostlist install_packages.yaml

## Spin Up Kubernetes Cluster on Master Node

$ ansible-playbook -i hostlist k8s_master_setup.yaml -C
$ ansible-playbook -i hostlist k8s_master_setup.yaml

## Install `kubectl` on Ansible Host Machine

$ sh kubectl_install.sh
$ mkdir ~/.kube
$ cp -p /path/to/admin.conf ~/.kube/config
$ kubectl get nodes

## Join Worker Nodes to Kubernetes Cluster

$ ansible-playbook -i hostlist worker_node_join.yaml -C
$ ansible-playbook -i hostlist worker_node_join.yaml

## Label All Joined Worker Nodes

$ kubectl get nodes
$ kubectl label node worker-node01 node-role.kubernetes.io/worker=worker-new
$ kubectl label node worker-node02 node-role.kubernetes.io/worker=worker-new
$ kubectl label node worker-node03 node-role.kubernetes.io/worker=worker-new
$ kubectl get nodes

## Install Calico CNI

$ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml

## Apply Custom Resource for Pod CIDR

$ kubectl apply -f custom_resource.yaml

## Test Deployment with Bookinfo

$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/bookinfo/platform/kube/bookinfo.yaml
$ kubectl get all

