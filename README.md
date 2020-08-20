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
  2.  `ssh docker@<node-ip-address` which will prompt for password where password is `tcuser`. You can get the ip address of node by `docker-machine ip <node-name>`
- To make a swarm cluster first ssh to node1 and make it execute the below commands
  1.  `docker swarm init --advertise-addr <node1-ip-address>`
  2.  `docker swarm join-token manager`
- The last command will give you another to command to be executed on other nodes to be joined them as manager nodes in the cluster
  1.  `<output from join-token manager> --advertise-addr <node n ip address>`
- Now we will connect all swarm nodes with overlay network. To create an overlay network execute following from node1 `docker network create --attachable --driver overlay first-network` where `first-network` is name of network. If it is successful and then you can check the network from any of the cluster nodes by executing `docker netwokr ls`. There, network scope will be **swarm**.
- Now copy the fabric-sample downloaded earlier from your local pc to `node1`. To do that execute the below command
  - `scp fabric-samples/ docker@<node1-ip-address>:/home/docker/`, it will ask for a password which is _tcuser_
- Now follow the below commands to copy the configuration file
  - `cd raft-4node-swarm`
  - `cp ../first-network/crypto-config.yaml .`
  - `cp ../first-network/configtx.yaml .`
- And generate the required materials to boot up the network
  - `../bin/cryptogen generate --config=./crypto-config.yaml`
  - `export FABRIC_CFG_PATH=$PWD`
  - `mkdir channel-artifacts`
  - `../bin/configtxgen -profile SampleMultiNodeEtcdRaft -outputBlock ./channel-artifacts/genesis.block`
  - `../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel`
  - `../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP`
  - `../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP`
- Now let's deploy our docker stack. To deploy it use the following command
  - `docker stack deploy -c docker_stack.yaml hlf` where _docker_stack.yaml_ is docker stack config file and _hlf_ is docker stack name.
  - Docker stack supports only version greater than or equals to 3.
  - You can check the docker stack by `docker stack ls`
  - There will be 10 services running across the swarm cluster. Five of orderer, four of peer and one cli service to interact with peers.
  - You can check the total replication of services by `docker service ls`
  - If you want to view specifics of one service you can do with `docker service ps <service-name>`
  - If some service is in pending state or failing you can get the extend view of it by `docker service ps --no-trunc <service-name>`
  - You can see the logs of the service by `docker service logs <service-name>`
- Now let's create a channel and let all peers join it
  - You can do docker ps and can get the cli container name or id to use it further
  - Create channel genesis block for _mychannel_ `docker exec <cli-container-name> peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem`
  - Join _peer0_org1_ to _mychannel_ `docker exec <cli-container-name> peer channel join -b mychannel.block`
