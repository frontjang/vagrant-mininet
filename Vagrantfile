# -*- mode: ruby -*-
# vi: set ft=ruby :

$init = <<SCRIPT
	sudo sed -i 's/archive.ubuntu.com/ftp.daum.net/g' /etc/apt/sources.list
	sudo aptitude update
	export DEBIAN_FRONTEND='noninteractive'
SCRIPT

$ovs = <<SCRIPT
	sudo apt-get install -y build-essential fakeroot
	wget -qO- http://openvswitch.org/releases/openvswitch-2.5.1.tar.gz | tar xzvf -
	cd openvswitch-2.5.1
	sudo apt-get install -y module-assistant graphviz autoconf automake bzip2 debhelper dh-autoreconf libssl-dev libtool openssl procps python-all python-twisted-conch python-zopeinterface python-six
	sudo apt-get install -y dkms libc6-dev make openssl kmod module-init-tools netbase procps python-argparse uuid-runtime iproute2 
	#racoon breaks vagrant ssh Actually, as ipsec-tools is a dependency of racoon
	DEB_BUILD_OPTIONS='parallel=8 nocheck' fakeroot debian/rules binary  
	cd ..
	mv openvswitch-ipsec_2.5.1-1_amd64.deb openvswitch-2.5.1/
	sudo dpkg -i *.deb
SCRIPT

$mininet = <<SCRIPT
  sudo apt-get install -y git
  git clone git://github.com/mininet/mininet
  cd mininet
  git checkout -b 2.2.1 2.2.1
  ./util/install.sh -fn
SCRIPT

$wireshark = <<SCRIPT
  sudo apt-get install -y wireshark
  sudo groupadd wireshark
  sudo usermod -a -G wireshark vagrant
  sudo chgrp wireshark /usr/bin/dumpcap
  sudo chmod 750 /usr/bin/dumpcap
  sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/dumpcap
  sudo getcap /usr/bin/dumpcap
  logout
SCRIPT

$cleanup = <<SCRIPT
  aptitude clean
  rm -rf /tmp/*
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", "2048"]
  end

  ## Guest config
  config.vm.hostname = "sdnlab"
  config.ssh.forward_agent = true
  config.ssh.forward_x11 = true
	
  ## Provisioning
  #http://foo-o-rama.com/vagrant--stdin-is-not-a-tty--fix.html
  config.vm.provision "fix-no-tty", type: "shell" do |s|
    s.privileged = false
    s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
  end
  
  config.vm.provision :shell, privileged: false, :inline => $init
  config.vm.provision :shell, privileged: false, :inline => $ovs
  config.vm.provision :shell, privileged: false, :inline => $mininet
  config.vm.provision :shell, privileged: false, :inline => $wireshark

end
