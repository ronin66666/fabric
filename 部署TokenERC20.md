部署`token-erc20`

### 启动测试网络

进入`test-network`目录

```bash
cd fabric-samples/test-network
```

启用管理员，先关闭网络再启动并创建默认通道`myChannel`

```
sudo su
./network.sh down
./network.sh up createChannel
```

### 安装tokenERC20.go依赖

进入到`fabric-samples`项目的`toekn-erc-20/chaincode-go`目录

```bash
go mod vendor
```

### `peer`命令工具环境变量

进入`test-network`目录

```bash
cd fabric-samples/test-network
```

导出终端级别的`peer CLI`命令 

```bash
export PATH=${PWD}/../bin:$PATH
```

设置 `FABRIC_CFG_PATH`变量

```bash
export FABRIC_CFG_PATH=$PWD/../config/
```

检查peer版本

```bash
peer version
```

输出

```bash
peer:
 Version: 2.4.4
 Commit SHA: 1473ecae6
 Go version: go1.18.2
 OS/Arch: linux/amd64
 Chaincode:
  Base Docker Label: org.hyperledger.fabric
  Docker Namespace: hyperledger 
```

### 开启日志

```bash
cd fabric-samples/test-network
./monitordocker.sh fabric_test
```

### 创建链码包

使用 [peer lifecycle chaincode package](https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-package)来创建链码包

```bash
peer lifecycle chaincode package tokenERC20.tar.gz --path ../token-erc-20/chaincode-go/ --lang golang --label tokenERC20_1.0
```



### 安装链码包

安装链码到每个背书交易的节点，这里设置的背书策略为`Org1`和`Org2`，需要将链码安装到这两个组织的所有节点中

- peer0.org1.example.com
- peer0.org2.example.com

安装到`Org1`的节点中

`Org1`节点环境变量

```bash
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

使用`peer lifecycle chaincode install` 安装到系节点中， 该命令会返回package 的标识， package identifier 将用于后面的批准

```bash
peer lifecycle chaincode install tokenERC20.tar.gz
```

安装到`Org2`下的节点中

`Org2`节点环境变量

```bash
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

安装链码

```bash
peer lifecycle chaincode install tokenERC20.tar.gz
```

### 批准链码定义

查询已安装的链码

```bash
peer lifecycle chaincode queryinstalled
```

设置链码package ID 为环境变量

```bash
export CC_PACKAGE_ID=tokenERC20_1.0:218dac0c83471acc8b986ffb79eb648c628801716825fe4db258eadaef0fb502
```

授权链码

```bash
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name tokenERC20 --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```

`Org1`组织授权

环境设置

```bash
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:7051
```

授权

```bash
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name tokenERC20 --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```

### 提交

当有足够的组织授权成功后，其中一个组织可提交链码到通道中

查看授权状态

```bash
peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name tokenERC20 --version 1.0 --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --output json
```

提交链码

```bash
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name tokenERC20 --version 1.0 --sequence 1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"
```

查看已提交节点

```
peer lifecycle chaincode querycommitted --channelID mychannel --name tokenERC20 --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```

提交成功输出

```
Committed chaincode definition for chaincode 'tokenERC20' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
```



### 调用智能合约

初始化合约



` Mint(amount: int)`代币

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n tokenERC20 --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" -c '{"function":"Mint","Args":["300000000"]}'
```

查询`ClientAccountID`: 也作为支付地址

```
peer chaincode query -C mychannel -n tokenERC20 -c '{"function":"ClientAccountID","Args":[]}'
```

`Org1`

`eDUwOTo6Q049QWRtaW5Ab3JnMS5leGFtcGxlLmNvbSxPVT1hZG1pbixMPVNhbiBGcmFuY2lzY28sU1Q9Q2FsaWZvcm5pYSxDPVVTOjpDTj1jYS5vcmcxLmV4YW1wbGUuY29tLE89b3JnMS5leGFtcGxlLmNvbSxMPVNhbiBGcmFuY2lzY28sU1Q9Q2FsaWZvcm5pYSxDPVVT`

`eDUwOTo6Q049QWRtaW5Ab3JnMi5leGFtcGxlLmNvbSxPVT1hZG1pbixMPVNhbiBGcmFuY2lzY28sU1Q9Q2FsaWZvcm5pYSxDPVVTOjpDTj1jYS5vcmcyLmV4YW1wbGUuY29tLE89b3JnMi5leGFtcGxlLmNvbSxMPVNhbiBGcmFuY2lzY28sU1Q9Q2FsaWZvcm5pYSxDPVVT`

查询余额 `BalanceOf(account string)`

```
peer chaincode query -C mychannel -n tokenERC20 -c '{"function":"BalanceOf","Args":["eDUwOTo6Q049QWRtaW5Ab3JnMS5leGFtcGxlLmNvbSxPVT1hZG1pbixMPVNhbiBGcmFuY2lzY28sU1Q9Q2FsaWZvcm5pYSxDPVVTOjpDTj1jYS5vcmcxLmV4YW1wbGUuY29tLE89b3JnMS5leGFtcGxlLmNvbSxMPVNhbiBGcmFuY2lzY28sU1Q9Q2FsaWZvcm5pYSxDPVVT"]}'
```

查询`ClientAccountBalance`

```
peer chaincode query -C mychannel -n tokenERC20 -c '{"function":"ClientAccountBalance","Args":[]}'
```

