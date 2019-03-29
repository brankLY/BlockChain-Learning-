# 中本聪的比特币白皮书
>Satoshi Nakamoto satoshin@gmx.com www.bitcoin.org

##比特币网络相关数据结构解析
本文以技术设计和非技术设计两方面来分析，技术部分又以what，why，how三步来进行解析。
###技术方面
####相关数据结构
##### What 
######区块
区块大体分为两个部分，区块头和区块体。

区块头存储父块哈希，当前时间戳（timestamp），随机数，默克尔树（Merkle tree）根结点。

######区块链

由于每一个区块头存有上一个区块的哈希，在逻辑上所有区块成链式结构链接，所有区块链链接在一起即称为区块链。

###### 默克尔树（Merkle tree or Hash tree）
默克尔树即哈希树，由Ralph Merkle于1979年命名，其结构顾名思义，是由左右叶子结点哈希得到父结点，自下而上得来的，其叶子结点由实际数据块哈希得来，如下图。
![markdown](https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Hash_Tree.svg/620px-Hash_Tree.svg.png "HashTree")


###### 事务/交易（Transactions）
在比特币网络中transactions可直接理解为交易（其他区块链网络中往往扩展为事务），
是区块链的最小单元。
###### 事件序列（sequence）
##### Why
###### 默克尔树（Merkle tree）
Merkel tree是区块链世界非常重要的一个底层区块使用的数据结构，
由于区块体本身用于存储事务（具体见下），即使存储的事务是哈希过的64位哈希值，
其区块体的数据量也由于长期积累，变的异常巨大，那么在检索时便非常不方便，中本聪为了解决该问题使用了
默克尔树作为区块体内事务哈希的存储结构，并将根结点存在区块头。
###### 事务/交易（Transactions）
比特币网络是设计用来解决电子现金交易过程中的第三方问题的，其本身就是一个交易系统设计，故最小单元为一次交易。

##### How
以下内容主要来源于https://github.com/bitcoin/bitcoin源码解析

###### 事务/交易（Transactions）2019.03.29
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

######区块 2019.03.29
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

######区块链 2019.03.29
[源码定义](https://github.com/bitcoin/bitcoin/blob/master/src/primitives/block.h)
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