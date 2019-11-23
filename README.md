# 3org1ch_143
Rewrite 3org1ch with latest code (1.4.3)

This repo is used for creating a fabric network in a Hyperledger Fabric node.
It is assumed you have a Hyperledger Fabric node, with the prerequisite installation done and all required components, including the docker images, tools and fabric samples.

## Instruction
Step 1: clone this repo in fabric-samples directory
```
cd fabric-samples
git clone https://github.com/kctam/3org1ch.git
cd 3org1ch
```

Step 2: generate the required crypto material for organizations
```
../bin/cryptogen generate --config=./crypto-config.yaml
```

Step 3: generate the channel artifacts
```
mkdir channel-artifacts && export FABRIC_CFG_PATH=$PWD
../bin/configtxgen -profile ThreeOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
../bin/configtxgen -profile ThreeOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
```

Step 4: bring up all the containers, and you should see total 8 containers up and running
```
docker-compose -f docker-compose-cli.yaml up -d
docker ps
```

Step 5: bring up three terminals for easy demonstration

For Org1
```
docker exec -it cli bash 
```

For Org2
```
docker exec -e "CORE_PEER_LOCALMSPID=Org2MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp" -e "CORE_PEER_ADDRESS=peer0.org2.example.com:7051" -it cli bash
```

For Org3
```
docker exec -e "CORE_PEER_LOCALMSPID=Org3MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp" -e "CORE_PEER_ADDRESS=peer0.org3.example.com:7051" -it cli bash 
```

Step 5: create and join channel

Org1
```
peer channel create -o orderer.example.com:7050 /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -c mychannel -f ./channel-artifacts/channel.tx

peer channel join -b mychannel.block
```
Org2
```
peer channel join -b mychannel.block
```
Org3
```
peer channel join -b mychannel.block
```

Step 6: If everything looks good, you now have a fabric network with 3 organizations set, and ready for testing any chaincode.

Step 7: clean up: always good practice for cleaning up stuff after testing
```
docker-compose -f docker-compose-cli.yaml --volumes --remove-orphans
docker rm $(docker ps -aq)
docker rmi $(docker images dev-* -q)
```

## Reuse the crypto material and channel artifacts
You can keep the content inside *crypto-config* and *channel-artifacts* for next test. If you have them already you can start with Step 4.

Enjoy Hyperledger Fabric!

