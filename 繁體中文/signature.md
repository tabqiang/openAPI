## 簽名:

    簽名相關參數和簽名都放請求頭部 說明如下
    -x-ts: 當前時間戳字符串(與服務器時間相差不得超過600S)
    -x-nonce: 隨機字符串
    -x-key: app key
    簽名方式:

        1. 簽名字符串連接: 所有請求參數拼接起來的字符串 + 時間戳字符串 + 隨機字符串 + app_secrete  # (不包含+)
        2. 然後進行sha256 簽名寫到請求頭的-x-sign裏面

        舉例如下:
            - 請求參數: {'a':12,'b':'中文','c':"c c"}
            - 請求參數拼接
                1. 所有參數按照key首字母排序 也就是a在前 然後是b, c
                2. 參數需要進行urlencode 也就是 '中文' => %E4%B8%AD%E6%96%87 , "c c" => "c+c"
                3. 請求參數拼接之後的字符串為: a=12&b=%E4%B8%AD%E6%96%87&c=c+c
            - 簽名
                - 時間戳: 1533203011
                - 隨機字符串: aaa
                - app_secret: EjmkjrcQdmhTsUOc0aLPgSgVnkR5tMdPuiWKS2itnPTHZim9HumCRXpi3I2fU94X
                - 簽名原始字符串為: a=12&b=%E4%B8%AD%E6%96%87&c=c+c1533203011aaaEjmkjrcQdmhTsUOc0aLPgSgVnkR5tMdPuiWKS2itnPTHZim9HumCRXpi3I2fU94X
            - 最終生成簽名(sha256):
                1dfff694ade12ed7a97247454d6fff2bb5457a41787d374276b18db691a6de49

## Python Demo

```python
# coding: utf8

import time
import random
import hashlib
import json
import string
import requests
import urllib
import urlparse
import unittest



def _build_api_sign(secret, ts, nonce, req_data):
    for k in req_data:
        value = req_data[k]
        if isinstance(value, unicode):
            value = value.encode('utf8')
        else:
            value = str(value)
        req_data[k] = urllib.quote_plus(value)  # a a -> a+a
    d = sorted(req_data.iteritems(), key=lambda x: x[0])
    req_str = "&".join(["%s=%s" % (x[0], x[1]) for x in d])
    sign_str = "{req_str}{ts}{nonce}{secret}".format(req_str=req_str, ts=ts, nonce=nonce, secret=secret)
    return hashlib.sha256(sign_str).hexdigest()


def do_sign_request(url, data, app_key, app_secrete, method='post', headers=None):
    assert method == 'get' or method == 'post'
    assert isinstance(data, dict)
    headers = headers or {}
    headers['Content-type'] = 'application/json'
    ts = str(int(time.time()))
    nonce = ''.join(random.sample(string.ascii_letters, 6))
    headers['-x-ts'] = ts  # 时间戳
    headers['-x-nonce'] = nonce  # 随机字符串
    headers['-x-key'] = app_key
    sign_data = data or {}
    if method == "get":
        query = urlparse.urlparse(url).query
        sign_data = dict([(k, v[0]) for k, v in urlparse.parse_qs(query).items()])

    sign = _build_api_sign(app_secrete, ts, nonce, sign_data)
    headers['-x-sign'] = sign
    try:
        # print headers
        data = json.dumps(data) if method == 'post' else data
        req = getattr(requests, method)(url=url, data=data, headers=headers, timeout=3)
        print "result content; ", req.content  # {"data": {}, "reason": "", "ok": true}  # ok=true表示请求成功
        return json.loads(req.content)
    except Exception as ex:
        print "bad: ", ex
        # logging.error('[do fetch request content]: ' + repr(ex))
        return None



class TestApiToken(unittest.TestCase):
    host = "https://a.yunex.io"
    app_key = "835f4c0e1ba5d7a7b20ffad2a562f673816993239bb3d69fa306e48729917102"
    app_secrete = "1"

    app_key_readonly= "c86e420a142ba381aafd5cbbf99323138e84a35e25cc1bd9b5411ea8062f57ee"
    app_secrete_readonly = "2"


    def test_query_order(self):
        res = do_sign_request(
                url="%s/api/v1/order/query?symbol=btc_usdt&order_id=B18072612051288996137086" % self.host,
                data={},
                app_key=self.app_key,
                app_secrete=self.app_secrete,
                method="get"
            )
        self.assertEqual(res['ok'], 1)

        res = do_sign_request(
            url="%s/api/v1/order/query?symbol=btc_usdt&order_id=B18072612051288996137086" % self.host,
            data={},
            app_key=self.app_key_readonly,
            app_secrete=self.app_secrete_readonly,
            method="get"
        )
        self.assertEqual(res['ok'], 1)

    def test_buy_order(self):
        res = do_sign_request(
                url="%s/api/v1/order/buy" % self.host,
                data={
                    "price": "3.25",
                    "volume": "1",
                    "symbol": "yun_usdt"
                },
                app_key=self.app_key,
                app_secrete=self.app_secrete,
                method="post"
            )
        self.assertEqual(res['ok'], 1)

        res = do_sign_request(
                url="%s/api/v1/order/buy" % self.host,
                data={
                    "price": "3.25",
                    "volume": "1",
                    "symbol": "yun_usdt"
                },
                app_key=self.app_key_readonly,
                app_secrete=self.app_secrete_readonly,
                method="post"
            )
        self.assertEqual(res['ok'], 0)

    def test_sell_order(self):
        res = do_sign_request(
                url="%s/api/v1/order/sell" % self.host,
                data={
                    "price": "4.25",
                    "volume": "1",
                    "symbol": "yun_usdt"
                },
                app_key=self.app_key,
                app_secrete=self.app_secrete,
                method="post"
            )
        self.assertEqual(res['ok'], 1)

        res = do_sign_request(
            url="%s/api/v1/order/sell" % self.host,
            data={
                "price": "4.25",
                "volume": "1",
                "symbol": "yun_usdt"
            },
            app_key=self.app_key_readonly,
            app_secrete=self.app_secrete_readonly,
            method="post"
        )
        self.assertEqual(res['ok'], 0)

    def test_cancel_order(self):
        res = do_sign_request(
            url="%s/api/v1/order/cancel" % self.host,
            data={
                "order_id": "",
                "symbol": "yun_usdt"
            },
            app_key=self.app_key,
            app_secrete=self.app_secrete,
            method="post"
        )
        self.assertEqual(res['ok'], 1)

        res = do_sign_request(
            url="%s/api/v1/order/cancel" % self.host,
            data={
                "order_id": "",
                "symbol": "yun_usdt"
            },
            app_key=self.app_key_readonly,
            app_secrete=self.app_secrete_readonly,
            method="post"
        )
        self.assertEqual(res['ok'], 0)


    def test_unfinished_order(self):
        res = do_sign_request(
                url="%s/api/v1/order/unfinished?symbol=yun_usdt&direction=1&start=1&count=1&more=1" % self.host,
                data={},
                app_key=self.app_key,
                app_secrete=self.app_secrete,
                method="get"
            )
        self.assertEqual(res['ok'], 1)

        res = do_sign_request(
            url="%s/api/v1/order/unfinished?symbol=yun_usdt&direction=1&start=1&count=1&more=1" % self.host,
            data={},
            app_key=self.app_key_readonly,
            app_secrete=self.app_secrete_readonly,
            method="get"
        )
        self.assertEqual(res['ok'], 1)


if __name__ == '__main__':
    unittest.main()
```
