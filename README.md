# Infrastructure as Code with Ansible



### Ansible Controller and Agent nodes set up with Vagrant
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what

# MULTI SERVER/VMs environment
#
Vagrant.configure("2") do |config|

# creating first VM called web
  config.vm.define "web" do |web|

    web.vm.box = "bento/ubuntu-18.04"
   # downloading ubuntu 18.04 image

    web.vm.hostname = 'web'
    # assigning host name to the VM

    web.vm.network :private_network, ip: "192.168.33.10"
    #   assigning private IP

    config.hostsupdater.aliases = ["development.web"]
    # creating a link called development.web so we can access web page with this link instread of an IP

    end

# creating second VM called db
  config.vm.define "db" do |db|

    db.vm.box = "bento/ubuntu-18.04"

    db.vm.hostname = 'db'

    db.vm.network :private_network, ip: "192.168.33.11"

    config.hostsupdater.aliases = ["development.db"]
    end

# creating are Ansible controller
  config.vm.define "controller" do |controller|
    
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
      
  end
end
```
#### set hosts for other machines inside ansible machine
```
[web]
192.168.33.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant

enter this code inside hosts file using `sudo nano ~/etc/ansible/hosts`

we can test all terminals associated with the ansible controller by running the command `ansible all -m ping`

for individual servers use `ansible web -m ping`
``` 
#### ansible ad-hoc commands
Date
- ansible db -a "date" 
Free memory
- ansible all -a "free -m" -i ansible_hosts
- ansible db -a "free"     
view files directory in specific machine
- ansible web -m shell -a "ls -a"
Uptime
- ansible all -m command -a uptime 
- ansible all -m shell -a uptime 
- ansible all -a uptime
Physical memory
- ansible all -m shell -a "cat /proc/meminfo|head -2" 
Execute a command as root user
- ansible multi -m shell -a "cat /etc/passwd|grep -i vagrant"

##### example of .YAML file

Command to run YML files ansible-playbook filename.yml

when creating a yaml file inside vm, use commad `sudo nano nignx.yml` 

An example of the type of script you would write is:
```
# define the agent nodes name host name

- hosts: web
# see logs
  gather_facts: yes

# sudo permission with become = true
  become: true

# install nginx on web server
  tasks:
  - name: Install nginx
    apt: pkg=nginx state=present
```
Make sure you pay attention to indentation as it makes a difference and could cause the file to not execute properly.

#### YML to install mongodb using ansible
```
---
# configuring mongodb

- hosts: db

  gather_facts: yes

  become: true

  tasks:
  - name: install mongodb
    apt: pkg=mongodb state=present
```

#### Hosts setup to connect VMS
```
This is the default ansible 'hosts' file.
[web]
192.168.33.10 ansible_user=vagrant ansible_ssh_pass=vagrant

[db]
192.168.33.11 ansible_user=vagrant ansible_ssh_pass=vagrant
```

It should live in /etc/ansible/hosts

 - Comments begin with the '#' character
 - Blank lines are ignored
 - Groups of hosts are delimited by [header] elements
 - You can enter hostnames or ip addresses
 - A hostname/ip can be a member of multiple groups

 command to disable `host_key_checking` incase of ssh pass error:
 ```
 sed -i '/#host_key_checking = False/c\host_key_checking = True' /etc/ansible/ansible.cfg
 ```
 