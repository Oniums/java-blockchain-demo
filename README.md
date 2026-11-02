# Java 区块链与挖矿演示

大学课程设计的 Java 学习项目，用简化的区块链展示 SHA-256 哈希、工作量证明（PoW）、JSON 持久化和局域网节点广播。

> 状态：历史课设。当前快照缺少 `Block` 数据模型，无法独立编译；本文按现有源码整理，不代表完整区块链实现或运行验收。

## 项目内容

- 生成创世区块，用前一区块哈希连接后续区块。
- 尝试通过调整 nonce 寻找满足目标值的哈希。
- 校验新区块的索引、前序哈希和内容哈希。
- 使用 Gson 读写本地 JSON 区块记录。
- 通过 UDP 向局域网广播新区块并接收其他节点消息。
- 使用 JDK 自带 HTTP Server 展示启动时读取的区块数据。

## 文件导航

| 文件 | 用途 |
| --- | --- |
| [MainClass.java](blockchain/MainClass.java) | 入口：启动 HTTP、读取区块、启动监听与挖矿线程 |
| [BlockChain.java](blockchain/BlockChain.java) | 区块生成、校验、PoW 与文件读写 |
| [Hash256.java](blockchain/Hash256.java) | SHA-256 计算 |
| [Miner.java](blockchain/Miner.java) | 挖矿循环与 UDP 广播 |
| [P2PListen.java](blockchain/P2PListen.java) | 接收消息并尝试加入本地区块链 |
| [Http.java](blockchain/Http.java) | HTTP 数据展示 |
| [blocks.json](blockchain/blocks.json) | 历史区块样例 |
| [gson-2.7.jar](blockchain/gson-2.7.jar) | 随仓库保留的 JSON 依赖 |

## 阅读与复现

工程使用普通 Java 包结构，没有 Maven 或 Gradle 配置。入口为 `blockchain.MainClass`。

复现前需要先处理以下条件：

1. 找回或补齐 `blockchain.Block`：仓库没有 `Block.java`，也没有提供包含该类的项目依赖。
2. 配置数据文件：`BlockChain.java` 和 `Http.java` 都写死了 `D:/blockchain/blocks.json`，需要准备目录与文件，或统一修改路径。
3. 配置局域网：`Miner.java` 使用 `192.168.31.255:8123` 广播，监听端口同为 UDP `8123`，应与实验网络匹配。
4. 确认 HTTP 端口 `8080` 可用。入口先启动 HTTP，再初始化区块链，因此数据文件需要提前准备。

补齐缺失类后，可在仓库根目录使用以下入口；**当前快照编译会因缺少 `Block` 失败**：

```bash
mkdir -p out
javac -encoding UTF-8 -cp blockchain/gson-2.7.jar -d out blockchain/*.java
java -cp "out:blockchain/gson-2.7.jar" blockchain.MainClass
```

以上为 Bash 示例，Windows 的 Java classpath 分隔符使用 `;`。HTTP 入口为 `http://localhost:8080/`。

## 当前实现的边界

- HTTP 返回启动时读取的数据快照，不随挖矿实时刷新。
- 接收区块时未实现完整 PoW 校验、分叉选择和全链同步；替换链逻辑尚未实现。
- PoW 目标值使用 `double`，命中后又重新生成区块，不能视为严谨的共识实现。
- 线程循环、网络异常与共享状态仍是课程实验写法；运行会持续挖矿和广播，需手动停止。

## 来源与维护

保留大学时期的学习实现，整理重点是说明代码结构与缺失条件，没有补写缺失功能或把历史样例当作新运行结果。

仓库目前没有项目级 LICENSE；附带依赖的权利与许可由其原作者规定。
