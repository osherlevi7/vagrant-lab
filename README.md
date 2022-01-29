# vegrant-lab
Ansible using vegrant as a docker cluster
</br>


# First Step 
### Download VirtualBox - 
> https://www.virtualbox.org/wiki/Downloads
</br>

### Download Vagrant - 
> https://www.vagrantup.com/downloads
</br>

### vagrant cheatsheet - 
> https://gist.github.com/osherlevi7/9676dee6bc8837b0e2d939a1a0a54bd2
</br>


# Manaaging the servers using Ansible
### init the VMs from the project dir 
    $ vagrant up 
</br>

### ssh to the master server 
    $ vagrant ssh control
</br>

### Set DNS between master and the nodes 
    < use the host file on this repo for this mission >
     $ sudo cp /vagrant/hosts /etc/hosts

    < ping node1, node2, node3 to test the name resolution>
     $ ping node1
</br>

### install ansible & sshpass on the control server 
    $ sudo apt-get install ansible -y
    $ sudo apt-get install sshpass
</br>

### make hosts SSH accessible
    $ ssh-keygen
    $ ssh-copy-id node1 && ssh-copy-id node2 && ssh-copy-id node3
</br>

### test ansible
    $ cd /vagrant/ansible
    $ ansible nodes -i myhosts -m command -a hostname
</br>

### install python with plybook on myhosts inventory file
    $ ansible nodes -i myhosts -m command -a 'sudo apt-get -y install python-simplejson'
</br>

### Run playbook docker-install on myhosts inventory
    $ ansible-playbook -i myhosts -k docker-install.yml
</br>

    


# Containerizing application with Docker 
### build and tag you docker image with docker hub 
    inside your docker dir run - 
    $ docker build -t <your_username>/my-private-repo .
    push it to your DockerHub - 
    $ docker push <your_username>/my-private-repo
</br>

### login to node1 to make sure docker is installed properlly
    $ docker run hello-world
    < it will pull docker image of 'Hello World>
* in case you made a mistake in one of the file, re-build the docker image with the following command - 
    $ docker-compose build
</br>



# Orchestrating application with SWARM

### go to Control server /vagrant dir
    $ git clone git clone https://github.com/devopsjourney1/ansible-swarm-playbook
</br>



### create a new inventory file inside the cloning repository
    $ cp vagrant@node1:/home/vagarent/ansible/myhosts .
    
</br>

### add manager and worker to the hosts file for the swarm orchesration
    $ vim /vagarent/ansible-swarm-playbook/myhosts

    [manager]
    node1

    [worker]
    node2
    node3

    $ vim swarm.yml 

    *Note I had to change *eth0 to *eth1 in this, since ip address didn't match.
</br>

## Verifaid the docker swarm setup 

ssh to the node1 and run the command -
    $ sudo docker node ls

    ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
    z03ohmerd1pbobhobfe3yz7yb *   node1      Ready     Active         Leader           20.10.7
    lo8kp82drv51oadc6cqkygqk8     node2      Ready     Active                          20.10.7
    02gzlyiot8b9zfebgfl0a4e6m     node3      Ready     Active                          20.10.7

</br>


