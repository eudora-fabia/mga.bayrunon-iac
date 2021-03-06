# Vagrant Getting Started

cd to directory

Create .vault_pass file with content `secretpassword` - this is the password to group_vars/vault.yml

vagrant up

- this will run `ansible` and `live` Ubuntu 18.04LTS servers

To manually run ansible-playbook, do this in `ansible` server

`cd /vagrant`

`sudo ansible-playbook -i hosts provision.yml -e env=prod --vault-password-file ~/.vault_pass`

To run only specific tags:

`sudo ansible-playbook -i hosts provision.yml -e env=prod --vault-password-file ~/.vault_pass --tags "lamp-server"`

To check exim4 logs in `live`

`sudo eximstats /var/log/exim4/mainlog`

List email queue

`sudo mailq`

Remove all email queue

`sudo exim -bp | awk '/^ *[0-9]+[mhd]/{print "exim -Mrm " $3}' | sudo bash`

To send test email

`echo "This is test mail from my machine." | mail -s Testing hello@jimbalatero.com`

To view email

`exim -Mvb [id]`

# In newly rebuilt Cloudcone server - Ubuntu 18.04 LTS, do this first in ctrl node

```shell
ssh-copy-id root@s02cloudcone.jimbalatero.com
# it will ask you for root password sent by Cloudcone via Email
# somewhere in the playbook, the root password will be changed, 
# so Cloudcone generated password will be invalid anymore.

ansible all -i hosts -m ping #should return SUCCESS
```

Do this in ansible control node. This assumes, ssh user logged in is sudo password-less.
```bash
cd ~
git clone git@github.com:eudora-fabia/mga.bayrunon-iac.git
cd mga.bayrunon-iac
# Change it with the password
echo 'secretpassword' > .vault_pass
# Make sure file is only Read and Writable by the owner
sudo chmod 600 .vault_pass

# create hosts file
echo '[prod]' > hosts
echo 's02cloudcone.jimbalatero.com ansible_ssh_user=root' >> hosts

# ssh copy and test ansible connection
ssh-copy-id root@s02cloudcone.jimbalatero.com
ansible all -i hosts -m ping #should return SUCCESS

# do this to include installing the requirements
sudo bash install.sh -H s02cloudcone.jimbalatero.com -p .vault_pass -e env=prod
# or this, if reqs are already installed
# ansible-playbook provision.yml
# For some reason, jnv.debian-backports var: backports_uri doesn't work in all.yml
```

facettectl library restore -i dump.tar.gz && sudo systemctl restart facette
sudo rm -f /var/lib/facette/data.db && sudo systemctl restart facette

sudo curl http://localhost:12003/api/v1/library/graphs

# Prep the server

Login as root in the prod server

Create a user account for Ansible to do its thing through:

```
useradd -m -d /home/ansible ansible
passwd ansible
echo 'ansible ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/ansible
```

# In the control node

```
# ssh copy and test ansible connection
ssh-copy-id s02cloudcone.jimbalatero.com
ansible all -m ping #should return SUCCESS
```

# In the control node

```
# ssh copy and test ansible connection
ssh-copy-id s02cloudcone.jimbalatero.com
ansible all -i hosts -m ping #should return SUCCESS

# Random port generator

`curl -sSL https://git.io/v7Hch | bash`

# To test logcheck new rules

this will send an email

```
sudo -u logcheck logcheck -t -d
```

Note:

* You cannot access `live` via `vagrant ssh live` because of disabled password auth. You need to go through `vagrant ssh ansible` first, and then do `ssh live`
* `sudo env EDITOR=nano ansible-vault edit group_vars/vault.yml`

# Perfect Linux machine setup with Ansible

In 2020, having a Linux IaaS machine configured and setup should be easy and fast: after all, it's the basic task anybody has to do in order to start working on more-meaningful and productive activities.

However, this is not always the case: you work in different environments (different cloud providers, on-premises environments, lab environments) where you have different base images and different tools, and creating the very same machine is still quite a daunting - or at least *time-consuming* - task.

This project aims to solve that, by providing a **boilerplate yet complete** and **easy-to-use** and **easy-to-customize** way to spin up your own Linux machine.

## Usage

You will need [Ansible](https://ansible.com) on your workstation in order to run this project, and you will need to first install a base Linux image on your target system. The target system can be a Debian-based system or an Ubuntu-based system, since the [apt](https://it.wikipedia.org/wiki/Advanced_Packaging_Tool) package is used.

You can start the fun with the [install.sh](/install.sh) script which takes in input the hostname to configure.

In the output, you will receive the passwords for the default user and the `root` user.
The script will add your local SSH keys as trusted keys for remote login, and will harden the Linux installation quite a bit.
In the end, you will also have several utilities installed and a Docker daemon.

The playbook also uses the [olivomarco/dotfiles](https://github.com/olivomarco/dotfiles) repo in order to setup a custom and productive shell for the users of the system.

## Parameters

Please note that some of the parameters provided by this template repo **MUST BE CHANGED** before running the script.

Customization starts by modifying files in [this folder](/group_vars/). In particular, consider that:

- in [all.yml](/group_vars/all/all.yml) you must modify the `default_username` variable
- in [all.yml](/group_vars/all/all.yml) you must modify the `dot_forward_email` variable
- the [vault.yml](/group_vars/prod/vault.yml) is an [ansible-vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) encrypted file (default password: `secretpassword`) which you must customize with your SMTP parameters

## Final notes

You can, of course, customize everything and add your own tools and configurations: **if you think you have found something useful which can be beneficial to any user using this package, please open a feature request and I will gladly evaluate it**.
