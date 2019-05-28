# DigiFinex Websocket接口(2019-5-28)
# 基本信息
* 本篇所列出的所有wss接口的baseurl为: **wss://stream.digifinex.com:9443**
* 所有stream均可以直接访问，或者作为组合streams的一部分。
* 直接访问时URL格式为 **/ws/\<streamName\>**
* 组合streams的URL格式为 **/stream?streams=\<streamName1\>/\<streamName2\>/\<streamName3\>**
* 订阅组合streams时，事件payload会以这样的格式封装 **{"stream":"\<streamName\>","data":\<rawPayload\>}**
* stream名称中所有交易对均为**小写**
* 每个到**stream.binance.com**的链接有效期不超过24小时，请妥善处理断线重连。
* 每3分钟，服务端会发送ping帧，客户端应当在10分钟内回复pong帧，否则服务端会主动断开链接。允许客户端发送不成对的pong帧(即客户端可以以高于10分钟每次的频率发送pong帧保持链接)。
# Stream 详细定义
## 归集交易
归集交易与逐笔交易的区别在于，同一个taker在同一价格与多个maker成交时，会被归集为一笔成交。

**Stream 名称:** \<symbol\>@aggTrade

**Payload:**
```javascript
{
  "e": "aggTrade",  // 事件类型
  "E": 123456789,   // 事件时间
  "s": "BNBBTC",    // 交易对
  "a": 12345,       // 归集交易ID
  "p": "0.001",     // 成交价格
  "q": "100",       // 成交数量
  "f": 100,         // 被归集的首个交易ID
  "l": 105,         // 被归集的末次交易ID
  "T": 123456785,   // 成交时间
  "m": true,        // 买方是否是做市方。如true，则此次成交是一个主动卖出单，否则是一个主动买入单。
  "M": true         // 请忽略该字段
}
```

## 逐笔交易
逐笔交易推送每一笔成交的信息。**成交**，或者说交易的定义是仅有一个吃单者与一个挂单者相互交易。

**Stream 名称:** \<symbol\>@trade

**Payload:**
```javascript
{
  "e": "trade",     // 事件类型
  "E": 123456789,   // 事件时间
  "s": "BNBBTC",    // 交易对
  "t": 12345,       // 交易ID
  "p": "0.001",     // 成交价格
  "q": "100",       // 成交数量
  "b": 88,          // 买方的订单ID
  "a": 50,          // 卖方的订单ID
  "T": 123456785,   // 成交时间
  "m": true,        // 买方是否是make单。如true，则此次成交是一个主动卖出单，否则是一个主动买入单。
  "M": true         // 请忽略该字段
}
```


## 有限档深度信息
每秒推送有限档深度信息。levels表示几档买卖单信息, 可选 5/10/20档

**Stream 名称:** \<symbol\>@depth\<levels\>

**Payload:**
```javascript
{
  "lastUpdateId": 160,  // 末次更新ID
  "bids": [             // 买单
    [
      "0.0024",         // 价
      "10",             // 量
      []                // 忽略
    ]
  ],
  "asks": [             // 卖单
    [
      "0.0026",         // 价
      "100",            // 量
      []                // 忽略
    ]
  ]
}
```

## 增量深度信息stream
第一次会将当前队列推送过去，后续则只推送增量
每秒推送orderbook的变化部分（如果有）

**Stream 名称:** \<symbol\>@depth

**Payload:**
```javascript
{
  "e": "depthUpdate", // 事件类型
  "E": 123456789,     // 事件时间
  "s": "BNBBTC",      // 交易对
  "T": 0,             // 类型 0表示现有队列，1表示增量
  "U": 157,           // 从上次推送至今新增的第一个 update Id
  "u": 160,           // 从上次推送至今新增的最后一个 update Id
  "b": [              // 变动的买单深度
    [
      "0.0024",       // 价
      "10",           // 量
      []              // Ignore
    ]
  ],
  "a": [              // 变动的卖单深度
    [
      "0.0026",       // 价
      "100",          // 量
      []              // Ignore
    ]
  ]
}
```

## 如何正确在本地维护一个orderbook副本



