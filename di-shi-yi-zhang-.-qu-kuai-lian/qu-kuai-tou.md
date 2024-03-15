# 区块头

\
区块头部由如下表11-2所示的区块元数据组成。&#x20;

Table 11-2. 区块头部的结构

| 大小   | 字段                          | 描述                         |
| ---- | --------------------------- | -------------------------- |
| 4字节  | 版本(Version)                 | 最初是一个版本字段；随着时间的推移，它的用途已经发展 |
| 32字节 | 之前的块哈希(Previous Block Hash) | 链中前一个（父）块的哈希               |
| 32字节 | 默克尔根(Merkle Root)           | 该块交易的默克尔树根哈希               |
| 4字节  | 时间戳(Timestamp)              | 该区块的大致创建时间（Unix纪元时间）       |
| 4字节  | 目标(Target)                  | 该区块的工作量证明目标的紧凑编码           |
| 4字节  | 只用一次的随机数 (Nonce)            | 工作证明算法使用的任意数据              |

nonce（随机数）、目标值和时间戳在挖矿过程中使用，并将在第12章中进行更详细的讨论。