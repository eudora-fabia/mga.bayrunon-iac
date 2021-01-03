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
  
  live_server_ip = "192.168.33.20"
  
  config.vm.define "live" do |m|
	m.vm.hostname = "live"
	m.vm.network "private_network", ip: live_server_ip
	m.vm.network "forwarded_port", guest: 22, host: 2220
	m.vm.network "forwarded_port", guest: 2812, host: 2812
	m.vm.provision :shell, :inline =>"
	 echo 'Updating /etc/hosts'
	 echo '192.168.33.10 ansible' | sudo tee -a /etc/hosts
	 
	 echo 'Updating /etc/ssh/sshd_config to allow password authentication'
	 sudo sed -i 's|PasswordAuthentication no|PasswordAuthentication yes|' /etc/ssh/sshd_config
	 sudo systemctl restart sshd
    ", privileged: false
  end
  
  
  config.vm.define "ansible" do |m|
	m.vm.hostname = "ansible"
	m.vm.network "private_network", ip: "192.168.33.10"
	m.vm.network "forwarded_port", guest: 22, host: 2210
	m.vm.provision "file", source: ".vault_pass", destination: "/home/vagrant/.vault_pass"
	m.vm.provision :shell, :inline =>"
	 sudo chmod 600 /home/vagrant/.vault_pass
	
	 echo 'Updating /etc/hosts'
	 echo '#{live_server_ip} live' | sudo tee -a /etc/hosts
	 
	 echo 'Generating SSH Key'
	 ssh-keygen -t rsa -f ~/.ssh/id_rsa -N ''
	 
	 echo 'Generating SSH Config'
	 echo 'StrictHostKeyChecking no' >> ~/.ssh/config
	 chmod -R 600 /home/vagrant/.ssh/config
	 
	 echo 'Installing sshpass'
	 sudo apt -y update
	 sudo apt -y install sshpass
	 
	 echo 'Copying SSH Key to Live'
	 sshpass -p 'vagrant' ssh-copy-id live
	 
	 echo 'Installing ansible'
	 sudo apt-add-repository -y ppa:ansible/ansible
	 sudo apt install -y ansible python-pip virtualenv
	 echo 'Updating /etc/ansible/ansible.cfg'
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