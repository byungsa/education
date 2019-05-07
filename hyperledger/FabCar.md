# Writing Your First Application

1. 개발환경 설정
2. FabCar를 이용한 스마트컨트랙트 연습
3. FabCar를 이용한 샘플앱 개발


## 블록체인 네트워크 설정

만일 Building Your First Network 을 작동중이라면 기존의 tutorial을 정지합니다.
    ./byfn.sh down
    
만일 이 튜토리얼을 반복하는 것이라면 작동중인 도커 컨테이너를 정지하고 도커 이미지를 삭제합니다.
    docker rm -f $(docker ps -aq)
    docker rmi -f $(docker images | grep fabcar | awk '{print $3}')


fabcar 디렉토리로 이동합니다.
    cd ~/fabric-samples/fabcar
    cd ~/fabric-samples/fabcar/javascript
    npm install
    
------------------------

# Composer Tutorial

## 사전설치
    curl -O https://hyperledger.github.io/composer/latest/prereqs-ubuntu.sh
    chmod u+x prereqs-ubuntu.sh
    ./prereqs-ubuntu.sh
    
## Installing components (sudo 또는 su 권한으로 설치하지 말것)

    npm install -g composer-cli@0.20
    npm install -g composer-rest-server@0.20
    npm install -g generator-hyperledger-composer@0.20
    npm install -g yo
    npm install -g composer-playground@0.20
    mkdir ~/fabric-dev-servers && cd ~/fabric-dev-servers
    curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
    tar -xvf fabric-dev-servers.tar.gz
    export FABRIC_VERSION=hlfv12
    ./downloadFabric.sh
    cd ~/fabric-dev-servers
    export FABRIC_VERSION=hlfv12
    ./startFabric.sh
    ./createPeerAdminCard.sh
    
## Start the web app ("Playground")
    composer-playground
    
## Step One: Creating a business network structure
    yo hyperledger-composer:businessnetwork
