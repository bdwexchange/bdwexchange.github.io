### 一.Java,demo(API接口文档)
#### 获取签名方式

```

        private static final String API_HOST = "39.100.79.158";
        private static final String SIGNATURE_METHOD = "HmacSHA256";
        private static final String SIGNATURE_VERSION = "2";
        private static final ZoneId ZONE_GMT = ZoneId.of("Z");
        private static final DateTimeFormatter DT_FORMAT = DateTimeFormatter.ofPattern("uuuu-MM-dd'T'HH:mm:ss");


    private static String createSignature(String method, String path, String apiKey, String timeStamp,
                                          Map map, String secretKey) throws Exception{
        StringBuilder sb = new StringBuilder(1024);
        // GET
        sb.append(method.toUpperCase()).append('\n')
                // Host
                .append(API_HOST.toLowerCase()).append('\n')
                // path
                .append(path).append('\n');



        StringJoiner joiner = new StringJoiner("&");
        joiner.add("accessKeyId=" + apiKey)
                .add("signatureMethod=" + SIGNATURE_METHOD)
                .add("signatureVersion=" + SIGNATURE_VERSION)
                .add("timestamp=" + encode(timeStamp));


        //拼接 遍历map
        Iterator<Map.Entry<String, String>> entries = map.entrySet().iterator();
        while (entries.hasNext()){
            Map.Entry<String, String> entry = entries.next();
            joiner.add(entry.getKey()+"="+entry.getValue());
        }
        log.info("sb={},joiner={}",sb.toString(),joiner.toString());
        return sign(sb.toString() + joiner.toString(), secretKey);
    }

    public static String sign(String message, String secret) {
        try {
            Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
            SecretKeySpec secretKeySpec = new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), "HmacSHA256");
            sha256_HMAC.init(secretKeySpec);
            byte[] hash = sha256_HMAC.doFinal(message.getBytes());
            return Base64.getEncoder().encodeToString(hash);
        } catch (Exception e) {
            throw new RuntimeException("Unable to sign message.", e);
        }
    }

    private static String encode(String code) {
        try {
            return URLEncoder.encode(code, "UTF-8").replaceAll("\\+", "%20");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            return null;
        }
    }

```
#### 下单接口示例,获取签名

```
/**                    
    * 下单接口示例,获取签名              
    * @param args         
    * @throws Exception   
    */                    
    public static void main(String[] args) throws Exception{
        String s = Instant.now().atZone(ZONE_GMT).format(DT_FORMAT);
        String timeStamp = URLEncoder.encode(s, "UTF-8").replaceAll("\\+", "%20");
        System.out.println( timeStamp);
        Map<String,String> map = new TreeMap<>();
        map.put("memberId","9");
        map.put("direction","SELL");
        map.put("symbol","BTC/USDT");
        map.put("price","6750");
        map.put("amount","1");
        map.put("type","LIMIT_PRICE");
        String signature = createSignature("POST","/open-api/user/add_order",
                "5f79608e-e270-401c-9c13-f7ff8800952d",timeStamp,
                map,"989eb428-af34-49e2-a793-a4f14fc28f44");
        System.out.println(signature);
    }

```

### 二.获取签名

1. 获取签名signature(示例如上)

参数: 

- method:GET或POST

- path:请求路径(/open-api/user/add_order)

- apiKey:申请的APIkey

- secretKey:申请的APIsecret

- timeStamp:时间戳格式("uuuu-MM-dd'T'HH:mm:ss")

- map:请求的参数集合(字典顺序),没有时传空对象

2. 获取签名signature后进行URL编码

3. 请求示例

https:// api.nr3d.cn/open-api/user/add_order?accessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&memberId=1234567890&signatureMethod=HmacSHA256&signatureVersion=2&timestamp=2017-05-11T15%3A19%3A30&signature=EZkLJhNMHn6ApWGFna%2FrTBWqTxwlH0k2GQUrDJJNAls%3D

所有的API请求都以GET或者POST形式发出。

4. 返回内容主体格式

```
{
	"data": null,
	"code": 0,
	"message": "success",
	"total": null
}

```

### 三.接口信息

需要签名的接口(请求示例如上)
需要签名的接口公共参数(必传):
- signatureMethod:HmacSHA256
- signatureVersion:2
- timestamp:时间戳格式("uuuu-MM-dd'T'HH:mm:ss")
- signature:计算出的signature
- accessKeyId:申请的APIkey

### 四.apiDemo

