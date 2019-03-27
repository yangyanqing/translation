# 交易所适配器

原文：https://www.docs.melonport.com/chapters/exchanges.html

## 综述

通过交易所适配器合约将每个交易所集成到 water**melon** 协议中。Exchange适配器合约每个都继承自标准合约ExchangeAdapter.sol 。然后，每个适配器将此功能映射到特定交换合约的已定义功能，从而覆盖基本合约的公共方法。  

## ExchangeAdapter.sol

#### 源码

https://github.com/melonproject/protocol/blob/master/src/contracts/exchanges/ExchangeAdapter.sol

#### 描述

ExchangeAdapter 定义了其他交换适配器的接口。

三个函数 `makeOrder()`、`takeOrder()`和`cancelOrder()` 具有相同的参数。参数总结如下：

- `address targetExchange` - 与订单有关的预期交换合约的地址

- `address[6] orderAddresses` - 包含按订单下面的地址的数组
  - `orderAddresses[0]` - 下单者地址
  - `orderAddresses[1]` - 吃单者地址
  - `orderAddresses[2]` - 下单代币合约地址
  - `orderAddresses[3]` - 吃单代币合约地址
  - `orderAddresses[4]`- 收费人的地址，`feeRecipientAddress`
  - `orderAddresses[5]`- 发起者的地址，`msg.sender`

- `uint[8] orderValues` - 一个数组，其中包含与订单下面相关的值
  - `orderValues[0]` - 下单资产数量
  - `orderValues[1]` - 吃单资产数量
  - `orderValues[2]` - 下单手续费
  - `orderValues[3]` - 吃单手续费
  - `orderValues[4]` - 订单的到期时间（以秒为单位）
  - `orderValues[5]` - salt/nonce 用于区分类似资产和数量的订单
  - `orderValues[6]` - 订单成交数量，即实际成交的代币数
  - `orderValues[7]` - 特定于Dexy交易所的签名模

- `bytes32 identifier` - 订单标识符
- `bytes makerAssetData` - 针对挂单资产的编码数据
- `bytes takerAssetData` - 针对吃单资产的编码数据
- `bytes signature` - 挂单者签名

#### 继承自
无
#### 构造函数
无
#### 结构
无
#### 枚举
无
#### 修饰器

`modifier onlyManager()`

此功能修饰器要求`msg.sender`在执行功能之前是投资经理地址。  

`modifier notShutDown()`

此功能修饰器要求基金在执行功能之前不处于关闭状态。  

`modifier onlyCancelPermitted(address exchange, address asset)`

此功能修饰器要求`msg.sender`是投资经理地址，基金处于关闭状态或订单在执行功能之前已过期。  

#### 事件

无 

#### 公共状态变量
无
#### 函数

`function getTrading() internal view returns (Trading)`

内部查询函数，将当前合约地址作为`Trading`合约返回。  

`function getHub() internal view returns (Hub)`

内部查询函数，返回与`Hub`合约对应的`Trading`合约。  

`function getAccounting() internal view returns (Accounting)`

内部查询函数，返回`Accounting`与当前`Hub`合约对应的合约。  

`function hubShutDown() internal view returns (bool)`

内部查询函数，返回一个布尔值，表示基金的合约是否是关闭状态。  

`function getManager() internal view returns (address)`

内部查询函数，返回当前基金的投资经理的地址。  

`function ensureNotInOpenMakeOrder(address _asset) internal view`

内部查询函数，确认输入资产地址不在未关闭的订单中。  

`function makeOrder( address targetExchange, address[6] orderAddresses, uint[8] orderValues, bytes32 identifier, bytes makerAssetData, bytes takerAssetData, bytes signature)`

该公共函数必须确保：

基金没有关闭，资产代币价格是近期的，适用的风险政策被评估，待交易的资产代币被批准（如果需要），订单交易被发送到交易所合约，订单在交易所登记合约（如果可能），所获得的资产代币（如果尚未持有）将被添加到`ownedAssets`。

该函数在特定交易所创建一个指令，用于指定要交换的资产代币以及订单所需的相应数量（隐含价格）。此函数在基础合约中未实现。

`function takeOrder( address targetExchange, address[6] orderAddresses, uint[8] orderValues, bytes32 identifier, bytes makerAssetData, bytes takerAssetData, bytes signature)`

该公共函数必须确保：

