# -*- mode: ruby -*-
# vi: set ft=ruby :
NUM_WORKER_NODES    = 3
IP_NW="192.168.56."
IP_START=10
CPUS_MASTER_NODE    = 4
MEMORY_MASTER_NODE  = 2048
CPUS_WORKER_NODE    = 2
MEMORY_WORKER_NODE  = 1024
VAGRANT_BOX_IMAGE   = "bento/ubuntu-22.04"
CPUS_ANSIBLE_NODE   = 2
MEMORY_ANSIBLE_NODE = 1024

Vagrant.configure("2") do |config|

  config.vm.provision "shell", env: {"IP_NW" => IP_NW, "IP_START" => IP_START}, inline: <<-SHELL
      apt-get update -y
      echo "$IP_NW$((IP_START))  master-node  master-node.hellocloud.io" >> /etc/hosts
      echo "$IP_NW$((IP_START+1))  worker-node01  worker-node01.hellocloud.io" >> /etc/hosts
      echo "$IP_NW$((IP_START+2))  worker-node02  worker-node02.hellocloud.io" >> /etc/hosts
      echo "$IP_NW$((IP_START+3))  worker-node03  worker-node03.hellocloud.io" >> /etc/hosts
      echo "$IP_NW$((IP_START+4))  ansible-node  ansible-node.hellocloud.io" >> /etc/hosts
  SHELL

  config.vm.define "master" do |master|
    config.ssh.private_key_path = "/home/kms/4VM/.ssh/node_rsa"
    config.ssh.forward_agent = true
    config.ssh.username = "vagrant"
    config.ssh.password = "vagrant"
    master.vm.box = VAGRANT_BOX_IMAGE
    master.vm.hostname = "master-node.hellocloud.io"
    master.vm.synced_folder "./master", "/home/vagrant/"
    master.vm.network "private_network", ip: IP_NW + "#{IP_START}"
    master.vm.provider "virtualbox" do |vb|
      vb.name = "master-node"
      vb.cpus = CPUS_MASTER_NODE
      vb.memory = MEMORY_MASTER_NODE
      vb.gui = false
    end
    master.vm.provision "shell", run: "always", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install net-tools zip curl jq tree unzip wget siege apt-transport-https ca-certificates software-properties-common gnupg lsb-release bash-completion -y
      # Enable passwordless authentication
      sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
      sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
      sudo systemctl restart ssh
      netstat -tunlp
      echo "Hello from Master-node"
    SHELL
  end

  (1..NUM_WORKER_NODES).each do |i|
    config.vm.define "node0#{i}" do |node|
      config.ssh.private_key_path = "/home/kms/4VM/.ssh/node_rsa"
      config.ssh.forward_agent = true
      config.ssh.username = "vagrant"
      config.ssh.password = "vagrant"
      node.vm.hostname = "worker-node0#{i}.hellocloud.io"
      node.vm.box = VAGRANT_BOX_IMAGE
      node.vm.synced_folder "./node0#{i}", "/home/vagrant/"
      node.vm.network "private_network", ip: IP_NW + "#{IP_START + i}"
      node.vm.provider "virtualbox" do |vb|
        vb.name = "worker-node0#{i}"
        vb.cpus = CPUS_WORKER_NODE
        vb.memory = MEMORY_WORKER_NODE
        vb.gui = false
      end
      node.vm.provision "shell", run: "always", inline: <<-SHELL
        sudo apt-get update
        sudo apt-get install net-tools zip curl jq tree unzip wget siege apt-transport-https ca-certificates software-properties-common gnupg lsb-release bash-completion -y
        # Enable passwordless authentication
        sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
        sudo systemctl restart ssh
        netstat -tunlp
        echo "Hello from Worker-node#{i}"
      SHELL
    end
  end

  config.vm.define "ansible" do |ansible|
    config.ssh.private_key_path = "/home/kms/4VM/.ssh/node_rsa"
    config.ssh.forward_agent = true
    config.ssh.username = "vagrant"
    config.ssh.password = "vagrant"
    ansible.vm.box = VAGRANT_BOX_IMAGE
    ansible.vm.hostname = "ansible-node.hellocloud.io"
    ansible.vm.synced_folder "./ansible", "/home/vagrant/"
    ansible.vm.network "private_network", ip: IP_NW + "#{IP_START + NUM_WORKER_NODES + 1}"
    ansible.vm.provider "virtualbox" do |vb|
      vb.name = "ansible-node"
      vb.cpus = CPUS_ANSIBLE_NODE
      vb.memory = MEMORY_ANSIBLE_NODE
      vb.gui = false
    end

    ansible.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
      sudo apt-get install -y software-properties-common
      sudo add-apt-repository --yes --update ppa:ansible/ansible
      sudo apt-get install -y ansible
      sudo apt-get install -y sshpass

      # Generate SSH keys
      # ssh-keygen -t rsa -b 2048 -f /home/vagrant/.ssh/id_rsa -q -N ""
      # cat /home/vagrant/.ssh/id_rsa.pub > /home/vagrant/.ssh/authorized_keys

      # Copy SSH keys to all nodes
      # for i in 0 1 2 3; do
      #   sshpass -p "vagrant" ssh-copy-id -o StrictHostKeyChecking=no vagrant@#{IP_NW}$((IP_START + i))
      # done
    SHELL
  end

end
