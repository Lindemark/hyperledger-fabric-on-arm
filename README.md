# Hyperledger On Arm
Makes hyperledger version 1.0.0-beta working on arm architectures as well (tested on armv7l - Raspberry Pi 3 Model B)

### Steps to make the getting_started to work :

Currently still working with the beta 1.0.0 tutorial on : http://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html

So open that link and execute this commands : 

```
go get github.com/ArvsIndrarys/hyperledger-fabric-on-arm
cd ~/go/src/github.com/ArvsIndrarys/hyperledger-fabric-on-arm
```
and next :
```
./bootstrap.sh
```
From now on, the tutorial shall work with only one change, add -t 20 at the end of the peer create channel command.

### In case the tutorial doesn't work

You can also complete it by followind theses commands then reading the getting_started and checking how it works :  
```
./generate_artifacts
```
 

> In case you don't use ./generate_artifacts, you have to modify the base/peer-base.yaml file on the line `CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=` to correspond to `CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=(folder with docker-compose-cli.bash)_default` either by modifying it yourself or by using :
```
NETWORK_CC=$(basename $PWD | tr '[:upper:]' '[:lower:]')
```
  
>then 
 
``` 
sed -i -e "/NETWORKMODE/c\      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE="$NETWORK_CC"_default" base/peer-base.yaml
```  

From this on, we can continue with :  
```
CHANNEL_NAME=mychannel TIMEOUT=100 docker-compose -f docker-compose-cli.yaml -d

docker exec -it cli bash

peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME \
-f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem \
-t 20

peer channel join -b mychannel.block

peer chaincode install -n mycc -v 1.0 \
-p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 

peer chaincode instantiate -o orderer.example.com:7050 \
--tls $CORE_PEER_TLS_ENABLED \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem \
-C $CHANNEL_NAME -n mycc -v 1.0 \
-p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 \
-c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem  \
-C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```
  
  
 ### WARNINGS
 As presented on the tutorial page, the scripts network.sh and scripts/script.sh aren't working. They have been removed so it is unlikely that they
 would work as most of the files have been adapted (docker-compose-cli.yaml for example).  
After some modifications between two runs, it may be necessary to remove the old chaincode image for the instantiation to work properly.
