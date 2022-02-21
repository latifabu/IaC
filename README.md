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

![Ansible diagram](https://user-images.githubusercontent.com/98215575/154823142-7e8d8d77-1657-43a2-9e73-13b2e48dec26.png)





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
-  `sudo nano hosts` anf enter the below to gain ssh access through the controller.
  ```

  [web] 
  192.168.56.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant 
  [db] \n192.168.56.11 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant" | sudo tee --append /etc/ansible/hosts
  ```

  If that does not work enter into hosts or the terminal:
  ```
  echo -e "[web] \n192.168.56.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant \n[db] \n192.168.56.11 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant \n\n" | sudo tee --append /etc/ansible/hosts
  ```
- Enter `sudo ansible web -m ping` and `sudo ansible db -m ping` 
- Or `sudo ansible all -m ping` 
- Create `README.md`
- Enter `ansible web -m ansible.builtin.copy -a "src=/etc/ansible/README.md dest=/home/vagrant/README.md"`
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
## Ansible playbooks

What are ansible playbooks?
YAML/yml files with script to implment config managment
WHy use them?
- Save time and they are re usable
- Not many changes are needed when using a playbook
- If we use the playbook on the cloud instead of local host all we need to do is change the ip
- how to create a playbook <file>.yml/yaml
- Interprester knows the YAML/yml `---`
- In python we use import pytest
- In ansilbe we can import file.yml
- Run playbooks with `ansible-playbook <filename>.yml` and add `-v` to get additional logs

## Using playbooks within our controller VM to control/send commands to our `web` and `db` VMs.

### Install nginx on web using this playbook:
- Wiithin our controller VM run the following:
- `sudo nano install_nginx.yml`:
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
### Change vagrant file
- By syncing the app file in our current dir to our controller VM by adding :
- `controller.vm.synced_folder "./app", "/home/vagrant/app"`
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

       controller.vm.synced_folder "./app", "/home/vagrant/app" 
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

### Copy app to web, Install nodejs version 6 and npm with using this playbook:
- `install_app_depenedencies.yml`
```
---
# web vm
- hosts: web

# gather logs
  gather_facts: yes

# give admin access
  become: true

# install dependencies
  tasks:
  - name: Copy app folder from local to web node
    synchronize:
      src: /home/vagrant/app/
      dest: /home/vagrant/

  - name: Add nodejs apt key
    apt_key:
      url: https://deb.nodesource.com/gpgkey/nodesource.gpg.key
      state: present

  - name: add nodejs 6.x version
    apt_repository:
      repo: deb https://deb.nodesource.com/node_6.x bionic main
      update_cache: yes

  - name: Install nodejs
    apt: pkg=nodejs state=present

  - name: Install npm
    apt: pkg=npm state=present
```
### Run update and upgrades on all VMs controller by ansible with
- file name `update_upgrade.yml`
```
---
- hosts: all
  become: yes
  tasks:
  - name: Update and upgrade apt packages
    apt:
      upgrade: yes
      update_cache: yes

```

### Install mongodb on db:
- `install_mongod.yml`:
```
 Install mongo in db VM
---
- hosts: db

# Gather facts
  gather_facts: yes
# admin access
  become: true

# install mongo
  tasks:
  - name: Installing mongodb in db vm
    apt: pkg=mongodb state=present
```
### Alternatively 
- `setting_up_mongodb.yml` :
- Install mongodb, edit mongodb.conf so the bind ip is `0.0.0.0` so all networks have access to mongodb with:
  ```
    ---
  - hosts: db
    gather_facts: yes
    become: true
    tasks:
    - name: Installing mongo 
    apt: pkg=mongodb state=present
    - name: allow 0.0.0.0
      ansible.builtin.lineinfile:
        path: /etc/mongodb.conf
        regexp: '^bind_ip = '
        insertafter: '^#bind_ip = '
        line: bind_ip = 0.0.0.0
    - name: Resart and enable mongodb
      service: name=mongodb state=restarted enabled=yes
    
  ```
### Install mongodb pre-requistes with
- `mongodb_pre_req.ywl`:
```
---
-  hosts: db
   gather_facts: yes
   become: true
   tasks:
   - name: installing mongo pre-requisites
     shell: sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927; echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodbs

   - name: restart mongod
     service: name=mongodb state=restarted

   - name: mongod enable
     service: name=mongodb enabled=yes
```
### Add .bashrc file to the web db with
- This will allow us to access web to access db
- `source_bashrc.yml`
```
---
- hosts: web

  gather_facts: yes

  become: true
  tasks:
    - name: sourcing .bashrc
      shell: source /home/vagrant/.bashrc
      args:
        executable: /bin/bash
```
### Now create add the environmental variable, DB_HOST to .bashrc

```
---
-  hosts: web
   gather_facts: yes
   become: yes
   tasks:
     - name: Insert line at end of file
       lineinfile:
         path: /home/vagrant/.bashrc
         line: export DB_HOST='mongodb://192.168.33.11:27017/posts
```
### Add reverse proxy to web app:
- Create a file called `default` in working dir outside of our VMs:
- Within the default file add the code for reverse proxy and hide port 3000
  
```
server {
    listen 80;



    server_name _;



    location / {

        proxy_pass http://localhost:3000;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;

        proxy_set_header Connection 'upgrade';

        proxy_set_header Host $host;

        proxy_cache_bypass $http_upgrade;

    }

}

```
- Create file name `reverse_proxy.yml`:

```
---
# reverse proxy
- hosts: web

  gather_facts: yes

  become: yes

  tasks:
   - name: reverse proxy
     synchronize:
       src: /home/vagrant/app/default
       dest: /etc/nginx/sites-available/default

   - name: restart nginx
     service: name=nginx state=restarted

   - name: enable nginx
     service: name=nginx enabled=yes
```
- You may need to restart mongodb so run `setting_up_mongodb` again.
- `ssh` into web VM.
- `cd app`
- Enter `node seeds/seed.js`
- Enter `npm start`
- App should be running on `192.168.33.10` along with `/posts`.
By running the above playbooks from our controller VM we can send multiple commands to the other VMs all.

## Ansible hybrid cloud deployment
- Create or use a new VM
- Inside VM run update upgrade commands:
- `sudo apt-get update -y && sudo apt-get upgrade -y`
- 
- Install tree (Optional) with ` sudo apt-get install tree`
- This commands downloads folder for ansible and the specific folder needed: `sudo apt-add-repository --yes --update ppa:ansible/ansible`
- Install ansible: `sudo apt-get install ansible -y`
- Install python and pip: `sudo apt-get install python3-pip`
- Using pip we can install awscli and boto3 
- Install awscii `pip3 install awscli`
- Install boto3 `pip3 install boto boto3`
- Run update and upgrade commands to check if anything does need to be updated.
- Check if the right versions of the above dependencies have been installed with: `aws --version`'
 If the following appears, open a new git bash terminal as admin:
 
 ![image](https://user-images.githubusercontent.com/98215575/155037923-694d1655-6c4d-488b-bc8f-be95c8300738.png)

- `cd /etc/ansible`
- Folder structure for vault:
 
- `/etc/ansible/group_vars/all/file.yml` group and all folders are not created yet.
- Use `sudo mkdir group_vars` to create `group_vars` folder
- sudo `mkdir all`
- Inside `all` folder create this file
- `sudo ansible-vault create pass.yml`
- Enter password if prompted
- Inside the file add
```
aws_access_key: YOURACCESSKEY
aws_secret_key: YOURSECRETKEY
```
- Type `ii` to insert and then edit file
- To save press `esc` and type `:wq!`
- File structure should be the follwing:
  ![tree structure ansible](https://user-images.githubusercontent.com/98215575/155037587-e54be4b7-a93b-4852-b3c7-78b0d839e6f9.png)
 
- To not save type :q!
- `sudo cat <file>.yml` print an encrypted key
- `sudo chmod 600 pass.yml` give the file permission to be read
- `--ask-vault-pass` at the end of any command when
- `sudo ansible all -m ping --ask-vault-pass` here we are checking if it is working and it is verifying the keys we have entered

We have now created a secure authentication method using ansible for hybrid cloud.
- Enter `cd` and come out or currend `pwd`
- Enter `cd ~/.ssh`
- Run `ssh-keygen -t rsa -b 4096` and create public and private keys.
AWS credentials that were encrypted, so we did not need to expose the keys at any stage. 
- The SSH key pair (pub and private) is used to authenticate the identity of a user or process that wants to access a remote system using the SSH protocol. 
- The public key is used by both the user and the remote server to encrypt messages. 
- On the remote server side, it is saved in a file that contains a list of all authorized public keys.

How did you secure the data on prem? 
- vagrant password is needed to enter VMs
- On cloud AWS credentials are encypted on transit between ansible and cloud is secure.
- IaC is codify everything written what we have as code.

Launching EC2 instance using playbook
- Now `cd /etc/ansible` 
- Create playbook called `ec2.yml`

```
---
- hosts: localhost
  connection: local
  gather_facts: yes
  vars_files:
  - /etc/ansible/group_vars/all/pass.yml
  vars:
    key_name: <private_key_create_above> # private key just created
    region: eu-west-1
    image: ami-07d8796a2b0f8d29c # Ubuntu AMI
    id: "<my-name>-ansible5"
    sec_group: "{{ id }}-sec"
  tasks:
    - name: Provisioning EC2 instances
      block:
      - name: Upload public key to AWS
        ec2_key:
          name: "{{ key_name }}"                          # public key just created
          key_material: "{{ lookup('file', '/home/vagrant/.ssh/<public_key>') }}"
          region: "{{ region }}"
          aws_access_key: "{{aws_access_key}}"
          aws_secret_key: "{{aws_secret_key}}"
      - name: Create security group
        ec2_group:
          name: "{{ sec_group }}"
          description: "Sec group for app {{ id }}"
          region: "{{ region }}"
          aws_access_key: "{{aws_access_key}}"
          aws_secret_key: "{{aws_secret_key}}"
          rules:
            - proto: tcp
              ports:
                - 22
              cidr_ip: 0.0.0.0/0
              rule_desc: allow all on ssh port
            - proto: tcp
              ports:
                - 80
              cidr_ip: 0.0.0.0/0
              rule_desc: allow all on http port
        register: result_sec_group
      - name: Provision instance(s)
        ec2:
          aws_access_key: "{{aws_access_key}}"
          aws_secret_key: "{{aws_secret_key}}"
          key_name: "{{ key_name }}"
          id: "{{ id }}"
          group_id: "{{ result_sec_group.group_id }}"
          image: "{{ image }}"
          instance_type: t2.micro
          region: "{{ region }}"
          wait: true
          count: 1
          instance_tags:
            name: eng103a_<my_name>_ansible
      tags: ['never', 'create_ec2']

```
- Check if pwd is `/etc/ansible` 
- Enter to launch instance and use python3 as interpreter:
```
sudo ansible-playbook ec2.yml --ask-vault-pass --tags create_ec2 --tags=ec2-create -e "ansible_python_interpreter=/usr/bin/python3" 
```
- Edit hosts file and add the IP of the instance just created.
- 
```
[aws]
<instance_ip> ssh_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=/home/vagrant/.ssh/<private_key_name>
```
- Test connection with: `sudo ansible aws -m ping --ask-vault-pass`
- Install nginx with a playbook called `install_nginx`:
- `sudo nano install_nginx`
- Enter:
```
---
- hosts: aws # Name of host in host file is <aws>
  gather_facts: yes
  become: true
  tasks:
  - name: Installing Nginx web-sever
    apt: pkg=nginx state=present
```
- Enter: `sudo ansible-playbook install_nginx.yml --ask-vault-pass`
- Nginx will be running on aws instance.
