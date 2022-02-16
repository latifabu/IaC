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


![ansible](https://user-images.githubusercontent.com/98215575/154273691-b2b7853f-2609-4caf-9853-c7ac3a92bbbb.png)

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

- get upgrade
- get update
- install dependencies
- install ansible
- tree
- set up agent nodes
- default folder strcutures/ etc/ ansible
- hosts file - agent node called web ip of the web.
- 
