# Chaincode Upgrade


### 도커에 접속

    docker exec -it cli bash


### 새로운 버전의 chaincode를 peer에 등록하기

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    peer chaincode install -n mycc -v 2.0 -p github.com/chaincode/chaincode_example02_2/go
