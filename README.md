# Multi-Host Hyperledger Fabric Network Setup with Docker Swarm approach

- We are using fabric version 1.4.4 in which raft mechanism has been introduced. Raft is native in the orderer service from 1.4.1. You can verify this by checking `configtx.yaml` where orderer profile is specified as `SampleMultiNodeEtcdRaft`.

> This demo/tutorial is based on cluster of nodes created by docker-machine in which docker-machine uses default driver `--driver virtualbox`. So you need to [install Oracle Virtual Box]([I'm an inline-style link](https://www.virtualbox.org/wiki/Linux_Downloads) in order to follow this tutorial.

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

scp host1_stack.yaml docker@192.168.99.108:/home/docker/fabric-samples/raft-4node-swarm
