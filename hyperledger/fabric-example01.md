# Fabric Node 설치

## 개발환경
* OS : Ubuntu 16.04 LTS
* Curl : 7.47.0
* Docker : 18.09.0
* Docker Compose : 1.8.0
* Golang : 1.11.1
* Node.js : 8.9
* Python : 2.7


# 두 개의 서버에서 네트워크 구동하기

* Server1 이하 S1
* Server2 이하 S2

## S1, S2 사전설치하기
    sudo apt-get update && curl -fsSL https://get.docker.com/ | sudo sh && sudo apt install -y docker-compose && cd && wget https://dl.google.com/go/go1.11.1.linux-amd64.tar.gz && tar zxvf go1.11.1.linux-amd64.tar.gz && cd && curl -sL https://deb.nodesource.com/setup_8.x -o nodesource_setup.sh && sudo bash nodesource_setup.sh && sudo apt-get install -y nodejs &&  sudo npm install npm@5.6.0 -g && sudo chown -R $USER:$(id -gn $USER) /home/ubuntu/.config  && sudo apt-get install -y gcc g++ make
    

## S1, S2 Docker 권한 부여하기
    sudo usermod -aG docker $USER


## S1, S2 환경변수 설정하기


    echo "
    export GOROOT=$HOME/go
    export GOPATH=$HOME/workspace
    export PATH=$PATH:$GOPATH/bin:$GOROOT/bin
    " >> ~/.profile
     

'exit' 로그아웃 후 로그인.

    exit

## S1, S2 fabric 이미지 받기
    cd
    curl -sSL http://bit.ly/2ysbOFE | bash -s

***

# 각종 파일 생성하기

### S1 인증서 설정파일 수정

    vi ~/fabric-samples/first-network/crypto-config.yaml
    
orderer와 Org1의 인증서만 설치
Org2 부분은 주석처리
````
# - Name: Org2
#   Domain: org2.example.com
#   EnableNodeOUs: true
#   Template:
#     Count: 2
#   Users:
#     Count: 1
````


### S1 인증서 생성
    cd ~/fabric-samples/first-network/
    ../bin/cryptogen generate --config=./crypto-config.yaml
    

결과

    org1.example.com


### S2 인증서 설정파일 수정

    vi ~/fabric-samples/first-network/crypto-config.yaml
    
Org2 의 인증서만 설치
orderer와 Org1 부분 주석처리하고 아래부분만 남김
````
PeerOrgs:
  - Name: Org2
    Domain: org2.example.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
````


### S2 인증서 생성
    cd ~/fabric-samples/first-network/
    ../bin/cryptogen generate --config=./crypto-config.yaml
    

결과

    org2.example.com

## MSP 복사

### S1의 MSP를 S2로 복사하기
S1의 MSP중에서 orderer에 해당하는 인증서를 S2로 복사

아래 디렉토리를 sftp를 이용해서 복사해서 S2의 동일한 경로로 붙여넣기

    ~/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    ~/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/msp
    ~/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

    
### S2의 MSP를 S1으로 복사하기

    ~/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/msp
    ~/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt

***

## 블록 및 트랜잭션 생성

### S1 : Orderer Genesis Block 생성

    cd ~/fabric-samples/first-network/
    export FABRIC_CFG_PATH=$PWD
    ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
    
결과. 디렉토리 channel-artifacts에 genesis.block 이 생성됨

    2018-12-05 19:21:56.301 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
    2018-12-05 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
    2018-12-05 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block
    
    
### S1 : Channel configuration transaction 생성
    cd ~/fabric-samples/first-network/
    export CHANNEL_NAME=mychannel
    ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

결과. 디렉토리 channel-artifacts에 channel.tx (channel configuration transaction)가 channel.tx 생성됨

    2018-12-05 09:39:41.270 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
    2018-12-05 09:39:41.294 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
    2018-12-05 09:39:41.295 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx
   
   
## Anchor peer 정의 transaction 생성

