
# How to create a Docker swarm (mode) cluster setup.

### Requirements
I want to walk you through the process of enabling a Docker swarm manager and then joining two hosts as nodes

Three local/cloud machine with Ubuntu Server 16.04 and above installed. Out of which once server will act as a Manager node and two servers will act as Worker node.

A static IP address is configured on all the machine. Here, we will use IP address.

Details are below.
```
192.168.0.100 for Manager node.
192.168.0.101 for Worker node01.
192.168.0.102 for Worker node02.
```
Each node must also be running Docker. I have use docker version 18.05.0-ce.


### First we install a docker.

The installation process is well documented elsewhere and you can read Docker’s documentation if you’d like the play by play. The simplest method is to run the following commands on each machine:
```
$ curl -sSL https://get.docker.com | sh
```
This will install the latest version of Docker engine.

### Creating your Swarm Manager
Creating the swarm on your manager can be done with a single command. Remember, our manager is at host 192.168.0.100, so the command to initialize the manager is:
```
$ sudo docker swarm init --advertise-addr 192.168.0.100
```

You should see the following output:
```
Swarm initialized: current node (viwovkb0bk0kxlk98r78apopo) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3793hvb71g0a6ubkgq8zgk9w99hlusajtmj5aqr3n2wrhzzf8z-1s38lymnir13hhso1qxt5pqru 192.168.0.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
Note: Remember the token from the above output. This will be used to join worker nodes to the manager node later.


#### You can also see the list of nodes in your cluster with the following command:
```
$ sudo docker node ls
```

Output:
```
ID                             HOSTNAME         STATUS     AVAILABILITY        MANAGER STATUS
viwovkb0bk0kxlk98r78apopo  *   ubuntu-16.04     Ready      Active              Leader
```

### Join the Worker nodes to the Swarm Manager.
Manager node is now ready. Next, you will need to add Worker node to the Manager node.

You can do this by running docker swarm join command on both Worker node as follows:
```
$ sudo docker swarm join --token SWMTKN-1-3793hvb71g0a6ubkgq8zgk9w99hlusajtmj5aqr3n2wrhzzf8z-1s38lymnir13hhso1qxt5pqru 192.168.0.100:2377
```
Output:
```
This node joined a swarm as a worker.
```

On the manager node, run the following command to check the node status, whether the nodes are active or not:
```
$ sudo docker node ls
```
If everything went fine, you should see the following output:
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
viwovkb0bk0kxlk98r78apopo *   managernode         Ready               Active              Leader
yf6nb2er69pydlp6drijdfmwd     workernode1         Ready               Active
yyavdslji7ovmw5fd3v7l62g8     workernode2         Ready               Active
```

## Deploying your first nginx service
With the active nodes, we can now deploy a service. Go back to the Docker swarm manager and let's create an nginx webserver service with the command:
```
$ sudo docker service create -p 80:80 --name webserver --replica 3 nginx
```
Output:
```
bky6uhg2agdxeqgc2b1a5tcsm
overall progress: 3 out of 3 tasks
1/3: running   [==================================================>]
2/3: running   [==================================================>]
3/3: running   [==================================================>]
verify: Service converged
```
The above command will create a service with name webserver and containers will be launched from docker image "nginx". containers are deployed across the cluster nodes such as, Manager node, Worker node1 and node2.

Now, you can list and check the status of the service with the following command:
```
$ sudo docker service ls
```
Output:
```
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
bky6uhg2agdx        webservice          replicated          3/3                 httpd:latest        *:80->80/tcp
```

#### Now verify the Service from Manager node and see whether a new container is started or not:
```
$ sudo docker service ps webservice
```
Output:
```
ID                  NAME            IMAGE               NODE                DESIRED STATE       CURRENT STATE                  ERROR           PORTS
xsa5wb0eg2ln        webserver.1     httpd:latest        managernode         Running             Running about 10 minutes ago                                       
3sbscs7m9lnh        webserver.2     httpd:latest        workernode2         Running             Running about 10 minutes ago                                      
yao2m5wi54kz        webserver.3     httpd:latest        workernode2         Running             Running about 10 minutes ago                                 
```
#### At the moment, our webserver service is only running on one of our Docker machines. If we have the manager and two other nodes, we can scale the service up for our swarm (currently 3 machines), with the following command:
```
$ sudo docker service scale webserver=5
```

#### Again now verify the Service from Manager node and see whether a new container is started or not:
```
$ sudo docker service ps webserver
```
Output:
```
ID                  NAME               IMAGE               NODE                DESIRED STATE       CURRENT STATE       ERROR           PORTS
xsa5wb0eg2ln        webserver.1        httpd:latest        managernode         Running             Running about 10 minutes ago                                       
3sbscs7m9lnh        webserver.2        httpd:latest        workernode2         Running             Running about 10 minutes ago                                      
yao2m5wi54kz        webserver.3        httpd:latest        workernode2         Running             Running about 10 minutes ago                                 
dfg2mswa52sf        webserver.4        httpd:latest        workernode1         Running             Running about 15 seconds ago                                 
kah1j5hs14as        webserver.5        httpd:latest        workernode1         Running             Running about 15 seconds ago                                 
```

You may also use yaml file for deploying your service and save with nginx.yml

like:-
```
version: '3.3'

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
```

you can deploy by this command
```
$ sudo docker stack deploy -c nginx.yml nginx
```

#### Clustering made easy
That's the gist of creating a Docker swarm and creating a service on your new cluster. To learn more about what Docker swarm can do, issue the command docker swarm —help to see the other commands you can use in conjunction with Docker swarm.