[apiDemo](https://github.com/bdwexchange/api.git)

#### 需要签名的接口

1 获取平台id

- 请求路径: /open-api/user/get/account
- 请求方式: GET
- 参数类型: form 表单
- 参数:     无

返回参数示例:

```

{
	"data": {
		"memberId": 9
	},
	"code": 0,
	"message": "SUCCESS",
	"total": null
}

```

2 查询个人账户信息

- 请求路径: /open-api/user/account
- 请求方式: GET
- 参数类型: form表单
- 请求参数: memberId

参数示例: 

参数名 | 参数值
---|---
memberId | 9

返回参数示例:
```

{
	"data": [{
			"memberId": 9,              //账户id
			"coinId": "BTC",            //币种
			"balance": 998361.01453347, //余额
			"frozenBalance": 1.63349428,//冻结余额
			"address": null,            //地址
			"isLock": 0                 //是否锁定
		},
		{
			"memberId": 9,
			"coinId": "ETH",
			"balance": 996602.51137612,
			"frozenBalance": 23.59323104,
			"address": null,
			"isLock": 0
		}
	],
	"code": 0,
	"message": "SUCCESS",
	"total": null
}

```
3 下单

- 请求路径: /open-api/user/add_order
- 请求方式: POST
- 参数类型: form 表单

参数示例:

参数名 | 参数值
---|---
memberId | 9
direction | BUY/SELL
symbol | ETH/USDT
price | 1.2
amount | 1
type | MARKET_PRICE,LIMIT_PRICE

返回参数示例:

```
{
	"data": {
		"orderId": E155722091722695
	},
	"code": 0,
	"message": "SUCCESS",
	"total": null
}

```
5 根据订单Id查询订单详情

- 请求路径: /open-api/user/query/order_detail
- 请求方式: GET
- 参数类型: form 表单

参数示例:

参数名 | 参数值
---|---
orderId | E155722091722695

返回参数

```
{
	"data": {
		"orderId": "E155722091722695",              //订单id
		"memberId": 9,                              //会员id
		"type": "LIMIT_PRICE",                      //订单类型
		"amount": 0.1899,                           //数量
		"symbol": "BTC/USDT",                       //交易对
		"tradedAmount": 0.1899,                     //成交量                     
		"turnover": 1128.138031,                    //成交额，对市价买单有用
		"coinSymbol": "BTC",                        //币单位
		"baseSymbol": "USDT",                       //结算单位
		"status": "COMPLETED",                      //订单状态
		"direction": "BUY",                         //订单方向
		"price": 5942.83,                           //挂单价格
		"triggerPrice": 0,                          //触发价
		"time": 1557220917226,                      //挂单时间
		"completedTime": 1557220917243,             //交易完成时间
		"canceledTime": null,                       //取消时间
		"marginTrade": 0,                           //是否来自杠杆交易
		"orderResource": "API",                   //订单来源
		"detail": [{                              //订单详情
				"orderId": "E155722091722695",    //订单id    
				"price": 5939.72,                 //价格
				"amount": 0.079,                  //数量
				"turnover": 469.23788,            //成交额
				"fee": 0.00158,                   //手续费
				"time": 1557220917243             //时间
			},
			{
				"orderId": "E155722091722695",
				"price": 5941.39,
				"amount": 0.1109,
				"turnover": 658.900151,
				"fee": 0.002218,
				"time": 1557220917263
			}
		],
		"amountStr": null,
		"priceStr": null,
		"completed": true
	},
	"code": 0,
	"message": null,
	"total": null
}

```
6 根据用户id和交易对查询当前委托订单列表
- 请求路径: /open-api/user/query/order
- 请求方式: POST
- 参数类型: form 表单

参数示例:

参数名 | 参数值 | 备注
---|---|---
memberId | 9 |
symbol | BTC/USDT | 必传
direction | BUY,SELL | 非必传
startTime | 1558946032000 |
endTime | 1558946032000 |
pageNum | 1 |
pageSize | 2 |

返回参数

 ```
 
 {
	"data": {
		"content": [{
			"orderId": "E155722139978120",      //订单号
			"memberId": 9,                      //会员id
			"type": "LIMIT_PRICE",              //挂单类型
			"amount": 0.51,                     //买入或卖出量
			"symbol": "ETH/USDT",               //交易对
			"tradedAmount": 0,                  //成交量
			"turnover": 0,                      //成交额
			"coinSymbol": "ETH",                //币单位
			"baseSymbol": "USDT",               //结算单位
			"status": "TRADING",                //订单状态
			"direction": "BUY",                 //订单方向
			"price": 177.33,                    //挂单价格
			"triggerPrice": 0,                  //触发价
			"time": 1557221399781,              //挂单时间
			"completedTime": null,              //交易完成时间
			"canceledTime": null,               //取消时间
			"marginTrade": 0,                   //是否来自杠杆交易
			"orderResource": "API",             //订单来源
			"detail": [],                       //订单详情
			"amountStr": null,
			"priceStr": null,
			"completed": false
		}],
		"totalElements": 21,
		"totalPages": 21,
		"last": false,
		"size": 1,
		"number": 0,
		"first": true,
		"sort": [{
			"direction": "DESC",
			"property": "time",
			"ignoreCase": false,
			"nullHandling": "NATIVE",
			"ascending": false,
			"descending": true
		}],
		"numberOfElements": 1
	},
	"code": 0,
	"message": null,
	"total": null
}
 
```

7 根据订单号取消订单

- 请求路径: /open-api/user/cancel_order
- 请求方式: GET
- 参数类型: form 表单

参数示例

参数名 | 参数值 | 备注
---|---|---
orderId | E155722139978120 |
memberId | 9 |

返回参数

```

{
	"data": null,
	"code": 0,
	"message": "执行成功",
	"total": null
}

```

8 查询用户历史委托

- 请求路径: /open-api/user/history
- 请求方式: POST
- 参数类型: form 表单

参数示例

参数名 | 参数值 | 备注
---|---|---
memberId | 9 |
symbol | BTC/USDT | 非必传
direction | BUY/SELL | 非必传
type | MARKET_PRICE,LIMIT_PRICE | 非必传
startTime | long型时间戳 | 非必传
endTime | long型时间戳 | 非必传
pageNum | 1 | 非必传
pageSize | 10 | 非必传

返回参数

```
{
	"data": {
		"content": [{
			"orderId": "E155722139978120",      //订单号
			"memberId": 9,                      //会员id
			"type": "LIMIT_PRICE",              //挂单类型
			"amount": 0.51,                     //买入或卖出量
			"symbol": "ETH/USDT",               //交易对
			"tradedAmount": 0,                  //成交量
			"turnover": 0,                      //成交额
			"coinSymbol": "ETH",                //币单位
			"baseSymbol": "USDT",               //结算单位
			"status": "TRADING",                //订单状态
			"direction": "BUY",                 //订单方向
			"price": 177.33,                    //挂单价格
			"triggerPrice": 0,                  //触发价
			"time": 1557221399781,              //挂单时间
			"completedTime": null,              //完成时间
			"canceledTime": null,               //创建时间
			"marginTrade": 0,                   //是否来自杠杆
			"orderResource": "API",             //订单来源
			"detail": [],                       //订单详情
			"amountStr": null,                  //数量
			"priceStr": null,                   //价格
			"completed": false                  //完全成交
		}],
		"totalElements": 21,
		"totalPages": 21,
		"last": false,
		"size": 1,
		"number": 0,
		"first": true,
		"sort": [{
			"direction": "DESC",
			"property": "time",
			"ignoreCase": false,
			"nullHandling": "NATIVE",
			"ascending": false,
			"descending": true
		}],
		"numberOfElements": 1
	},
	"code": 0,
	"message": null,
	"total": null
}

```

#### 非apiKey验证接口(不需要验证)
1 获取支持交易对symbol_thumb讯息

- 请求路径: /open-api/open/symbol_thumb
- 请求方式: GET
- 参数类型: form 表单

参数示例：无

返回示例: 无

2 获取历史K线图 根据时间(只查询最近一天的数据,最大100条)

- 请求路径: /open-api/open/history/kline
- 请求方式: POST
- 参数类型: form 表单

参数示例 

参数名 | 参数值 | 备注
---|---|---
symbol | ETH/USDT |
period | 1min 5min 15min 30min 1D 1week 1month |
size | 100 |

返回示例

```

{
	"data": [{
		"symbol": null,             //交易对
		"openPrice": 163.69,        //开
		"highestPrice": 163.7,      //高
		"lowestPrice": 163.69,      //低
		"closePrice": 163.7,        //关
		"time": 1557135900000,      //时间
		"period": "1min",           //时间粒度
		"count": 6,                 //成交笔数
		"volume": 1.12,             //成交量
		"turnover": 183.3407        //成交额
	}],
	"code": 0,
	"message": "SUCCESS",
	"total": null
}

```

3 获取盘口信息

- 请求路径: /open-api/open/trade_plate
- 请求方式: POST
- 参数类型: form 表单

参数示例 

参数名 | 参数值 | 备注
---|---|---
symbol | ETH/USDT |
size | 100 |

返回示例 无

4 获取成交历史信息(最大100条)

- 请求路径: /open-api/open/trade_history
- 请求方式: POST
- 参数类型: form 表单

参数示例

参数名 | 参数值 | 备注
---|---|---
symbol | ETH/USDT |
size | 100 |

返回示例

```

{
	"data": [{
		"symbol": "ETH/USDT",               //交易对
		"price": 178.25,                    //价格
		"priceStr": "178.2500",             //String 价格
		"amount": 0.6,                      //数量
		"amountStr": "0.60",                //String 数量
		"buyTurnover": 106.95,              //买卖等值
		"sellTurnover": 106.95,             //买卖等值
		"direction": "SELL",                //描述
		"buyOrderId": "E155722235445529",   //买单id
		"sellOrderId": "E155722238683592",  //卖单id
		"time": 1557222386842               //时间
	}],
	"code": 0,
	"message": "SUCCESS",
	"total": null
}


```