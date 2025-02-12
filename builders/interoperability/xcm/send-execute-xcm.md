---
title: 发送&执行XCM消息
description: 学习如何通过组合和试验不同的XCM指令来构建自定义XCM消息并在Moonbeam上本地执行以查看结果。
---

# 发送和执行XCM消息

## 概览 {: #introduction }

XCM消息由[一系列的指令](/builders/interoperability/xcm/overview/#xcm-instructions){target=_blank}组成，由跨共识虚拟机（XCVM）执行。这些指令的组合会执行预定的操作，例如跨链Token转移。您可以通过组合各种XCM指令创建自定义XCM消息。

[X-Tokens](/builders/interoperability/xcm/xc20/xtokens){target=_blank}和[XCM Transactor](/builders/interoperability/xcm/xcm-transactor/){target=_blank}等Pallet提供带有一组预定义的XCM指令的函数，用于发送[XC-20s](/builders/interoperability/xcm/xc20/overview/){target=_blank}或通过XCM在其他链上远程执行。然而，要更好地了解组合不同XCM指令的结果，您可以在Moonbeam上本地构建和执行自定义XCM消息。你也可以发送自定义XCM消息至其他链（这将以[`DescendOrigin`](https://github.com/paritytech/xcm-format#descendorigin){target=_blank}指令开始）。但是，要成功执行XCM消息，目标链需要理解这些指令。

要执行或发送自定义XCM消息，你可以直接使用[Polkadot XCM Pallet](#polkadot-xcm-pallet-interface)或者尝试通过带有[XCM-Utilities预编译](/builders/pallets-precompiles/precompiles/xcm-utils){target=_blank}的以太坊API。在本教程中，您将学习如何使用这两种方式在Moonbase Alpha上本地执行和发送自定义的XCM消息。

本教程假设您已熟悉XCM基本概念，例如[基本的XCM专业术语](/builders/interoperability/xcm/overview/#general-xcm-definitions){target=_blank}和[XCM指令](/builders/interoperability/xcm/overview/#xcm-instructions){target=_blank}。您可以访问[XCM概览](/builders/interoperability/xcm/overview){target=_blank}文档获取更多信息。

## Polkadot XCM Pallet接口 {: #polkadot-xcm-pallet-interface }

### Extrinsics {: #extrinsics }

Polkadot XCM Pallet包含以下相关extrinsics（函数）：

- **execute**(message, maxWeight) - **仅支持Moonbase Alpha** - 给定SCALE编码的XCM版本化的XCM消息和要消耗的最大权重，执行自定义XCM消息
- **send**(dest, message) - **仅支持Moonbase Alpha** - 给定要发送消息的目标链的multilocation和要发送的SCALE编码的XCM版本化的XCM消息，发送自定义消息。要成功执行XCM消息，目标链需要理解消息中的指令

### 存储函数 {: #storage-methods }

Polkadot XCM Pallet包含以下相关只读存储函数：

- **assetTraps**(Option<H256>) - 给定`MultiAssets`对的Blake2-256哈希，返回中断资产的现有次数。如果哈希出现omitted的错误，则返回所有中断资产
- **palletVersion**() - 提供正在使用的Polkadot XCM Pallet的版本

## 查看先决条件 {: #checking-prerequisites }

开始操作本教程之前，请先准备以下内容：

- 您的账户必须拥有一些DEV Token
  --8<-- 'text/_common/faucet/faucet-list-item.md'

## 本地执行XCM消息 {: #execute-an-xcm-message-locally }

这一部分涵盖了通过两种不同的方法来构建要在本地（即在Moonbeam中）执行的自定义XCM消息：Polkadot XCM Pallet的`execute`函数和[XCM-Utilities预编译](/builders/pallets-precompiles/precompiles/xcm-utils){target=_blank}的`xcmExecute`函数。此功能为您提供了试验不同的XCM指令并直接查看这些试验结果的平台。这也有助于确定与Moonbeam上给定XCM消息相关联的[费用](/builders/interoperability/xcm/fees){target=_blank}。

在以下示例中，您将在Moonbase Alpha上从一个账户转移DEV Token至另一个账户。为此，您需要构建一个XCM消息以包含以下XCM指令，这些指令将在本地执行（在本示例中为Moonbase Alpha）：

- [`WithdrawAsset`](https://github.com/paritytech/xcm-format#withdrawasset){target=_blank} - 移除资产并将其放入暂存处
- [`DepositAsset`](https://github.com/paritytech/xcm-format#depositasset){target=_blank} - 将资产从暂存处取出并存入等值资产至接收方账户中

!!! 注意事项
    通常情况下，当您发送XCM消息跨链至目标链时，需要用到[`BuyExecution`指令](https://github.com/paritytech/xcm-format#buyexecution){target=_blank}用于支付远程执行。但是，对于本地执行，此指令非必要，因为您已通过extrinsic调用支付费用。

### 使用Polkadot.js API执行XCM消息 {: #execute-an-xcm-message-with-polkadotjs-api }

在本示例中，您将使用Polkadot.js API在Moonbase Alpha上本地执行自定义XCM消息，以直接与Polkadot XCM Pallet交互。

Polkadot XCM Pallet的`execute`函数接受两个参数：`message`和`maxWeight`。您可以执行以下步骤组装这些参数：

1. 构建`WithdrawAsset`指令，其将要求您定义：

    - Moonbase Alpha上DEV token的multilocation
    - 要转移的DEV token数量

    ```js
    const instr1 = {
      WithdrawAsset: [
        {
          id: { Concrete: { parents: 0, interior: { X1: { PalletInstance: 3 } } } },
          fun: { Fungible: 100000000000000000n }, // 0.1 DEV
        },
      ],
    };
    ```

2. 构建`DepositAsset`指令，其将要求您定义：

    - DEV token的多资产标识符。您可以使用允许通配符匹配的[`WildMultiAsset` format](https://github.com/paritytech/xcm-format/blob/master/README.md#6-universal-asset-identifiers){target=_blank}来识别资产
    - Moonbase Alpha上接收账户的multilocation

    ```js
    const instr2 = {
      DepositAsset: {
        assets: { Wild: { AllCounted: 1 } },
        beneficiary: {
          parents: 0,
          interior: {
            X1: {
              AccountKey20: {
                key: moonbeamAccount,
              },
            },
          },
        },
      },
    };
    ```

3. 将XCM指令合并至版本化的XCM消息中：

    ```js
    const message = { V3: [instr1, instr2] };
    ```

4. 指定`maxWeight`，其中包括您需要定义的`refTime`和`proofSize`值

    - `refTime`是可用于执行的计算时间量。在本示例中，您可以设置为`400000000n`
    - `proofSize`是可使用的存储量（以字节为单位）。在本示例中，您可以设置为`14484n`

    ```js
    const maxWeight = { refTime: 400000000n, proofSize: 14484n } ;
    ```

现在，您已经有了每个参数的值，您可以为交易编写脚本了。为此，您需要执行以下步骤：

1. 提供调用的输入数据，这包含：

     - 用于创建提供商的Moonbase Alpha端点URL
     - `execute`函数的每个参数的值

2. 创建一个用于发送交易的Keyring实例
3. 创建[Polkadot.js API](/builders/build/substrate-api/polkadot-js-api/){target=_blank}提供商
4. 使用`message`和`maxWeight`值制作`polkadotXcm.execute` extrinsic
5. 使用`signAndSend` extrinsic和在第二个步骤创建的Keyring实例发送交易

!!! 请记住
    本教程的操作仅用于演示目的，请勿将您的私钥存储至JavaScript文档中。

```js
--8<-- 'code/builders/interoperability/xcm/send-execute-xcm/execute/executeWithPolkadot.js'
```

!!! 注意事项
    您可以使用以下编码的调用数据在[Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=wss://wss.api.moonbase.moonbeam.network#/extrinsics/decode/0x1c03030800040000010403001300008a5d784563010d010204000103003cd0a705a2dc65e5b1e1205896baa2be8a07c6e002105e5f51e2){target=_blank}上查看上述脚本的示例，该脚本将1个DEV发送给Moonbeam上Bob的账户：`0x1c03030800040000010403001300008a5d784563010d010204000103003cd0a705a2dc65e5b1e1205896baa2be8a07c6e002105e5f51e2`。

交易处理后，0.1 DEV Token和相关联的XCM费用从Alice的账户提取，Bob将在其账户收到0.1 DEV Token。`polkadotXcm.Attempted`事件将与结果一同发出。

### 使用XCM Utilities预编译执行XCM交易 {: #execute-xcm-utils-precompile }

在这一部分，您将使用[XCM Utilities预编译](/builders/pallets-precompiles/precompiles/xcm-utils){target=_blank}的`xcmExecute`函数（该函数仅支持Moonbase Alpha）以本地执行XCM消息。XCM Utilities预编译位于以下地址：

```text
{{ networks.moonbase.precompiles.xcm_utils }}
```

在Hood下，XCM Utilities预编译的`xcmExecute`函数调用Polkadot XCM Pallet的`execute`函数，即用Rust编码的Substrate pallet。使用XCM Utilities预编译调用`xcmExecute`的好处是您可以通过以太坊API完成此操作并使用[Ethers.js](/builders/build/eth-api/libraries/ethersjs){target=_blank}等以太坊库。

`xcmExecute`函数接受两个参数：要执行SCALE编码的版本化XCM消息和要消耗的最大权重。

首先，您将了解如何生成编码的调用数据，然后您将了解如何使用编码的调用数据与 XCM Utilities预编译交互。

#### 生成XCM消息的编码调用数据 {: #generate-encoded-calldata }

要获取XCM消息的编码调用数据，您可以创建一个类似于在[使用Polkadot.js API执行XCM消息](#execute-an-xcm-message-with-polkadotjs-api)部分创建的脚本。您将构建消息来获取编码的调用数据，而不是构建消息并发送交易。为此，您需要执行以下步骤：

 1. 提供调用的输入数据，这包含：

     - 用于创建提供商的Moonbase Alpha端点URL
     - [使用Polkadot.js API执行XCM消息](#execute-an-xcm-message-with-polkadotjs-api)部分定义的`execute`函数的每个参数的值

 2. 创建[Polkadot.js API](/builders/build/substrate-api/polkadot-js-api/){target=_blank}提供商
 3. 使用`message`和`maxWeight`值制作`polkadotXcm.execute` extrinsic
 4. 使用交易获取编码的调用数据

整个脚本如下所示：

```js
--8<-- 'code/builders/interoperability/xcm/send-execute-xcm/execute/generateEncodedCalldata.js'
```

#### 执行XCM消息 {: #execute-xcm-message }

现在，您已拥有SCALE编码的XCM消息，您可以使用以下代码片段通过您选择的以太坊库以编程方式调用XCM-Utilities预编译的`xcmExecute`函数：

1. 创建提供商和签署者
2. 创建用于交互的XCM Utilities Precompile的实例
3. 定义`xcmExecute`函数所需的参数，这些参数将是XCM消息的编码调用数据以及用于执行消息的最大权重。您可以将`maxWeight`设置为`400000000n`，它对应于`refTime`。`proofSize`将自动设置为默认值，即64KB
4. 执行XCM消息

!!! 请记住
    请勿将您的私钥存储至JavaScript或Python文件中。

=== "Ethers.js"

    ```js
    --8<-- 'code/builders/interoperability/xcm/send-execute-xcm/execute/ethers.js'
    ```

=== "Web3.js"

    ```js
    --8<-- 'code/builders/interoperability/xcm/send-execute-xcm/execute/web3.js'
    ```

=== "Web3.py"

    ```py
    --8<-- 'code/builders/interoperability/xcm/send-execute-xcm/execute/web3.py'
    ```

## 跨链发送XCM消息 {: #send-xcm-message }

这一部分涵盖了通过两种不同的方法来跨链发送自定义XCM消息（即从Moonbeam到目标链，如中继链）：Polkadot XCM Pallet的`send`函数和[XCM-Utilities预编译](/builders/pallets-precompiles/precompiles/xcm-utils){target=_blank}的`xcmSend`函数。

要成功执行XCM消息，目标链需要理解消息中的指令。相反，您将在目标链上看到`Barrier`过滤器。为保证安全，XCM消息前会加上[`DecendOrigin`](https://github.com/paritytech/xcm-format#descendorigin){target=_blank}指令以防止XCM代表源链的主权账户执行操作。**如上所述，此部分的示例仅用于演示目的**。

在以下示例中，您将构建一个包含以下XCM指令的XCM消息，这些指令将在Alphanet中继链中执行：

 - [`WithdrawAsset`](https://github.com/paritytech/xcm-format#withdrawasset){target=_blank} - 移除资产并将其放入暂存处
 - [`BuyExecution`](https://github.com/paritytech/xcm-format#buyexecution){target=_blank} - 从暂存处获取资产以支付执行费用。支付的费用由目标链决定
 - [`DepositAsset`](https://github.com/paritytech/xcm-format#depositasset){target=_blank} - 将资产从暂存处取出并存入等值资产至接收方账户中

这些指令的目的是将中继链的原生资产（即Alphanet中继链的UNIT）从Moonbase Alpha转移到中继链上的一个账户。此示例仅用于演示目的，以演示如何跨链发送自定义XCM消息。 请记住，目标链需要理解消息中的指令才可执行。

### 使用Polkadot.js API发送XCM消息 {: #send-xcm-message-with-polkadotjs-api }

在本示例中，您将使用Polkadot.js API在Moonbase Alpha上本地执行自定义XCM消息，以直接与Polkadot XCM Pallet交互。

Polkadot XCM Pallet的`send`函数接受两个参数：`dest`和`message`。您可以执行以下步骤开始组装这些参数：

1. 为`dest`构建中继链Token UNIT的multilocation：

    ```js
    const dest = { V3: { parents: 1, interior: null } };
    ```

2. 构建`WithdrawAsset`指令，这将要求您定义：

    - 中继链上UNIT token的multilocation
    - 要提现的UNIT token数量

    ```js
    const instr1 = {
      WithdrawAsset: [
        {
          id: { Concrete: { parents: 1, interior: null } },
          fun: { Fungible: 1000000000000n }, // 1 UNIT
        },
      ],
    };
    ```

3. 构建`BuyExecution`指令，这将要求您定义：

    - 中继链上UNIT token的multilocation
    - 用于执行的UNIT token数量
    - 权重限制

    ```js
    const instr2 = {
      BuyExecution: [
        {
          id: { Concrete: { parents: 1, interior: null } },
          fun: { Fungible: 1000000000000n }, // 1 UNIT
        },
        { Unlimited: null }
      ],
    };
    ```

4. 构建`DepositAsset`指令，这将要求您定义：

    - UNIT token的多资产标识符。您可以使用允许通配符匹配的[`WildMultiAsset` format](https://github.com/paritytech/xcm-format/blob/master/README.md#6-universal-asset-identifiers){target=_blank}来识别资产
    - 中继链上接收账户的multilocation

    ```js
    const instr3 = {
      DepositAsset: {
        assets: { Wild: 'All' },
        beneficiary: {
          parents: 1,
          interior: {
            X1: {
              AccountId32: {
                id: relayAccount,
              },
            },
          },
        },
      },
    };
    ```

5. 将XCM指令合并至版本化的XCM消息中：

    ```js
    const message = { V3: [instr1, instr2, instr3] };
    ```

现在，您已经有了每个参数的值，您可以为交易编写脚本了。为此，您需要执行以下步骤：

1. 提供调用的输入数据，这包含：

     - 用于创建提供商的Moonbase Alpha端点URL
     - `send`函数的每个参数的值

2. 创建用于发送交易的Keyring实例
3. 创建[Polkadot.js API](/builders/build/substrate-api/polkadot-js-api/){target=_blank}提供商
4. 使用`dest`和`message`值制作`polkadotXcm.execute` extrinsic
5. 使用`signAndSend` extrinsic和在第二个步骤创建的Keyring实例发送交易

!!! 请记住
    本教程的操作仅用于演示目的，请勿将您的私钥存储至JavaScript文档中。

```js
--8<-- 'code/builders/interoperability/xcm/send-execute-xcm/send/sendWithPolkadot.js'
```

!!! 注意事项
    您可以使用以下编码的调用数据在[Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=wss://wss.api.moonbase.moonbeam.network#/extrinsics/decode/0x1c03030800040000010403001300008a5d784563010d010204000103003cd0a705a2dc65e5b1e1205896baa2be8a07c6e002105e5f51e2){target=_blank}上查看上述脚本的示例，该脚本将1个UNIT发送给中继链上Bob的账户：`0x1c00030100030c000400010000070010a5d4e81300010000070010a5d4e8000d0100010101000c36e9ba26fa63c60ec728fe75fe57b86a450d94e7fee7f9f9eddd0d3f400d67`。

交易处理后，`polkadotXcm.sent`事件将与发送的XCM消息详情一同发出。

### 使用XCM Utilities预编译发送XCM交易 {: #send-xcm-utils-precompile }

在这一部分，您将使用[XCM Utilities预编译](/builders/pallets-precompiles/precompiles/xcm-utils){target=_blank}的`xcmSend`函数（该函数仅支持Moonbase Alpha）以跨链发送XCM消息。XCM-Utilities预编译位于以下地址：

=== "Moonbase Alpha"

    ```text
    {{ networks.moonbase.precompiles.xcm_utils }}
    ```

在Hood下，XCM-Utilities预编译的`xcmSend`函数调用Polkadot XCM Pallet的`send`函数，即用Rust编码的Substrate pallet。使用XCM-Utilities预编译调用`send`的好处是您可以通过以太坊API完成此操作并使用[Ethers.js](/builders/build/eth-api/libraries/ethersjs){target=_blank}等以太坊库。要成功执行XCM消息，目标链需要了解消息中的指令。

`xcmSend`函数接受两个参数：目标链的multilocation和要发送的SCALE编码的版本化XCM消息。

首先，您将了解如何生成用于XCM消息的编码调用数据，然后您将了解如何使用编码的调用数据与 XCM Utilities预编译交互。

#### 生成XCM消息的编码调用数据 {: #generate-encoded-calldata }

要获取XCM消息的编码调用数据，您可以创建一个类似于在[使用Polkadot.js API执行XCM消息](#send-xcm-message-with-polkadotjs-api)部分创建的脚本。您将构建消息来获取编码的调用数据，而不是构建消息并发送交易。为此，您需要执行以下步骤：

 1. 提供调用的输入数据，这包含：

     - 用于创建提供商的Moonbase Alpha端点URL
     - [使用Polkadot.js API执行XCM消息](#send-xcm-message-with-polkadotjs-api)部分定义的`send`函数的每个参数的值

 2. 创建[Polkadot.js API](/builders/build/substrate-api/polkadot-js-api/){target=_blank}提供商
 3. 使用`message`和`maxWeight`值制作`polkadotXcm.execute` extrinsic
 4. 使用交易获取编码的调用数据

完整脚本如下所示：

```js
--8<-- 'code/builders/interoperability/xcm/send-execute-xcm/send/generateEncodedCalldata.js'
```

#### 发送XCM消息 {: #send-xcm-message }

在发送XCM消息前，您需要构建目标的multilocation。在本示例中，您将以Moonbase Alpha作为源链的中继链：

```js
const dest = [
  1, // Parents: 1 
  [] // Interior: Here
];
```

现在，你已拥有SCALE编码的XCM消息和目标multilocation，您可以使用以下代码片段选择[以太坊库](/builders/build/eth-api/libraries/){target=_blank}以编程方式调用XCM Utilities预编译的`xcmSend`函数。通常，您需要执行以下步骤：

1. 创建提供商和签署者
2. 创建用于交互的XCM Utilities Precompile的实例
3. 定义`xcmSend`函数所需的参数，该参数将是XCM消息的目标链和编码的调用数据
4. 发送XCM消息

!!! 请记住
    以下代码片段仅用于演示目的，请勿将您的私钥存储至JavaScript或Python文件中。

=== "Ethers.js"

    ```js
    --8<-- 'code/builders/interoperability/xcm/send-execute-xcm/send/ethers.js'
    ```

=== "Web3.js"

    ```js
    --8<-- 'code/builders/interoperability/xcm/send-execute-xcm/send/web3.js'
    ```

=== "Web3.py"

    ```py
    --8<-- 'code/builders/interoperability/xcm/send-execute-xcm/send/web3.py'
    ```

这样就可以了！您已成功使用Polkadot XCM Pallet和XCM-Utilities预编译从Moonbase Alpha上发送消息至另一条链。
