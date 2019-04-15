# Hyperledger Fabric
逻辑架构分为MSP（Membership service provider/成员用户服务）、区块链channel、链码/智能合约三部分
## MSP
### 描述
MSP将颁发与校验证书，以及用户认证背后的所有密码学机制与协议都抽象了出来。一个MSP可以自己定义身份，以及身份的管理（身份验证）与认证（生成与验证签名）规则。

在fabric区块链网络中，用户首先是在MSP进行注册，然后以MSP颁发的合法身份在整个网络上活动，因此MSP在某种程度上具有踢出(屏蔽)用户的权利，这也是目前联盟链和公链的最主要区别之一。

### 实现
MSP需要进行密钥和证书的生成，该部分一般是CA来完成的，fabric提供Hyperledger Fabric CA用于产生所需的密钥及证书。

每一个peer节点和orderer节点都需要在本地指定其配置，并在channel上启用peer节点、orderer节点及client的身份的验证与各自的签名验证。channel侧一般在network.yaml里进行配置来指定该org所使用的mspid。

在peer&orderer侧，需要对应MSP配置相关的证书和密钥。主要分为order和peer两部分。

## 区块链channel
### 描述
fabric的区块链网络是通过创建通道，加入节点，实例化智能合约后，客户端通过sdk调用智能合约来交互使用的。

channel是fabric区块链网络中账本的基础，每一个channel内的所有节点维护一个共享的账本，来进行

### 实现

channel配置中的

