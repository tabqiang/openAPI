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


def do_sign_request(url, data, app_key, app_secrete, method='post', headers=None):
    method = method.lower()
    assert method == 'get' or method == 'post'
    assert isinstance(data, dict)
    headers = headers or {}
    headers['Content-type'] = 'application/json'
    ts = str(int(time.time()))
    nonce = ''.join(random.sample(string.ascii_letters, 6))
    headers['-x-ts'] = ts  # 時間戳
    headers['-x-nonce'] = nonce  # 隨機字符串
    headers['-x-key'] = app_key

    if method == "get":
        query_str = urlparse.urlparse(url).query if not data else \
            urllib.urlencode({k: (v.encode("utf8") if isinstance(v, unicode) else v) for k, v in data.items()})
    else:
        query_str = json.dumps(data)
    sign_str = "{req_str}{ts}{nonce}{secret}".format(req_str=query_str, ts=ts, nonce=nonce, secret=app_secrete)
    sign = hashlib.sha256(sign_str).hexdigest()
    print "--sign_str: ", sign_str, "  \nsign=", sign
    headers['-x-sign'] = sign
    try:
        data = json.dumps(data) if method == 'post' else data
        req = getattr(requests, method)(url=url, data=data, headers=headers, timeout=3)
        print "result content; ", req.content  # {"data": {}, "reason": "", "ok": true}  # ok=true表示請求成功
        return json.loads(req.content)
    except Exception as ex:
        print "bad: ", ex
        # logging.error('[do fetch request content]: ' + repr(ex))
        return None


if __name__ == '__main__':
    do_sign_request(
        url="http://a.yunex.io/api/coin/bonus/transfer/",
        data={
            "bonus": [
                {
                     "to_uid": 16,
                     "symbol": "kt",
                     "amount": "1.13",
                     "date": "2018-10-16",
                     "order_id": "B20181016"
                }
            ]
        },
        app_key="91991e4def97bdd517e678ccd5c0ac856724ab7f6f3e9de14caa8325df51cccf",
        app_secrete="W8ZLD2x4N11PeCpZt2mCDtTSe8KpQtS6FkSgtS1qwnrWTK3QPGIDfpYEW061UlEe",
        method="post",
    )
```
