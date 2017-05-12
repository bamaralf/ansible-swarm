# Swarm Cluster on AWS with Docker 

Quickly put, Swarm is now embedded in the Docker engine and provides some advanced networking features like load-balancing and virtual IPs for services.

The main playbook creates a cluster of Ubuntu 16.04 LTS machines on AWS.
Then you can init a Docker Swarm cluster.

## Prerequisites 

Docker-engine 1.12+

## Start the cluster

Check the content of the main play and run it.
It will create a SSH key pair and a swarm security group for you.

```
$ ansible-playbook swarm.yml
```

Check the inventory file and after a bit of time, you should have Docker running everywhere:

```
$ ansible swarm -m shell -a "sudo docker ps"
52.208.57.134 | SUCCESS | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

52.51.96.89 | SUCCESS | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

head | SUCCESS | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

52.30.66.217 | SUCCESS | rc=0 >>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## Create your Swarm

Initialize your Swarm head

```
$ ansible head -b -m shell -a "docker swarm init"
```

Join the nodes in your Swarm cluster, note that we only run this on the _nodes_ defined in the inventory.
Get the private IP of the head node from the inventory (because the security group is not wide open...)

```
$ ansible nodes -b -m shell -a "docker swarm join <PRIVATE_IP_HEAD>:2377"
```

Test it

```
$ ansible head -m shell -b -a "docker node list"
```

## Now use it

On the head node create a service

```
$ docker service create --name game --publish 80:80 --replicas 4 runseb/2048
2xia8q3b6io1nbb946g95zh00
```

You will see it listed

```
$ sudo docker service ls
ID            NAME  REPLICAS  IMAGE        COMMAND
2xia8q3b6io1  game  4/4       runseb/2048  
```

And a service is made of tasks that are individual containers.

```
$ docker service tasks game
ID                         NAME    SERVICE  IMAGE        LAST STATE              DESIRED STATE  NODE
4tceqzyflwiz0f88kxq9dlqsz  game.1  game     runseb/2048  Running About a minute  Running        ip-172-31-0-198
58grjto24nx5dqtqicp21mtbd  game.2  game     runseb/2048  Running About a minute  Running        ip-172-31-13-247
5lh0wfhw5mqnhodcszomjit37  game.3  game     runseb/2048  Running About a minute  Running        ip-172-31-13-248
2y6zuue6v67elzfxpxf2uo41l  game.4  game     runseb/2048  Running About a minute  Running        ip-172-31-13-248
```

