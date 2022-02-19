# IAC with Ansible


### Let's create Vagrantfile to create Three VMs for Ansible architecture
#### Ansible controller and Ansible agents 

```

# -*- mode: ruby -*-
 # vi: set ft=ruby :
 
 # All Vagrant configuration is done below. The "2" in Vagrant.configure
 # configures the configuration version (we support older styles for
 # backwards compatibility). Please don't change it unless you know what
 
 # MULTI SERVER/VMs environment 
 #
 Vagrant.configure("2") do |config|
 # creating are Ansible controller
   config.vm.define "controller" do |controller|
     
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
    # config.hostsupdater.aliases = ["development.controller"] 
    
   end 
 # creating first VM called web  
   config.vm.define "web" do |web|
     
     web.vm.box = "bento/ubuntu-18.04"
    # downloading ubuntu 18.04 image
 
     web.vm.hostname = 'web'
     # assigning host name to the VM
     
     web.vm.network :private_network, ip: "192.168.33.10"
     #   assigning private IP
     
     #config.hostsupdater.aliases = ["development.web"]
     # creating a link called development.web so we can access web page with this link instread of an IP   
         
   end
   
 # creating second VM called db
   config.vm.define "db" do |db|
     
     db.vm.box = "bento/ubuntu-18.04"
     
     db.vm.hostname = 'db'
     
     db.vm.network :private_network, ip: "192.168.33.11"
     
     #config.hostsupdater.aliases = ["development.db"]     
   end
 
 
 end
```
# IaC
![Ansible diagram](https://user-images.githubusercontent.com/98215575/154822757-7241e846-742e-49f6-9875-8d7a52edac92.png)




- We need a different infrastructure. So we use infrastructure as code, can launch to many machines not just one conf like provision.db
- Any file that has an extension of YAML/YML – YET ANOTHER MARKUP LANGUAGE 
- YAML/YML can be used on multiple platforms : can be used on aws, etc
- YAML/TML is useful because it is reusable thus saving us time
What is Ansible?
- An automation code: a configuration management tool, we can config everything regardless of where the machine is, on prem, hybrid, public or multi-cloud
- It can talk to all machines if the description is right, can connect to 100s of servers with one file and do different tasks.
- Aws has special services, management, can go serverless, like automated driving. Like a car without a driver a serverless services can run by itself. 
- In serverless, you put the instructions in, and it does everything on its own 

Setting up Ansible Controller
Steps we will make:
- get upgrade
- get update
- install dependencies
- install ansible
- Use tree (allows us to easily see what is within dir )
- set up agent nodes
- default folder strcutures/ etc/ ansible
- hosts file - agent node called web ip of the web.

Preprations steps:
- Enter terminal on local host
- Make sure you're not in any file and do no create any
- Clone repo
- Remove `.git`
- Create repo and follow the steps
- Push to the newly created repo
  
Step 1) Setting up vagrant VMs
- Enter terminal
- cd to where clone repo is
- Enter `vagrant up`
- Check `vagrant status`
- Run update and upgrade commands for each VM 
- SSH into controller first 
- Enter the standard commands `sudo apt-get update -y` and `sudo apt-get upgrade -y`
- ssh into `vagrant web` and `vagrant db` and run the update and upgrade commands

Step 2) Ansible controller and Ansible agents 
- ssh into `vagrant controller`
- Enter `sudo apt-get install software-properties-common`
- Next `sudo apt-add-repository ppa:ansible/ansible`
- `sudo apt-get install ansible` or `sudo apt-get install ansible -y` to save time
- Check ansible version is 2.9.27 with `ansible --version`. Python version 2.7.17 should also be visible
- Now `cd /etc`
- `cd ansible/` then ls to see : ansible.cfg, hosts and roles
- `sudo apt install tree` : the same files above should be visible in a tree like format
- **Now this is key**
- While in `/etc/ansible` enter `ssh vagrant@192.168.33.10` (The ip of the web VM. Can be found in Vagrantfile. Make the IP is entered correctly )
- Enter password `vagrant`
- After entering `web` VM via run `sudo apt-get update -y` to see if all packages necessary are there.
- Exit and return to controller VM.
- Enter db  VM by entering `ssh vagrant@192.168.33.10` and again enter `sudo apt-get update -y`. We ensure internet connection by running these commands
- Exit and return to `/etc/ansible`
Step 3) Accessing the other VMS through controller only
- If we run ping in `web` or `db` by entering `ping <ip>` it will ping continously
- Enter and change hosts `sudo nano hosts`


  ```
  
  [web] 
  192.168.56.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant 
  [db] \n192.168.56.11 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant" | sudo tee --append /etc/ansible/hosts
  ```

  If that does not work enter into the terminal
  ```
  echo -e "[web] \n192.168.56.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant \n[db] \n192.168.56.11 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant \n\n" | sudo tee --append /etc/ansible/hosts
  ```
- Enter `sudo ansible web -m ping` and `sudo ansible db -m ping` 
- Or `sudo ansible all -m ping` 
- Create `README.md`
- Enter `ansible web -m ansible.builtin.copy -a "src=/etc/ansible/README.md dest=/home/vagrant/README.md"`
- 
- Executing command in all instances ansible all -a "SOME COMMAND"
  
Ad hoc commands for ansible
They allow us to interact with all agents from controller.

Useful commands:

```
sudo ansible all/web/db -m ping
sudo ansible all/web/db -a "uname -a"
sudo ansible db -a "ls-a"
sudo ansible web -a "free" # view memory
ansible all -m copy -a "src=/etc/ansible/README.md dest=/home/vagrant/README.md" # copy files from one location to another
```
Ansible playbooks

What are ansible playbooks?
YAML/yml files with script to implment config managment
WHy use them?
Save time and they are re usable
- not many changes are needed when using a playbook
- if we use the playbook on the cloud instead of local host all we need to do is change the ip
- how to create a playbook <file>.yml/yaml
- Interprester knows the YAML/yml `---`
- IN python we use import pytest
- In ansilbe we can import file.yml

Ansible playbooks
Install nginx on web using this playbook:

```
# YAML/YML file to create a playbook to configure nginx in our web instance

---
# Starts with three dashes

# add the name of the host/instance/VM
- hosts: web

# collect logs or gather facts -
  gather_facts: yes

# need admin access to install anything
  become: true

# add the instructions - install nginx - in web server
  tasks:
  - name: Installing Nginx web-sever in our app machine
    apt: pkg=nginx state=present


# HINT: be mindful of indentation
# use 2 spaces - avoid using tab
```
Install nodejs using this playbook
```
#Yml file to create a playbook to set up nodejs
---
- hosts: web

  gather_facts: yes

  become: true

  tasks:
  - name: load a specific version of nodejs
    shell: curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

  - name: install the required packages
    apt:
      pkg:
        - nginx
        - nodejs
        - npm
      update_cache: yes
  - name: install and run the app
    shell:
       cd app/app; npm install; screen -d -m npm start
```
