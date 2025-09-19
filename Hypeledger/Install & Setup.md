# **Hyperledger Fabric Installation and Setup Guide**

**🔹 Step 1: Prerequisites**

Make sure your system has these installed:

    Git

    cURL

    Docker (>=20.x)

    Docker Compose (>=2.x)

    Go (>=1.20, for Go chaincode)

    Node.js + npm (>=16.x, for JavaScript/TypeScript chaincode)

**👉 Check versions:**
    git --version
    curl --version
    docker --version
    docker compose version   # or docker-compose --version
    go version
    node -v
    npm -v


**🔹 Step 2: Download Fabric Samples, Binaries, and Docker Images**

**Make a workspace:**

    cd ~
    mkdir fabric-samples && cd fabric-samples


**Download using the official script:**

    curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.5.12 1.5.15


This installs:

    Fabric Binaries: peer, orderer, configtxgen, cryptogen
    Docker Images: Fabric peer, orderer, CA, CouchDB
    Sample networks inside fabric-samples/test-network

**🔹 Step 3: Start the Fabric Test Network**

    Move to the test-network folder:
    cd ~/fabric-samples/test-network

Bring the network up with a channel and CouchDB:

    ./network.sh down   # cleanup old network
    ./network.sh up createChannel -c mychannel -ca -s couchdb


This will start:

    2 Orgs (Org1 & Org2), each with 1 peer
    1 Orderer
    CouchDB as state database

    A channel named mychannel

**🔹 Step 4: Deploy Chaincode**

    Deploy the JavaScript sample chaincode:

        ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript -ccl javascript

**🔹 Step 5: Interact with the Network**

    Set environment variables for Org1:

        export PATH=${PWD}/../bin:$PATH
        export FABRIC_CFG_PATH=${PWD}/../config/

        export CORE_PEER_TLS_ENABLED=true
        export CORE_PEER_LOCALMSPID="Org1MSP"
        export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
        export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
        export CORE_PEER_ADDRESS=localhost:7051


Now test:

    peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'

    peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
    -C mychannel -n basic -c '{"Args":["CreateAsset","asset7","blue","20","Tom","300"]}'

**🔹 Step 6: Stop the Network**

    When done, shut everything down:

        ./network.sh down


✅ You now have Hyperledger Fabric downloaded, set up, and running a test network with CouchDB and deployed chaincode.