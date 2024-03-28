# Ansible Set-Up Documentation on Windows using Vagrant

#### Install Vagrant : https://developer.hashicorp.com/vagrant/downloads 

#### Pre-Req : Vagrant needs at least one Hyper-Visor (Oracle VirtualBox : https://www.virtualbox.org/wiki/Downloads)

<br> Vagrantfile </br>

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  
  # Will not check for box updates during every startup.
  config.vm.box_check_update = false


  # Master node where ansible will be installed
  config.vm.define "controller" do |controller|
    controller.vm.box = "geerlingguy/ubuntu2004"
    controller.vm.hostname = "controller.anslab.com"
    controller.vm.network "private_network", ip: "192.168.10.3"
    controller.vm.provider "virtualbox" do |vb|
      vb.memory = 1048
      vb.cpus = 1
    end
  end

  # Managed node 1.
  config.vm.define "server1" do |server1|
    server1.vm.box = "geerlingguy/ubuntu2004"
    server1.vm.hostname = "server1.anslab.com"
    server1.vm.network "private_network", ip: "192.168.10.4"
    server1.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
      vb.cpus = 1
    end
  end

end

```
<br>Instructions to spin up VMs </br>
```
# Run this command from the folder where Vagrantfile is present
vagrant up
vagrant status
# Once both the VMs are running fine, get into the controller VM
vagrant ssh controller
```

<br> Instructions to Install Ansible in Ubuntu VM </br>
```
# Run these commands inside the 'controller' VM
sudo apt-get update -y
sudo apt-add-repository -y ppa:ansible/ansible
sudo apt-get update -y
sudo apt-get install ansible
ansible --version

# Let us try some adhoc commands against localhost
ansible localhost -m "ping"
ansible localhost -a "hostname"

```
<br> Ansible configurations to run play-books </br>

```
# Auto-generate /etc/ansible/ansible.cfg
# You would need root privilege to create & modify ansible.cfg
sudo su
cd /etc/ansible
ansible-config init --disabled > ansible.cfg

# In order to disable strict key checking, modify the following flag in ansible.cfg
vi ansible.cfg
# Search for the following line and edit it as follows
host_key_checking=False

```

<br> Let us make an entry in /etc/ansible/hosts file to add details about our 'ubuntu1' VM so that we can control it </br>

```
# Need root privilege for this
sudo su
cd /etc/ansible
vi hosts

# Append the following at the end of the file
[server1]
192.168.10.4

[server1:vars]
ansible_connection=ssh
ansible_user=vagrant
ansible_ssh_pass=vagrant

```

<br> All settings are now done, let us try running some ad-hoc commands against the server ubuntu1

```
# You can run these commands from any location and you do not need root privilege
# -vvvv is for printing verbose information
ansible server1 -m ping -vvvv
ansible server1 -a "hostname"
ansible server1 -a "date"
ansible server1 -a "whoami"

# Let us copy a file from our controller to server1
# First, create a file in the home folder - hello.txt - with some contents

echo "Hello World" >> hello.txt
ansible server1 -m copy -a "src=~/hello.txt dest=~"

# Let us check if this file has been copied onto server1 (inside home folder)
ansible server1 -m stat -a "path=~/hello.txt"

# Create a group
ansible server1 -b -m group -a "name=accenture state=present"
# Add a user to this group
ansible server1 -b -m user -a "name=anand group=accenture createhome=yes"

```
