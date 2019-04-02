# 中本聪的比特币白皮书
>Satoshi Nakamoto satoshin@gmx.com www.bitcoin.org

##比特币网络相关数据结构解析
本文以技术设计和非技术设计两方面来分析，技术部分又以what(是什么)，why(为什么需要)，how(怎么实现的)三步来进行解析。
##技术方面
###相关数据结构
数据结构部分只提取一些区块链世界较为普遍使用的结构或类似结构，挖坑相关的并未解析，后续可能会以POW机制相关的形式单独分析。
### What 
####区块
区块大体分为两个部分，区块头和区块体。

区块头存储父块哈希，当前时间戳（timestamp），随机数，默克尔树（Merkle tree）根结点。

####区块链
由于每一个区块头存有上一个区块的哈希，在逻辑上所有区块成链式结构链接，所有区块链链接在一起即称为区块链。
在实际存储时，将区块索引顺序存储在一起，见数据结构中CChain类。

#### 默克尔树（Merkle tree or Hash tree）
默克尔树即哈希树，由Ralph Merkle于1979年命名，其结构顾名思义，是由左右叶子结点哈希得到父结点，自下而上得来的，其叶子结点由实际数据块哈希得来，如下图。
![markdown](https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Hash_Tree.svg/620px-Hash_Tree.svg.png "HashTree")

#### merkleblock
默克尔树区块，用于响应SPV节点请求验证路径时的返回数据。

####节点
比特币网络节点主体分为全节点和轻量级节点/SPV（Simplified Payment Verification）节点。
全节点需要同步所有的区块信息，即拥有完整区块链账本的节点，全节点能够独立自主地校验所有交易，而不需借由任何外部参照。
SPV节点仅同步所有区块的区块头信息，不可以校验交易，只能验证支付。


#### 事务/交易（Transactions）
在比特币网络中transactions可直接理解为交易（其他区块链网络中往往扩展为事务），
是区块链的最小单元。

### Why

####区块
区块大体分为区块头和区块体，主要是由于区块体中包含交易相关信息，数据量相对庞大，
为了便于验证交易，将区块头和区块体拆开，用户可以通过只加载区块头来完成验证相关的基本操作，降低节点存储负担。
#### 默克尔树（Merkle tree）
Merkel tree是区块链世界非常重要的一个底层区块使用的数据结构，
由于区块体本身用于存储事务（具体见下），即使存储的事务是哈希过的64位哈希值，
其区块体的数据量也由于长期积累，变的异常巨大，那么在检索时便非常不方便，中本聪为了解决该问题使用了
默克尔树作为区块体内事务哈希的存储结构，并将根结点存在区块头。
#### merkleblock
默克尔树区块，用于响应SPV节点请求验证路径时的返回数据。当SPV节点进行验证请求验证路径时，
#### 事务/交易（Transactions）
比特币网络是设计用来解决电子现金交易过程中的第三方问题的，其本身就是一个交易系统设计，故最小单元为一次交易。
####节点
全节点作为比特币网络数据承载的物理单位，是比特币网络中最为重要的基本成员。
在比特币网络中，所有的全节点是平等的，具有同等的权利（记账和竞争挖矿），
其作用在于实现共识和通过庞大的数量来保证网络安全和去中心化可信。
### How
以下内容主要来源于https://github.com/bitcoin/bitcoin源码解析

