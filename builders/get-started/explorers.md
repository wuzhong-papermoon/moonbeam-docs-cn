---
title: 区块浏览器
description: 当前在Moonbeam上支持导航Substrate和以太坊层的区块浏览器概述
---

# 区块浏览器

## 概览 {: #introduction }

区块浏览器如同区块链的搜寻引擎，允许用户搜索余额、合约和交易等资讯。更多高级区块浏览器还提供索引编辑的功能，使用户能够在网络中直接提供像ERC-20 Token一样完整的一套信息，甚至还有可能提供API服务，可直接通过外部设备连接。

Moonbeam现在提供两种不同的浏览器：一种用于查询以太坊API，另一种专为查询Substrate API。所有基于EVM的交易都可以通过以太坊API访问，而Substrate API可以依赖于Substrate 的原生功能，如治理和质押。Substrate API也包括基于EVM交易的相关信息，但仅显示有限信息。

## 相关链接 {: #quick-links }

--8<-- 'text/builders/get-started/explorers/explorers.md'

## 以太坊API {: #ethereum-api }

### Moonscan {: #Moonscan }

[Moonscan](https://moonscan.io/){target=_blank}是Moonbeam主要的以太坊API区块浏览器。Moonscan由Etherscan团队创建，为用户提供强大、直观且功能丰富的体验。除了全面的交易和区块数据，Moonscan还提供一系列的[数据和图表](https://moonbeam.moonscan.io/charts){target=_blank}，如平均Gas价格、日交易量和区块大小图表。

Moonscan其他功能如下：

 - [收集人排行榜](https://moonbeam.moonscan.io/collators){target=_blank}，通过收集人的表现进行排名
 - [合约源码验证](/builders/build/eth-api/verify-contracts/block-explorers/){target=_blank}，可通过网页界面和API访问
 - 能够读写已验证的智能合约的状态数据
 - [Token授权](https://moonscan.io/tokenapprovalchecker){target=_blank}，您可以查看和撤销任何之前的Token授权
 - [添加Token信息](/builders/get-started/token-profile/){target=_blank}，并为部署到基于Moonbeam的网络的ERC-20、ERC-721和ERC-1155创建资料页面。资料页面可以包括您的项目连接、社交媒体、价格数据和与代币有关的其他信息

![Moonbeam Moonscan](/images/builders/get-started/explorers/explorers-1.png)

### Expedition {: #expedition }

以Moonbeam为主题的[Expedition](https://github.com/xops/expedition){target=_blank}浏览器版可以在[此链接](https://moonbeam-explorer.netlify.app/){target=_blank}找到。它是一个简单的JSON-RPC浏览器。

默认情况下，该浏览器连接至Moonbeam。您可通过以下步骤切换至Moonriver、Moonbase Alpha或连接至本地开发节点：

 1. 点击网页右上角网络名称，这里您可以选择所有的网络，包括在`{{ networks.development.rpc_url }}`上运行的**Moonbeam Development Node**

  2. 如果您想要连接至特定的PRC URL，选择**Add Custom Chain**，输入URL。例如：`http://localhost:9937`

![Expedition Explorer](/images/builders/get-started/explorers/explorers-2.png)

## Substrate API {: #substrate-api }

### Subscan {: #subscan }

[Subscan](https://moonbeam.subscan.io/){target=_blank} 是Moonbeam主要的Substrate API区块浏览器，它能够解析标准或自定义模块。举例而言，这个功能对展示关于质押、治理和EVM pallet（或是模块）非常有帮助。所有代码都是开源的，并且可以在[Subscan Essentials](https://github.com/subscan-explorer/subscan-essentials){target=_blank}找到。

![Subscan Moonbeam](/images/builders/get-started/explorers/explorers-3.png)

### Polkadot.js {: #polkadotjs }

虽然Polkadot.js Apps不是功能齐全的区块浏览器，但是一个方便的选项，尤其是对于运行本地开发节点的用户，使其可以查看事件和查询交易哈希。Polkadot.js Apps使用WebSocket端点与网络进行交互。Polkadot.js Apps支持[Moonbeam](https://polkadot.js.org/apps/?rpc=wss://wss.api.moonbeam.network#/explorer){target=_blank}、[Moonriver](https://polkadot.js.org/apps/?rpc=wss://wss.api.moonriver.moonbase.moonbeam.network#/explorer){target=_blank}和[Moonbase Alpha](https://polkadot.js.org/apps/?rpc=wss://wss.api.moonbase.moonbeam.network#/explorer){target=_blank}。

![Polkadot.js Moonbeam](/images/builders/get-started/explorers/explorers-4.png)

要将其连接到 Moonbeam 开发节点，您可以按照 [将Polkadot.js应用程序连接到本地Moonbeam节点](/builders/get-started/networks/moonbeam-dev/#connecting-polkadot-js-apps-to-a-local-moonbeam-node){target=_blank}部分的[开始使用 Moonbeam 开发节点](/builders/get-started/networks/moonbeam-dev/){target=_blank} 指南。默认端口为`9944`。

![Polkadot.js Local Node](/images/builders/get-started/explorers/explorers-5.png)

