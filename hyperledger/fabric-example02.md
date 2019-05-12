# Fabric Node Org3 추가

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

Org3 추가 한 번에 하기

    ./eyfn.sh up

네트워크 종료 및 도커 컨테이너 삭제 한 번에 하기

    ./eyfn.sh down
    
# Org3 관련 파일 생성

## cli 컨테이너의 설정 변경

container의 디버그 설정을 켬

    vi ~/fabric-samples/first-network/docker-compose-cli.yaml
    
아래처럼 수정
    
    68       - FABRIC_LOGGING_SPEC=DEBUG
    69       #- FABRIC_LOGGING_SPEC=INFO
    
## Org1, Org2 생성 및 기존 컨테이너 실행    

    ./byfn.sh generate
    ./byfn.sh up
    
    
## Org3를 위한 컨테이너 파일 설정파일 만듬.

 
컨테이너 설정파일의 디버그 설정

    vi ~/fabric-samples/first-network/docker-compose-org3.yaml
    
아래처럼 수정
    
    74       #- FABRIC_LOGGING_SPEC=INFO
    75       - FABRIC_LOGGING_SPEC=DEBUG

## Org3 인증서 생성을 위한 설정파일

    vi ~/fabric-samples/first-network/org3-artifacts/org3-crypto.yaml
    vi ~/fabric-samples/first-network/org3-artifacts/configtx.yaml
    
## Org3 인증서 생성

    cd ~/fabric-samples/first-network/org3-artifacts/
    ../../bin/cryptogen generate --config=./org3-crypto.yaml
    
org3-crypto.yaml 설정파일은 Org3 그룹만 있고 orderer에 대한 정보가 없어서 아래 오류가 나오지만 무시함.

    WARN 003 Default policy emission is deprecated, please include policy specifications for the orderer org group Org3MSP in configtx.yaml 
    

## Org3 정책 정의파일 org3.json 생성

    export FABRIC_CFG_PATH=$PWD && ../../bin/configtxgen -printOrg Org3MSP > ../channel-artifacts/org3.json

같은 오류가 발생.

    WARN 003 Default policy emission is deprecated, please include policy specifications for the orderer org group Org3MSP in configtx.yaml


## orderer MSP 인증서파일을 작업중인 Org3 디렉토리로 복사

    cp -r ~/fabric-samples/first-network/crypto-config/ordererOrganizations ~/fabric-samples/first-network/org3-artifacts/crypto-config/
    
***

# Org3 추가하기

## CLI 환경설정

docker cli 연결

    docker exec -it cli bash
    
    
환경변수 설정

    export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  && export CHANNEL_NAME=mychannel


## Fetch the Configuration

    peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA


결과 : config_block.pb, binary protobuf channel configuration block protobuf 파일생성.


## Convert the Configuration to JSON and Trim It Down

    configtxlator proto_decode --input config_block.pb --type common.Block | jq .data.data[0].payload.data.config > config.json


결과 : config.json 생성. 기존의 조직과 정책에 대한 파일.


## Org3 인증관련 파일 추가


    jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"Org3MSP":.[1]}}}}}' config.json ./channel-artifacts/org3.json > modified_config.json


결과 : modified_config.json 생성. config.json에 Org3의 정보를 담고 있는 org3.json의 내용이 합쳐져서 생성



config.json을 protobuf로 인코딩

    configtxlator proto_encode --input config.json --type common.Config --output config.pb


결과 : config.pb 생성. 


modified_config.json을 protobuf로 인코딩


    configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb


결과 : modified_config.pb 생성


원래의 config.pb와 수정된 modified_config.pb 새롭게 업데이트 될 protobuf bynary 생성

    configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_config.pb --output org3_update.pb


결과 : org3_update.pb 생성


수정가능한 org3_update.json 형태 파일 생성

    configtxlator proto_decode --input org3_update.pb --type common.ConfigUpdate | jq . > org3_update.json

결과 : org3_update.json 생성


json 양식에 맞게 header 등의 정보 추가.

    echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":'$(cat org3_update.json)'}}}' | jq . > org3_update_in_envelope.json


결과 : org3_update_in_envelope.json 생성


org3_update_in_envelope.json을 protobuf 인코딩

    configtxlator proto_encode --input org3_update_in_envelope.json --type common.Envelope --output org3_update_in_envelope.pb


결과 : org3_update_in_envelope.pb


## 서명 및 업데이트

Org1 Admin 에서의 서명

    peer channel signconfigtx -f org3_update_in_envelope.pb

    

Org2 Admin 에서의 서명

    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_ADDRESS=peer0.org2.example.com:9051
    peer channel update -f org3_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.example.com:7050 --tls --cafile $ORDERER_CA


***

## Org3 Peer의 채널 참여

Org3 컨테이너 실행

    cd ~/fabric-samples/first-network/
    docker-compose -f docker-compose-org3.yaml up -d



Org3cli 접속

    docker exec -it Org3cli bash
    

환경변수 입력
    export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem && export CHANNEL_NAME=mychannel


0번블록 생성 및 채널참여

    peer channel fetch 0 mychannel.block -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
    peer channel join -b mychannel.block


peer1.org3의 채널 참여
    export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer1.org3.example.com/tls/ca.crt && export CORE_PEER_ADDRESS=peer1.org3.example.com:12051
    peer channel join -b mychannel.block


Org3에서 mycc 2.0으로 업데이트 함.

    peer chaincode install -n mycc -v 2.0 -p github.com/chaincode/chaincode_example02/go/


Org1, Org2에서도 mycc 2.0으로 업데이트해야함.
Org1과 Org2 에서 mycc를 업데이트함.

    peer chaincode install -n mycc -v 2.0 -p github.com/chaincode/chaincode_example02/go/


Org1 또는 Org2에서 mycc Endosement 정책 업데이트를 함.

    peer chaincode upgrade -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 2.0 -c '{"Args":["init","a","90","b","210"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer','Org3MSP.peer')"


    
