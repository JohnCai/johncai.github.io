---
layout: post
title: "How-to: Kafka上手"
category: [How-to]
tags: [Kafka]
---
想使用 Linux 上手 Kafka。可我只有一台 Windows 的笔记本。

### 在 Azure 上创建一个 Ubuntu 的虚拟机
- 到 https://portal.azure.com/ 上按照 wizzard， 创建一个 Ubuntu 的 Virtual Machine。 
  - 内存选大点，至少 8G。
  - 使用推荐的ssh连接（自动创建私钥公钥）
  - 创建过程中，按提示下载私钥。
- 创建并启动虚拟机后，打开Powershell，使用 SSH 连接

### 安装 Kafka
- Kafka 需要 Java。经测试使用 OpenJDK 即可 `sudo apt-get install default-jdk`
- download Kafka `wget https://mirror.bit.edu.cn/apache/kafka/2.6.0/kafka_2.13-2.6.0.tgz`
- 解压 `tar -xzf kafka_2.13-2.6.0.tgz`
- 进入目录 `cd kafka_2.13-2.6.0`

### 玩起来
- 参照官方文档即可：https://kafka.apache.org/quickstart 
- 每次新开一个 terminal 均按以下步骤
    - 打开Powershell
    - 连接虚拟机：`ssh -i <private key path> <username>@<pip>`
    - `cd kafka_2.13-2.6.0`
    - 执行 Kafka 命令