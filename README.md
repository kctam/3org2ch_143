# 3org2ch_143
Rewrite 3org2ch with latest code (1.4.3)

This repo is used for creating a fabric network in a Hyperledger Fabric node.
It is assumed you have a Hyperledger Fabric node, with the prerequisite installation done and all required components, including the docker images, tools and fabric samples.

## Instruction
Step 1: clone this repo in fabric-samples directory
```
cd fabric-samples
git clone https://github.com/kctam/3org2ch_143.git
cd 3org2ch_143
```

Step 2: generate the required crypto material for organizations
```
../bin/cryptogen generate --config=./crypto-config.yaml
```

Step 3: generate the channel artifacts
```
mkdir channel-artifacts && export FABRIC_CFG_PATH=$PWD

../bin/configtxgen -profile OrdererGenesis -outputBlock ./channel-artifacts/genesis.block

export CHANNEL_ONE_NAME=channelall
export CHANNEL_ONE_PROFILE=ChannelAll
export CHANNEL_TWO_NAME=channel12
export CHANNEL_TWO_PROFILE=Channel12

../bin/configtxgen -profile ${CHANNEL_ONE_PROFILE} -outputCreateChannelTx ./channel-artifacts/${CHANNEL_ONE_NAME}.tx -channelID $CHANNEL_ONE_NAME

../bin/configtxgen -profile ${CHANNEL_TWO_PROFILE} -outputCreateChannelTx ./channel-artifacts/${CHANNEL_TWO_NAME}.tx -channelID $CHANNEL_TWO_NAME

../bin/configtxgen -profile ${CHANNEL_ONE_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors_${CHANNEL_ONE_NAME}.tx -channelID $CHANNEL_ONE_NAME -asOrg Org1MSP

../bin/configtxgen -profile ${CHANNEL_ONE_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors_${CHANNEL_ONE_NAME}.tx -channelID $CHANNEL_ONE_NAME -asOrg Org2MSP

../bin/configtxgen -profile ${CHANNEL_ONE_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org3MSPanchors_${CHANNEL_ONE_NAME}.tx -channelID $CHANNEL_ONE_NAME -asOrg Org3MSP

../bin/configtxgen -profile ${CHANNEL_TWO_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors_${CHANNEL_TWO_NAME}.tx -channelID $CHANNEL_TWO_NAME -asOrg Org1MSP

../bin/configtxgen -profile ${CHANNEL_TWO_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors_${CHANNEL_TWO_NAME}.tx -channelID $CHANNEL_TWO_NAME -asOrg Org2MSP
```

Step 4: bring up all the containers, and you should see total 5 containers up and running
```
docker-compose up -d
docker ps
```

Step 5: bring up three terminals for easy demonstration and set the orderer CA

*Org1*
```
docker exec -it cli bash 

export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

*Org2*
```
docker exec -e "CORE_PEER_LOCALMSPID=Org2MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp" -e "CORE_PEER_ADDRESS=peer0.org2.example.com:7051" -it cli bash

export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

*Org3*
```
docker exec -e "CORE_PEER_LOCALMSPID=Org3MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp" -e "CORE_PEER_ADDRESS=peer0.org3.example.com:7051" -it cli bash

export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

Step 5: create and join channel

For channelall

*Org1*
```
peer channel create -o orderer.example.com:7050 -c channelall -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/channelall.tx --tls --cafile $ORDERER_CA

peer channel join -b channelall.block --tls --cafile $ORDERER_CA

peer channel update -o orderer.example.com:7050 -c channelall -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org1MSPanchors_channelall.tx --tls --cafile $ORDERER_CA
```

*Org2*
```
peer channel join -b channelall.block --tls --cafile $ORDERER_CA

peer channel update -o orderer.example.com:7050 -c channelall -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org2MSPanchors_channelall.tx --tls --cafile $ORDERER_CA
```

*Org3*
```
peer channel join -b channelall.block --tls --cafile $ORDERER_CA

peer channel update -o orderer.example.com:7050 -c channelall -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org3MSPanchors_channelall.tx --tls --cafile $ORDERER_CA
```

For channel12

*Org1*
```
peer channel create -o orderer.example.com:7050 -c channel12 -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/channel12.tx --tls --cafile $ORDERER_CA

peer channel join -b channel12.block --tls --cafile $ORDERER_CA

peer channel update -o orderer.example.com:7050 -c channel12 -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org1MSPanchors_channel12.tx --tls --cafile $ORDERER_CA
```

*Org2*
```
peer channel join -b channel12.block --tls --cafile $ORDERER_CA

peer channel update -o orderer.example.com:7050 -c channel12 -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org2MSPanchors_channel12.tx --tls --cafile $ORDERER_CA
```

Step 6: Check each peer the channel(s) it has joint. 

For each terminal,
```
peer channel list
```

You should see org1 and org2 has two channels, while org3 only on channelall.

If everything looks good, you now have a fabric network with 3 organizations set, and ready for testing any chaincode.

Step 7: clean up: always good practice for cleaning up stuff after testing
```
docker-compose down -v
docker rm $(docker ps -aq)
docker rmi $(docker images dev-* -q)
```

## Reuse the crypto material and channel artifacts
You can keep the content inside *crypto-config* and *channel-artifacts* for next test. If you have them already you can start with Step 4.

Enjoy Hyperledger Fabric!

