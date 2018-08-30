# 接口文檔
## 默認聲明
1. 接口host： `https://a.yunex.io`
2. 響應數據默認數據結構

    | 字段名 | 數據類型 | 說明 | 取值範圍 |
    | --- | --- | --- | --- |
    | ok | bool,int | 請求處理結果 | true, false, 1, 0 |
    | reason | int | 請求錯誤碼 | 詳情請參考[errcode.md](./errcode.md) |
    | data | object | 響應數據返回值 | 根據具體接口而定 |
   - 示例

        ```
        {
            "data": {},
            "reason": 111,
            "ok": false
        }
        ```

## 幣詳情、交易對詳情相關
1. 獲取所有幣種
    - 請求方式： GET
    - url： `{host}/api/v1/base/coins/list`
    - 驗證簽名： 不需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 | 取值範圍 |
        | --- | --- | --- | --- | --- |
        | base | int | N | 是否只獲取基礎幣種 | 1 |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | coin_id | string | 幣種ID |
        | name | string | 幣種名稱 |
        | symbol | string | 幣種標識 |
        | precision | string | 幣種精度 |
        | min_precision | int | 最小展示精度 |
        | base | bool | 是否為基礎幣種,不是則不返回 |
    - 示例
        ```
        {
            "data": {
                "coin_id": 1,
                "name": "USDT",
                "symbol": "usdt",
                "precision": 8,
                "min_precision": 8,
                "base": true
            },
            "reason": "",
            "ok": true
        }
        ```

1. 獲取交易對列表
    - 請求方式： GET
    - url： `{host}/api/v1/base/coins/tradepair`
    - 驗證簽名： 不需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 | 取值範圍 |
        | --- | --- | --- | --- | --- |
        | basecoin | string | N | 獲取指定基礎幣種ID的交易對列表 |  |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | pair_id | int | 交易對ID |
        | name | string | 交易對名稱 |
        | symbol | string | 交易對標識 |
        | coin_id | int | 交易幣種ID |
        | base_coin_id | int | 基礎幣種ID |
        | allow_trade | int | 是否允許交易 |
        | allow_order | int | 是否允許下單 |
        | display | int | 是否展示 |
    - 示例
        ```
        {
            "data": {
                "pair_id": 2,
                "name": "BTC/ETH",
                "symbol": "btc_eth",
                "coin_id": 3,
                "base_coin_id": 2,
                "allow_trade": 1,
                "allow_order": 1,
                "display": 1
            },
            "reason": "",
            "ok": true
        }
        ```

## 用戶資產相關
1. 獲取幣種資產
    - 請求方式： GET
    - url： `{host}/api/v1/coin/balance`
    - 驗證簽名： 需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 | 取值範圍 |
        | --- | --- | --- | --- | --- |
        | cid | int | N | 獲取指定幣種資產 |  |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | coin_id | int | 幣種ID |
        | name | string | 最小提幣數 |
        | symbol | string | 幣種標識 |
        | min_withdraw | string | 最小提幣數 |
        | max_withdraw | string | 最大提幣數 |
        | allow_withdraw | bool | 是否允許提幣 |
        | allow_deposit | bool | 是否允許充幣 |
        | allow_withdraw_free_fee | bool | 是否免費提幣到CoinHot.io |
        | dep_addr | string | 充幣地址 |
        | precision | int | 幣種精度 |
        | withdraw_fee | string | 提幣手續費 |
        | withdraw_fee_rate | string | 提幣費率 |
        | usable | string| 幣種可用資產 |
        | freezed | string | 幣種被凍結資產 |
        | total | string | 幣種總資產 |
        | logo | string | 幣種圖標url |
    - 示例
        ```
        {
            "data": {
                "coin": [{
                    "allow_withdraw": true,
                    "usable": "913529.29070061",
                    "max_withdraw": "100",
                    "name": "USDT",
                    "allow_deposit": true,
                    "symbol": "usdt",
                    "dep_addr": "n166L3FK2msWDaMeuAWmYaU2bhw7aVBQR6",
                    "allow_withdraw_free_fee": true,
                    "precision": 8,
                    "coin_id": 1,
                    "withdraw_fee_rate": "0.1",
                    "withdraw_fee": "0.1",
                    "total": "913529.29070061",
                    "min_withdraw": "0.01",
                    "freezed": "0",
                    "logo": "https://s.yunex.io/static/uploads/201805/66fbeac1db5fdb8.png",
                }],
                "total_usdt": "913529.290700"
            },
            "reason": "",
            "ok": true
        }
        ```

