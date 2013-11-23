salt-node-docker-cluster
========================

Vagrant + Salt + Node + Docker = Cluster yes.

## What is this?

    This project revolves around a vagrant file (and VirtualBox) for testing out the 
    deployment of a multinode saltstack based cluster.  Node.JS and Docker 
    (with a salt-minion nodejs ubuntu image) are all included and built 
    during the "vagrant up" command. 
   

    The vagrant file deploys a salt master, and three salt minions.  All four 
    virtual machines install docker with a salt-minion nodejs docker image. 
    Whenever this image is used to create containers, it will automatically 
    communicate with the master and exchange keys.   This means all 
    spawned linux containers run salt-minion which opens up infinite possibilities 
    for rapid deployment of linux environments

## Requirements

    By default, the ubuntu cloud image for VirtualBox / Vagrant is set to 1 CPU and 512 MB of RAM. 
    Keep this in mind when deploying processes to docker containers.  If more RAM is needed
    update the Vagrantfile and vagrant reload.

## Installation and Running the cluster

##### Description

    This cluster runs on a private vbox network on the 192.168.200 subnet.
    Salt server runs on 192.168.200.5 and the minions are 192.168.200.[6,7,8].
    
##### Cmds   
    $ git clone https://github.com/PortalGNU/salt-node-docker-cluster.git
    $ cd salt-node-docker-cluster
    $ vagrant up
    $ vagrant ssh [salt, minion-01, minion-02, minion-03] 
    
## Spawning minions to do your bidding

##### Start a salted node container

    Here we are logging into one of our minion docker servers and spawning
    a minion docker container for a salted node env.  We are assigning
    the docker a useful name (node) so that we can filter it out in our 
    salt-master state configs.
    
    Note:  In order for memory allocation to actually happen, there must
    be swap space allocated on the minion server.  

##### Cmds

    $ vagrant ssh minion-[01, 02, 03]
    $ sudo docker run -d -name node -h node -m 279969792 -v /vagrant:/vagrant -p 3000 nodebuntu
    $ exit
    

#### Accept and list our minions

    $ vagrant ssh salt
    $ sudo salt-key -A
    $ sudo salt-key -L 
    
## Verify our node container minion is online

##### Description
    We will use the active module and state scripts for dockerio in this project.
    Included in the srv folder are _modules and _states subfolders with the dockerio
    py files.
    
##### Cmds
    
    $ vagrant ssh salt
    $ sudo salt 'minion-*' docker.get_containers all=false
 
##### Output:
    minion-01:
        ----------
        comment:
            All containers in out
        id:
            None
        out:
            ----------
            - Command:
                /usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf
            - Created:
                1385235040
            - Id:
                f99a4e4730d6ef9d00f5369aa11af386b8fc7d538faea196953815684ca1a551
            - Image:
                nodebuntu:latest
            - Names:
                - /node
            - Ports:
                ----------
                - IP:
                    0.0.0.0
                - PrivatePort:
                    3000
                - PublicPort:
                    49155
                - Type:
                    tcp
            - SizeRootFs:
                0
            - SizeRw:
                0
            - Status:
                Up 3 hours
        status:
            True
    minion-02:
        ----------
        comment:
            We did not get any expectable answer from docker
        id:
            None
        out:
            None
        status:
            False
    minion-03:
        ----------
        comment:
            We did not get any expectable answer from docker
        id:
            None
        out:
            None
        status:
            False 
    
## Make demands to our node container minion

#### Description

    For a quick example of states, we'll double check nodejs (and deps)
    installed into the container during our docker image build.
    
    This project includes a srv folder that is copied to /srv on the master.
    Please see these files for config details.

##### Cmds

    $  sudo salt 'node' state.highstate
    

    node:
    ----------
        State: - npm
        Name:      coffee-script
        Function:  installed
            Result:    True
            Comment:   Package coffee-script satisfied by coffee-script@1.6.3
            Changes:   
    ----------
        State: - npm
        Name:      bower
        Function:  installed
            Result:    True
            Comment:   Package bower satisfied by bower@1.2.7
            Changes:   
    ----------
        State: - npm
        Name:      express
        Function:  installed
            Result:    True
            Comment:   Package express satisfied by express@3.4.4
            Changes:   
    ----------
        State: - pkg
        Name:      python
        Function:  installed
            Result:    True
            Comment:   Package python is already installed
            Changes:   
    
    Summary
    ------------
    Succeeded: 4
    Failed:    0
    ------------
    Total:     4