基金没有关闭，基金没有成交自己的挂单，行情信息中包含资产代币价格，资产代币价格是近期的，适用的风险政策被评估，资产代币交易被批准（如果被要求），吃单交易被发送到交换合约，订单将从交换合约中移除（如果可能的话），所获得的资产代币（如果尚未持有）将被添加到`ownedAssets`。

该函数采是特定交易所现有挂单相对应的，交付挂单所需的资产代币，以换挂单提供的资产代币，其数量（或部分订单填充的更少）指定。此函数在基础合约中未实现。  

`function cancelOrder( address targetExchange, address[6] orderAddresses, uint[8] orderValues, bytes32 identifier, bytes makerAssetData, bytes takerAssetData, bytes signature)`

此函数必须确保：

`msg.sender`是订单所有者，订单已过期或基金被关闭，订单从基金内部订单跟踪队列中删除，交易所的订单被撤销。

该函数撤销并删除基金内及其所在交易所的所有订单信息。此函数在基础合约中未实现。   

`function getOrder( address onExchange, uint id, address makerAsset) view returns ( address, address, uint, uint)`

此公共查询功能旨在返回给定交换合约地址，订单标识符和挂单资产代币合约地址参数的相关订单信息（挂单代币地址，吃单代币地址，挂单代币数量和吃单代币数量） 。基本合约上的功能仍未实现并恢复。  

## EngineAdapter.sol

#### 源码

https://github.com/melonproject/protocol/blob/master/src/contracts/exchanges/EngineAdapter.sol

#### 描述

该合约是 water**melon** 引擎的适配器，用作从 water**melon** 基金到 water**melon** 引擎的接口，用于发送 water**melon** 基金交易 MLN 到 water**melon** 引擎以换取 WETH。  

#### 继承自