### S1

    cd ~/fabric-samples/first-network/
    export CHANNEL_NAME=mychannel
    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
   
### S2

    cd ~/fabric-samples/first-network/
    export CHANNEL_NAME=mychannel
    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP


결과 

    2018-12-05 09:42:07.568 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
    2018-12-05 09:42:07.591 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
    2018-12-05 09:42:07.591 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
    
***

# Docker 설정 수정

## S1 ~/fabric-samples/first-network/docker-compose-cli.yaml 수정

    vi ~/fabric-samples/first-network/docker-compose-cli.yaml
    
S1에는 다음 컨테이너를 구동

    orderer.example.com 
    peer0.org1.example.com
    peer1.org1.example.com 
    
다른 서버와 볼륨은 모두 주석처리
/etc/hosts 호스트 등록처리

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
    extra_hosts:
      - "orderer.example.com:10.0.1.28"      
      - "peer0.org1.example.com:10.0.1.28"
      - "peer1.org1.example.com:10.0.1.28"
      - "peer0.org2.example.com:10.0.1.55"
      - "peer1.org2.example.com:10.0.1.55"
    networks:
      - byfn

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org1.example.com
    extra_hosts:
      - "orderer.example.com:10.0.1.28"      
      - "peer0.org1.example.com:10.0.1.28"
      - "peer1.org1.example.com:10.0.1.28"
      - "peer0.org2.example.com:10.0.1.55"
      - "peer1.org2.example.com:10.0.1.55"
    networks:
      - byfn

  peer1.org1.example.com:
    container_name: peer1.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org1.example.com
    extra_hosts:
      - "orderer.example.com:10.0.1.28"      
      - "peer0.org1.example.com:10.0.1.28"
      - "peer1.org1.example.com:10.0.1.28"
      - "peer0.org2.example.com:10.0.1.55"
      - "peer1.org2.example.com:10.0.1.55"
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
    extra_hosts:
      - "orderer.example.com:10.0.1.28"      
      - "peer0.org1.example.com:10.0.1.28"
      - "peer1.org1.example.com:10.0.1.28"
      - "peer0.org2.example.com:10.0.1.55"
      - "peer1.org2.example.com:10.0.1.55"
    networks:
      - byfn
````

## S2 ~/fabric-samples/first-network/docker-compose-cli.yaml 수정

    vi ~/fabric-samples/first-network/docker-compose-cli.yaml
    
S2에는 다음 컨테이너를 구동

    peer0.org2.example.com
    peer1.org2.example.com 
    
다른 서버와 볼륨은 모두 주석처리

````
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

volumes:
  # orderer.example.com:
  # peer0.org1.example.com:
  # peer1.org1.example.com:
  peer0.org2.example.com:
  peer1.org2.example.com:

networks:
  byfn:

services:

  # orderer.example.com:
  #   extends:
  #     file:   base/docker-compose-base.yaml
  #     service: orderer.example.com
  #   container_name: orderer.example.com
  #   networks:
  #     - byfn

  # peer0.org1.example.com:
  #   container_name: peer0.org1.example.com
  #   extends:
  #     file:  base/docker-compose-base.yaml
  #     service: peer0.org1.example.com
  #   networks:
  #     - byfn

  # peer1.org1.example.com:
  #   container_name: peer1.org1.example.com
  #   extends:
  #     file:  base/docker-compose-base.yaml
  #     service: peer1.org1.example.com
  #   networks:
  #     - byfn

  peer0.org2.example.com:
    container_name: peer0.org2.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org2.example.com
    extra_hosts:
      - "orderer.example.com:10.0.1.28"      
      - "peer0.org1.example.com:10.0.1.28"
      - "peer1.org1.example.com:10.0.1.28"
      - "peer0.org2.example.com:10.0.1.55"
      - "peer1.org2.example.com:10.0.1.55"
    networks:
      - byfn

  peer1.org2.example.com:
    container_name: peer1.org2.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org2.example.com
    extra_hosts:
      - "orderer.example.com:10.0.1.28"      
      - "peer0.org1.example.com:10.0.1.28"
      - "peer1.org1.example.com:10.0.1.28"
      - "peer0.org2.example.com:10.0.1.55"
      - "peer1.org2.example.com:10.0.1.55"
    networks:
      - byfn

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
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      # - orderer.example.com
      # - peer0.org1.example.com
      # - peer1.org1.example.com
      - peer0.org2.example.com
      - peer1.org2.example.com
    extra_hosts:
      - "orderer.example.com:10.0.1.28"      
      - "peer0.org1.example.com:10.0.1.28"
      - "peer1.org1.example.com:10.0.1.28"
      - "peer0.org2.example.com:10.0.1.55"
      - "peer1.org2.example.com:10.0.1.55"
    networks:
      - byfn
