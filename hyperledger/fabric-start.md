# Fabric Node 설치

## 개발환경
* OS : Ubuntu 16.04 LTS
* Curl : 7.47.0
* Docker : 18.09.0
* Docker Compose : 1.8.0
* Golang : 1.11.1
* Node.js : 8.9
* Python : 2.7

## 한 번에 사전설치
    sudo apt-get update && curl -fsSL https://get.docker.com/ | sudo sh && sudo apt install -y docker-compose && cd && wget https://dl.google.com/go/go1.11.1.linux-amd64.tar.gz && tar zxvf go1.11.1.linux-amd64.tar.gz && cd && curl -sL https://deb.nodesource.com/setup_8.x -o nodesource_setup.sh && sudo bash nodesource_setup.sh && sudo apt-get install -y nodejs &&  sudo npm install npm@5.6.0 -g && sudo chown -R $USER:$(id -gn $USER) /home/ubuntu/.config  && sudo apt-get install -y gcc g++ make
    sudo usermod -aG docker $USER

환경변수 설정하기

    vi ~/.profile

    export GOROOT=$HOME/go
    export GOPATH=$HOME/workspace
    export PATH=$PATH:$GOPATH/bin:$GOROOT/bin

로그아웃 후 로그인.

***

# Fabric 실습

## 이미지 받기
    cd
    curl -sSL http://bit.ly/2ysbOFE | bash -s
    
    
다음 옵션을 통해 다른 버전을 받을 수도 있음

    curl -sSL http://bit.ly/2ysbOFE | bash -s <fabric> <fabric-ca> <thirdparty>
    curl -sSL http://bit.ly/2ysbOFE | bash -s -- 1.4.1 1.4.1 0.4.15

## Fabric 네트워크 한번에 시작하기

first-network로 이동

    cd ~/fabric-samples/first-network/


인증서, 채널설정 한 번에 하기

    ./byfn.sh generate


네트워크 시작 한번에 하기

    ./byfn.sh up

네트워크 종료 및 도커 컨테이너 삭제 한 번에 하기

    ./byfn.sh down
    
    
# 두 개의 서버에서 네트워크 구동하기

Server1 이하 S1
Server2 이하 S2

## S1 
### 사전설치하기
    sudo apt-get update && curl -fsSL https://get.docker.com/ | sudo sh && sudo apt install -y docker-compose && cd && wget https://dl.google.com/go/go1.11.1.linux-amd64.tar.gz && tar zxvf go1.11.1.linux-amd64.tar.gz && cd && curl -sL https://deb.nodesource.com/setup_8.x -o nodesource_setup.sh && sudo bash nodesource_setup.sh && sudo apt-get install -y nodejs &&  sudo npm install npm@5.6.0 -g && sudo chown -R $USER:$(id -gn $USER) /home/ubuntu/.config  && sudo apt-get install -y gcc g++ make
    sudo usermod -aG docker $USER

### 환경변수 설정하기

    vi ~/.profile

    export GOROOT=$HOME/go
    export GOPATH=$HOME/workspace
    export PATH=$PATH:$GOPATH/bin:$GOROOT/bin

로그아웃 후 로그인.

### 이미지 받기
    cd
    curl -sSL http://bit.ly/2ysbOFE | bash -s


## S2
### 사전설치하기
    sudo apt-get update && curl -fsSL https://get.docker.com/ | sudo sh && sudo apt install -y docker-compose && cd && wget https://dl.google.com/go/go1.11.1.linux-amd64.tar.gz && tar zxvf go1.11.1.linux-amd64.tar.gz && cd && curl -sL https://deb.nodesource.com/setup_8.x -o nodesource_setup.sh && sudo bash nodesource_setup.sh && sudo apt-get install -y nodejs &&  sudo npm install npm@5.6.0 -g && sudo chown -R $USER:$(id -gn $USER) /home/ubuntu/.config  && sudo apt-get install -y gcc g++ make
    sudo usermod -aG docker $USER

### 환경변수 설정하기

    vi ~/.profile

    export GOROOT=$HOME/go
    export GOPATH=$HOME/workspace
    export PATH=$PATH:$GOPATH/bin:$GOROOT/bin

로그아웃 후 로그인.

### 이미지 받기
    cd
    curl -sSL http://bit.ly/2ysbOFE | bash -s


## S1에서 각종 트랜잭션 파일 만들기

### 인증서 생성
    cd ~/fabric-samples/first-network/
    ../bin/cryptogen generate --config=./crypto-config.yaml
    

결과
    org1.example.com
    org2.example.com
    
### Orderer Genesis Block 생성
    export FABRIC_CFG_PATH=$PWD
    ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
    
결과. 디렉토리 channel-artifacts에 genesis.block 이 생성됨

    2018-12-05 19:21:56.301 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
    2018-12-05 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
    2018-12-05 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block
    
### Channel configuration transaction 생성
    export CHANNEL_NAME=mychannel
    ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

결과. 디렉토리 channel-artifacts에 channel.tx (channel configuration transaction)가 channel.tx 생성됨

    2018-12-05 09:39:41.270 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
    2018-12-05 09:39:41.294 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
    2018-12-05 09:39:41.295 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx
    
### Anchor peer 정의 transaction 생성
    export CHANNEL_NAME=mychannel
    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP

결과 

    2018-12-05 09:42:07.568 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
    2018-12-05 09:42:07.591 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
    2018-12-05 09:42:07.591 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
    
