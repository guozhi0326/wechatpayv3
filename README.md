# 微信支付 API v3 Python SDK

[![PyPI version](https://badge.fury.io/py/wechatpayv3.svg)](https://badge.fury.io/py/wechatpayv3)
[![Download count](https://img.shields.io/pypi/dw/wechatpayv3)](https://img.shields.io/pypi/dw/wechatpayv3)

## 介绍

微信支付接口V3版python库。

欢迎微信支付开发者扫码进QQ群讨论：

![image](qq.png)

## 适用对象

**wechatpayv3**支持微信支付直连商户，接口说明详见 [官网](https://pay.weixin.qq.com/wiki/doc/apiv3/index.shtml)。

## 特性

1. 平台证书自动更新，无需开发者关注平台证书有效性，无需手动下载更新；
2. 支持本地缓存平台证书，初始化时指定平台证书保存目录即可；
3. 敏感信息直接传入明文参数，SDK内部自动加密，无需手动处理；
4. 回调通知自动验证回调消息，自动解密resource对象，并返回解密后的数据。

## 适配进度

微信支付V3版API接口

其中：

#### 基础支付

    JSAPI支付 已适配
    APP支付 已适配
    H5支付 已适配
    Native支付 已适配
    小程序支付 已适配
    合单支付 已适配

#### 经营能力

    支付即服务 已适配

#### 行业方案

    智慧商圈 已适配
    微信支付分停车服务 已适配

#### 营销工具

    图片上传(营销专用) 已适配

### 资金应用

    分账 已适配

#### 其他能力

    图片上传 已适配
    视频上传 已适配

#### 需要的接口还没有适配怎么办？

由于**wechatpayv3**包内核心的core.py已经封装了请求签名和消息验证过程，开发者无需关心web请求细节，直接根据官方文档参考以下基础支付的[申请退款](https://pay.weixin.qq.com/wiki/doc/apiv3/apis/chapter3_1_9.shtml)接口代码自行适配，测试OK的话，欢迎提交代码。
必填的参数建议加上空值检查，可选的参数默认传入None。参数类型对照参考下表：

| 文档声明 | **wechatpayv3** |
| --- | --- |
| string | string |
| int | int |
| object | dict: {} |
| array  | list: [] |
| boolean | bool: True, False |

```python
def refund(self,
           out_refund_no,
           amount,
           transaction_id=None,
           out_trade_no=None,
           reason=None,
           funds_account=None,
           goods_detail=None,
           notify_url=None):
    """申请退款
    :param out_refund_no: 商户退款单号，示例值：'1217752501201407033233368018'
    :param amount: 金额信息，示例值：{'refund':888, 'total':888, 'currency':'CNY'}
    :param transaction_id: 微信支付订单号，示例值：'1217752501201407033233368018'
    :param out_trade_no: 商户订单号，示例值：'1217752501201407033233368018'
    :param reason: 退款原因，示例值：'商品已售完'
    :param funds_account: 退款资金来源，示例值：'AVAILABLE'
    :param goods_detail: 退款商品，示例值：{'merchant_goods_id':'1217752501201407033233368018', 'wechatpay_goods_id':'1001', 'goods_name':'iPhone6s 16G', 'unit_price':528800, 'refund_amount':528800, 'refund_quantity':1}
    :param notify_url: 通知地址，示例值：'https://www.weixin.qq.com/wxpay/pay.php'
    """
    params = {}
    params['notify_url'] = notify_url or self._notify_url
    # 
    if out_refund_no:
        params.update({'out_refund_no': out_refund_no})
    else:
        raise Exception('out_refund_no is not assigned.')
    if amount:
        params.update({'amount': amount})
    else:
        raise Exception('amount is not assigned.')
    if transaction_id:
        params.update({'transaction_id': transaction_id})
    if out_trade_no:
        params.update({'out_trade_no': out_trade_no})
    if reason:
        params.update({'reason': reason})
    if funds_account:
        params.update({'funds_account': funds_account})
    if goods_detail:
        params.update({'goods_detail': goods_detail})
    path = '/v3/refund/domestic/refunds'
    return self._core.request(path, method=RequestType.POST, data=params)
```

## 源码

[github](https://github.com/minibear2021/wechatpayv3)

[gitee](https://gitee.com/minibear2021/wechatpayv3)

## 安装

```
$ pip install wechatpayv3
```

## 使用方法

### 准备
参考微信官方文档准备好密钥, 证书文件和配置([证书/密钥/签名介绍](https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay3_0.shtml))

+ **商户 API 证书私钥：PRIVATE_KEY**。商户申请商户 API 证书时，会生成商户私钥，并保存在本地证书文件夹的文件 apiclient_key.pem 中。
> :warning: 不要把私钥文件暴露在公共场合，如上传到 Github，写在客户端代码等。
+ **商户API证书序列号：CERT_SERIAL_NO**。每个证书都有一个由 CA 颁发的唯一编号，即证书序列号。扩展阅读 [如何查看证书序列号](https://wechatpay-api.gitbook.io/wechatpay-api-v3/chang-jian-wen-ti/zheng-shu-xiang-guan#ru-he-cha-kan-zheng-shu-xu-lie-hao) 。
+ **微信支付 APIv3 密钥：APIV3_KEY**，是在回调通知和微信支付平台证书下载接口中，为加强数据安全，对关键信息 `AES-256-GCM` 加密时使用的对称加密密钥。

### 初始化

``` python
from wechatpayv3 import WeChatPay, WeChatPayType

# 微信支付商户号
MCHID = '1230000109'
# 商户证书私钥
with open('path_to_key/apiclient_key.pem') as f:
    PRIVATE_KEY = f.read()
# 商户证书序列号
CERT_SERIAL_NO = '444F4864EA9B34415...'
# API v3密钥， https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay3_2.shtml
APIV3_KEY = 'MIIEvwIBADANBgkqhkiG9w0BAQE...'
# APPID
APPID = 'wxd678efh567hg6787'
# 回调地址，也可以在调用接口的时候覆盖
NOTIFY_URL = 'https://www.weixin.qq.com/wxpay/pay.php'
# 微信支付平台证书缓存目录
CERT_DIR = './cert'

wxpay = WeChatPay(
    wechatpay_type=WeChatPayType.MINIPROG,
    mchid=MCHID,
    private_key=PRIVATE_KEY,
    cert_serial_no=CERT_SERIAL_NO,
    apiv3_key=APIV3_KEY,
    appid=APPID,
    notify_url=NOTIFY_URL,
    cert_dir=CERT_DIR)
```

### 接口

``` python

# 统一下单
def pay():
    code, message = wxpay.pay(
        description='demo-description',
        out_trade_no='demo-trade-no',
        amount={'total': 100},
        payer={'openid': 'demo-openid'}
    )
    print('code: %s, message: %s' % (code, message))

# 订单查询
def query():
    code, message = wxpay.query(
        transaction_id='demo-transation-id'
    )
    print('code: %s, message: %s' % (code, message))

# 关闭订单
def close():
    code, message = wxpay.close(
        out_trade_no='demo-out-trade-no'
    )
    print('code: %s, message: %s' % (code, message))

# 申请退款
def refund():
    code, message=wxpay.refund(
        out_refund_no='demo-out-refund-no',
        amount={'refund': 100, 'total': 100, 'currency': 'CNY'},
        transaction_id='1217752501201407033233368018'
    )
    print('code: %s, message: %s' % (code, message))

# 退款查询
def query_refund():
    code, message = wxpay.query_refund(
        out_refund_no='demo-out-refund-no'
    )
    print('code: %s, message: %s' % (code, message))

# 申请交易账单
def trade_bill():
    code, message = wxpay.trade_bill(
        bill_date='2021-04-01'
    )
    print('code: %s, message: %s' % (code, message))

# 申请资金流水账单
def fundflow_bill():
    code, message = wxpay.fundflow_bill(
        bill_date='2021-04-01'
    )
    print('code: %s, message: %s' % (code, message))

# 下载账单
def download_bill():
    code, message = wxpay.download_bill(
        url='https://api.mch.weixin.qq.com/v3/billdownload/file?token=demo-token'
    )
    print('code: %s, message: %s' % (code, message))

# 合单支付下单
def combine_pay():
    code, message = wxpay.combine_pay(
        combine_out_trade_no='demo_out_trade_no',
        sub_orders=[{'mchid':'1900000109',
                     'attach':'深圳分店',
                     'amount':{'total_amount':100,'currency':'CNY'},
                     'out_trade_no':'20150806125346',
                     'description':'腾讯充值中心-QQ会员充值',
                     'settle_info':{'profit_sharing':False, 'subsidy_amount':10}}]
    )
    print('code: %s, message: %s' % (code, message))

# 合单订单查询
def combine_query():
    code, message = wxpay.combine_query(
        combine_out_trade_no='demo_out_trade_no'
    )
    print('code: %s, message: %s' % (code, message))

# 合单订单关闭
def combine_close():
    code, message = wxpay.combine_close(
        combine_out_trade_no='demo_out_trade_no', 
        sub_orders=[{'mchid': '1900000109', 'out_trade_no': '20150806125346'}]
    )
    print('code: %s, message: %s' % (code, message))

# 计算签名供调起支付时拼凑参数使用
# 注意事项：注意参数顺序，某个参数为空时不能省略，以空字符串''占位
def sign():
    print(wxpay.sign(['wx888','1414561699','5K8264ILTKCH16CQ2502S....','prepay_id=wx201410272009395522657....']))

# 验证并解密回调消息，把回调接口收到的headers和body传入
# 这里以flask框架为例，其他web框架如果遇到InvalidSignature，请确认传入的body和收到的一致，没有做额外的预处理
def decrypt_callback(headers=request.headers, body=request.data):
    print(wxpay.decrypt_callback(headers, body))

# 智慧商圈积分通知
def points_notify():
    code, message = wxpay.points_notify(
        transaction_id='4200000533202000000000000000',
        openid='otPAN5xxxxxxxxrOEG6lUv_pzacc',
        earn_points=True,
        increased_points=100,
        points_update_time='2020-05-20T13:29:35.120+08:00'
    )
    print('code: %s, message: %s' % (code, message))

# 智慧商圈积分授权查询
def user_authorization():
    code, message = wxpay.user_authorizations(
        openid='otPAN5xxxxxxxxrOEG6lUv_pzacc'
    )
    print('code: %s, message: %s' % (code, message))

# 支付即服务人员注册
def guides_register():
    code, message = wxpay.guides_register(
        corpid='1234567890',
        store_id=1234,
        userid='rebert',
        name='robert',
        mobile='13900000000',
        qr_code='https://open.work.weixin.qq.com/wwopen/userQRCode?vcode=xxx',
        avatar='http://wx.qlogo.cn/mmopen/ajNVdqHZLLA3WJ6DSZUfiakYe37PKnQhBIeOQBO4czqrnZDS79FH5Wm5m4X69TBicnHFlhiafvDwklOpZeXYQQ2icg/0',
        group_qrcode='http://p.qpic.cn/wwhead/nMl9ssowtibVGyrmvBiaibzDtp/0'
    )
    print('code: %s, message: %s' % (code, message))

# 支付即服务人员分配
def guides_assign():
    code, message = wxpay.guides_assign(
        guide_id='LLA3WJ6DSZUfiaZDS79FH5Wm5m4X69TBic',
        out_trade_no='20150806125346'
    )
    print('code: %s, message: %s' % (code, message))

# 支付即服务人员查询
def guides_query():
    code, message = wxpay.guides_query(
        store_id=1234,
        userid='robert',
        mobile='13900000000',
        work_id='robert',
        limit=5,
        offset=0
    )
    print('code: %s, message: %s' % (code, message))

# 支付即服务人员信息更新
def guides_update():
    code, message = wxpay.guides_update(
        guide_id='LLA3WJ6DSZUfiaZDS79FH5Wm5m4X69TBic',
        name='robert',
        mobile='13900000000',
        qr_code='https://open.work.weixin.qq.com/wwopen/userQRCode?vcode=xxx',
        avatar='http://wx.qlogo.cn/mmopen/ajNVdqHZLLA3WJ6DSZUfiakYe37PKnQhBIeOQBO4czqrnZDS79FH5Wm5m4X69TBicnHFlhiafvDwklOpZeXYQQ2icg/0',
        group_qrcode='http://p.qpic.cn/wwhead/nMl9ssowtibVGyrmvBiaibzDtp/0'
    )
    print('code: %s, message: %s' % (code, message))

# 图片上传
def image_upload():
    code, message = wxpay.image_upload(
        filepath='./media/demo.png'
    )
    print('code: %s, message: %s' % (code, message))

# 视频上传
def video_upload():
    code, message = wxpay.video_upload(
        filepath='./media/demo.mp4'
    )
    print('code: %s, message: %s' % (code, message))

# 查询车牌服务开通信息
def parking_service_find():
    code, message = wxpay.parking_service_find(
        plate_number='粤B888888',
        plate_color='BLUE',
        openid='oUpF8uMuAJOM2pxb1Q'
    )
    print('code: %s, message: %s' % (code, message))

# 创建停车入场
def parking_enter():
    code, message = wxpay.parking_enter(
        out_parking_no='1231243',
        plate_number='粤B888888',
        plate_color='BLUE',
        start_time='2017-08-26T10:43:39+08:00',
        parking_name='欢乐海岸停车场',
        free_duration=3600
    )
    print('code: %s, message: %s' % (code, message))

# 停车扣费受理
def parking_order():
    code, message = wxpay.parking_order(
        description='停车场扣费',
        out_trade_no='20150806125346',
        total=888,
        parking_id='5K8264ILTKCH16CQ250',
        plate_number='粤B888888',
        plate_color='BLUE',
        start_time='2017-08-26T10:43:39+08:00',
        end_time='2017-08-26T10:43:39+08:00',
        parking_name='欢乐海岸停车场',
        charging_duration=3600,
        device_id='12313'
    )
    print('code: %s, message: %s' % (code, message))

# 查询停车扣费订单
def parking_order_query():
    code, message = wxpay.parking_order_query(
        out_trade_no='20150806125346'
    )
    print('code: %s, message: %s' % (code, message))

# 图片上传(营销专用)
def marking_image_upload():
    code, message = wxpay.marketing_image_upload(
        filepath='./media/demo.png'
    )
    print('code: %s, message: %s' % (code, message))

# 请求分账
def profitsharing_order():
    code, message = wxpay.profitsharing_order(
        transaction_id='4208450740201411110007820472',
        out_order_no='P20150806125346',
        receivers={{'type': 'MERCHANT_ID', 'account': '86693852', 'amount': 888, 'description': '分给商户A'}},
        unfreeze_unsplit=True
    )
    print('code: %s, message: %s' % (code, message))

# 查询分账结果
def profitsharing_order_query():
    code, message = wxpay.profitsharing_order_query(
        transaction_id='4208450740201411110007820472',
        out_order_no='P20150806125346'
    )
    print('code: %s, message: %s' % (code, message))

# 请求分账回退
def profitsharing_return():
    code, message = wxpay.profitsharing_return(
        order_id='3008450740201411110007820472',
        out_return_no='R20190516001',
        return_mchid='86693852',
        amount=888,
        description='用户退款')
    print('code: %s, message: %s' % (code, message))

# 查询分账回退结果
def profitsharing_return_query():
    code, message = wxpay.profitsharing_return_query(
        out_order_no='P20150806125346',
        out_return_no='R20190516001'
    )
    print('code: %s, message: %s' % (code, message))

# 解冻剩余资金
def profitsharing_unfreeze():
    code, message = wxpay.profitsharing_unfreeze(
        transaction_id='4208450740201411110007820472',
        out_order_no='P20150806125346',
        description='解冻全部剩余资金'
    )
    print('code: %s, message: %s' % (code, message))

# 查询剩余待分金额
def profitsharing_amount_query():
    code, message = wxpay.profitsharing_amount_query(
        transaction_id='4208450740201411110007820472'
    )
    print('code: %s, message: %s' % (code, message))

# 添加分账接收方
def profitsharing_add_receiver():
    code, message = wxpay.profitsharing_add_receiver(
        account_type='MERCHANT_ID',
        account='86693852',
        relation_type='CUSTOM',
        name='腾讯充值中心',
        custom_relation='代理商'
    )
    print('code: %s, message: %s' % (code, message))

# 删除分账接收方
def profitsharing_delete_receiver():
    code, message = wxpay.profitsharing_delete_receiver(
        account_type='MERCHANT_ID',
        account='86693852'
    )
    print('code: %s, message: %s' % (code, message))

# 申请分账账单
def profitsharing_bill():
    code, message = wxpay.profitsharing_bill(
        bill_date='2021-04-01'
    )
    print('code: %s, message: %s' % (code, message))

```

## 回调验证失败处理
开发者遇到的难点之一就是回调验证失败的问题，由于众多的python web框架对回调消息的处理不完全一致，如果出现回调验证失败，请务必确认传入的headers和body的值和类型。
通常框架传过来的headers类型是dict，而body类型是bytes。使用以下方法可直接获取到解密后的实际内容。

### flask框架

直接传入request.headers和request.data即可。
```python
result = wxpay.decrypt_callback(headers=request.headers, body=request.data)
```
### django框架
由于django框架特殊性，会将headers做一定的预处理，可以参考以下方式调用。
```python
headers = {}
headers.update({'Wechatpay-Signature': request.META.get('HTTP_WECHATPAY_SIGNATURE'))
headers.update({'Wechatpay-Timestamp': request.META.get('HTTP_WECHATPAY_TIMESTAMP'))
headers.update({'Wechatpay-Nonce': request.META.get('HTTP_WECHATPAY_NONCE'))
headers.update({'Wechatpay-Serial': request.META.get('HTTP_WECHATPAY_SERIAL'))
result = wxpay.decrypt_callback(headers=headers, body=request.body)
```

### 其他框架
参考以上处理方法，大原则就是保证传给decrypt_callback的参数值和收到的值一致，不要转换为dict，也不要转换为string。

## 签名、验签、加密、解密的内部实现

一般应用开发者可以不用向下看了，有心了解这几项内部实现的可以参考一下。如果计划抽出来单独使用的，需要引入[cryptography](https://pypi.org/project/cryptography/)包。

### 构造签名信息

对应v3版微信支付api文档的[签名生成](https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay4_0.shtml)部分。

```python
def build_authorization(path,
                        method,
                        mchid,
                        serial_no,
                        mch_private_key,
                        data=None,
                        nonce_str=None):
    timeStamp = str(int(time.time()))
    nonce_str = nonce_str or ''.join(str(uuid.uuid4()).split('-')).upper()
    body = json.dumps(data) if data else ''
    sign_str = '%s\n%s\n%s\n%s\n%s\n' % (method, path, timeStamp, nonce_str, body)
    signature = rsa_sign(private_key=mch_private_key, sign_str=sign_str)
    authorization = 'WECHATPAY2-SHA256-RSA2048 mchid="%s",nonce_str="%s",signature="%s",timestamp="%s",serial_no="%s"' % (mchid, nonce_str, signature, timeStamp, serial_no)
    return authorization
```


### 验证签名

对应v3版微信支付api文档的[签名验证](https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay4_1.shtml)部分。

```python
def rsa_verify(timestamp, nonce, body, signature, certificate):
    sign_str = '%s\n%s\n%s\n' % (timestamp, nonce, body)
    public_key = certificate.public_key()
    message = sign_str.encode('UTF-8')
    signature = b64decode(signature)
    try:
        public_key.verify(signature, sign_str.encode('UTF-8'), PKCS1v15(), SHA256())
    except InvalidSignature:
        return False
    return True
```


### 回调信息解密

对应v3版微信支付api文档的[证书和回调报文解密](https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay4_2.shtml)部分。

```python
def aes_decrypt(nonce, ciphertext, associated_data, apiv3_key):
    key_bytes = apiv3_key.encode('UTF-8')
    nonce_bytes = nonce.encode('UTF-8')
    associated_data_bytes = associated_data.encode('UTF-8')
    data = b64decode(ciphertext)
    aesgcm = AESGCM(key=key_bytes)
    try:
        result = aesgcm.decrypt(nonce=nonce_bytes, data=data, associated_data=associated_data_bytes).decode('UTF-8')
    except InvalidTag:
        result = None
    return result
```


### 敏感信息加密

对应v3版微信支付api文档的[敏感信息加解密](https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay4_3.shtml)的加密部分。

```python
def rsa_encrypt(text, certificate):
    data = text.encode('UTF-8')
    public_key = certificate.public_key()
    cipherbyte = public_key.encrypt(
        plaintext=data,
        padding=OAEP(mgf=MGF1(algorithm=SHA1()), algorithm=SHA1(), label=None)
    )
    return b64encode(cipherbyte).decode('UTF-8')
```


### 敏感信息解密

对应v3版微信支付api文档的[敏感信息加解密](https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay4_3.shtml)的解密部分。

```python
def rsa_decrypt(ciphertext, private_key):
    data = private_key.decrypt(ciphertext=b64decode(ciphertext), padding=OAEP(mgf=MGF1(algorithm=SHA1), algorithm=SHA1))
    result = data.decode('UTF-8')
    return result
```
