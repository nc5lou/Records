## Hyperledger Fabric V1.2 环境搭建

#### 1 新建用户

```shell
adduser fabric
usermod -aG sudo fabric
su fabric
cd ~
```



#### 2 依赖安装

1. 下载执行自动安装脚本

   ```shell
    curl -O https://hyperledger.github.io/composer/latest/prereqs-ubuntu.sh
    chmod u+x prereqs-ubuntu.sh
    ./prereqs-ubuntu.sh
   ```

   上述脚本会安装 node 8.x， git， npm，docker，docker-compose 等依赖环境



2. 除了上述环境外，我们还需要手动安装 golang 环境，目前 Fabric 最新版本（v1.4）要求 golang 1.2 版本

   ```shell
   wget https://dl.google.com/go/go1.12.6.linux-amd64.tar.gz
   tar -xvf go1.12.6.linux-amd64.tar.gz -C /usr/local
   # config the golang environment
   vi /etc/profile 添加 export PATH=$PATH:/usr/local/go/bin
   source /etc/profile
   vi ~/.bashrc 添加 export GOROOT=/usr/local/go/bin
   export GOPATH=/opt/goPath
   source ~/.bashrc
   ```



#### 3 配置Fabric

```shell
wget https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh
chmod u+x ./bootstrap.sh
./bootstarp.sh
```

上述命令会安装fabric必要的镜像，产生一个具有例程的 fabric-samples 文件夹



#### 4 fabric 工作流程

