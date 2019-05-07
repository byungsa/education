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