## S1 ~/fabric-samples/first-network/docker-compose-cli.yaml 수정

    vi ~/fabric-samples/first-network/docker-compose-cli.yaml
    
S1에는 다음 컨테이너를 구동

    orderer.example.com 
    peer0.org1.example.com
    peer1.org1.example.com 
    
다른 서버와 볼륨은 모두 주석처리

````
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

volumes:
  orderer.example.com:
  peer0.org1.example.com:
  peer1.org1.example.com:
  # peer0.org2.example.com:
  # peer1.org2.example.com:

networks:
  byfn:

services:

  orderer.example.com:
    extends:
      file:   base/docker-compose-base.yaml
      service: orderer.example.com
    container_name: orderer.example.com
    networks:
      - byfn

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org1.example.com
    networks:
      - byfn

  peer1.org1.example.com:
    container_name: peer1.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org1.example.com
    networks:
      - byfn

  # peer0.org2.example.com:
  #   container_name: peer0.org2.example.com
  #   extends:
  #     file:  base/docker-compose-base.yaml
  #     service: peer0.org2.example.com
  #   networks:
  #     - byfn

  # peer1.org2.example.com:
  #   container_name: peer1.org2.example.com
  #   extends:
  #     file:  base/docker-compose-base.yaml
  #     service: peer1.org2.example.com
  #   networks:
  #     - byfn

  cli:
    container_name: cli
    image: hyperledger/fabric-tools:$IMAGE_TAG
    tty: true
    stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      #- FABRIC_LOGGING_SPEC=DEBUG
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - orderer.example.com
      - peer0.org1.example.com
      - peer1.org1.example.com
      # - peer0.org2.example.com
      # - peer1.org2.example.com
    networks:
      - byfn
````

## S1 ~/fabric-samples/first-network/base/docker-compose-base.yaml


    vi ~/fabric-samples/first-network/base/docker-compose-base.yaml
    
S1에는 다음 컨테이너를 구동

    orderer.example.com 
    peer0.org1.example.com
    peer1.org1.example.com 
    
다른 서버와 볼륨은 모두 주석처리

````
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

services:

  orderer.example.com:
    container_name: orderer.example.com
    extends:
      file: peer-base.yaml
      service: orderer-base
    volumes:
        - ../channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
        - ../crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp
        - ../crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/:/var/hyperledger/orderer/tls
        - orderer.example.com:/var/hyperledger/production/orderer
    ports:
      - 7050:7050

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer0.org1.example.com
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org1.example.com:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.example.com:8051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
    volumes:
        - /var/run/:/host/var/run/
        - ../crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/fabric/msp
        - ../crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls:/etc/hyperledger/fabric/tls
        - peer0.org1.example.com:/var/hyperledger/production
    ports:
      - 7051:7051

  peer1.org1.example.com:
    container_name: peer1.org1.example.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer1.org1.example.com
      - CORE_PEER_ADDRESS=peer1.org1.example.com:8051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:8051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org1.example.com:8052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:8052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.example.com:8051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
    volumes:
        - /var/run/:/host/var/run/
        - ../crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp:/etc/hyperledger/fabric/msp
        - ../crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls:/etc/hyperledger/fabric/tls
        - peer1.org1.example.com:/var/hyperledger/production

    ports:
      - 8051:8051

  # peer0.org2.example.com:
  #   container_name: peer0.org2.example.com
  #   extends:
  #     file: peer-base.yaml
  #     service: peer-base
  #   environment:
  #     - CORE_PEER_ID=peer0.org2.example.com
  #     - CORE_PEER_ADDRESS=peer0.org2.example.com:9051
  #     - CORE_PEER_LISTENADDRESS=0.0.0.0:9051
  #     - CORE_PEER_CHAINCODEADDRESS=peer0.org2.example.com:9052
  #     - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:9052
  #     - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org2.example.com:9051
  #     - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org2.example.com:10051
  #     - CORE_PEER_LOCALMSPID=Org2MSP
  #   volumes:
  #       - /var/run/:/host/var/run/
  #       - ../crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp:/etc/hyperledger/fabric/msp
  #       - ../crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls:/etc/hyperledger/fabric/tls
  #       - peer0.org2.example.com:/var/hyperledger/production
  #   ports:
  #     - 9051:9051

  # peer1.org2.example.com:
  #   container_name: peer1.org2.example.com
  #   extends:
  #     file: peer-base.yaml
  #     service: peer-base
  #   environment:
  #     - CORE_PEER_ID=peer1.org2.example.com
  #     - CORE_PEER_ADDRESS=peer1.org2.example.com:10051
  #     - CORE_PEER_LISTENADDRESS=0.0.0.0:10051
  #     - CORE_PEER_CHAINCODEADDRESS=peer1.org2.example.com:10052
  #     - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:10052
  #     - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org2.example.com:10051
  #     - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org2.example.com:9051
  #     - CORE_PEER_LOCALMSPID=Org2MSP
  #   volumes:
  #       - /var/run/:/host/var/run/
  #       - ../crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/msp:/etc/hyperledger/fabric/msp
  #       - ../crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls:/etc/hyperledger/fabric/tls
  #       - peer1.org2.example.com:/var/hyperledger/production
  #   ports:
  #     - 10051:10051
````
