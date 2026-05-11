# 阿里云短信

## 配置
修改./config/app.php文件，配置自己的阿里云短信服务的配置：
```php
   /**
     * 阿里云配置
     */
    'aliYun' => array(
        'accessKeyId' => '', // TODO： 修改成你的key
        'accessSecret' => '', // TODO： 修改成你的secret
    )
```

## 使用接口

 + 阿里云发送短信接口：App.Sms.AliyunSendSms
 + 阿里云查看短信发送记录和发送状态：App.Sms.AliyunQuerySendDetails
 + 校验短信验证码： App.Sms.CheckSmsCaptcha

## 示例效果

1、调用阿里云的SendSms发送短信，传递号码，短信签名名称，短信模板ID和短信模板变量对应的实际值，JSON格式。例如返回：
```
curl 'http://你的接口域名/api/app.php?s=App.Sms.AliyunSendSms&PhoneNumbers=号码&SignName=短信签名名称
&TemplateCode=短信模板ID&TemplateParam={"code":"1111"}'
{
    "ret": 200,
    "data": {
        "Message": "OK",
        "RequestId": "4674148E-F8CE-4CBA-AF26-D3F13B7F5162",
        "BizId": "734404282527581474^0",
        "Code": "OK"
    },
    "msg": ""
}
```
![](http://cd7.yesapi.net/89E670FD80BA98E7F7D7E81688123F32_20200224221308_036fb728567de98f8da9eaee2567c304.jpeg)

2. 阿里云查看短信发送记录和发送状态
```
curl 'http://你的接口域名/api/app.php?s=App.Sms.AliyunQuerySendDetails&PhoneNumber=号码&SendDate=日期'
{
    "ret": 200,
    "data": {
        "TotalCount": 1,
        "Message": "OK",
        "RequestId": "FC95F662-4FC1-4692-ADEE-2B1AD4BBB90F",
        "SmsSendDetailDTOs": {
            "SmsSendDetailDTO": [
                {
                    "SendDate": "2020-02-24 14:59:41",
                    "SendStatus": 3,
                    "ReceiveDate": "2020-02-24 14:59:48",
                    "ErrCode": "DELIVERED",
                    "TemplateCode": "SMS_177253065",
                    "Content": "【YesApi】您的验证码1234，该验证码15分钟内有效，请勿泄漏于他人！",
                    "PhoneNum": "15914283683"
                }
            ]
        },
        "Code": "OK"
    },
    "msg": ""
}
```

3. 检验短信验证码
    + 检验的接口保存在pp_sms_cache表中，验证码将在15分钟内有效，可尝试5次，成功校验的失败。
    + 检验结果：0成功，1验证码不存在或已过期，2验证码已失效（失败次数过多），3验证码错误




