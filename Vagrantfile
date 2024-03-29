# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "debian/stretch64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"
  # config.vm.network "private_network"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true

    # Customize the amount of memory on the VM:
    vb.memory = "2048"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    swapoff -a
    perl -n -i -e 'm/swap/ || print' /etc/fstab
    apt-get update
    apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
    echo deb https://download.docker.com/linux/debian stretch stable | tee /etc/apt/sources.list.d/docker.list
    wget -qO - https://download.docker.com/linux/debian/gpg | apt-key add -
    apt-get update
    apt-get install -y docker-ce
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
    apt-get update
    apt-get upgrade -y
    apt-get install -y kubelet kubeadm kubectl kubernetes-cni
    kubeadm config images pull
  SHELL
  
  config.vm.define "master", primary: true do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "172.28.128.254"
    master.vm.provision "shell", inline: <<-SHELL
      mkdir -p ~vagrant/.kube
      cp -i /etc/kubernetes/admin.conf ~vagrant/.kube/config
      chown vagrant:vagrant ~vagrant/.kube/config
      wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml -O ~vagrant/kube-flannel.yml
      perl -p -i -e 's/(kube-subnet-mgr)/\1\n        - --iface=eth1/' ~vagrant/kube-flannel.yml
      kubeadm init --pod-network-cidr 10.244.0.0/16 \
        --apiserver-advertise-address 172.28.128.254 \
        --apiserver-cert-extra-sans kubernetes.cluster.home
      kubectl apply -f ~vagrant/kube-flannel.yml
    SHELL
  end
  (0..2).each do |id| 
    config.vm.define "node#{id}" do |node|
      node.vm.hostname = "node#{id}"
      node.vm.network "private_network", type: "dhcp"
    end
  end
end
