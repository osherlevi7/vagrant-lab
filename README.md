# vegrant-lab
Ansible using vegrant as a docker cluster


# First Step 
### Download VirtualBox - 
> https://www.virtualbox.org/wiki/Downloads

### Download Vagrant - 
> https://www.vagrantup.com/downloads
    # vagrant cheatsheet - https://gist.github.com/osherlevi7/9676dee6bc8837b0e2d939a1a0a54bd2


# Manaaging the servers using Ansible
### init the VMs from the project dir 
    $ vagrant up 

### ssh to the master server 
    $ vagrant ssh control

### Set DNS between master and the nodes 
    < use the host file on this repo for this mission >
     $ sudo cp /vagrant/hosts /etc/hosts

    < ping node1, node2, node3 to test the name resolution>
     $ ping node1

### install ansible on the control server
    $ sudo apt-get install ansible -y

### make hosts SSH accessible
    $ ssh-keygen
    $ ssh-copy-id node1 && ssh-copy-id node2 && ssh-copy-id node3

### test ansible
    $ ansible nodes -i myhosts -m command -a hostname