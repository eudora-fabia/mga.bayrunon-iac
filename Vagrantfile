# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false
  
  public_key = File.read("id_rsa.pub")
  
  live_server_ip = "192.168.33.20"
  
  config.vm.define "live" do |m|
	m.vm.hostname = "live"
	m.vm.network "private_network", ip: live_server_ip
	m.vm.network "forwarded_port", guest: 22, host: 2220
	m.vm.network "forwarded_port", guest: 2812, host: 2812
	m.vm.provision :shell, :inline =>"
     echo 'Copying ansible public SSH Keys to live'
     mkdir -p /home/vagrant/.ssh
     chmod 700 /home/vagrant/.ssh
     echo '#{public_key}' >> /home/vagrant/.ssh/authorized_keys
     chmod -R 600 /home/vagrant/.ssh/authorized_keys
     echo 'StrictHostKeyChecking no' >> /home/vagrant/.ssh/config
	 echo 'UserKnownHostsFile /dev/null' >> /home/vagrant/.ssh/config
	 chmod -R 600 /home/vagrant/.ssh/config
	 
	 echo 'Updating /etc/hosts'
	 echo '192.168.33.10 ansible' | sudo tee -a /etc/hosts
    ", privileged: false
  end
  
  config.vm.define "ansible" do |m|
	m.vm.hostname = "ansible"
	m.vm.network "private_network", ip: "192.168.33.10"
	m.vm.network "forwarded_port", guest: 22, host: 2210
	m.vm.provision "file", source: "id_rsa", destination: "/home/vagrant/.ssh/id_rsa"
	m.vm.provision "file", source: "id_rsa.pub", destination: "/home/vagrant/.ssh/id_rsa.pub"
	m.vm.provision "file", source: ".vault_pass", destination: "/home/vagrant/.vault_pass"
	m.vm.provision :shell, :inline =>"
	 sudo chmod -R 700 /home/vagrant/.ssh
	 sudo chmod 600 /home/vagrant/.vault_pass
	 echo 'Updating /etc/hosts'
	 echo '#{live_server_ip} live' | sudo tee -a /etc/hosts
	 echo 'Installing ansible'
	 sudo apt-add-repository -y ppa:ansible/ansible
	 sudo apt install -y ansible python-pip virtualenv
	 echo 'Updating /etc/ansible/ansible.cfg - private_key_file = /home/vagrant/.ssh/id_rsa'
	 sudo sed -i 's|#private_key_file = /path/to/file|private_key_file = /home/vagrant/.ssh/id_rsa|g' /etc/ansible/ansible.cfg
	 sudo sed -i 's|#host_key_checking = False|host_key_checking = False|g' /etc/ansible/ansible.cfg
	 cd /vagrant
	 echo '[prod]' > hosts
	 echo '#{live_server_ip}' >> hosts
	 sudo chmod +x install.sh
	 sudo bash install.sh -H live -p ~/.vault_pass
    ", privileged: false
  end
end

#	 echo 'Disabling IPv6'
#	 echo 'net.ipv6.conf.all.disable_ipv6 = 1
#      net.ipv6.conf.default.disable_ipv6 = 1
#      net.ipv6.conf.lo.disable_ipv6 = 1
#      net.ipv6.conf.enp0s3.disable_ipv6 = 1
#	  net.ipv6.conf.enp0s8.disable_ipv6 = 1' | sudo tee -a /etc/sysctl.conf
#     sudo sysctl -p
	
	 

# echo 'Host 192.168.*.*' >> /home/vagrant/.ssh/config
# echo 'StrictHostKeyChecking no' >> /home/vagrant/.ssh/config
# echo 'UserKnownHostsFile /dev/null' >> /home/vagrant/.ssh/config
# chmod -R 600 /home/vagrant/.ssh/config

#	m.vm.provision :shell, :inline =>"
#	 chmod -R 700 /home/vagrant/.ssh
#	 chmod 600 /home/vagrant/.vault_pass
#	 echo 'Updating /etc/hosts'
#	 echo '192.168.33.20 live' | sudo tee -a /etc/hosts
#	 echo 'Installing ansible'
#	 sudo apt-add-repository -y ppa:ansible/ansible
#	 sudo apt install -y ansible python-pip virtualenv
#	 cd /vagrant
#	 echo '[prod]' > hosts
#	 echo '192.168.33.20' >> hosts
#	 sudo chmod +x install.sh
#	 sudo bash install.sh -H 192.168.33.20 -p ~/.vault_pass
#    ", privileged: false
#