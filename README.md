
## 签名器 API接口

### SatoTx

我们部署了一个签名器 [https://api.satotx.com](https://api.satotx.com) ，可暂做测试，获取utxo的签名。支持的API如下：

### 部署方法

需要设置3个环境变量

* 配置监听端口 `LISTEN=0.0.0.0:8000`
* 配置私钥P `PINT=4500086422777284614649185698080302158559082854283623085071286437702609134945108376498504441947235804413771973268603081955797477873992855707579719874199`
* 配置私钥Q `QINT=644767523354888926443294407216811496298090339710859618436800524594486048803150814665399137334438348620190687521269075965377327693448171249204212488583`

1. 普通web API运行方式，
```
$ go build -v main.go     # 编译
$ PINT=313242 QINT=328471 LISTEN=:8000 ./satotx  # 运行
```
2. 使用aws lambda方式部署
```
$ GOOS=linux GOARCH=amd64 go build -v main_aws.go    # 编译
```

### 0. Welcome

可获知签名公钥等信息。

#### Request
- Method: **GET**
- URL:  ```/```

#### Response

data包括字段为：

- pubKey: rabin签名的公钥，hex编码
- contact: 联系方式
- github: 项目地址

- Body
```
{
  code: 0,
  msg: "Welcome to use sensing contract on Bitcoin SV!",
  data: {
    "pubKey": "25108ec89eb96b99314619eb5b124f11f00307a833cda48f5ab1865a04d4cfa567095ea4dd47cdf5c7568cd8efa77805197a67943fe965b0a558216011c374aa06a7527b20b0ce9471e399fa752e8c8b72a12527768a9fc7092f1a7057c1a1514b59df4d154df0d5994ff3b386a04d819474efbd99fb10681db58b1bd857f6d5",
    "contact": "",
    "job": "",
    "github": "https://github.com/sensing-contract"
  }
}
```

### 1. 对“某UTXO”签名


URL中需要的参数为：

1. txid: 产生UTXO的txid
2. index: UTXO的output index

Body中需要的json参数为：

1. txHex: 产生UTXO的rawtx内容


#### Request
- Method: **POST**
- Headers：`Content-Type: application/json`
- URL:  ```/utxo/{txid}/{index}```
  - 示例:  ```/utxo/4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b/0```
- Body:
```
{
  "txHex": "01000000010000000000000000000000000000000000000000000000000000000000000000ffffffff4d04ffff001d0104455468652054696d65732030332f4a616e2f32303039204368616e63656c6c6f72206f6e206272696e6b206f66207365636f6e64206261696c6f757420666f722062616e6b73ffffffff0100f2052a01000000434104678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5fac00000000"
}
```

#### Response

返回值 code == 0 为正确，其他都是错误

data包括字段为：

- txId: 同输入参数
- index: 同输入参数
- byTxId: 空
- sigBE: 签名，大端字节序，hex编码
- padding: 签名的padding，hex编码
- payload: 签名的内容，hex编码
- script: 脚本原始内容，hex编码

其中payload字节内容为：

    txid, index, value, hash160(script)

txid在payload中为原始字节序, index是小端4字节，value是小端8字节。

- Body
```
{
  "code": 0,
  "msg": "ok",
  "data": {
    "txId": "4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b",
    "index": 0,
    "byTxId": "",
    "sigBE": "07ffec4dddc4a881ccd12d7075946304cd5544525c02b366f643363267f8916d9b4f57374b6aa9867c3f7bacb8ecf528ef056b14e7fb2b2cc9e45bac10a8ab15d07dff74c6e8830977bda7c421b5a53c545d3aff1ac63757d2aed201148113e6fd6d6676ebc264f63c46e58528c708504dfc86dafbccbbaa57b3c0a89f1871f1",
    "padding": "0800",
    "payload": "3ba3edfd7a7b12b27ac72c3e67768f617fc81bc3888a51323a9fb8aa4b1e5e4a0000000000f2052a010000008424e7542477ef1a76cbab88d4b177d2fb5a96c1",
    "script": "4104678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5fac"
  }
}
```


### 2. 对“某UTXO被下一个Tx花费”签名

URL中需要的参数为：

- txid: 产生UTXO的txid
- index: UTXO的output index
- byTxid: 花费UTXO的txid

Body中需要的json参数为：
- txHex: 产生UTXO的rawtx内容
- byTxHex: 花费UTXO的rawtx内容

#### Request
- Method: **POST**
- Headers：`Content-Type: application/json`
- URL:  ```/utxo-spend-by/{txid}/{index}/{byTxid}```
    - 示例:  ```/utxo-spend-by/0437cd7f8525ceed2324359c2d0ba26006d92d856a9c20fa0241106ee5a597c9/0/f4184fc596403b9d638783cf57adfe4c75c605f6356fbc91338530e9831e9e16```
- Body:
```
{
  "txHex": "01000000010000000000000000000000000000000000000000000000000000000000000000ffffffff0704ffff001d0134ffffffff0100f2052a0100000043410411db93e1dcdb8a016b49840f8c53bc1eb68a382e97b1482ecad7b148a6909a5cb2e0eaddfb84ccf9744464f82e160bfa9b8b64f9d4c03f999b8643f656b412a3ac00000000",
  "byTxHex": "0100000001c997a5e56e104102fa209c6a852dd90660a20b2d9c352423edce25857fcd3704000000004847304402204e45e16932b8af514961a1d3a1a25fdf3f4f7732e9d624c6c61548ab5fb8cd410220181522ec8eca07de4860a4acdd12909d831cc56cbbac4622082221a8768d1d0901ffffffff0200ca9a3b00000000434104ae1a62fe09c5f51b13905f07f06b99a2f7159b2225f374cd378d71302fa28414e7aab37397f554a7df5f142c21c1b7303b8a0626f1baded5c72a704f7e6cd84cac00286bee0000000043410411db93e1dcdb8a016b49840f8c53bc1eb68a382e97b1482ecad7b148a6909a5cb2e0eaddfb84ccf9744464f82e160bfa9b8b64f9d4c03f999b8643f656b412a3ac00000000"
}
```

#### Response

返回值 code == 0 为正确，其他都是错误

data包括字段为：

- txId: 同输入参数
- index: 同输入参数
- byTxId: 同输入参数
- sigBE: 签名，大端字节序，hex编码
- padding: 签名的padding，hex编码
- payload: 签名的内容，hex编码
- script: 脚本原始内容，hex编码

其中payload字节内容为：

    txid, index, value, hash160(script), bytxid

txid在payload中为原始字节序, index是小端4字节，value是小端8字节。

- Body
```
{
  "code": 0,
  "msg": "ok",
  "data": {
    "txId": "0437cd7f8525ceed2324359c2d0ba26006d92d856a9c20fa0241106ee5a597c9",
    "index": 0,
    "byTxId": "f4184fc596403b9d638783cf57adfe4c75c605f6356fbc91338530e9831e9e16",
    "sigBE": "04e71de4aab8b5065e7f9ab0c6b503c26d917e2d869142c0bada2eabd5977e6dcc7a337a9f030bec405d6aec4efac3a4aea217c78af31a1f20966c7b60cdde30883638067d69655d78250faaa937f3b67bbfa0f304dee8564505ef0e4a51f8cab1ad767f797f8ea065110c148495198ce7aef67f7f06d04e31a30fc7a530abbf",
    "padding": "0100",
    "payload": "c997a5e56e104102fa209c6a852dd90660a20b2d9c352423edce25857fcd37040000000000f2052a01000000e01507f88b6dcc026c7062029c03adb11553de10169e1e83e930853391bc6f35f605c6754cfead57cf8387639d3b4096c54f18f4",
    "script": "410411db93e1dcdb8a016b49840f8c53bc1eb68a382e97b1482ecad7b148a6909a5cb2e0eaddfb84ccf9744464f82e160bfa9b8b64f9d4c03f999b8643f656b412a3ac"
  }
}
```
