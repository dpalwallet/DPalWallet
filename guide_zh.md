# 介绍

DPAL WALLET 是一款专为 DOGECOIN 设计的插件钱包，设计的初衷是为了让 DOGE 也能有自己的插件钱包平台，并且让狗狗币的web开发者集成狗狗币公链更加方便和简单，同时满足安全的需要。

# 为什么需要 DPal?

### web应用集成 dogecoin 公链的问题

要了解 DPal 的设计初衷我们需要清楚我们在步入去中心化的时代，如果没有第三方（中心化）平台单纯想要在web应用中集成DOGE币支付会有哪些问题和困难，以 Tesla 的狗币支付方案为例。

* 订单和地址绑定还是交易HASH?

简单来说，Tesla 是将订单和地址绑定，Tesla 构建了一个复杂的地址管理分发系统，将每个订单与地址绑定，交易完成后再统一将地址的余额汇总。虽然这样做没有什么问题，但是当客户退款时系统会遇到一点管理上的麻烦，这大大增加了研发和维护的成本，而对中小企业来说，构建维护这类系统成本过于高昂。所以问题就回到了标题所提到的，将订单和地址绑定时成本过高的系统，而如果订单能和交易绑定，那么只需管理一个地址不需要地址管理系统了，这大大降低了成本。即使出现退款，再次付款等支付问题时，也是相当容易处理的，因为每订单和哪笔交易进行了绑定。（所以有必要提出统一的二维码支付标准我称之为 “DogePayCode”,如果有空我会提出这个标准，并希望能够标准化。）
 
 * 利用DPAL将订单和交易 映射
  
 DPAL 能简单的完成商户订单和交易的绑定，成本只需要几行的代码即可集成，这大大降低了系统开发维护的难度，如果你是中小网站你肯定不会觉得对你来说是个很大的问题
 
 
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
 
 通过DPAL 所有的网站可以简化用户系统了，如果你愿意拥抱去中心化以及保护用户的隐私的话，简单的调用连接、签名方法即可帮助用户登录和验证，并以狗狗币地址作为用户的唯一ID, 生成对应的数据访问token，既可以告别繁琐的邮件系统，邮件系统可以退出KYC并作为辅助用户。
 

### 如何集成 DPal
* [API and demo](./api.md)