````

***

## S1, S2에서 Docker 컨테이너 실행하기

S1, S2에 각각 접속해서 fabric 컨테이너를 실행함.

    cd ~/fabric-samples/first-network/
    docker-compose -f docker-compose-cli.yaml up -d


결과 S1

    Creating network "net_byfn" with the default driver
    Creating volume "net_peer1.org1.example.com" with default driver
    Creating volume "net_peer0.org1.example.com" with default driver
    Creating volume "net_orderer.example.com" with default driver
    Creating peer1.org1.example.com
    Creating peer0.org1.example.com
    Creating orderer.example.com
    Creating cli


결과 S2

    Creating network "net_byfn" with the default driver
    Creating volume "net_peer0.org2.example.com" with default driver
    Creating volume "net_peer1.org2.example.com" with default driver
    Creating peer0.org2.example.com
    Creating peer1.org2.example.com
    

컨테이너 확인하기

    docker ps
    
    
결과 S1

    CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS              PORTS                      NAMES
    8f5103e26d79        hyperledger/fabric-tools:latest     "/bin/bash"         49 seconds ago      Up 48 seconds                                  cli
    d85a5bcd4855        hyperledger/fabric-orderer:latest   "orderer"           54 seconds ago      Up 50 seconds       0.0.0.0:7050->7050/tcp     orderer.example.com
    15abb3e17781        hyperledger/fabric-peer:latest      "peer node start"   54 seconds ago      Up 52 seconds       0.0.0.0:7051->7051/tcp     peer0.org1.example.com
    35cfe9c7b062        hyperledger/fabric-peer:latest      "peer node start"   54 seconds ago      Up 51 seconds       0.0.0.0:9051->8051/tcp     peer1.org1.example.com


결과 S2

    CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS              PORTS                      NAMES
    f36dad7aa063        hyperledger/fabric-peer:latest      "peer node start"   54 seconds ago      Up 52 seconds       0.0.0.0:8051->9051/tcp     peer0.org2.example.com
    1ef857b24afc        hyperledger/fabric-peer:latest      "peer node start"   54 seconds ago      Up 49 seconds       0.0.0.0:10051->10051/tcp   peer1.org2.example.com


    
    
## hosts 설정하기

다음 3가지 중의 하나를 사용해야 host 호출이 가능함
* 공인 DNS 서비스
* 사설 DNS 서비스
* /ets/hosts 등록

DNS 설정에는 전문적인 기술이 필요하므로 /etc/hosts에 ip와 host를 등록해야함.

docker 설정에서 extra_hosts 옵션을 통해 /etc/hosts 자동 등록 설정
    
***

# Fabric 실습

## Fabric 실습
Fabric cli 컨테이너로 접속

    docker exec -it cli bash


결과

    root@8f5103e26d79:/opt/gopath/src/github.com/hyperledger/fabric/peer#

이 컨테이너에 처음 접속한 것이므로 초기상태임. 새로운 프로그램이나 설정들은 새로 해야 함.

환경변수설정

    export CHANNEL_NAME=mychannel


## Fabric 채널 생성

### S1-cli : orderer 에 mychannel 생성한 후 mychannel.tx 생성

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem



### S2-cli : orderer에 접속해서 mychannel.tx 가져옴.

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    CORE_PEER_ADDRESS=peer0.org2.example.com:9051
    CORE_PEER_LOCALMSPID="Org2MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt  
    peer channel fetch 0 $CHANNEL_NAME.block -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem








## 피어를 채널에 참여시킴

### S1-cli : peer0.org1 을 mychannel에 참여

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    peer channel join -b $CHANNEL_NAME.block


결과

    2018-12-05 11:00:24.809 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
    2018-12-05 11:00:24.930 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel



### S1-cli : peer1.org1 을 mychannel에 참여


    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer1.org1.example.com:8051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
    peer channel join -b $CHANNEL_NAME.block
    
    
    
### S2-cli : peer0.org2 을 mychannel에 참여

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    CORE_PEER_ADDRESS=peer0.org2.example.com:9051
    CORE_PEER_LOCALMSPID="Org2MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt  
    peer channel join -b $CHANNEL_NAME.block

    
    
### S2-cli : peer1.org2 을 mychannel에 참여


    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    CORE_PEER_ADDRESS=peer1.org2.example.com:10051
    CORE_PEER_LOCALMSPID="Org2MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt
    peer channel join -b $CHANNEL_NAME.block    


## Anchor Peer Update

모든 피어가 채널에 참여했으면 각 조직(Org1, Org2)별로 Anchor 피어 설정을 하며, Peer0.org1, Peer0.org2를 Anchor 피어로 업데이트한다

### S1-cli : peer0.org1 Anchor Peer 설정

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem



결과

    2018-12-05 11:13:02.551 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
    2018-12-05 11:13:02.565 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update


### S2-cli : peer0.org2 Anchor Peer 설정
    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    CORE_PEER_ADDRESS=peer0.org2.example.com:9051
    CORE_PEER_LOCALMSPID="Org2MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    

## Chaincode 인스톨과 초기화
체인코드를 사용하기 위해서 체인코드를 인스톨하고 초기화한다. 

peer의 명령어 옵션은 체인코드를 설치하기 위한 install, 데이터를 쓰기 위한 invoke, 데이터를 조회하는 query가 있다.

###  S1-cli : peer0.org1 에 체인코드 인스톨

    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go


###  S1-cli : peer1.org1 에 체인코드 인스톨

    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer1.org1.example.com:8051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
    peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go


###  S2-cli : peer0.org2 에 체인코드 인스톨

    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    CORE_PEER_ADDRESS=peer0.org2.example.com:9051
    CORE_PEER_LOCALMSPID="Org2MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go


###  S2-cli : peer1.org2 에 체인코드 인스톨

    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    CORE_PEER_ADDRESS=peer1.org2.example.com:10051
    CORE_PEER_LOCALMSPID="Org2MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt
    peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go

### chaincode_example02.go 
https://github.com/hyperledger/fabric-samples/blob/release-1.3/chaincode/chaincode_example02/go/chaincode_example02.go


## Chaincode 초기화
###  S1-cli : peer0.org1에서 체인코드를 초기화한다.

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.member','Org2MSP.member')"


결과

    2018-12-05 11:51:13.950 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
    2018-12-05 11:51:13.950 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc


## Chaincode-level Endorsement 정책
Endorsement 정책

-P 옵션 : Endorsement 정책. 

"AND('Org1.member', 'Org2.member’)”

AND 조건 : Org1 멤버중의 하나 && Org2 멤버중의 하나

