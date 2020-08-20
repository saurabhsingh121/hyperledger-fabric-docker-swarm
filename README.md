# Multi-Host Hyperledger Fabric Network Setup with Docker Swarm approach

- We are using fabric version 1.4.4 in which raft mechanism has been introduced. Raft is native in the orderer service from 1.4.1. You can verify this by checking `configtx.yaml` where orderer profile is specified as `SampleMultiNodeEtcdRaft`.

> This demo/tutorial is based on cluster of nodes created by docker-machine in which docker-machine uses default driver `--driver virtualbox`. So you need to [install Oracle Virtual Box](https://www.virtualbox.org/wiki/Linux_Downloads) in order to follow this tutorial.

## Steps to do launch the network

- Make sure your system has docker and go language installed. If not,
  1. Install docker and docker compose by `sudo apt-get -y install docker-compose`
  2. And as standard practice on docker installation `sudo usermod -aG docker $USER`
  3. Check if docker is installed by `docker -v` and `docker-compose -v`
  4. Install go programming language and set up environment vairables ny using following commands
     - `wget https://golang.org/dl/go1.11.11.linux-amd64.tar.gz`
     - `sudo tar -xvf go1.11.11.linux-amd64.tar.gz`
     - `export GOPATH=$HOME/go`
     - `export PATH=$PATH:$GOPATH/bin`
- Install Hyperledger Fabric Components consisting docker images, binary tools and fabric samples using `curl -sSL http://bit.ly/2ysbOFE | bash -s 1.4.4`
- We can verify the docker images pulled by `docker images`. You can see there are two types of tag for same docker image.
- This tutorial runs on four nodes cluster. To make a four node cluster run the below commands.

  1. `docker-machine create node1`
  2. `docker-machine create node2`
  3. `docker-machine create node3`
  4. `docker-machine create node4`

- You can ssh to these nodes by two ways. Choose either of below given methodes
  1.  `docker-machine ssh <node-name>`
  2.  `ssh docker@<node-ip-address` which will prompt for password where password is `tcuser`
- To make a swarm cluster first ssh to node1 and make it execute the below commands
  1.  `docker swarm init --advertise-addr <node1-ip-address>`
  2.  `docker swarm join-token manager`
- The last command will give you another to command to be executed on other nodes to be joined them as manager nodes in the cluster
  1.  `<output from join-token manager> --advertise-addr <node n ip address>`

scp host1_stack.yaml docker@192.168.99.108:/home/docker/fabric-samples/raft-4node-swarm