DSMath，TokenUser，[ExchangeAdapter](#ExchangeAdapter.sol)

#### 构造函数
无


#### 结构
无


#### 枚举
无


#### 修饰器
无


#### 事件
无


#### 公共状态变量
无


#### 公共函数

`function takeOrder ( 
    address targetExchange, 
    address[6] orderAddresses, 
    uint[8] orderValues, 
    bytes32 identifier, 
    bytes makerAssetData, 
    bytes takerAssetData, 
    bytes signature) 
    onlyManager notShutDown`

函数使用以下参数：

- `targetExchange`-  water**melon** 引擎交换合约的地址
- `orderAddresses[2]` - WETH 代币合约的地址（挂单资产代币）
- `orderAddresses[3]` - MLN 代币合约的地址（吃单资产代币）
- `orderValues[0]` - 从引擎返回的最小 ETH 数量
- `orderValues[1]` - MLN代币的数量，以18位小数精度表示
- `orderValues[6]` - 与 orderValues [1] 相同

此功能要求`msg.sender`是投资经理和基金未关闭。该功能要求 water**melon** 基金批准所需的MLN代币交易数量。该功能调用 water**melon** 引擎获得相应数量的ETH。 water**melon** 引擎功能`sellAndBurnMln`被调用时，作为资金转账 MLN 为WETH，因为WETH被收到 water**melon** 基金，并转移到 water**melon** 基金的账户。如果需要，最后使用新位置更新`ownedAssets`，订单状态更新。  

## ZeroExV2Adapter.sol

#### 源码

https://github.com/melonproject/protocol/blob/master/src/contracts/exchanges/ZeroExV2Adapter.sol

#### 描述

此合约是用户适配 0x 交易所 v2 版本合约的交换适配器，用作从 water**melon** 基金到 0x 交易所的接口，用于交换交易所上列出的资产代币。  

#### 继承自

[ExchangeAdapter](#ExchangeAdapter.sol)，DSMath（链接）

#### 构造函数
无


#### 结构
无


#### 枚举
无


#### 修饰器
无


#### 事件
无


#### 公共状态变量
无


#### 公共函数

`function makeOrder(
    address targetExchange,
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes32 identifier,
    bytes wrappedMakerAssetData,
    bytes takerAssetData,
    bytes signature
) onlyManager notShutDown`

请参阅上面的参数说明。

这项公共函数要求`msg.sender`是投资经理的地址，基金尚未关闭。该函数在 0x v2 交换合约上创建一个 make 订单。它确保挂单代币目前没有列入 water**melon** 基金在任何交易所可能拥有的任何其他开放式订单中。吃单资产代币被初步添加到 water**melon** 基金的自有资产中，并且开放的订单被添加到 water**melon** 基金的内部订单跟踪中。最后，订单在0x v2交换合约上预先签署，授权经理代表交易合约签字。  

function cancelOrder(
    address targetExchange,
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes32 identifier,
    bytes wrappedMakerAssetData,
    bytes takerAssetData,
    bytes signature
) onlyCancelPermitted(targetExchange, orderAddresses[2])

请参阅上面的参数说明。

此公共函数通过调用0x v2交换合约的`cancelOrder()`函数来取消0x v2交换合约上的现有订单。该函数应用`onlyCancelPermitted`修饰器，仅允许取消在以下条件之一的提交：投资经理取消订单，基金关闭或订单已过期。资产代币最终从 water**melon** 基金的内部订单跟踪中删除。  

`function getOrder(
    address onExchange,
    uint id,
    address makerAsset)
view returns (
    address,
    address,
    uint,
    uint)`

该公共视图函返回在给定交换合约地址的订单信息，如：挂单资产代币地址，吃单资产代币地址，挂单资产代币数量和吃单资产代币数量。该功能可以确定订单是已部分还是全部成交（返回剩余的挂单资产代币数量和剩余的吃单资产代币数量）。

`function approveTakerAsset(
    address targetExchange,
    address takerAsset,
    bytes takerAssetData,
    uint fillTakerQuantity)
internal`

此内部查询功能从保管库中提取吃单资产的 `fillTakerQuantity` 金额，并授权与资产代理相同的金额 。

`function approveMakerAsset(
    address targetExchange,
    address makerAsset,
    bytes makerAssetData,
    uint makerQuantity)
internal`

此内部查询功能从保险库中提取挂单资产的makerQuantity金额，并授权与资产代理相同的金额。

`function constructOrderStruct(
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes makerAssetData,
    bytes takerAssetData)
internal
view
    returns (LibOrder.Order memory order)`

内部查询函数，根据提供的参数值返回`order`的结构。  

`function getAssetProxy(
    address targetExchange,
    bytes assetData)
internal
view
    returns (address assetProxy)`

内部查询函数，给定 Ethfinex 交换合约的地址和提供的资产数据，返回资产代理的地址。  

`function getAssetAddress(
    bytes assetData)
internal
view
    returns (address assetAddress)`

内部查询函数，根据提供的资产数据，返回资产的地址。  

## EthfinexAdapter.sol

#### 源码

https://github.com/melonproject/protocol/blob/master/src/contracts/exchanges/EthfinexAdapter.sol

#### 描述

该合约是 Ethfinex Exchange 合约的交换适配器，用作从 water**melon** 基金到 Ethfinex 交易所的接口，用于交换Ethfinex交易所上市的资产代币。  

#### 继承自

[ExchangeAdapter](#ExchangeAdapter.sol)，DSMath，DBC

#### 构造函数
无


#### 结构
无


#### 枚举
无


#### 修饰器
无


#### 事件
无


#### 公共状态变量
无


#### 公共函数

`function makeOrder(
    address targetExchange,
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes32 identifier,
    bytes wrappedMakerAssetData,
    bytes takerAssetData,
    bytes signature
) onlyManager notShutDown`

请参阅上面的参数说明。

这项公共函数要求`msg.sender`是投资经理和基金未关闭。该函数在 Ethfinex 交换合约上创建一个 make 订单。它确保挂单资产代币目前没有列入 water**melon** 基金在任何交易所可能拥有的任何其他打开的订单中。该功能对订单进行签名并确保签名有效。吃单资产代币被初步添加到 water**melon** 基金的自有资产中，并且打开的订单被添加到 water**melon** 基金的内部订单跟踪中。最后，订单被添加到 Ethfinex 交换合约中。  

`function cancelOrder(
    address targetExchange,
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes32 identifier,
    bytes wrappedMakerAssetData,
    bytes takerAssetData,
    bytes signature
) onlyCancelPermitted(targetExchange, orderAddresses[2])`

请参阅上面的参数说明。

此公共函数通过调用 Ethfinex 交换合约的`cancelOrder()` 功能撤销 Ethfinex 交换合约的现有订单。该函数应用`onlyCancelPermitted`修饰器，仅允许取消在满足以下条件之一下提交：投资经理取消订单，基金关闭或订单已过期。资产代币最终从 water**melon** 基金的内部订单跟踪中删除。  

`function withdrawTokens(
    address targetExchange,
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes32 identifier,
    bytes makerAssetData,
    bytes takerAssetData,
    bytes signature)`

请参阅上面的参数说明。

该公共函数部门在开放式订单中提取 Ethfinex 交易所持有的所有资产代币，然后通过调用交易合约来移除 water**melon** 基金内部订单跟踪的生产订单`removeOpenMakeOrder()`。最后，该函数将资产标记返回到 Vault 并将资产标记添加到`ownedAssets`。  

`function getOrder(
    address onExchange,
    uint id,
    address makerAsset)
view returns (
    address,
    address,
    uint,
    uint)`

该公共视图函数在给定交换合约地址，订单标识符和挂单资产代币合约地址参数的情况下返回相关订单信息（挂单资产代币地址，吃单资产代币地址，挂单资产代币数量和吃单资产代币数量）。该功能可以确定订单是已部分还是全部填写（返回剩余的挂单资产代币数量和剩余的吃单资产代币数量）  

`function wrapMakerAsset(
  address targetExchange,
  address makerAsset,
  bytes wrappedMakerAssetData,
  uint makerQuantity,
  uint orderExpirationTime)
internal`

这个内部功能从 water**melon** 基金的保险库中提取挂单资产代币，根据 Ethfinex 的打包注册表包装挂单代币。  

`function constructOrderStruct(
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes makerAssetData,
    bytes takerAssetData)
internal
view
    returns (LibOrder.Order memory order)`

内部查询函数，根据提供的参数值返回 `order` 结构。  

`function getAssetProxy(
    address targetExchange,
    bytes assetData)
internal
view
    returns (address assetProxy)`

内部查询函数，给定 Ethfinex 交换合约的地址和提供的资产数据，返回资产代理的地址。  

`function getAssetAddress( bytes assetData) internal view returns (address assetAddress)`

给定提供的资产数据，内部查询函数，返回资产的地址。  

`function getAssetAddress(
    bytes assetData)
internal
view
    returns (address assetAddress)`

内部查询函数，给定提供的资产代币合约的地址，返回 Ethfinex 指定的包装资产代币合约的地址。  

## KyberAdapter.sol

#### 源码

https://github.com/melonproject/protocol/blob/master/src/contracts/exchanges/KyberAdapter.sol

#### 描述

该合约是 Kyber 网络合约的交换适配器，用作从 water**melon** 基金到 Kyber 网络合约的接口，用于交换 Kyber 网络上列出的资产代币。  

#### 继承自

[ExchangeAdapter](#ExchangeAdapter.sol)，DSMath，DBC

#### 构造函数
无


#### 结构
无


#### 枚举
无


#### 修饰器
无


#### 事件
无


#### 公共状态变量

`address public constant ETH_TOKEN_ADDRESS = 0x00eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee`

此公共常量状态变量表示 ETH 代币地址。  

#### 公共函数

`function takeOrder(
    address targetExchange,
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes32 identifier,
    bytes makerAssetData,
    bytes takerAssetData,
    bytes signature)
onlyManager notShutDown`

请参阅上面的参数说明。在Kyber交换合约的命名（swapToken（）函数）的上下文中，以下参数采用指定的映射：

- `orderAddresses[2]` - Src代币地址。 
- `orderAddresses[3]` - Dest代币地址。 
- `orderValues[0]` - Src代币数量。 
- `orderValues[1]` - Dest代币数量。

该公共函数必须确保：

- 基金没有关闭
- `msg.sender`是投资经理
- 吃单成交数量等于吃单资产数量
- 资产代币价格是最近的并计算最低要求汇率
- 评估适用的风险政策
- 批准交易的资产代币（如果需要）
- Kyber 交易所的交换订单已执行合约
- 将获得的资产代币（如果尚未持有）添加到`ownedAssets`。最后，将获得的资产代币返还给 water**melon** 基金的保险库，并更新订单状态。

最后，该功能将获得的代币资产返回到基金的金库，并更新基金的内部订单跟踪。  

`function dispatchSwap(
    address targetExchange,
    address srcToken,
    uint srcAmount,
    address destToken,
    uint minRate)
internal
    returns (uint actualReceiveAmount)`

此内部函数根据所涉及的资产代币路由调用逻辑。三种可能性是：

- 基金交易ETH其他资产代币—被`swapNativeAssetToToken()` 调用
- 基金交易其他资产代币ETH—被`swapTokenToNativeAsset()` 调用
- 基金交易非ETH代币—被`swapTokenToToken()`调用

该函数采用以下参数：

- `targetExchange` - 预期交换合约的地址（即Kyber网络）
- `srcToken` - 基金将提供的资产代币的地址
- `srcAmount` - 交付资产代币的数量
- `destToken` - 基金将收到的资产代币的地址
- `minRate` - 接收资产代币的最小可接受数量

该函数返回基金收到的资产代币数量。  

`function swapNativeAssetToToken(
    address targetExchange,
    address nativeAsset,
    uint srcAmount,
    address destToken,
    uint minRate)
internal
    returns (uint receivedAmount)`

此内部功能启动交易，其中 water**melon** 基金为 `destToken` 地址参数指定的接收资产代币提供ETH 。该功能从基金的保险库中提取指定数量的ETH，将ETH数量发送给Kyber Network交换合约，调用其`swapEtherToToken()`功能。

该函数采用以下参数：

- `targetExchange` - 预期交换合约的地址（即Kyber网络）
- `nativeAsset` - 基金将提供的本地资产代币的地址，即ETH
- `srcAmount` - 交付资产代币的数
- `destToken` - 基金将收到的资产代币的地
- `minRate` - 接收资产代币的最小可接受数量。如果`minRate`未定义，则使用来自Kyber网络的预期费率。

该函数返回`destToken`收到的数量。Kyber网络合约功能将资产转移到此（适配器）合约，然后将其转移回`takeOrder`功能中的保险库。 

`function swapTokenToNativeAsset(
    address targetExchange,
    address srcToken,
    uint srcAmount,
    address nativeAsset,
    uint minRate)
internal
    returns (uint receivedAmount)`

该内部功能启动交易，其中 water**melon** 基金提供由`srcToken`地址参数指定的资产代币以接收ETH。该功能从基金的保险库中提取指定数量的交割资产代币，并将数量发送到称为其`swapTokenToEther()`功能的Kyber Network交换合约。然后，该功能批准交换的转移数量。最后，该函数将收到的任何ETH转换为WETH。

该函数采用以下参数：

- `targetExchange` - 预期交换合约的地址（即Kyber网络）
- `srcToken` - 基金将提供的资产代币的地址
- `srcAmount` - 交付资产代币的数量
- `nativeAsset` - 基金将收到的本地资产代币的地址，即ETH
- `minRate` - 接收资产代币的最小可接受数量。

该函数返回`nativeAsset`接收的数量ETH。Kyber网络合约功能将资产转移到此（适配器）合约，然后将其转移回`takeOrder`功能中的保险库。  

`function swapTokenToToken(
    address targetExchange,
    address srcToken,
    uint srcAmount,
    address destToken,
    uint minRate)
internal
    returns (uint receivedAmount)`

该内部功能启动交易，其中 water**melon** 基金提供由`srcToken`地址参数指定的资产代币，用于接收由`destToken`地址参数指定的资产代币。该功能从基金的保险库中提取指定数量的交割资产代币，并将数量发送到称为其`swapTokenToToken()`功能的Kyber Network交换合约。然后，该功能批准交换的转移数量。最后，该函数将收到的任何ETH转换为WETH。

该函数采用以下参数：

- `targetExchange` - 预期交换合约的地址（即Kyber网络）
- `srcToken` - 基金将提供的资产代币的地址
- `srcAmount` - 交付资产代币的数量
- `destToken` - 基金将收到的资产代币的地址
- `minRate`- 接收资产代币的最小可接受数量（`destToken`）。如果`minRate`未定义，则使用来自Kyber网络的预期费率。

该函数返回`destToken`收到的数量。Kyber网络合约功能将资产转移到此（适配器）合约，然后将其转移回`takeOrder`功能中的保险库。  

`function calcMinRate(
    address srcToken,
    address destToken,
    uint srcAmount,
    uint destAmount)
internal
view
    returns (uint minRate)`

内部查询函数，根据给定参数的从行情源返回吃单资产代币的最小可接受交换数量：

- `srcToken` - 基金将提供的资产代币的地址
- `destToken` - 基金将收到的资产代币的地址
- `srcAmount`- 交付资产代币的数量
- `destAmount` - 接收资产代币的数量

## MatchingMarketAdapter.sol

#### 源码
https://github.com/melonproject/protocol/blob/master/src/contracts/exchanges/MatchingMarketAdapter.sol

#### 描述

该合约是 OasisDex 撮合市场交换合约的交换适配器，用作从 water**melon** 基金到 OasisDex 撮合市场的接口，用于交换OasisDex 撮合市场上列出的资产代币。  

#### 继承自

[ExchangeAdapter](#ExchangeAdapter.sol)，DSMath 

#### 构造函数
无


#### 结构
无


#### 枚举
无


#### 修饰器
无


#### 事件

`event OrderCreated(uint id)`

在交换合约上创建订单时会发出此事件。事件记录并广播订单的标识符。

#### 公共状态变量
无


#### 公共函数

`function makeOrder(
    address targetExchange,
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes32 identifier,
    bytes makerAssetData,
    bytes takerAssetData,
    bytes signature)
onlyManager notShutDown`

请参阅上面的参数说明。

该公共函数必须确保：

- 基金没有关闭
- `msg.sender`是投资经理
- 资产代币价格是近期的
- 评估适用的风险政策
- 批准交易的资产代币（如果需要）
- 将订单交易发送到交换合约
- 订单在交换合约上注册（如果可能）
- 获得的资产代币（如果尚未持有）将被添加到`ownedAssets`。

该函数在 OasisDex 撮合市场交易合约上创建一个挂单，指定要交换的资产代币和订单所需的相应数量（隐含价格）。一个`orderID > 0`表示该订单被成功提交 OasisDex 撮合的市场交易合约，并将挂单加入 water**melon**基金的内部订单跟踪。最后，函数发出带有 orderID 的 `OrderCreated()`消息。  

`function takeOrder(
    address targetExchange,
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes32 identifier,
    bytes makerAssetData,
    bytes takerAssetData,
    bytes signature)
onlyManager notShutDown`

请参阅上面的参数说明。值得注意的是参数：

- `orderValues[6]` - 吃单资产代币的填充数量
-  `identifier` - 事件订单的orderID

该公共函数必须确保：

- 基金没有关闭，`msg.sender` 是投资经理
- 基金未成交自己的订单
- 资产代币价格对存在于价格源中
- 资产代币价格为近期
- 适用的风险政策进行评估
- 待交易的资产代币被批准（如果需要）
- 采购订单交易如果发送到交换合约
- 则该订单将从交换合约中移除（如果可能）
- 所获得的资产代币（如果尚未持有）将被添加到`ownedAssets`。

该函数采用 OasisDex 撮合市场交换合约上现有生产订单的对手单，交付下单所需的资产代币，以换取下单提供的资产代币指定数量（或更少的部分订单成交）。最后，该功能将获得的代币资产返回到基金的金库，并更新基金的内部订单跟踪。  

`function cancelOrder(
    address targetExchange,
    address[6] orderAddresses,
    uint[8] orderValues,
    bytes32 identifier,
    bytes wrappedMakerAssetData,
    bytes takerAssetData,
    bytes signature
) onlyCancelPermitted(targetExchange, orderAddresses[2])`

请参阅上面的参数说明。值得注意的是参数：

- `orderAddresses[2]` - 订单挂单资产代币
- `identifier` - 有效订单的orderID。

此公共函数通过调用 OasisDex 撮合市场合约的`cancel()`功能取消 OasisDex 撮合市场交换合约的现有订单。该函数应用`onlyCancelPermitted`修饰器，仅允许取消满足以下条件之一的提交：投资经理取消订单，基金关闭或订单已过期。资产代币最终从 water**melon** 基金的内部订单跟踪中移除，挂单资产代币返回到 water**melon** 基金的金库。  

`function getOrder(
    address targetExchange,
    uint id,
    address makerAsset)
view returns (
    address,
    address,
    uint,
    uint)`

该公共视图函数在给定交换合约地址，订单标识符和挂单资产代币合约地址参数的情况下返回当前相关订单信息（挂单资产代币地址，吃单资产代币地址，挂单资产代币数量和吃单资产代币数量）。已部分成交的订单将返回剩余的相应代币数量。  