* "AND('Org1.peer', 'Org2.peer’)” : AND 조건 : Org1 Peer 중의 하나 && Org2 Peer중의 하나
* OR('Org1.member', 'Org2.member’) : OR 조건 : Org1 멤버중의 하나 || Org2 멤버중의 하나
* OR(   AND('Org1.member', 'Org2.member’), AND('Org1.member', 'Org3.member’), AND('Org2.member', 'Org3.member’)  )


|옵션|설명|
|-----|-----|
| 'Org0.admin’ |Org0 MSP의 어드민중 아무나|
| 'Org1.member’|Org1 MSP의 멤버중 아무나|
| 'Org1.client’|Org1 MSP 클라이언트중 아무나|
| 'Org1.peer’|Org1 MSP 피어중 아무나|


## 초기화 결과 Query

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'


결과

    100
    
    
## Invoke
### S1-cli : a의 10을 b로 이체하는 체인코드 invoke

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'



결과

    2018-12-05 12:22:01.254 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200


## Query 확인

### S1-cli : Invoke의 결과로 a에 90이 저장됐는지 확인.

    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

결과

    90


### 다른 피어, peer0.org2에 원장이 복제되었는지 확인

### S2-cli : peer0.org2에서 조회 쿼리
    export CHANNEL_NAME=mychannel
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    CORE_PEER_ADDRESS=peer0.org2.example.com:9051
    CORE_PEER_LOCALMSPID="Org2MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'


결과

    90
    
    
***

    
***

# Hyperledger Explorer

## 기존에 설치 실패했을 경우 기존 소스 삭제
    cd
    rm -rf blockchain-explorer
    

## 사전설치

nodejs 8.14.x (9.x 버전은 지원하지 않음), PostgreSQL 9.5 or greater, Jq[https://stedolan.github.io/jq/]

    sudo apt install -y postgresql jq


### Hyperledger Explorer 설치

    cd
    git clone https://github.com/hyperledger/blockchain-explorer


### postgresql 접속 설정

    vi ~/blockchain-explorer/app/explorerconfig.json (안해도 됨)


### DB 생성

    cd ~/blockchain-explorer/app/persistence/fabric/postgreSQL/db 
    sudo -u postgres ./createdb.sh
    (원래 ./createdb.sh 만 해도 되나 오류가 발생하는 경우가 많음)

### DB 생성확인 (안해도 됨)

    sudo -u postgres psql
    postgres=# \l


### 접속종료

    postgres-# \q


### Explorer에 인증서 경로 지정

    vi ~/blockchain-explorer/app/platform/fabric/config.json
    > :%s /fabric-path/home\/ubuntu/g

### 오류수정

    vi ~/blockchain-explorer/client/src/components/View/LandingPage.spec.js
    > getBlockActivity: jest.fn(), // 14열에 추가

### Explorer 빌드
터미널을 하나 더 열고 실행

    cd ~/blockchain-explorer
    npm install
    cd ~/blockchain-explorer/client/
    npm install
    npm test -- -u --coverage
    npm run build


### Explorer 실행
    cd ~/blockchain-explorer
    ./start.sh

### Explorer 종료
    cd ~/blockchain-explorer
    ./stop.sh

***

### 실시간 로그보기
    tail -f blockchain-explorer/logs/console/console-2019-05-22.log
    
# 실시간 로그 확인하기

    docker exec -it cli bash
    docker logs -f orderer.example.com
    docker logs -f peer0.org1.example.com
    docker logs -f peer1.org1.example.com
    docker logs -f peer0.org2.example.com
    docker logs -f peer1.org2.example.com

# Docker 명령어

## Container


| 내용             | 명령어                |
|-----------------|-----------------------|
| 조회(작동중)   | $ docker ps |
| 조회(모두)   | $ docker ps -a |
| 정지   | $ docker stop [CONTAINER ID], [CONTAINER ID] |
| 정지(모두)   | $ docker stop \`docker ps -aq\` |
| 삭제  | $ docker rm [CONTAINER ID], [CONTAINER ID] |
| 삭제(모두) | $ docker rm \`docker ps -aq\` |
| 볼륨삭제(모두) | $ docker volume rm $(docker volume ls -qf dangling=true) |
  
 
## Images

| 내용             | 명령어                |
|-----------------|-----------------------|
| 조회   | $ docker images |
| 삭제  | $ docker rmi [IMAGE ID], [IMAGE ID] |
| 삭제(모두) | $ docker rmi \`docker images -q\` |
| 삭제(강제) | $ docker rmi -f \`docker images -q\` |


