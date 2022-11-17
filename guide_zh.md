# 介绍

DPAL WALLET 是一款专为 DOGECOIN 设计的插件钱包，设计的初衷是为了让 DOGE 也能有自己的插件钱包平台，并且让狗狗币的web开发者集成狗狗币公链更加方便和简单，同时满足安全的需要。

# 为什么需要 DPal?

### web应用集成 dogecoin 公链的问题

要了解 DPal 的设计初衷，首先要搞清楚如果没有第三方支付（中心化）平台或者是DPal Wallet这类插件钱包，要在web应用中集成DOGE币支付会有哪些问题和困难，我以 Tesla 的狗币支付方案为例来解释这些问题。

* 订单和地址绑定还是交易HASH?

简单来说，Tesla 是将订单和地址绑定，Tesla 构建了一个复杂的地址管理、分发系统，将每个订单与地址绑定，地址收到狗狗币之后，再统一将不同地址的余额汇总。虽然这样做没有什么问题，但客户遇到退款、再次付款等支付售后时，系统会遇到管理无数地址（实际上管理的地址数量与订单的并发数量正相关）的麻烦，这会增加研发和维护成本。对中小企业来说，构建维护这类系统成本过于高昂。所以问题就回到了标题所提到的，将订单和地址绑定时系统的成本过高，而如果订单能和交易绑定，那就只要管理一个地址就够了，这大大降低了成本。即使出现退款，再次付款等支付问题时，也是相当容易处理的，因为订单和哪笔交易进行了绑定这和传统的网络交易无异。
 
 * 利用DPAL将订单和交易 映射
  
 DPAL 能简单的完成商户订单和交易的绑定，成本只需要几行的代码即可集成，这大大降低了系统开发维护的难度，如果你是中小网站，你肯定不会觉得对你来说是个很大的问题
 
 
 ```javascript
if (await doge.isEnabled()) {
  const rs = await doge.useDoge(cost, toAddress, 'Buy Things Info');
  if (rs?.txid) {
    // successed
    // you can track the transaction is confirmed by txid in doge chain
    // map the transaction id with your webapps orderid.
    // you don't need to build a complex address allocator for your system anymore.
  }
}
```

![Demo](https://github.com/dpalwallet/DPalWallet/blob/main/Untitled_%20Oct%2017%2C%202022%2012_54%20PM.gif)


 * 简化KYC
 
 通过DPAL 所有的网站可以简化用户系统了，如果你愿意拥抱去中心化以及保护用户的隐私的话，简单的调用连接、签名方法即可帮助用户登录和验证，并以狗狗币地址作为用户的唯一ID, 生成对应的数据访问的授权token，既可以告别繁琐的邮件系统，邮件系统可以退出KYC并作为辅助用户。
 
  * 密钥安全和设计原理
    1.  钱包的助及词及密钥经过严格的加密处理并存储在本地，钱包对你设置的密码强度有一定要求，所以即便你的电脑遭遇黑客，只要未能泄露密码，你的 DOGE 仍然是安全的 
    2.  和所有支持 Chrome Extension V2 （METAMASK等）的钱包不同，DPAL 支持 V3 , V2 的缺点是数据会长时间停留在 background.js 内存中，如果你的电脑遭遇黑客，那么会有暴露密钥的风险，而DPAL仅在使用时短时间停留在内存，且交易需要输入密码解密，换言之你的钱包更加安全。
    3.  黑名单白名单计划，所有web3应用在与DPAL 交互时会通过安全列表过滤，有安全风险的网站是不被允许与DPAL交互的。
    4.  Your keys, Your Doge
   
   * 开源和计划
    1. 插件实际上对技术人员是开放的，所有人都可以通过简单的chrome 自带的源代码查看功能，追踪插件的运行过程
    2. DPAL 会在 2023 Q1 随着集成了DPAL 的 web3 应用同步开源所有代码
  
  * 实验：使用代理模式让智能合约记录以狗币地址为资产地址
 
https://github.com/dpalwallet/DPalWallet/blob/main/SmartContract_DOGE.MD

### 如何集成 DPal
* [API and demo](./api.md)

### 路线图
* 2023 Q1 构建基于钱包的 web 应用 : 以 DOGE 为唯一代币的 自组织 DAO
* 2023 Q1-Q2 手机版本（以支付宝和PayPal 为蓝本）
* 基于 DOGE 的金融服务
