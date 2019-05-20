# 새로운 Peer 추가하기

### crypto-config.yaml 수정하기

기존의 2에서 3으로 수정. 

    Org1.Template.Count : 3

새로운 Peer에 대한 MSP 생성

    ../bin/cryptogen extend --config=./crypto-config.yaml

읽기 권한 부여

    sudo chmod 755 -R ~/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/
    
### 새로운 Peer에 대한 docker-compose-new-peer.yaml 설정

    
    services:

      peer2.org1.example.com:
        container_name: peer2.org1.example.com
        extends:
          file: peer-base.yaml
          service: peer-base
        environment:
          - CORE_PEER_ID=peer2.org1.example.com
          - CORE_PEER_ADDRESS=peer2.org1.example.com:11051
          - CORE_PEER_LISTENADDRESS=0.0.0.0:11051
          - CORE_PEER_CHAINCODEADDRESS=peer2.org1.example.com:11052
          - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:11052
          - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051
          - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer2.org1.example.com:11051
          - CORE_PEER_LOCALMSPID=Org1MSP
        volumes:
            - /var/run/:/host/var/run/
            - ../crypto-config/peerOrganizations/org1.example.com/peers/peer2.org1.example.com/msp:/etc/hyperledger/fabric/msp
            - ../crypto-config/peerOrganizations/org1.example.com/peers/peer2.org1.example.com/tls:/etc/hyperledger/fabric/tls
            - peer2.org1.example.com:/var/hyperledger/production
        ports:
          - 11051:11051
        networks:
          - byfn