#### 事务/交易（Transactions）2019.03.29
[源码定义](https://github.com/bitcoin/bitcoin/blob/master/src/primitives/transaction.h)
其设计实现主要分为CTransaction类，CTxIn类，CTxOut类，下面是CTransaction类（部分数据定义）：

| 数据项        |  数据类型  | 描述 |
| --------   | :----:  | :----:  |
| vin      |   vector<CTxIn>     |    输入    |
| vout        |   vector<CTxOut>   |    输出    |
| nVersion        |  int32_t  |    版本号   |
| nLockTime        |  uint32_t  |    交易锁定时间，默认是0（[用于延时交易](https://bitcoin.org/en/developer-guide#locktime-and-sequence-number))   |

由上表可见，该类主要定义了输入四个数据项，分别为输入vector、输出vector、版本号和交易锁定时间。

CTxIn类（部分数据定义）：

| 数据项        |  数据类型  | 描述 |
| --------   | :----:  | :----:  |
| prevout      |   COutPoint     |    上一个交易的输出    |
| scriptSig        |   CScript   |    脚本签名  （包含signature（付款人的私钥的签名） + pub key（付款人的公钥））  |
| nSequence        |  uint32_t  |    交易可替换的标志，默认是UINT_MAX（当nSequence为非最大值，可配合nLockTime来替换交易）   |
| scriptWitness        |  CScriptWitness  |    隔离见证(仅当交易被序列化时才参与,将scriptSig里的主要内容转移至该数据项，减少交易size)   |

CTxOut类（部分数据定义）：

| 数据项        | 大小(Byte)   |  数据类型  | 描述 |
| --------   | -----:  | :----:  | :----:  |
| nValue      | 64   |   CAmount     |    交易输出    |
| scriptPubKey|   Varies   |   CScript   |    收款公钥    |

####区块 2019.03.29
[源码定义](https://github.com/bitcoin/bitcoin/blob/master/src/primitives/block.h)
其设计实现主要分为CBlockHeader类，CBlock类，下面是CBlock类（部分数据定义）：

CBlock类继承CBlockHeader类（部分数据定义）：

| 数据项        |  数据类型  | 描述 |
| --------   | :----:  | :----:  |
| vtx      |   vector<CTransactionRef>     |    该块的交易集合    |
| fChecked|   bool   |    是否已经通过合法性验证（包括工作量检查验证和交易信息检查验证）    |

CBlockHeader类（部分数据定义）：

| 数据项        |   数据类型  | 描述 |
| --------   | :----:  | :----:  |
| nVersion      |    int32_t     |    版本号    |
| hashPrevBlock|   uint256   |  前一个区块头的哈希  |
| hashMerkleRoot | uint256   |    默克尔树（Merkle tree）根    |
| nTime|   uint32_t   |  时间戳  |
| nBits| uint32_t   |    挖坑难度    |
| nNonce|   uint32_t   |  随机数  |

####区块链 2019.03.29
[源码定义](https://github.com/bitcoin/bitcoin/blob/master/src/primitives/chain.h)
其设计实现主要分为CBlockFileInfo类，CBlockIndex类，CDiskBlockIndex类，CChain类。

CChain类（部分数据定义）：
| 数据项        |   数据类型  | 描述 |
| --------   | :----:  | :----:  |
| vChain      |    vector<CBlockIndex*>     |    所有的区块索引构成的一个vector   |

CDiskBlockIndex类继承CBlockIndex类（部分数据定义）：

| 数据项        |   数据类型  | 描述 |
| --------   | :----:  | :----:  |
| hashPrev      |    uint256     |    前任区块索引    |

CBlockIndex类（部分数据定义）：

| 数据项        |   数据类型  | 描述 |
| --------   | :----:  | :----:  |
| phashBlock      |    const uint256*     |    区块哈希值    |
| pprev      |    CBlockIndex*     |    前一个CBlockIndex指针    |
| pskip|   CBlockIndex*   |  更远处某个CBlockIndex指针  |
| nHeight | int   |    区块总数，也叫块高    |
| nFile|   int   |  存储文件在磁盘上的存储编号  |
| nDataPos| unsigned int   |    该区块在存储文件内部的序号    |
| nUndoPos|   unsigned int   |  该区块在撤销数据文件内部的序号  |
| nChainWork | arith_uint256   |    所有区块的工作量总和    |
| nTx|   unsigned int   |  该区块交易总数量  |
| nChainTx| unsigned int   |    所有区块的交易数量总数量    |
| nStatus|   uint32_t   |  本区块的校验状态  |
| nVersion      |    int32_t     |    版本号    |
| hashMerkleRoot | uint256   |    默克尔树（Merkle tree）根    |
| nTime|   uint32_t   |  时间戳  |
| nBits| uint32_t   |    挖坑难度    |
| nNonce|   uint32_t   |  随机数  |
| nSequenceId|   int32_t   |  区块序号  |
| nTimeMax|   unsigned int   |  所有区块的最大时间戳  |

CBlockFileInfo类（部分数据定义）：
主要用于存储block时判断当前文件是否可以有足够空间。

| 数据项        |   数据类型  | 描述 |
| --------   | :----:  | :----:  |
| nBlocks      |    unsigned int     |    文件中区块的数量    |
| nSize      |    unsigned int     |    区块文件中的字节数    |
| nUndoSize|   unsigned int   |  undo文件中的字节数  |
| nHeightFirst | unsigned int   |    该文件中最低的区块号    |
| nHeightLast|   unsigned int   |  该文件中最高的区块号  |
| nTimeFirst| uint64_t   |    该文件中最早的区块时间    |
| nTimeLast|   uint64_t   |  该文件中最后的区块时间  |

####默克尔树（Merkle tree）/merkleblock 2019.04.02
[源码定义](https://github.com/bitcoin/bitcoin/blob/master/src/merkleblock.h)
其设计实现主要分为CPartialMerkleTree类，CMerkleBlock类。

CPartialMerkleTree类（部分数据定义）：

| 数据项        |   数据类型  | 描述 |
| --------   | :----:  | :----:  |
| nTransactions      |    unsigned int     |    区块中交易的数量    |
| vBits      |    vector<bool>     |    用于生成Merkle Proof的Flag值列表    |
| vHash|   vector<uint256>   |  用于生成Merkle Proof的Hash值列表和  |
| fBad | bool   |    数据有效性标志位    |

CMerkleBlock类（部分数据定义）：

| 数据项        |   数据类型  | 描述 |
| --------   | :----:  | :----:  |
| header      |    CBlockHeader     |    区块头    |
| txn      |    CPartialMerkleTree     |    默克尔树路径    |
| vMatchedTxn|   vector<std::pair<unsigned int, uint256> >   |  布隆筛选符合的交易  |