## 交易相關
1. 下買單
    - 請求方式： POST
    - url： `{host}/api/v1/order/buy`
    - 驗證簽名： 需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 |
        | --- | --- | --- | --- |
        | price | string | Y | 買入價格 |
        | volume | string | Y | 買入數量 |
        | symbol | string | Y | 交易對標識 |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | order_id | string | 訂單ID |
    - 示例
        ```
        {
            "data": {
                "order_id": "A18033116451362751420294"
            },
            "ok": true
        }

        ```

1. 下賣單
    - 請求方式： POST
    - url： `{host}/api/v1/order/sell`
    - 驗證簽名： 需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 |
        | --- | --- | --- | --- |
        | price | string | Y | 賣出價格 |
        | volume | string | Y | 賣出數量 |
        | symbol | string | Y | 交易對標識 |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | order_id | string | 訂單ID |
    - 示例
        ```
        {
            "data": {
                "order_id": "A18033116451362751420294"
            },
            "ok": true
        }

        ```

1. 申請取消委托訂單
    - 請求方式： POST
    - url： `{host}/api/v1/order/cancel`
    - 驗證簽名： 需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 |
        | --- | --- | --- | --- |
        | symbol | string | Y | 交易對標識 |
        | order_id | string | Y | 訂單ID |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | order_id | int | 委托訂單ID |
    - 示例
        ```
        {
            "data": {},
            "ok": true
        }

        ```

1. 查詢訂單狀態
    - 請求方式： POST
    - url： `{host}/api/v1/order/query`
    - 驗證簽名： 需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 |
        | --- | --- | --- | --- |
        | symbol | string | Y | 交易對標識 |
        | order_id | string | Y | 訂單ID |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | pair_id | int | 交易對ID |
        | order_id | string | 訂單ID |
        | user_id | int | 用戶ID |
        | type | int | 訂單類型,1:賣,2:買 |
        | timestamp | int | 訂單創建時間,納秒級別 |
        | price | string | 委托價格 |
        | volume | string | 委托數量 |
        | trade_volume | string | 成交數量 |
        | status | int | 訂單狀態,1:等待成交,2:已完成,3:已取消,4:失敗 |
    - 示例
        ```
        {
            "data": {
                "pair_id": 8,
                "order_id": "B18061514193903039730059",
                "user_id": 16,
                "type": 2,
                "timestamp": 1529043579030382874,
                "price": "4.871453",
                "volume": "11",
                "trade_volume": "0.0184",
                "status": 3
            },
            "ok": 1
        }
        ```

1. 查詢未結束成交訂單
    - 請求方式： POST
    - url： `{host}/api/v1/order/unfinished`
    - 驗證簽名： 需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 |
        | --- | --- | --- | --- |
        | symbol | string | Y | 交易對標識,傳空字符串則查詢所有交易對 |
        | direction | int | N | 查詢方向,1:正序，2:倒序 |
        | start | int | N | 分頁起始ID |
        | count | int | N | 每頁數量 |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | pair_id | int | 交易對ID |
        | order_id | string | 訂單ID |
        | user_id | int | 用戶ID |
        | type | int | 訂單類型,1:賣,2:買 |
        | timestamp | int | 訂單創建時間,納秒級別 |
        | price | string | 委托價格 |
        | volume | string | 委托數量 |
        | trade_volume | string | 成交數量 |
        | status | int | 訂單狀態,1:等待成交,2:已完成,3:已取消,4:失敗 |
        | id | int | 分頁ID |
    - 示例
        ```
        {
            "data": {
                "count": 1,
                "list": [{
                    "pair_id": 3,
                    "order_id": "B18072319025470778740386",
                    "user_id": 80,
                    "type": 2,
                    "timestamp": 1532343774707775786,
                    "price": "1.00000000",
                    "volume": "1.00000",
                    "trade_volume": "0.00000",
                    "amount": "1.00000000",
                    "status": 1,
                    "id": 1406365
                }]
            },
            "ok": 1
        }
        ```

1. 分月查詢成交記錄
    - 請求方式： POST
    - url： `{host}/api/v1/order/trade/history`
    - 驗證簽名： 需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 | 取值範圍 |
        | --- | --- | --- | --- | --- |
        | symbol | string | Y | 交易對標識 |  |
        | direction | int | N | 查詢方向,1:正序，2:倒序 |  |
        | start | int | N | 分頁起始ID |  |
        | count | int | N | 每頁數量 |  |
        | status | int | Y | 記錄狀態 | 0 |
        | yearmonth | string | Y | 查詢年月 | "yyyymm" |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | count | int | 總數量 |
        | pair_id | int | 交易對ID |
        | trade_id | string | 成交ID |
        | type | int | 訂單類型,1:賣,2:買 |
        | timestamp | int | 成交時間,納秒級別 |
        | price | string | 成交價格 |
        | volume | string | 成交數量 |
        | amount | string | 成交額 |
        | id | int | 分頁ID |
    - 示例
        ```
        {
            "data": {
                "count": 1,
                "list": [
                    {
                        "pair_id": 1,
                        "trade_id": "T180514234007c4f7250d159",
                        "timestamp": 1526267586992620300,
                        "price": "852.115243",
                        "volume": "0.01629",
                        "amount": "13.88096",
                        "type": 2,
                        "id": 64
                    }
                ]
            },
            "ok": 1
        }
        ```

