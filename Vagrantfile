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

	#TODO https://www.vagrantup.com/docs/multi-machine/#specifying-a-primary-machine
	config.vm.define "jenkins" do |jenkins|
	  # Every Vagrant development environment requires a box. You can search for
	  # boxes at https://atlas.hashicorp.com/search.
		jenkins.vm.box = "ubuntu/trusty64"

		  # Disable automatic box update checking. If you disable this, then
		  # boxes will only be checked for updates when the user runs
		  # `vagrant box outdated`. This is not recommended.
		  # config.vm.box_check_update = false

		  # Create a forwarded port mapping which allows access to a specific port
		  # within the machine from a port on the host machine. In the example below,
		  # accessing "localhost:8080" will access port 80 on the guest machine.
		jenkins.vm.network "forwarded_port", guest: 8080, host: 18080

		  # Create a private network, which allows host-only access to the machine
		  # using a specific IP.
		jenkins.vm.network "private_network", ip: "192.168.33.10"

		  # Create a public network, which generally matched to bridged network.
		  # Bridged networks make the machine appear as another physical device on
		  # your network.
		  # config.vm.network "public_network"

		  # Share an additional folder to the guest VM. The first argument is
		  # the path on the host to the actual folder. The second argument is
		  # the path on the guest to mount the folder. And the optional third
		  # argument is a set of non-required options.
		  # config.vm.synced_folder "../data", "/vagrant_data"
		  jenkins.vm.synced_folder "./sync", "/vagrant"
		  
		  
		  # Provider-specific configuration so you can fine-tune various
		  # backing providers for Vagrant. These expose provider-specific options.
		  # Example for VirtualBox:
		  #
		jenkins.vm.provider "virtualbox" do |vb|
			#   # Display the VirtualBox GUI when booting the machine
			#
			#vb.gui = true
			#
			#   # Customize the amount of memory on the VM:
			#vb.memory = "4096"
			vb.memory = "2048"
			vb.cpus = "2"
		end
		  
		  #
		  # View the documentation for the provider you are using for more
		  # information on available options.

		  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
		  # such as FTP and Heroku are also available. See the documentation at
		  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
		  # config.push.define "atlas" do |push|
		  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
		  # end

		  # Enable provisioning with a shell script. Additional provisioners such as
		  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
		  # documentation for more information about their specific syntax and use.
		  
		  # Setup hostnames for VB internal net
		  # https://stackoverflow.com/questions/20681190/can-multiple-vagrant-vms-communicate-by-vm-hostname
		jenkins.vm.provision "shell", inline: <<-SHELL
			wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
			sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
			sudo apt-get update
			sudo add-apt-repository ppa:openjdk-r/ppa
			sudo apt-get update
			sudo apt-get -y install openjdk-8-jdk
			sudo apt-get -y install jenkins
			sudo apt-get -y install git
			sudo echo '192.168.33.20 slave' >> /etc/hosts
			sudo echo '192.168.33.30 registry' >> /etc/hosts
			sudo cat /var/lib/jenkins/secrets/initialAdminPassword
		SHELL
	
	end

	config.vm.define "slave" do |slave|
		# Every Vagrant development environment requires a box. You can search for
		# boxes at https://atlas.hashicorp.com/search.
		slave.vm.box = "ubuntu/trusty64"

		# slave.vm.network "forwarded_port", guest: 8080, host: 18080
		slave.vm.network "private_network", ip: "192.168.33.20"

		# Create a public network, which generally matched to bridged network.
		# Bridged networks make the machine appear as another physical device on
		# your network.
		# config.vm.network "public_network"

		slave.vm.synced_folder "./sync", "/vagrant"


		# Provider-specific configuration so you can fine-tune various
		# backing providers for Vagrant. These expose provider-specific options.
		# Example for VirtualBox:
		#
		slave.vm.provider "virtualbox" do |vb|
			vb.memory = "1024"
			vb.cpus = "1"
		end

		# Enable provisioning with a shell script. Additional provisioners such as
		# Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
		# documentation for more information about their specific syntax and use.
		#slave.vm.provision "shell", inline: <<-SHELL
		#	sudo apt-get update
		#	sudo apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
		#	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
		#	sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu trusty stable"
		#	sudo apt-get update
		#	sudo apt-get -y install docker-ce
		#	sudo usermod -a -G docker vagrant
		#	sudo docker pull registry
		#	sudo docker run -d -p 5000:5000 --restart always --name registry registry:2
		#SHELL
	end

	config.vm.define "registry" do |registry|
		# Every Vagrant development environment requires a box. You can search for
		# boxes at https://atlas.hashicorp.com/search.
		registry.vm.box = "ubuntu/trusty64"

		# registry.vm.network "forwarded_port", guest: 8080, host: 18080
		registry.vm.network "private_network", ip: "192.168.33.30"

		# Create a public network, which generally matched to bridged network.
		# Bridged networks make the machine appear as another physical device on
		# your network.
		# config.vm.network "public_network"

		registry.vm.synced_folder "./sync", "/vagrant"


		# Provider-specific configuration so you can fine-tune various
		# backing providers for Vagrant. These expose provider-specific options.
		# Example for VirtualBox:
		#
		registry.vm.provider "virtualbox" do |vb|
			vb.memory = "2048"
			vb.cpus = "1"
		end

		# Enable provisioning with a shell script. Additional provisioners such as
		# Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
		# documentation for more information about their specific syntax and use.
		registry.vm.provision "shell", inline: <<-SHELL
			sudo apt-get update
			sudo apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
			curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
			sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu trusty stable"
			sudo apt-get update
			sudo apt-get -y -V install docker-ce=17.03.0~ce-0~ubuntu-trusty
			sudo usermod -a -G docker vagrant
			sudo docker pull registry
			sudo docker run -d -p 5000:5000 --restart always --name registry registry:2
			sudo mkdir -p /etc/systemd/system/docker.service.d
			sudo su root
			echo [Service] > /etc/systemd/system/docker.service.d/local_insecure.conf
			echo Environment="DOCKER_OPTS=--insecure-registry 127.0.0.1:5000" >> /etc/systemd/system/docker.service.d/local_insecure.conf
			exit
			sudo service docker restart
		SHELL
	end
end
