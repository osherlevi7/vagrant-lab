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


## deploy swarm service using my docker image on github
</br>

    deploy - 
    $ docker stack deploy --compose-file docker-compose.yml myapp
    ---------------------------------------------------------------------------------------------------------------------------------------

    ls your stack application - 
    $ docker stack ls 

    NAME      SERVICES   ORCHESTRATOR
    myapp     1          Swarm
    ---------------------------------------------------------------------------------------------------------------------------------------

    ls the docker service
    $ docker service ls

    ID             NAME        MODE         REPLICAS   IMAGE                           PORTS
    prgk0fsm5cyb   myapp_web   replicated   1/1        osherlevi7/vagrant-lab:latest   *:5000->5000/tcp
    ---------------------------------------------------------------------------------------------------------------------------------------

    get stack container info - 
    $ docker service ps myapp_web
    
    ID             NAME          IMAGE                           NODE      DESIRED STATE   CURRENT STATE           ERROR     PORTS
    lb62tfwnythn   myapp_web.1   osherlevi7/vagrant-lab:latest   node3     Running         Running 4 minutes ago     
    ---------------------------------------------------------------------------------------------------------------------------------------

    Scal up your services using swarm  -
    $ docker service scale myapp_web=<x>

    myapp_web scaled to 3
    overall progress: 1 out of 3 tasks 
    1/3: preparing [=================================>                 ] 
    2/3: preparing [=================================>                 ] 
    3/3: running   [==================================================>] ---------------------------------------------------------------------------------------------------------------------------------------

    Check your service ps - 
    $ docker service ps myapp_web
    
    ID             NAME          IMAGE                           NODE      DESIRED STATE   CURRENT STATE                ERROR     PORTS
    lb62tfwnythn   myapp_web.1   osherlevi7/vagrant-lab:latest   node3     Running         Running 13 minutes ago                 
    avx176m6ozwn   myapp_web.2   osherlevi7/vagrant-lab:latest   node2     Running         Running about a minute ago             
    2snxw610e4k6   myapp_web.3   osherlevi7/vagrant-lab:latest   node1     Running         Running about a minute ago  
    ---------------------------------------------------------------------------------------------------------------------------------------

    Scale the swrvice to 6 to get 2 service on each Vagrant node - 
    $ docker service scale myapp_web=6

    myapp_web scaled to 6
    overall progress: 6 out of 6 tasks 
    1/6: running   [==================================================>] 
    2/6: running   [==================================================>] 
    3/6: running   [==================================================>] 
    4/6: running   [==================================================>] 
    5/6: running   [==================================================>] 
    6/6: running   [==================================================>] 
    verify: Service converged 
    ---------------------------------------------------------------------------------------------------------------------------------------

    check that the new 3 services are registered - 
    $ docker service ps myapp_web

    ID             NAME          IMAGE                           NODE      DESIRED STATE   CURRENT STATE                ERROR     PORTS
    lb62tfwnythn   myapp_web.1   osherlevi7/vagrant-lab:latest   node3     Running         Running 16 minutes ago                 
    avx176m6ozwn   myapp_web.2   osherlevi7/vagrant-lab:latest   node2     Running         Running 4 minutes ago                  
    2snxw610e4k6   myapp_web.3   osherlevi7/vagrant-lab:latest   node1     Running         Running 5 minutes ago                  
    pbmc4yn73zxi   myapp_web.4   osherlevi7/vagrant-lab:latest   node2     Running         Running about a minute ago             
    m2fno54xxoxj   myapp_web.5   osherlevi7/vagrant-lab:latest   node3     Running         Running about a minute ago             
    pxlsqaww1fou   myapp_web.6   osherlevi7/vagrant-lab:latest   node1     Running         Running about a minute ago    
    ---------------------------------------------------------------------------------------------------------------------------------------

# TEST the swarm cluster

    Go to your vagrent control and check one node contain 2 services - 
    $ curl node1:5000

    Hello World! I am 69220cfc199f

    $ curl node1:5000

    Hello World! I am 419983bfc33e
    ---------------------------------------------------------------------------------------------------------------------------------------

    The same with node2 & node3 - 
    $ curl node2:5000

    Hello World! I am 69220cfc199f

    $ curl node2:5000

    Hello World! I am 419983bfc33e
    ---------------------------------------------------------------------------------------------------------------------------------------
    
    