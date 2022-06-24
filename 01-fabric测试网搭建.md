Hyperledger Fabric test network

## 测试网搭建

### 环境安装

https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html

### 安装Fabric 和Fabric Samples

新建文件夹`fabirc`存放`fabirc`相关项目和脚本

下载脚本，并设置脚本文件可执行权限

```bash
curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh
```

可使用 `-h` 查看使用方法

使用`docker`拉取镜像和 `Fabric sample`

```bash
./install-fabric.sh docker samples
```

或者

```bash
./install-fabric.sh d s
```

也可以使用`--fabric-version`指定版本，比如下载`v2.2.1 binaries`使用命令

```
./install-fabric.sh --fabric-version 2.2.1 binary
```

###  运行测试网

#### 启动网络

进入`fabric-samples/test-network`目录

```bash
cd fabric-samples/test-network
```

 `network.sh`脚本使用`Docker`在本地启动测试网，使用`-h`查看帮助

停止网络，移除之前运行的容器

```bash
./network.sh down
```

启动网络

```
./network.sh up
```

改命令创建了两个`peer` 节点，一个`ordering`节点， 改命令默认使用`cryptogen tool` 来启动网络，也可以使用[Certificate Authorities](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html#bring-up-the-network-with-certificate-authorities)来启动

```bash
Using docker and docker-compose
Starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb' with crypto from 'cryptogen'
LOCAL_VERSION=2.4.4
DOCKER_IMAGE_VERSION=2.4.4
/home/lc/dev/fabirc/fabric-samples/test-network/../bin/cryptogen
Generating certificates using cryptogen tool
Creating Org1 Identities
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output=organizations
org1.example.com
+ res=0
Creating Org2 Identities
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output=organizations
org2.example.com
+ res=0
Creating Orderer Org Identities
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output=organizations
+ res=0
Generating CCP files for Org1 and Org2
Creating network "fabric_test" with the default driver
Creating volume "compose_orderer.example.com" with default driver
Creating volume "compose_peer0.org1.example.com" with default driver
Creating volume "compose_peer0.org2.example.com" with default driver
Creating peer0.org2.example.com ... done
Creating orderer.example.com    ... done
Creating peer0.org1.example.com ... done
Creating cli                    ... done
CONTAINER ID   IMAGE                               COMMAND             CREATED         STATUS                  PORTS                                                                                                                             NAMES
b0a9927c561e   hyperledger/fabric-tools:latest     "/bin/bash"         2 seconds ago   Up Less than a second                                                                                                                                     cli
bd6de50866b7   hyperledger/fabric-orderer:latest   "orderer"           6 seconds ago   Up 1 second             0.0.0.0:7050->7050/tcp, :::7050->7050/tcp, 0.0.0.0:7053->7053/tcp, :::7053->7053/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   orderer.example.com
9c87e1c6b615   hyperledger/fabric-peer:latest      "peer node start"   6 seconds ago   Up 2 seconds            0.0.0.0:7051->7051/tcp, :::7051->7051/tcp, 0.0.0.0:9444->9444/tcp, :::9444->9444/tcp                                              peer0.org1.example.com
c41706419bfb   hyperledger/fabric-peer:latest      "peer node start"   6 seconds ago   Up 2 seconds            0.0.0.0:9051->9051/tcp, :::9051->9051/tcp, 7051/tcp, 0.0.0.0:9445->9445/tcp, :::9445->9445/tcp                                    peer0.org2.example.com

```

使用`docker ps -a` 查看正在运行的容器

```bash
CONTAINER ID   IMAGE                               COMMAND             CREATED         STATUS         PORTS                                                                                                                             NAMES
b0a9927c561e   hyperledger/fabric-tools:latest     "/bin/bash"         7 minutes ago   Up 7 minutes                                                                                                                                     cli
bd6de50866b7   hyperledger/fabric-orderer:latest   "orderer"           7 minutes ago   Up 7 minutes   0.0.0.0:7050->7050/tcp, :::7050->7050/tcp, 0.0.0.0:7053->7053/tcp, :::7053->7053/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   orderer.example.com
9c87e1c6b615   hyperledger/fabric-peer:latest      "peer node start"   7 minutes ago   Up 7 minutes   0.0.0.0:7051->7051/tcp, :::7051->7051/tcp, 0.0.0.0:9444->9444/tcp, :::9444->9444/tcp                                              peer0.org1.example.com
c41706419bfb   hyperledger/fabric-peer:latest      "peer node start"   7 minutes ago   Up 7 minutes   0.0.0.0:9051->9051/tcp, :::9051->9051/tcp, 7051/tcp, 0.0.0.0:9445->9445/tcp, :::9445->9445/tcp                                    peer0.org2.example.com

```

该测试网包含两个peer 组织： Org1 和 Org2, 和一个维护排序服务(`ordering service`)的`orderer`组织

每个`peer`都属于一个组织，该测试网站每个组织操作一个`peer`节点：`peer0.org1.example.com` 和 `peer0.org2.example.com`

每个`fabric`网络都至少包含一个`ordering service`

#### 创建通道

使用`network.sh` 脚本创建为`Org1`和`Org2`创建一个通道，并把它们的`peer`加入到通道

```bash
./network.sh createChannel
```

命令运行成功输出：

```
Channel 'mychannel' joined
```

也可以使用 `-c`来自定义通道名

```bash
./network.sh createChannel -c channel1
```

可以创建多个通道，创建一个`channel2`的通道：

```
./network.sh createChannel -c channel2
```

也可以使用一条命令来启动网络和创建通道

```bash
./network.sh up createChannel
```

### 

### 安装`chaincode`到节点和部署到通道

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

`deployCC` ：安装`asset-transfer-basic`链码到节点`peer0.org1.example.com` and `peer0.org2.example.com`，和部署到指定通道，(如果没有指定则使用`mychannel`)，如果第一次部署`chaincode`，脚本会安装相关依赖

`--c`：指定安装`GO、typescript、或JavaScript `版本的`chaincode`，在`asset-transfer-basic`包含了相应的链码

### 与测试网交互

如果前面安装了（Samples，Binaries和Docker Images）在`fabirc-samples/bin/`目录中可以看到`peer`的二进制文件，进入到`test-network`目录中执行命令添加这些`binaries`到`CLI PATH`中

```bash
export PATH=${PWD}/../bin:$PATH
```

将`fabric-samples/core.yaml`文件添加`FABRIC_CFG_PATH`环境变量中

```bash
export FABRIC_CFG_PATH=$PWD/../config/
```

为了方便操作`peer`节点，设置`Org1`环境变量如下

```bash
# Environment variables for Org1
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

 `CORE_PEER_TLS_ROOTCERT_FILE`（安全传输根证书） 和 `CORE_PEER_MSPCONFIGPATH` 环境变量引用的是 `Org1 crypto material `在 `organizations` 目录中

初始化账本

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'

```

运行成功后输出：

```bash
CST 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 
```

查询账本数据

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}
```































