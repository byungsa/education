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
    
## S1에서 Docker 실생

    vi ~/fabric-samples/first-network/docker-compose-cli.yaml
    
S1에는 orderer.example.com, peer0.org1.example.com, peer1.org1.example.com 컨테이너를 구동.
다른 서버와 볼륨은 모두 주석처리