![preview](https://pic4.zhimg.com/v2-b87fa5e601c88e3937d5a9aed913dafb_r.jpg)

##### 一 . 生成必要文件

1. 根据 crypto-config.yaml 定义的网络拓扑结构生成必要的密钥和证书文件

   ```shell
   cryptogen generate --config=./crypto-config.yaml
   ```

   crypto-config.yaml 结构如下:

   ```yaml
   # Copyright IBM Corp. All Rights Reserved.
   #
   # SPDX-License-Identifier: Apache-2.0
   #
   
   # ---------------------------------------------------------------------------
   # "OrdererOrgs" - Definition of organizations managing orderer nodes
   # ---------------------------------------------------------------------------
   OrdererOrgs:
     # ---------------------------------------------------------------------------
     # Orderer
     # ---------------------------------------------------------------------------
     - Name: Orderer
       Domain: example.com
       # ---------------------------------------------------------------------------
       # "Specs" - See PeerOrgs below for complete description
       # ---------------------------------------------------------------------------
       Specs:
         - Hostname: orderer
   # ---------------------------------------------------------------------------
   # "PeerOrgs" - Definition of organizations managing peer nodes
   # ---------------------------------------------------------------------------
   PeerOrgs:
     # ---------------------------------------------------------------------------
     # Org1
     # ---------------------------------------------------------------------------
     - Name: Org1
       Domain: org1.example.com
       # ---------------------------------------------------------------------------
       # "Specs"
       # ---------------------------------------------------------------------------
       # Uncomment this section to enable the explicit definition of hosts in your
       # configuration.  Most users will want to use Template, below
       #
       # Specs is an array of Spec entries.  Each Spec entry consists of two fields:
       #   - Hostname:   (Required) The desired hostname, sans the domain.
       #   - CommonName: (Optional) Specifies the template or explicit override for
       #                 the CN.  By default, this is the template:
       #
       #                              "{{.Hostname}}.{{.Domain}}"
       #
       #                 which obtains its values from the Spec.Hostname and
       #                 Org.Domain, respectively.
       # ---------------------------------------------------------------------------
       # Specs:
       #   - Hostname: foo # implicitly "foo.org1.example.com"
       #     CommonName: foo27.org5.example.com # overrides Hostname-based FQDN set above
       #   - Hostname: bar
       #   - Hostname: baz
       # ---------------------------------------------------------------------------
       # "Template"
       # ---------------------------------------------------------------------------
       # Allows for the definition of 1 or more hosts that are created sequentially
       # from a template. By default, this looks like "peer%d" from 0 to Count-1.
       # You may override the number of nodes (Count), the starting index (Start)
       # or the template used to construct the name (Hostname).
       #
       # Note: Template and Specs are not mutually exclusive.  You may define both
       # sections and the aggregate nodes will be created for you.  Take care with
       # name collisions
       # ---------------------------------------------------------------------------
       Template:
         Count: 1
         # Start: 5
         # Hostname: {{.Prefix}}{{.Index}} # default
       # ---------------------------------------------------------------------------
       # "Users"
       # ---------------------------------------------------------------------------
       # Count: The number of user accounts _in addition_ to Admin
       # ---------------------------------------------------------------------------
       Users:
         Count: 1
   ```

   产生的 crypto-config 文件夹如下：

   ![1560762889933](C:\Users\Liuqi\AppData\Roaming\Typora\typora-user-images\1560762889933.png)

2. 根据 configtx.yaml 生成创世区块文件 genesis.block 

   ```shell
   # set the configtx.yaml location
   export FABRIC_CFG_PATH=${PWD}
   configtxgen -profile OneOrgOrdererGenesis -outputBlock ./config/genesis.block
   ```

   configtx.yaml 配置文件给出了网络的定义指出了每个网络实体的加密材料存储位置。其常见的结构如下：

   ```yaml
   # Copyright IBM Corp. All Rights Reserved.
   #
   # SPDX-License-Identifier: Apache-2.0
   #
   
   ---
   ################################################################################
   #
   #   Section: Organizations
   #
   #   - This section defines the different organizational identities which will
   #   be referenced later in the configuration.
   #
   ################################################################################
   Organizations:
   
       # SampleOrg defines an MSP using the sampleconfig.  It should never be used
       # in production but may be used as a template for other definitions
       - &OrdererOrg
           # DefaultOrg defines the organization which is used in the sampleconfig
           # of the fabric.git development environment
           Name: OrdererOrg
   
           # ID to load the MSP definition as
           ID: OrdererMSP
   
           # MSPDir is the filesystem path which contains the MSP configuration
           MSPDir: crypto-config/ordererOrganizations/example.com/msp
   
       - &Org1
           # DefaultOrg defines the organization which is used in the sampleconfig
           # of the fabric.git development environment
           Name: Org1MSP
   
           # ID to load the MSP definition as
           ID: Org1MSP
   
           MSPDir: crypto-config/peerOrganizations/org1.example.com/msp
   
           AnchorPeers:
               # AnchorPeers defines the location of peers which can be used
               # for cross org gossip communication.  Note, this value is only
               # encoded in the genesis block in the Application section context
               - Host: peer0.org1.example.com
                 Port: 7051
   
   ################################################################################
   #
   #   SECTION: Application
   #
   #   - This section defines the values to encode into a config transaction or
   #   genesis block for application related parameters
   #
   ################################################################################
   Application: &ApplicationDefaults
   
       # Organizations is the list of orgs which are defined as participants on
       # the application side of the network
       Organizations:
   
   ################################################################################
   #
   #   SECTION: Orderer
   #
   #   - This section defines the values to encode into a config transaction or
   #   genesis block for orderer related parameters
   #
   ################################################################################
   Orderer: &OrdererDefaults
   
       # Orderer Type: The orderer implementation to start
       # Available types are "solo" and "kafka"
       OrdererType: solo
   
       Addresses:
           - orderer.example.com:7050
   
       # Batch Timeout: The amount of time to wait before creating a batch
       BatchTimeout: 2s
   
       # Batch Size: Controls the number of messages batched into a block
       BatchSize:
   
           # Max Message Count: The maximum number of messages to permit in a batch
           MaxMessageCount: 10
   
           # Absolute Max Bytes: The absolute maximum number of bytes allowed for
           # the serialized messages in a batch.
           AbsoluteMaxBytes: 99 MB
   
           # Preferred Max Bytes: The preferred maximum number of bytes allowed for
           # the serialized messages in a batch. A message larger than the preferred
           # max bytes will result in a batch larger than preferred max bytes.
           PreferredMaxBytes: 512 KB
   
       Kafka:
           # Brokers: A list of Kafka brokers to which the orderer connects
           # NOTE: Use IP:port notation
           Brokers:
               - 127.0.0.1:9092
   
       # Organizations is the list of orgs which are defined as participants on
       # the orderer side of the network
       Organizations:
   
   ################################################################################
   #
   #   Profile
   #
   #   - Different configuration profiles may be encoded here to be specified
   #   as parameters to the configtxgen tool
   #
   ################################################################################
   Profiles:
   
       OneOrgOrdererGenesis:
           Orderer:
               <<: *OrdererDefaults
               Organizations:
                   - *OrdererOrg
           Consortiums:
               SampleConsortium:
                   Organizations:
                       - *Org1
       OneOrgChannel:
           Consortium: SampleConsortium
           Application:
               <<: *ApplicationDefaults
               Organizations:
                   - *Org1
   
   ```

   在config文件夹下生成了一个genesis.block文件。

3. 生成通道配置文件

   ```shell
   configtxgen -profile OneOrgChannel -outputCreateChannelTx ./config/channel.tx -channelID mychannel
   ```

4. 为 org 1 生成 peer anchor（若有多个修改输出文件名即可执行多次）

   ```shell
   configtxgen -profile OneOrgChannel -outputAnchorPeersUpdate ./config/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
   ```

   此时生成环节完成，config 文件夹下应该有 channel.tx ，genesis.block 文件以及组织数目的Org1MSPanchors.tx文件。该环节可以通过一个generate.sh生成。

##### 二. 启动网络，创建通道，加入节点

1. 启动网络

hyperledger fabric 通过 docker-compose 实现多个容器的编排执行

```shell
docker-compose -f docker-compose.yml up -d ca.example.com orderer.example.com peer0.org1.example.com couchdb
```

docker-compose.yml 文件如下：

```yaml
#
# Copyright IBM Corp All Rights Reserved
#
# SPDX-License-Identifier: Apache-2.0
#
version: '2'

networks:
  basic:

services:
  ca.example.com:
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.example.com
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/4239aa0dcd76daeeb8ba0cda701851d14504d31aad1b2ddddbac6a57365e497c_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start -b admin:adminpw'
    volumes:
      - ./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.example.com
    networks:
      - basic

  orderer.example.com:
    container_name: orderer.example.com
    image: hyperledger/fabric-orderer
    environment:
      - ORDERER_GENERAL_LOGLEVEL=info
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    ports:
      - 7050:7050
    volumes:
        - ./config/:/etc/hyperledger/configtx
        - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/:/etc/hyperledger/msp/orderer
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/:/etc/hyperledger/msp/peerOrg1
    networks:
      - basic

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    image: hyperledger/fabric-peer
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_PEER_ID=peer0.org1.example.com
      - CORE_LOGGING_PEER=info
      - CORE_CHAINCODE_LOGGING_LEVEL=info
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 7051:7051
      - 7053:7053
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/msp/peer
        - ./crypto-config/peerOrganizations/org1.example.com/users:/etc/hyperledger/msp/users
        - ./config:/etc/hyperledger/configtx
    depends_on:
      - orderer.example.com
      - couchdb
    networks:
      - basic

  couchdb:
    container_name: couchdb
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 5984:5984
    networks:
      - basic

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=info
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
      - CORE_CHAINCODE_KEEPALIVE=10
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
    networks:
        - basic
    #depends_on:
    #  - orderer.example.com
    #  - peer0.org1.example.com
    #  - couchdb
```

2. 创建通道，节点加入

   ```shell
   docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
   
   docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel join -b mychannel.block
   
   或者是
   # 进入 cli 容器
   docker exec -it cli bash
   # 设置环境变量
   export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
   export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
   export CORE_PEER_LOCALMSPID="Org1MSP"
   export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
   
   > peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
   > peer channel join -b mychannel.block
   
   ```

   ##### 三. 链码交互

   链码交互包括了链码的部署和调用。

   1，链码的部署(install 和 instantiate)

   ```shell
   # install
   peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
   # instantiate
   peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
   
   ```

   2， 链码的调用。调用包括两种：query（读操作，不写账本，不生成块） 和 invoke（写操作，可修改账本，能产生块。）

   ```shell
   # invoke
   peer chaincode invoke -o orderer.example.com:7050 -C $CHANNEL_NAME -n mycc -c  "{\"Args\":[\"invoke\",\"a\",\"b\",\"10\"]}"
   
   # query
   peer chaincode query -C $CHANNEL_NAME -n mycc -c "{\"Args\":[\"query\",\"a\"]}"
   ```

   

   #### 5 Hyperledger 相关

   1，容器