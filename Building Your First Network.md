# Building Your First Network

# 建立你的第一个网络

> **注意**
>
> 该文档说明已经在最新稳定版本的 Docker 镜像和提供的 tar 文件中的预编译设置工具中验证通过。如果你使用当前 master 分支上的镜像或工具来运行这些命令，你可能会看到一些配置错误。

构建您的第一个网络（BYFN）方案提供了一个示例Hyperledger Fabric网络，该网络由两个组织组成，每个组织维护两个 peer 节点。它还将默认部署“Solo” Ordering 服务，但其他 Ordering 服务依然可用。

## 安装先决条件

在我们开始之前，如果您还没有这样做，您可能希望检查您是否已在将要开发区块链应用程序和/或运行Hyperledger Fabric的平台上安装了所有[先决条件](https://hyperledger-fabric.readthedocs.io/en/release-1.4/prereqs.html)。

您还需要[安装样本，二进制文件和Docker镜像](https://hyperledger-fabric.readthedocs.io/en/release-1.4/install.html)。您会注意到`fabric-samples`存储库中包含许多样本。我们将使用该 `first-network` 样例。我们现在打开那个子目录。

```
cd fabric-samples/first-network
```

> 注意
>
> 本文档中提供的命令**必须**从存储库克隆的`first-network`子目录运行 `fabric-samples`。如果您选择从其他位置运行命令，则各种提供的脚本将无法找到二进制文件。

## 想现在运行吗？

我们提供了一个完全注释的脚本 - `byfn.sh` - 利用这些Docker镜像快速引导Hyperledger Fabric网络，该网络默认由四个代表两个不同组织的 peer 节点和一个 orderer 节点组成。它还将启动一个容器来运行脚本执行，该脚本执行将 peer 连接到 channel，部署 chaincode 并根据部署的 chaincode 驱动事务执行。

这是`byfn.sh`脚本的帮助文本：

```
Usage:
byfn.sh <mode> [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-l <language>] [-o <consensus-type>] [-i <imagetag>] [-v]"
  <mode> - one of 'up', 'down', 'restart', 'generate' or 'upgrade'"
    - 'up' - bring up the network with docker-compose up"
    - 'down' - clear the network with docker-compose down"
    - 'restart' - restart the network"
    - 'generate' - generate required certificates and genesis block"
    - 'upgrade'  - upgrade the network from version 1.3.x to 1.4.0"
  -c <channel name> - channel name to use (defaults to \"mychannel\")"
  -t <timeout> - CLI timeout duration in seconds (defaults to 10)"
  -d <delay> - delay duration in seconds (defaults to 3)"
  -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)"
  -s <dbtype> - the database backend to use: goleveldb (default) or couchdb"
  -l <language> - the chaincode language: golang (default), node, or java"
  -o <consensus-type> - the consensus-type of the ordering service: solo (default), kafka, or etcdraft"
  -i <imagetag> - the tag to be used to launch the network (defaults to \"latest\")"
  -v - verbose mode"
byfn.sh -h (print this message)"

Typically, one would first generate the required certificates and
genesis block, then bring up the network. e.g.:"

  byfn.sh generate -c mychannel"
  byfn.sh up -c mychannel -s couchdb"
  byfn.sh up -c mychannel -s couchdb -i 1.4.0"
  byfn.sh up -l node"
  byfn.sh down -c mychannel"
  byfn.sh upgrade -c mychannel"

Taking all defaults:"
      byfn.sh generate"
      byfn.sh up"
      byfn.sh down"
```

如果您选择不提供参数，则脚本将使用默认值执行。

### 生成网络配置文件

准备好了吗？好吧！执行以下命令：

```
./byfn.sh generate
```

您将看到有关将发生什么的简要说明，以及 Y/n 命令行提示。输入`y`或按回车键执行所描述的操作。

```
Generating certs and genesis block for channel 'mychannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y
proceeding ...
/Users/xxx/dev/fabric-samples/bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################
org1.example.com
2017-06-12 21:01:37.334 EDT [bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.
...

/Users/xxx/dev/fabric-samples/bin/configtxgen
##########################################################
#########  Generating Orderer Genesis block ##############
##########################################################
2017-06-12 21:01:37.558 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-06-12 21:01:37.562 EDT [msp] getMspConfig -> INFO 002 intermediate certs folder not found at [/Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts]. Skipping.: [stat /Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts: no such file or directory]
...
2017-06-12 21:01:37.588 EDT [common/configtx/tool] doOutputBlock -> INFO 00b Generating genesis block
2017-06-12 21:01:37.590 EDT [common/configtx/tool] doOutputBlock -> INFO 00c Writing genesis block

#################################################################
### Generating channel configuration transaction 'channel.tx' ###
#################################################################
2017-06-12 21:01:37.634 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-06-12 21:01:37.644 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2017-06-12 21:01:37.645 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

#################################################################
#######    Generating anchor peer update for Org1MSP   ##########
#################################################################
2017-06-12 21:01:37.674 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-06-12 21:01:37.678 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2017-06-12 21:01:37.679 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

#################################################################
#######    Generating anchor peer update for Org2MSP   ##########
#################################################################
2017-06-12 21:01:37.700 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```

第一步为我们的各个网络实体生成所有证书和密钥，生成的 `genesis block` 用于引导 ordering 服务，以及配置[Channel](https://hyperledger-fabric.readthedocs.io/en/release-1.4/glossary.html#channel) 所需的一组配置 transaction。

### 启动网络

接下来，您可以使用以下命令之一启动网络：

```
./byfn.sh up
```

上面的命令将编译Golang chaincode 镜像并启动相应的容器。Go是默认的 chaincode 语言，但是也支持[Node.js](https://fabric-shim.github.io/)和[Java](https://fabric-chaincode-java.github.io/) chaincode 。如果您想通过 node chaincode 运行本教程，请传递以下命令：

```
# we use the -l flag to specify the chaincode language
# forgoing the -l flag will default to Golang

./byfn.sh up -l node
```

> 注意
>
> 有关Node.js shim的更多信息，请参阅其 [文档](https://fabric-shim.github.io/ChaincodeInterface.html)。

> 注意
>
> 有关Java shim的更多信息，请参阅其 [文档](https://fabric-chaincode-java.github.io/org/hyperledger/fabric/shim/Chaincode.html)。

要使样例使用 Java chaincode 运行，您必须指定如下：`-l java`

```
./byfn.sh up -l java
```

> 注意
>
> 不要同时运行两个命令。除非您关闭并重新创建网络，否则只能尝试一种语言。

除了支持多种 chaincode 语言外，您还可以指定参数，该参数将启动五个节点Raft ordering 服务或一个Kafka ordering 服务，而不是一个节点Solo orderer。有关当前支持的 ordering 服务实施的更多信息，请查看[The Ordering Service](https://hyperledger-fabric.readthedocs.io/en/release-1.4/orderer/ordering_service.html)。

要使用Raft ordering 服务启动网络，请发出：

```
./byfn.sh up -o etcdraft
```

要使用Kafka ordering 服务启动网络，请发出：

```
./byfn.sh up -o kafka
```

再次，系统将提示您是继续还是中止。回复`y`或按回车键：

```
Starting for channel 'mychannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n]
proceeding ...
Creating network "net_byfn" with the default driver
Creating peer0.org1.example.com
Creating peer1.org1.example.com
Creating peer0.org2.example.com
Creating orderer.example.com
Creating peer1.org2.example.com
Creating cli


 ____    _____      _      ____    _____
/ ___|  |_   _|    / \    |  _ \  |_   _|
\___ \    | |     / _ \   | |_) |   | |
 ___) |   | |    / ___ \  |  _ <    | |
|____/    |_|   /_/   \_\ |_| \_\   |_|

Channel name : mychannel
Creating channel...
```

日志将从那里继续。这将启动所有容器，然后驱动完整的端到端应用程序方案。成功完成后，它应在终端窗口中报告以下内容：

```
Query Result: 90
2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
===================== Query successful on peer1.org2 on channel 'mychannel' =====================

===================== All GOOD, BYFN execution completed =====================


 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

您可以滚动浏览这些日志以查看各种事务。如果您没有得到这个结果，那么请跳到[故障排除](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html#troubleshoot)部分，看看我们是否可以帮助您发现问题所在。

### 关闭网络

最后，让我们把它全部关闭，这样我们就可以一步一步地探索网络设置。以下内容将终止您的容器，删除加密材料和四个工件，并从Docker注册表中删除 chaincode 镜像：

```
./byfn.sh down
```

再一次，系统将提示您继续，回复`y`或按回车键：

```
Stopping with channel 'mychannel' and CLI timeout of '10'
Continue? [Y/n] y
proceeding ...
WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
Removing network net_byfn
468aaa6201ed
...
Untagged: dev-peer1.org2.example.com-mycc-1.0:latest
Deleted: sha256:ed3230614e64e1c83e510c0c282e982d2b06d148b1c498bbdcc429e2b2531e91
...
```

如果您想了解有关底层工具和引导机制的更多信息，请继续阅读。在接下来的部分中，我们将介绍构建全功能Hyperledger Fabric网络的各种步骤和要求。

> 注意
>
> 下面概述的手动步骤假定`FABRIC_LOGGING_SPEC`在`cli`容器被设置为`DEBUG`。您可以通过修改`first-network`目录中的`docker-compose-cli.yaml`文件来设置此项。例如
>
> ```
> cli:
>   container_name: cli
>   image: hyperledger/fabric-tools:$IMAGE_TAG
>   tty: true
>   stdin_open: true
>   environment:
>     - GOPATH=/opt/gopath
>     - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
>     - FABRIC_LOGGING_SPEC=DEBUG
>     #- FABRIC_LOGGING_SPEC=INFO
> ```
>
## 加密生成器


我们将使用该`cryptogen`工具为各种网络实体生成加密材料（x509证书和签名密钥）。这些证书代表身份，它们允许在我们的实体进行通信和交易时进行签名/验证身份验证。

### 它是如何工作的？

Cryptogen使用文件 - `crypto-config.yaml`包含网络拓扑，并允许我们为组织和属于这些组织的成员生成一组证书和密钥。每个组织都配置了一个唯一的根证书（`ca-cert`），它将特定成员（peers 和 orders）绑定到该组织。通过为每个组织分配唯一的CA证书，我们模仿典型的网络，其中参与的[成员](https://hyperledger-fabric.readthedocs.io/en/release-1.4/glossary.html#member)将使用其自己的证书颁发机构。Hyperledger Fabric中的事务和通信由实体的私钥（`keystore`）签名，然后通过公钥（`signcerts`）进行验证。

您会注意到此文件中的变量`count`。我们用它来指定每个组织的 peer 数量; 在我们的例子中，每个组织有两个 peer 。我们现在不会深入研究[x.509证书和公钥基础设施](https://en.wikipedia.org/wiki/Public_key_infrastructure)的细枝末节 。如果您有兴趣，可以在自己的时间内仔细阅读这些主题。

运行该`cryptogen`工具后，生成的证书和密钥将保存到标题为`crypto-config`的文件夹中。请注意，该`crypto-config.yaml` 文件列出了与 orders Orgs 关联的五个 Order。虽然该 `cryptogen`工具将为所有这五个 Order 创建证书，但除非使用Raft或Kafka Ordering 服务，否则这些 Order 中只有一个将用于Solo Ordering 服务实施，并用于创建系统 channel 和`mychannel`。

## 配置事务生成器

该`configtxgen`工具用于创建四个配置工件：

> - orderer ，`genesis block`
> - channel ，`configuration transaction`
> - 两个`anchor peer transactions` - 每个 Peer Org 各一个

有关此工具功能的完整说明，请参阅[configtxgen](https://hyperledger-fabric.readthedocs.io/en/release-1.4/commands/configtxgen.html)。

orderer块是 ordering 服务的[Genesis Block](https://hyperledger-fabric.readthedocs.io/en/release-1.4/glossary.html#genesis-block)，channel 配置交易文件在[Channel](https://hyperledger-fabric.readthedocs.io/en/release-1.4/glossary.html#channel)创建时广播到 orderer。正如名称所暗示的那样，锚点对等事务(Anchor Peer Transaction)在此 channel 上指定每个Org的[Anchor Peer](https://hyperledger-fabric.readthedocs.io/en/release-1.4/glossary.html#anchor-peer)。

### 它是如何工作的？

Configtxgen使用一个文件 - `configtx.yaml`包含示例网络的定义。有三个成员 - 一个Orderer Org（`OrdererOrg`）和两个Peer Orgs （`Org1`＆`Org2`），每个Org管理和维护两个 peer 节点。该文件还指定了一个联盟- `SampleConsortium`由我们的两个Peer Orgs组成。请特别注意此文件底部的“Profiles”部分。您会注意到我们有几个独特的配置文件。一些值得注意的是：

- `TwoOrgsOrdererGenesis`：为Solo Ordering服务生成创世块。
- `SampleMultiNodeEtcdRaft`：为Raft Ordering服务生成创世块。仅在您发出`-o`标志并指定时使用`etcdraft`。
- `SampleDevModeKafka`：为Kafka Ordering 服务生成创世块。仅在您发出`-o`标志并指定时使用`kafka`。
- `TwoOrgsChannel`：为我们的频道`mychannel`生成创世块。

这些头文件很重要，因为我们在创建工件时会将它们作为参数传递。

> 注意
>
> 请注意，我们`SampleConsortium`在系统级配置文件中定义，然后由我们的通道级配置文件引用。channel 存在于一个联盟的范围内，所有联盟必须在整个网络范围内定义。

此文件还包含两个值得注意的其他规范。首先，我们为每个Peer Org（`peer0.org1.example.com`＆`peer0.org2.example.com`）指定锚点对等体（anchor peers）。其次，我们指向每个成员的MSP目录的位置，从而允许我们在orderer genesis块中存储每个Org的根证书。这是一个关键概念。现在，与ordering 服务的任何网络实体都可以验证其数字签名。


## 运行工具

您可以使用`configtxgen`和`cryptogen`命令手动生成证书/密钥和各种配置工件。或者，您可以尝试调整byfn.sh脚本来实现目标。

### 手动生成工件

您可以在byfn.sh脚本中引用该函数`generateCerts`，以获取生成将用于`crypto-config.yaml`文件中定义的网络配置的证书所需的命令。但是，为方便起见，我们也将在此提供参考。

首先让我们运行该`cryptogen`工具。我们的二进制文件位于`bin` 目录中，因此我们需要提供工具所在位置的相对路径。

```
../bin/cryptogen generate --config=./crypto-config.yaml
```

您应该在终端中看到以下内容：

```
org1.example.com
org2.example.com
```

证书和密钥（即MSP材料）将输出到目录 - `crypto-config`- 目录的根`first-network`目录。

接下来，我们需要告诉`configtxgen`工具在哪里查找`configtx.yaml`它需要摄取的 文件。我们将在目前的工作目录中告诉它：

```
export FABRIC_CFG_PATH=$PWD
```

然后，我们将调用该`configtxgen`工具来创建orderer genesis块：

```
../bin/configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
```

要输出Raft订购服务的创世块，此命令应为：

```
../bin/configtxgen -profile SampleMultiNodeEtcdRaft -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
```

请注意`SampleMultiNodeEtcdRaft`此处使用的配置文件。

要输出Kafka订购服务的创世块，请发出：

```
../bin/configtxgen -profile SampleDevModeKafka -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
```

如果您不使用Raft或Kafka，您应该看到类似于以下内容的输出：

```
2017-10-26 19:21:56.301 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
2017-10-26 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
2017-10-26 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block
```

> 注意
>
> orderer genesis块和我们即将创建的后续工件将输出到`channel-artifacts`该项目根目录的目录中。上述命令中的channelID是系统通道的名称。
>



### 创建通道配置事务（Channel Configuration Transaction）

接下来，我们需要创建通道事务工件。请务必替换`$CHANNEL_NAME`或设置`CHANNEL_NAME`为可在整个说明中使用的环境变量：

```
# The channel.tx artifact contains the definitions for our sample channel

export CHANNEL_NAME=mychannel  && ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```

请注意，如果您使用的是Raft或Kafka Ordering服务，则无需为该频道发出特殊命令。该`TwoOrgsChannel`配置文件将使用创建网络创世纪块时指定的Ordering服务的配置。

如果您没有使用Raft或Kafka订购服务，您应该在终端中看到类似于以下内容的输出：

```
2017-10-26 19:24:05.324 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
2017-10-26 19:24:05.329 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2017-10-26 19:24:05.329 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx
```

接下来，我们将在我们构建的通道上为Org1定义锚点对等体。同样，请确保替换`$CHANNEL_NAME`或设置以下命令的环境变量。终端输出将模仿通道事务工件的输出：

```
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
```

现在，我们将在同一个通道上为Org2定义锚点对等体：

```
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
```

## 启动网络

注意

如果您运行`byfn.sh`上面的示例，请确保在继续之前已关闭测试网络（请参阅 [关闭网络](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html#bring-down-the-network)）。

我们将利用脚本来启动我们的网络。docker-compose文件引用我们之前下载的镜像，并使用之前生成的`genesis.block`引导程序引导 orderer。

我们希望手动完成命令，以便公开每个调用的语法和功能。

首先让我们开始我们的网络：

```
docker-compose -f docker-compose-cli.yaml up -d
```

如果要查看网络的实时日志，请不要提供该`-d`标志。如果您让日志流，那么您将需要打开第二个终端来执行CLI调用。



### 创建和加入 Channel

回想一下，我们使用上面`configtxgen`“ [创建通道配置事务”](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html#createchanneltx)部分中的 工具[创建了通道配置事务](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html#createchanneltx)。您可以使用`configtx.yaml`传递给`configtxgen`工具的相同或不同的配置文件重复该过程以创建其他通道配置事务。然后，您可以重复本节中定义的过程，以在您的网络中建立其他通道。

我们将使用以下命令进入CLI容器：`docker exec`

```
docker exec -it cli bash
```

如果成功，您应该看到以下内容：

```
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

要使以下CLI命令起作用，我们需要在命令前加上下面给出的四个环境变量。这些变量`peer0.org1.example.com`被烘焙到CLI容器中，因此我们可以在不传递它们的情况下进行操作。**但是**，如果要将调用发送到其他peers或orderer，请在进行任何CLI调用时覆盖环境变量，如下例所示：

```
# Environment variables for PEER0

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

接下来，我们将作为创建channel请求的一部分，将我们在[创建通道配置事务](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html#createchanneltx)部分（我们称之为“ [创建通道配置事务”](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html#createchanneltx)部分（我们称之为`channel.tx`））中生成的通道配置事务工件传递给orderer。

我们使用`-c`标志指定通道名称，使用`-f`标志指定通道配置事务。在这种情况下是`channel.tx`，但您可以使用其他名称挂载自己的配置事务。我们将再次`CHANNEL_NAME`在CLI容器中设置环境变量，以便我们不必显式传递此参数。通道名称必须全部小写，长度小于250个字符并与正则表达式匹配 `[a-z][a-z0-9.-]*`。

```
export CHANNEL_NAME=mychannel

# the channel.tx file is mounted in the channel-artifacts directory within your CLI container
# as a result, we pass the full path for the file
# we also pass the path for the orderer ca-cert in order to verify the TLS handshake
# be sure to export or replace the $CHANNEL_NAME variable appropriately

peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

> 注意
>
> 请注意`--cafile`我们作为此命令的一部分传递。它是orderer的根证书的本地路径，允许我们验证TLS握手。
>

此命令返回一个创世块 - `<CHANNEL_NAME.block>`我们将用它来加入频道。它包含指定的配置信息`channel.tx` 如果您没有对默认通道名称进行任何修改，那么该命令将返回一个标题为`mychannel.block`的原型。

> 注意
>
> 您将保留在CLI容器中以获取其余这些手动命令。您还必须记住在定位除了`peer0.org1.example.com`以外的peer 时，所有命令都带有相应的环境变量 。
>

现在让我们让`peer0.org1.example.com`加入 channel 吧。

```
# By default, this joins ``peer0.org1.example.com`` only
# the <CHANNEL_NAME.block> was returned by the previous command
# if you have not modified the channel name, you will join with mychannel.block
# if you have created a different channel name, then pass in the appropriately named block

 peer channel join -b mychannel.block
```

您可以根据需要通过对我们在上面的“ [创建和加入通道”](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html#peerenvvars) 部分中使用的四个环境变量进行适当更改来使其他对等方加入channel。

我们将简单地让`peer0.org2.example.com`加入，而不是加入每个对等体，以便我们可以正确地更新通道中的锚点对等体定义。由于我们将覆盖CLI容器中的默认环境变量，因此这个完整命令将如下所示：

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:9051 
CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt 
peer channel join -b mychannel.block
```

> 注意
>
> 在v1.4.1之前，docker网络中的所有对等端都使用了端口`7051`。如果在v1.4.1之前使用fabric-samples版本，请修改`CORE_PEER_ADDRESS`本教程中的所有实例以使用端口`7051`。
>

或者，您可以选择单独设置这些环境变量，而不是传入整个字符串。一旦设置完毕，您只需再次发出命令`peer channel join`，CLI容器将代表`peer0.org2.example.com`。

### 更新锚点对等点（anchor peers）

以下命令是通道更新，它们将传播到通道的定义。实质上，我们在通道的创世块之上添加了额外的配置信息。请注意，我们不是修改genesis块，而是简单地将增量添加到将定义锚点对等体的链中。

更新通道定义以将Org1的锚点对等体定义为`peer0.org1.example.com`：

```
peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

现在更新通道定义以将Org2的锚点对等体定义为`peer0.org2.example.com`。与Org2对等体`peer channel join`的命令相同，我们需要在此调用前加上适当的环境变量。

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:9051 
CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt 

peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

### 安装和实例化Chaincode

> 注意
>
> 我们将使用一个简单的现有链码。要了解如何编写自己的链代码，请参阅[Chaincode for Developers](https://hyperledger-fabric.readthedocs.io/en/release-1.4/chaincode4ade.html)教程。
>

应用程序通过`chaincode`与区块链账本交互。因此，我们需要在每个将执行和为 transaction 背书的peer 上安装链码，然后在通道上实例化链码。

首先，将示例Go，Node.js或Java链码安装到Org1中的peer0节点上。这些命令将指定的源代码放在我们的peer的文件系统上。

> 注意
>
> 每个链代码名称和版本只能安装一个版本的源代码。源代码存在于对等体的文件系统中，在链代码名称和版本的上下文中; 它与语言无关。类似地，实例化的链码容器将反映peer上安装的是哪种语言。
>

**Golang**

```
# this installs the Go chaincode. For go chaincode -p takes the relative path from $GOPATH/src
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```

**Node.js**

```
# this installs the Node.js chaincode
# make note of the -l flag to indicate "node" chaincode
# for node chaincode -p takes the absolute path to the node.js chaincode
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/
```

**Java**

```
# make note of the -l flag to indicate "java" chaincode
# for java chaincode -p takes the absolute path to the java chaincode
peer chaincode install -n mycc -v 1.0 -l java -p /opt/gopath/src/github.com/chaincode/chaincode_example02/java/
```

当我们在渠道上实例化链码时，背书政策将被设置为要求Org1和Org2中的对等方的认可。因此，我们还需要在Org2中的对等体上安装链码。

修改以下四个环境变量以在Org2中针对peer0发出install命令：

```
# Environment variables for PEER0 in Org2

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ADDRESS=peer0.org2.example.com:9051
CORE_PEER_LOCALMSPID="Org2MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```

现在将示例Go，Node.js或Java链码安装到Org2中的peer0上。这些命令将指定的源代码味道放在我们的对等文件系统上。

**Golang**

```
# this installs the Go chaincode. For go chaincode -p takes the relative path from $GOPATH/src
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```

**Node.js**

```
# this installs the Node.js chaincode
# make note of the -l flag to indicate "node" chaincode
# for node chaincode -p takes the absolute path to the node.js chaincode
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/
```

**Java**

```
# make note of the -l flag to indicate "java" chaincode
# for java chaincode -p takes the absolute path to the java chaincode
peer chaincode install -n mycc -v 1.0 -l java -p /opt/gopath/src/github.com/chaincode/chaincode_example02/java/
```

接下来，在通道上实例化链码。这将初始化通道上的链代码，设置链码的认可策略，并为目标对等方启动链代码容器。注意这个`-P` 论点。这是我们的政策，我们指定针对要验证的此链码的交易所需的认可级别。

在下面的命令中，您会注意到我们将策略指定为`-P "AND ('Org1MSP.peer','Org2MSP.peer')"` 。这意味着我们需要来自属于Org1 **和** Org2 的对等方的“认可” （即两个认可）。如果我们改变语法`OR`，那么我们只需要一个认可。

**Golang**

```
# be sure to replace the $CHANNEL_NAME environment variable if you have not exported it
# if you did not install your chaincode with a name of mycc, then modify that argument as well

peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

**Node.js**

> 注意
>
> Node.js链代码的实例化大约需要一分钟。命令没有挂; 而是在编译图像时安装fabric-shim图层。
>

```
# be sure to replace the $CHANNEL_NAME environment variable if you have not exported it
# if you did not install your chaincode with a name of mycc, then modify that argument as well
# notice that we must pass the -l flag after the chaincode name to identify the language

peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l node -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

**Java**

> 注意
>
> 请注意，Java链代码实例化可能需要一些时间，因为它编译链代码并使用java环境下载docker容器。
>

```
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l java -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"
```

有关背书策略实施的更多详细信息，请参阅[认可政策](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)文档。

如果您希望其他 peers 与ledger进行交互，则需要将它们连接到通道，并将链代码源的相同名称，版本和语言安装到相应的对等文件系统上。一旦他们尝试与特定的链代码进行交互，就会为每个对等体启动一个链代码容器。再次，要认识到Node.js镜像的编译速度会慢一些。

一旦链代码在通道上实例化，我们就可以放弃`l` 标志。我们只需传递 ChannelID 和链码的名称。

### 查询

让我们查询`a`的值，以确保链代码已正确实例化并填充状态数据库。查询语法如下：

```
# be sure to set the -C and -n flags appropriately

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```

### 调用

现在让我们转移`10`从`a`到`b`。此事务将剪切新块并更新状态DB。调用的语法如下：

```
# be sure to set the -C and -n flags appropriately

peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
```

### 查询

让我们确认我们之前的调用是否正确执行。我们用一个值`100`初始化了键`a`，并且刚刚使用我们之前的调用删除了`10`。因此，`a`应该返回一个查询`90`。查询的语法如下。

```
# be sure to set the -C and -n flags appropriately

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```

我们应该看到以下内容：

```
Query Result: 90
```

随意重新开始并操纵键值对和后续调用。

### 安装

现在我们将在第三个对等体，Org2中的peer1上安装链代码。修改以下四个环境变量，以便在Org2中针对peer1发出install命令：

```
# Environment variables for PEER1 in Org2

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_ADDRESS=peer1.org2.example.com:10051
CORE_PEER_LOCALMSPID="Org2MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt
```

现在将示例Go，Node.js或Java链代码安装到Org2中的peer1上。这些命令将指定的源代码味道放在我们的对等文件系统上。

**Golang**

```
# this installs the Go chaincode. For go chaincode -p takes the relative path from $GOPATH/src
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```

**Node.js**

```
# this installs the Node.js chaincode
# make note of the -l flag to indicate "node" chaincode
# for node chaincode -p takes the absolute path to the node.js chaincode
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/
```

**Java**

```
# make note of the -l flag to indicate "java" chaincode
# for java chaincode -p takes the absolute path to the java chaincode
peer chaincode install -n mycc -v 1.0 -l java -p /opt/gopath/src/github.com/chaincode/chaincode_example02/java/
```

### 查询

让我们确认我们可以在Org2中向Peer1发出查询。我们用一个值`100`初始化了键`a`，并且刚刚使用我们之前的调用删除了`10`。因此，查询`a`仍应返回`90`。

org2中的peer1必须首先加入该通道才能响应查询。可以通过发出以下命令来连接通道：

```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer1.org2.example.com:10051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt peer channel join -b mychannel.block
```

返回join命令后，可以发出查询。查询的语法如下。

```
# be sure to set the -C and -n flags appropriately

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```

我们应该看到以下内容：

```
Query Result: 90
```

随意重新开始并操纵键值对和后续调用。



### 幕后发生了什么？

> 注意
>
> 这些步骤描述了`script.sh`由'./byfn.sh up'运行的场景 。使用`./byfn.sh down`并清除网络并确保此命令处于活动状态。然后使用相同的docker-compose提示再次启动您的网络
>

- 脚本 - `script.sh`- 在CLI容器中执行。该脚本根据提供的通道名称驱动命令`createChannel`，并使用channel.tx文件进行通道配置。
- `createChannel`输出是一个创世块 - `<your_channel_name>.block`它存储在对等体的文件系统中，并包含从channel.tx指定的通道配置。
- `joinChannel`对所有四个对等体执行该命令，其将先前生成的生成块作为输入。此命令指示对等体加入`<your_channel_name>`并创建以`<your_channel_name>.block`开头的链。
- 现在我们有一个由四个peer和两个组织组成的频道。这是我们的`TwoOrgsChannel` profile。
- `peer0.org1.example.com`和`peer1.org1.example.com`属于Org1; `peer0.org2.example.com`和`peer1.org2.example.com`属于Org2
- 这些关系是通过`crypto-config.yaml`定义的和MSP路径是由我们的docker compose中指定的。
- 然后更新Org1MSP（`peer0.org1.example.com`）和Org2MSP（`peer0.org2.example.com`）的锚点对等体。我们通过将`Org1MSPanchors.tx`和`Org2MSPanchors.tx`工件传递给ordering服务以及我们的channel名称来实现这一点。
- 链码 - **chaincode_example02** - 安装在`peer0.org1.example.com`和 `peer0.org2.example.com`
- 然后将链代码“实例化” `mychannel`。实例化将链代码添加到通道，启动目标对等方的容器，并初始化与链代码关联的键值对。该示例的初始值为[“a”，“100”“b”，“200”]。这种“实例化”通过`dev-peer0.org2.example.com-mycc-1.0`起始名称产生一个容器。
- 实例化也传递了背书政策的论据。策略定义为 ，意味着任何事务必须由绑定到Org1和Org2的对等方签署。`-P "AND ('Org1MSP.peer','Org2MSP.peer')"`
- 发出针对“a”的值的查询`peer0.org2.example.com`。`dev-peer0.org2.example.com-mycc-1.0` 在实例化链代码时，启动了名称为Org2 peer0的容器。返回查询的结果。没有发生写入操作，因此对“a”的查询仍将返回值“100”。
- 一个调用被发送到`peer0.org1.example.com`和`peer0.org2.example.com` 从“a”到“b”的移动“10”
- 将查询发送给`peer0.org2.example.com`“a”的值。返回值90，正确反映上一个事务，在此事务期间，键“a”的值被修改为10。
- 链码 - **chaincode_example02** - 已安装在`peer1.org2.example.com`
- 将查询发送给`peer1.org2.example.com`“a”的值。这将启动名称为的第三个链代码容器`dev-peer1.org2.example.com-mycc-1.0`。返回值90，正确反映上一个事务，在此事务期间，键“a”的值被修改为10。

### 这证明了什么？

Chaincode **必须**安装在对等体上，以便它能够成功地对分类帐执行读/写操作。此外，`init`在针对该链代码执行传统事务（读/写）之前，不会为对等启动链代码容器（例如，查询“a”的值）。该transaction导致容器启动。此外，通道中的所有对等体都保持账本的特定副本，其包括用于以块的形式存储不可变的有序记录的区块链，以及用于维护当前状态的快照的状态数据库。这包括那些没有安装链代码的对等体（`peer1.org1.example.com`如上例所示）。最后，链码在安装后可以访问（比如`peer1.org2.example.com` 在上面的例子中）因为它已经被实例化了。

### 我如何看到这些交易？

检查CLI Docker容器的日志。

```
docker logs -f cli
```

您应该看到以下输出：

```
2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
Query Result: 90
2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
===================== Query successful on peer1.org2 on channel 'mychannel' =====================

===================== All GOOD, BYFN execution completed =====================


 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

您可以滚动浏览这些日志以查看各种事务。

### 如何查看链码日志？

检查各个链代码容器以查看针对每个容器执行的单独事务。以下是每个容器的组合输出：

```
$ docker logs dev-peer0.org2.example.com-mycc-1.0
04:30:45.947 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
ex02 Init
Aval = 100, Bval = 200

$ docker logs dev-peer0.org1.example.com-mycc-1.0
04:31:10.569 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
ex02 Invoke
Query Response:{"Name":"a","Amount":"100"}
ex02 Invoke
Aval = 90, Bval = 210

$ docker logs dev-peer1.org2.example.com-mycc-1.0
04:31:30.420 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
ex02 Invoke
Query Response:{"Name":"a","Amount":"90"}
```

## 了解Docker Compose拓扑

BYFN示例为我们提供了两种Docker Compose文件，这两种文件都是从`docker-compose-base.yaml`（位于`base` 文件夹中）扩展而来的。我们的第一个文件，`docker-compose-cli.yaml`为我们提供了一个CLI容器，以及一个orderer，四个peer。我们将此文件用于此页面上的所有说明。

> 注意
>
> 本节的其余部分介绍了为SDK设计的docker-compose文件。 有关运行这些测试的详细信息，请参阅[Node SDK](https://github.com/hyperledger/fabric-sdk-node) repo。
>

`docker-compose-e2e.yaml`构建第二种风格，使用Node.js SDK运行端到端测试。除了使用SDK之外，它的主要区别在于Fabric-ca服务器还有容器。因此，我们能够将REST调用发送到组织CA以进行用户注册和注册。

如果你想在`docker-compose-e2e.yaml`没有先运行byfn.sh脚本的情况下使用它，那么我们需要进行四次略微的修改。我们需要指向组织CA的私钥。您可以在crypto-config文件夹中找到这些值。例如，要找到Org1的私钥，我们将遵循此路径 - `crypto-config/peerOrganizations/org1.example.com/ca/`。私钥是一个长哈希值后跟`_sk`。Org2的路径是 - `crypto-config/peerOrganizations/org2.example.com/ca/`。

在`docker-compose-e2e.yaml`更新caAB和ca1的FABRIC_CA_SERVER_TLS_KEYFILE变量。您还需要编辑命令中提供的路径以启动ca服务器。您为每个CA容器提供两次相同的私钥。

## 使用CouchDB

状态数据库可以从默认（goleveldb）切换到CouchDB。CouchDB提供了相同的链代码功能，但是，根据链代码数据被建模为JSON，还可以对状态数据库数据内容执行丰富而复杂的查询。

要使用CouchDB而不是默认数据库（goleveldb），请按照前面概述的相同步骤生成工件，除非在启动网络传递时`docker-compose-couch.yaml`也是如此：

```
docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d
```

**chaincode_example02**现在应该使用下面的CouchDB。

注意

如果您选择实现fabric-couchdb容器端口到主机端口的映射，请确保您了解安全隐患。在开发环境中映射端口使CouchDB REST API可用，并允许通过CouchDB Web界面（Fauxton）可视化数据库。生产环境可能会避免实施端口映射，以限制对CouchDB容器的外部访问。

您可以使用上面列出的步骤对CouchDB状态数据库使用**chaincode_example02链**代码，但是为了运用CouchDB查询功能，您需要使用具有建模为JSON的数据的链代码（例如**marbles02**）。您可以在目录中找到**marbles02**链代码 `fabric/examples/chaincode/go`。

我们将按照相同的过程创建和加入频道，如上面的 createandjoin部分所述。将对等方加入频道后，请使用以下步骤与**marbles02**链代码进行交互：

- 安装并实例化链代码`peer0.org1.example.com`：

```
# be sure to modify the $CHANNEL_NAME variable accordingly for the instantiate command

peer chaincode install -n marbles -v 1.0 -p github.com/chaincode/marbles02/go
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.peer','Org1MSP.peer')"
```

- 制作一些大理石并移动它们：

```
# be sure to modify the $CHANNEL_NAME variable accordingly

peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'
```

- 如果您选择在docker-compose中映射CouchDB端口，现在可以通过打开浏览器并导航到以下URL，通过CouchDB Web界面（Fauxton）查看状态数据库：

  `http://localhost:5984/_utils`

您应该看到一个名为`mychannel`（或您的唯一通道名称）的数据库及其中的文档。

注意

对于以下命令，请务必适当更新$ CHANNEL_NAME变量。

您可以从CLI运行常规查询（例如，阅读`marble2`）：

```
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'
```

输出应显示以下内容的详细信息`marble2`：

```
Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}
```

您可以检索特定大理石的历史记录 - 例如`marble1`：

```
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'
```

输出应显示以下事务`marble1`：

```
Query Result: [{"TxId":"1c3d3caf124c89f91a4c0f353723ac736c58155325f02890adebaa15e16e6464", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}},{"TxId":"755d55c281889eaeebf405586f9e25d71d36eb3d35420af833a20a2f53a3eefd", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}},{"TxId":"819451032d813dde6247f85e56a89262555e04f14788ee33e28b232eef36d98f", "Value":}]
```

您还可以对数据内容执行丰富的查询，例如按所有者查询大理石字段`jerry`：

```
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'
```

输出应显示拥有的两个大理石`jerry`：

```
Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}},{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":70}}]
```

## 为何选择CouchDB

CouchDB是一种NoSQL解决方案。它是一个面向文档的数据库，其中文档字段存储为键值映射。字段可以是简单的键值对，列表或映射。除了LevelDB支持的键控/复合键/键范围查询之外，CouchDB还支持完全数据丰富的查询功能，例如针对整个区块链数据的非键查询，因为其数据内容以JSON格式存储，完全可查询。因此，CouchDB可以满足LevelDB不支持的许多用例的链代码，审计和报告要求。

CouchDB还可以增强区块链中的合规性和数据保护的安全性。因为它能够通过过滤和屏蔽事务中的各个属性来实现字段级安全性，并且只在需要时授权只读权限。

此外，CouchDB属于CAP定理的AP类型（可用性和分区容差）。它使用主 - 主复制模型。可以[在CouchDB文档的Eventual Consistency页面上](http://docs.couchdb.org/en/latest/intro/consistency.html)找到更多信息 。但是，在每个结构对等体下，没有数据库副本，对数据库的写入保证一致且持久（不）。`Eventual Consistency``Eventual Consistency`

CouchDB是Fabric的第一个外部可插拔状态数据库，可以而且应该有其他外部数据库选项。例如，IBM为其区块链启用了关系数据库。并且CP类型（一致性和分区容差）数据库也可能需要，以便在没有应用程序级别保证的情况下实现数据一致性。

## 关于数据持久性的注记

如果在对等容器或CouchDB容器上需要数据持久性，则可以选择将docker-host中的目录挂载到容器中的相关目录中。例如，您可以在`docker-compose-base.yaml`文件中的对等容器规范中添加以下两行：

```
volumes:
 - /var/hyperledger/peer0:/var/hyperledger/production
```

对于CouchDB容器，您可以在CouchDB容器规范中添加以下两行：

```
volumes:
 - /var/hyperledger/couchdb0:/opt/couchdb/data
```



## 故障排除

- 始终保持网络新鲜。使用以下命令删除工件，加密，容器和链代码图像：

  ```
  ./byfn.sh down
  ```

  注意

  如果不删除旧容器和图像，您**将**看到错误。

- 如果您看到Docker错误，请首先检查您的docker版本（[先决条件](https://hyperledger-fabric.readthedocs.io/en/release-1.4/prereqs.html)），然后尝试重新启动Docker进程。Docker的问题通常无法立即识别。例如，您可能会看到因无法访问容器中安装的加密材料而导致的错误。

  如果他们坚持删除你的图像并从头开始：

  ```
  docker rm -f $(docker ps -aq)
  docker rmi -f $(docker images -q)
  ```

- 如果在创建，实例化，调用或查询命令中看到错误，请确保已正确更新通道名称和链代码名称。提供的示例命令中有占位符值。

- 如果您看到以下错误：

  ```
  Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)
  ```

  您可能拥有先前运行的链代码图像（例如`dev-peer1.org2.example.com-mycc-1.0`或`dev-peer0.org1.example.com-mycc-1.0`）。删除它们然后再试一次。

  ```
  docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')
  ```

- 如果您看到类似于以下内容的内容：

  ```
  Error connecting: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
  Error: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
  ```

  确保您正在使用已被重新标记为“最新”的“1.0.0”图像运行您的网络。

- 如果您看到以下错误：

  ```
  [configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
  panic: Error reading configuration: Unsupported Config Type ""
  ```

  然后你没有`FABRIC_CFG_PATH`正确设置环境变量。configtxgen工具需要此变量才能找到configtx.yaml。返回并执行一个，然后重新创建您的通道工件。`export FABRIC_CFG_PATH=$PWD`

- 要清理网络，请使用以下`down`选项：

  ```
  ./byfn.sh down
  ```

- 如果您看到一条错误，指出您仍然有“活动端点”，则修剪您的Docker网络。这将擦除以前的网络，并为您提供一个全新的环境：

  ```
  docker network prune
  ```

  您将看到以下消息：

  ```
  WARNING! This will remove all networks not used by at least one container.
  Are you sure you want to continue? [y/N]
  ```

  选择`y`。

- 如果您看到类似于以下内容的错误：

  ```
  /bin/bash: ./scripts/script.sh: /bin/bash^M: bad interpreter: No such file or directory
  ```

  确保有问题的文件（本例中为**script.sh**）以Unix格式编码。这是最有可能通过不设置导致`core.autocrlf`对`false`使用Git的配置（见 [的Windows演员](https://hyperledger-fabric.readthedocs.io/en/release-1.4/prereqs.html#windows-extras)）。有几种方法可以解决这个问题。例如，如果您有权访问vim编辑器，请打开文件：

  ```
  vim ./fabric-samples/first-network/scripts/script.sh
  ```

  然后通过执行以下vim命令更改其格式：

  ```
  :set ff=unix
  ```

注意

如果您继续看到错误，请在[Hyperledger Rocket Chat](https://chat.hyperledger.org/home) 或[StackOverflow](https://stackoverflow.com/questions/tagged/hyperledger-fabric)上的**fabric-questions**通道上 共享您的日志 。