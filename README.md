# Multi-Host Hyperledger Fabric Network Setup with Docker Swarm approach

- We are using fabric version 1.4.4 in which raft mechanism has been introduced. Raft is native in the orderer service from 1.4.1. You can verify this by checking `configtx.yaml` where orderer profile is specified as `SampleMultiNodeEtcdRaft`.

scp host1_stack.yaml docker@192.168.99.108:/home/docker/fabric-samples/raft-4node-swarm
