# -*- mode: ruby -*-
# vi: set ft=ruby :

$vagrant_mem = 8192
$vagrant_cpus = 4
$local_mesos_dir = "../mesos"

Vagrant.configure(2) do |config|
  config.vm.box = "fedora/27-cloud-base"

  config.vm.provider "virtualbox" do |v|
    v.customize [
      "modifyvm", :id,
      "--memory", "#{$vagrant_mem}",
      "--cpus", "#{$vagrant_cpus}",
      "--paravirtprovider", "kvm"
    ]
  end

  config.vm.provider "vmware_fusion" do |v|
    v.memory = $vagrant_mem
    v.cpus = $vagrant_cpus
  end

  config.vm.network "forwarded_port", guest: 5050, host: 5050
  config.vm.network "private_network", type: "dhcp"

  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provision "shell", inline: <<-SHELL
    dnf -y upgrade

    # Development tools
    dnf -y install automake ccache cmake gcc-c++ git libtool patch

    # Mesos dependencies
    dnf -y install apr-devel apr-util-devel cyrus-sasl-devel cyrus-sasl-md5    \
                   elfutils-libelf-devel libblkid-devel libcurl-devel          \
                   libevent-devel libnl3-devel openssl-devel python-devel      \
                   redhat-rpm-config subversion-devel xfsprogs-devel
    dnf -y install java-1.8.0-openjdk-devel maven

    # Test dependencies
    dnf -y install bind-utils ethtool logrotate nmap-ncat perf
  SHELL

  config.vm.provision :reload

  config.vm.provision "shell", inline: <<-SHELL
    # Remove old kernel
    dnf -y remove $(dnf repoquery --installonly --latest-limit=-1 -q)

    # Sysdig
    curl -s https://s3.amazonaws.com/download.draios.com/stable/install-sysdig | bash
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    cd ~
    git clone https://github.com/apache/mesos.git
    cd mesos
    ./bootstrap
    mkdir build
    cd build
    ../configure --enable-port-mapping-isolator --enable-libevent --enable-ssl \
                 --enable-optimize --enable-xfs-disk-isolator --enable-grpc
  SHELL
end
