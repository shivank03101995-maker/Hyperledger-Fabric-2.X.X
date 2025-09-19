📘 **Documentation: Adding Org3 to an Existing Hyperledger Fabric Network**
   
    This guide explains step-by-step how to add Org3 into an already running Fabric network (with Org1, Org2, and Orderer).

✅ **Prerequisites**

    Hyperledger Fabric binaries and Docker images installed
    Running Fabric test network with Org1, Org2
    Channel already created (e.g., mychannel)

    Tools: cryptogen, configtxgen, configtxlator, jq

**1️⃣ Generate Crypto Material for Org3**
    Use cryptogen to generate Org3 certificates.

**📦 Command:**
    cd fabric-samples/test-network

    cryptogen generate --config=./organizations/cryptogen/crypto-config-org3.yaml \
    --output="organizations"


**📂 Output:**

    organizations/peerOrganizations/org3.example.com/

**2️⃣ Fetch Current Channel Config**

**📦 Command:**

    export CHANNEL_NAME=mychannel
    export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

    peer channel fetch config config_block.pb -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    -c $CHANNEL_NAME --tls --cafile $ORDERER_CA


Convert block to JSON:

    configtxlator proto_decode --input config_block.pb --type common.Block \
    | jq .data.data[0].payload.data.config > config.json

**3️⃣ Create Org3 Definition**

**📦 Command:**

    configtxgen -printOrg Org3MSP > org3.json


Merge with existing config:

    jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups":{"Org3MSP":.[1]}}}}}' \
    config.json org3.json > modified_config.json

**4️⃣ Create Config Update Transaction**

**📦 Command:**

# Encode configs
    configtxlator proto_encode --input config.json --type common.Config --output config.pb
    configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb

# Compute delta
    configtxlator compute_update --channel_id $CHANNEL_NAME \
    --original config.pb --updated modified_config.pb --output org3_update.pb

# Decode update
configtxlator proto_decode --input org3_update.pb --type common.ConfigUpdate \
  | jq . > org3_update.json

# Wrap in envelope
echo '{"payload":{"header":{"channel_header":{"channel_id":"'$CHANNEL_NAME'", "type":2}}, "data":{"config_update":'$(cat org3_update.json)'}}}' \
  | jq . > org3_update_in_envelope.json

# Encode final tx
configtxlator proto_encode --input org3_update_in_envelope.json \
  --type common.Envelope --output org3_update_in_envelope.pb

**5️⃣ Sign and Submit the Update**

**🔑 Org1 signs**

export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

peer channel signconfigtx -f org3_update_in_envelope.pb


**🔑 Org2 signs**

export CORE_PEER_LOCALMSPID=Org2MSP
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp

peer channel signconfigtx -f org3_update_in_envelope.pb


**📤 Submit update**

peer channel update -f org3_update_in_envelope.pb \
  -c $CHANNEL_NAME -o localhost:7050 \
  --ordererTLSHostnameOverride orderer.example.com \
  --tls --cafile $ORDERER_CA

**6️⃣ Start Org3 Peer Node**

Define docker-compose-org3.yaml (peer + optional CouchDB).

**📦 Command:**

docker-compose -f compose/docker/docker-compose-org3.yaml up -d

**7️⃣ Join Org3 to the Channel**

Set Org3 environment variables:

export CORE_PEER_LOCALMSPID=Org3MSP
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
export CORE_PEER_ADDRESS=localhost:11051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt


Fetch block and join:

peer channel fetch 0 mychannel.block \
  -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com \
  -c $CHANNEL_NAME --tls --cafile $ORDERER_CA

peer channel join -b mychannel.block

**8️⃣ Update Anchor Peer for Org3**

**📦 Command:**

configtxgen -outputAnchorPeersUpdate Org3MSPanchors.tx \
  -profile TwoOrgsChannel -asOrg Org3MSP -channelID $CHANNEL_NAME


**Submit update:**

peer channel update -o localhost:7050 \
  --ordererTLSHostnameOverride orderer.example.com \
  -c $CHANNEL_NAME -f Org3MSPanchors.tx \
  --tls --cafile $ORDERER_CA

**🎉 Org3 Successfully Added!**

Now Org3 can:

Install, approve, and commit chaincode

Query and submit transactions

Participate as a full member of the channel