1. 查詢指定交易對成交記錄
    - 請求方式： POST
    - url： `{host}/api/v1/order/trade/recent`
    - 驗證簽名： 不需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 | 取值範圍 |
        | --- | --- | --- | --- | --- |
        | symbol | string | Y | 交易對標識 |  |
        | direction | int | N | 查詢方向,1:正序，2:倒序 |  |
        | start | int | N | 分頁起始ID |  |
        | count | int | N | 每頁數量 |  |
        | status | int | Y | 記錄狀態 | 0 |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | pair_id | int | 交易對ID |
        | trade_id | string | 成交ID |
        | trade_by | int | 成交類型,1:賣,2:買 |
        | timestamp | int | 成交時間,納秒級別 |
        | price | string | 成交價格 |
        | volume | string | 成交數量 |
        | id | int | 分頁ID |
    - 示例
        ```
        {
            "data": {
                "list": [{
                    "pair_id": 3,
                    "trade_id": "T180823780b873599575b0c3",
                    "trade_by": 2,
                    "timestamp": 1535006629598808064,
                    "price": "3717.6633",
                    "volume": "0.0001",
                    "id": 4317
                }]
            },
            "ok": 1
        }
        ```

## 行情相關
1. 查詢盤口
    - 請求方式： GET
    - url： `{host}/api/market/depth`
    - 驗證簽名： 不需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 | 取值範圍 |
        | --- | --- | --- | --- | --- |
        | symbol | string | Y | 交易對標識 |  |
        | level | int | N | 盤口條數 | 1-15,默認5 |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | asks | object | 賣盤盤口,數組元素下標對應值描述,0:價格,1:數量 |
        | bids | object | 買盤盤口,數組元素下標對應值描述,0:價格,1:數量 |
    - 示例
        ```
        {
            "data": {
                "asks": [
                    ["8085.4879", "0.0123"],
                    ["8087.4032", "0.0133"]
                ],
                "bids": [
                    ["7971.112", "0.0254"],
                    ["7967.8713", "0.0161"]
                ]
            },
            "ok": 1
        }
        ```

1. 查詢K線
    - 請求方式： GET
    - url： `{host}/api/market/trade/kline`
    - 驗證簽名： 不需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 | 取值範圍 |
        | --- | --- | --- | --- | --- |
        | symbol | string | Y | 交易對標識 |  |
        | type | string | Y | k線類型 | '1min', '5min', '15min', '30min', '1hour', '4hour', '1day' |
        | start | int | N | 開始時間戳 | 默認當前時間戳 |
        | count | int | Y | 每頁數量 | 2-300 |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | ts | int | 服務器當前時間,秒級別 |
        | data->ts | int | k線數據時間戳,秒級別 |
        | v | object | 數組元素下標對應值描述 0:開盤價,1:收盤價,2:最高價,3:最低價,4:成交量,5:成交額 |
    - 示例
        ```
        {
            "data": [{
                "ts": 1532403420,
                "v": ["8071.2796", "8071.2796", "8071.2796", "8071.2796", "0", "0"]
            }, {
                "ts": 1532403480,
                "v": ["8071.2796", "8071.2796", "8071.2796", "8071.2796", "0", "0"]
            }],
            "ok": 1,
            "ts": 1532403534
        }
        ```

1. 市場行情
    - 請求方式： GET
    - url： `{host}/api/market/trade/info`
    - 驗證簽名： 不需要
    - 請求參數：

        | 字段名 | 數據類型 | 是否必須 | 說明 | 取值範圍 |
        | --- | --- | --- | --- | --- |
        | symbol | string | Y | 交易對標識 |  |
    - data數據描述：

        | 字段名 | 數據類型 | 說明 |
        | --- | --- | --- |
        | close_price | string | 收盤價 |
        | coin_pair | string | 交易對標識 |
        | cur_price | string | 當前價格 |
        | max_price | string | 最高價 |
        | min_price | string | 最低價 |
        | open_price | string | 開盤價 |
        | rate | string | 漲跌幅 |
        | amount | string | 24小時成交額 |
        | volume | string | 24小時成交量 |
    - 示例
        ```
        {
            "data": {
                "amount": "0",
                "close_price": "0",
                "coin_pair": "kt_usdt",
                "cur_price": "0.1499",
                "max_price": "0.1499",
                "min_price": "0.1499",
                "open_price": "0",
                "rate": "0",
                "volume": "0"
            },
            "ok": 1
        }
        ```
