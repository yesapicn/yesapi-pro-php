# 如何开发接口

使用PhalApi专业版开发接口，非常简单，大致流程如下。

## 目录结构

PhalApi Pro版的目录结构如下，

```
./
├── README.md # 简介
├── bin # 脚本目录
├── config # 配置目录
│   ├── app.php # 应用配置
│   ├── dbs.php # 数据库配置
│   ├── di.php # 依赖服务配置
│   └── sys.php #系统配置
├── data # 数据库
│   └── phalapi_pro.sql # 数据库安装时的文件
├── language # 翻译包
├── pro_admin # 管理后台的前端源代码，基于iview-admin
├── pro_platform # 开放平台的前端源代码，基于iview-admin
├── public # 对外访问的目录
│   ├── admin # 管理后台访问入口（相当于pro_admin打包构建后的dist目录）
│   ├── api # 接口访问入口（内分前台API和后台API）
│   ├── docs # 离线生成的HTML接口文档
│   ├── docs.php # 在线版接口文档访问入口
│   ├── index.php
│   ├── init.php # 全局初始化文件
│   ├── install # 安装向导（成功安装后建议删除此目录）
│   ├── platform # 开放平台访问入口（相当于pro_platform打包构建后的dist目录）
│   ├── static # 静态资源
│   ├── uploads # 上传目录（需要有写入权限）
│   └── wiki # 技术文档
├── runtime # 运行目录
│   ├── _install.lock # 安装锁定文件
│   ├── cache # 文件缓存
│   └── log # 文件日志
├── sdk # SDK包
├── src # 项目源代码，非常重要
│   ├── admin # 后台接口源代码（遵循ADM模式）
│   ├── app # 开放平台接口源代码（遵循ADM模式）
│   ├── base # 基础包源代码（放置底层公共的代码，不对外直接提供接口，即不提供Api层）
│   ├── platform # 开放平台接口源代码（遵循ADM模式）
│   ├── task # 计划任务接口源代码（遵循ADM模式）
│   └── view # 页面模板目录（如接口文档）
├── tests # 单元测试
└── vendor # composer包，不需要手动修改，通过composer install/update可进行安装和更新
```

PHP代码层次结构如下：  
![](./img/yesyesapi_20200416113656_6c74c2813bed5ce7f552c5ed2a1007cc.png)

## 编写Api接口层

如果需要编写开放接口，可以在./src/app/Api目录下新增一个PHP文件，类名和文件名一样（需要区分大小写）。并继承App\Common\Api基类即可。  

例如开放接口的Hello World示例，接口文件是：./src/app/Api/HelloWorld.php，类名是：App\Api\HelloWorld，对接的接口服务名称是：App.HelloWorld.Say。  

```php
<?php
namespace App\Api;
use App\Common\Api;

/**
 * Hello World示例
 */
class HelloWorld extends Api {
    // 接口参数配置
    public function getRules() {
        return array(
            'say' => array(
                'nickname' => array('name' => 'nickname', 'desc' => '昵称'),
            ),
        );
    }

    /**
     * 示例
     * @desc 第一个前台接口示例
     */
    public function say() {
        $nickname = $this->nickname;
        return array('content' => 'Hello world!');
    }
}
```

保存后，访问接口文档列表，可以看到以下新接口：  
![](./img/20220801-144843.png)

同时在接口详情文档可以查看到相应的详细接口文档：  
![](./img/_2022-08-01T06-49-25.594Z.png)

如果需要编写后台接口，则需要放置在./src/admin/Api目录下，并继承Admin\Common\Api基类。其他开发要求类似。

> 温馨提示：以下方法是系统保留的函数名，不以用于接口函数名称，否则会影响接口正常运行。系统保留接口函数名称有：tryToGetUid()、checkUserLogin()、tryToGetAppKey()、checkAppOnline()、getCurContext()、init()、createMemberValue()、getApiRules()、getApiCommonRules()、getRules()、filterCheck()、userCheck()、isServiceWhitelist()、equalOrIngore()。

## 如何取消接口令牌验证？

默认情况下，前台接口需要进行```access_token```令牌验证，以保护接口不被非法请求。如果不需要对指定的接口进行验证，可以在配置文件./config/app.php中的service_whitelist白名单中添加接口。例如上面的HelloWorld接口不需要接口验证，可以在最后追加配置：  

```php
'service_whitelist' => array(
        'Site.Index',
        'HelloWorld.Say', // 追加Hello World示例白名单
),
```

接口白名单配置，会取消过滤器，不进行任何校验和判断，此时不会相应调整应用的接口权限。如果需要让接口权限在界面上显示保持一致，可以配置接口权限规则。  

> 温馨提示：配置接口白名单，开放平台和管理后台的接口权限显示不会影响。  

## 如何取消接口权限判断？

+ 如何取消全部接口权限判断？  
可以修改./config/app.php里面的default_app_api_rigths_is_allow配置项为true，即可让开放接口默认拥有权限，相当于取消全部接口权限判断。  

+ 如何取消某个接口类的接口权限判断？

在你的接口具体子类中，重载```\App\Common\Api::userCheck()```方法，不进行任何操作即可。  
```php
/**
 * 平台接口基类
 */
class Api extends \App\Common\Api {
    
    /**
     * 进行接口权限判断
     * @throws BadRequestException
     */
    protected function userCheck() {
        // 不需要
    }
}

```
> 温馨提示：通过代码取消接口权限判断，开放平台和管理后台的接口权限显示不会影响。  

+ 如何取消一个接口的接口权限判断？  

在你的接口具体子类中，重载```\App\Common\Api::userCheckActionWhitelist()```方法，返回不需要进行接口权限判断的接口白名单。  
```php
<?php
namespace App\Api;
use App\Common\Api;

/**
 * Hello World示例
 */
class HelloWorld extends Api {
    
    protected function userCheckActionWhitelist() {
        return array('hiApp', 'hiMember');
    }
}

```
> 温馨提示：通过代码取消接口权限判断，开放平台和管理后台的接口权限显示不会影响。  

## 如何获取当前上下文信息？
如何获取当前上下文、登录会员ID和app_key？  

在App\Common\Api接口基类中，已经封装了针对于当前上下文、登录会员ID和app_key等接口，方便项目快速开发。  

以下是使用代码和相关说明。  

```php
class HelloWorld extends Api {
    public function say() {
        // 获取会员ID，未登录时异常返回
        $uid = $this->tryToGetUid();
        
        // 获取会员ID，未登录时返回0
        $uid = $this->tryToGetUid(false);
        
        // 检测会员是否已登录，未登录时异常返回
        $this->checkUserLogin();

        // 获取开发者ID，未指定时返回0
        $did = $this->tryToGetDid(false);
        
        // 获取app_key，未指定时异常返回
        $appKey = $this->tryToGetAppKey();
        
        // 获取app_key，未指定时返回空字符串
        $appKey = $this->tryToGetAppKey(false);
        
        // 检测是否已指定app_key
        $this->checkAppOnline();
        
        // 获取当前上下文
        $context = $this->getCurContext();
        var_dump($context->getUid()); // 会员ID
        var_dump($context->getAppKey()); // app_key
    }
}
````

## 如何隐藏access_token参数？

如果不需要在接口文档上显示```access_token```参数，可以在接口参数规则里这样配置（设置is_doc_hide为true即可）。  
```php
class HelloWorld extends Api {
    public function getRules() {
        return array(
            'say' => array(
                'accessToken' => array('name' => 'access_token', 'is_doc_hide' => true),
            ),
        )
    }
}
```

## 编写Domain领域层

Domain领域层主要用于封装复杂的业务逻辑、规则和算法。此部分PHP代码放置在./src/app/Domain目录下。此部分根据不同项目的业务需求，具体开发即可。

## 编写Model数据层

Model数据层主要用于操作MySQL数据库，全部的Model子类可继承Base\Model\Base基类，此基类封装了很多实用的方法和接口，极大减少了数据库封装的代码。例如对应配置表的配置Model类代码如下：  

```php
<?php
namespace App\Model;

class Config extends Base {
}
```
对应源代码文件是：./src/app/Model/Config.php。  

在继承```Base\Model\Base```数据库基类后，可以很方便进行数据库的操作和查询。  

更多关于数据库的连接、操作、查询、多数据库使用等，请参考[DataModel数据模型 - PhalApi 2.x 开发文档](http://docs.phalapi.net/#/v2.0/database-datamodel)。

## 如何新增API接口命名空间？  

默认情况，推荐将接口统一放置在App命名空间，即src/app目录下。如果项目有需要，可以新增自己的接口命名空间。  

首先，参考开源版文档[如何增加一个顶级命名空间？](http://docs.phalapi.net/#/v2.0/autoload?id=%e5%a6%82%e4%bd%95%e5%a2%9e%e5%8a%a0%e4%b8%80%e4%b8%aa%e9%a1%b6%e7%ba%a7%e5%91%bd%e5%90%8d%e7%a9%ba%e9%97%b4%ef%bc%9f)，增加一个新的顶级命名空间。  

然后，修改```./config/app.php```配置文件里的open_api_namespaces配置，追加你的命名空间。
```php
        // 开放接口的命名空间，配置后可提供接口权限分配，可配置多个
        'open_api_namespaces' => array('App', '新的顶级命名空间'),
```

例如，加了Task命名空间后： 
```php
        // 开放接口的命名空间，配置后可提供接口权限分配，可配置多个
        'open_api_namespaces' => array('App', 'Task'),
```

在管理后台可以自动管理该命名空间下的接口权限。  
接口权限分配：  
![](./img/yesyesapi_20200416114254_3c9a241aa03b33b0b66911583c8bd568.jpg)  
接口服务列表：  
![](./img/yesyesapi_20200416114450_1d98c314d1c0eef0f34aad607a4ce076.jpg)  

在开放平台，开发者也可自动看到新增命名空间下的接口。  
![](./img/yesyesapi_20200416114634_7ac7f4d70c1c4e138eac7206c2134e46.png)  

## 如何隐藏接口？

和开源版一样，在接口类或方法中添加```@ignore```注释。  

如：  
```php
<?php
namespace App\Api;
use App\Common\Api;

/**
 * Hello World示例
 * @ignore
 */
class HelloWorld extends Api {
    
    /**
     * 接口示例
     * @desc 一个开放接口示例，可直接访问，不需要任何验证，因为配置了service_whitelist白名单。熟悉后可删除此示例接口。
     * @ignore
     * @return string content 一句话
     */
    public function say() {
        return array('content' => 'Hello PhalApi Pro!');
    }
 }
```

## 如何开启接口参数加密传输？

为了保护客户端传递的参数不被外界非法获取，除了使用HTTPS协议外，也可以通过代码方式来增加。  

在PhalApi专业版，你可以：
  
** 客户端RSA公钥加密传输API接口参数 + 服务端RSA私钥解密API接口参数。**   

### 重要配置

首先，是RSA私钥和公钥的文件，分别是：  
 + rsa私钥文件：./config/phalapi_pro_rsa.pri
 + rsa公钥文件：./config/phalapi_pro_rsa.pub
 
在项目启动或开始时，你可以更换以上RSA配置文件，项目开始使用后，不建议再更换。   

其次，还可以开启./config/app.php配置文件中的encrypt_data公共参数，方便客户端查看可以传递此参数以及其格式要求说明。  

```php

    /**
     * 应用接口层的统一参数
     */
    'apiCommonRules' => array(
        'accessToken' => array('name' => 'access_token', 'default' => '', 'desc' => '访问令牌，仅当开启签名验证时需要传递，生成令牌可使用App.Auth.ApplyToken接口'),

        /** ----- 如果你需要使用第二套加密算法，请开启以下参数规则 ----- **/

        // 'app_key' => array('name' => 'app_key', 'default' => '', 'desc' => 'app_key，用于区分客户端应用，首次接入需要创建应用并等待管理员审核通过'),
        // 'sign' => array('name' => 'sign', 'desc' => '动态签名，签名算法是：<br/><ul><li>1、全部参数（排除sign），按key进行字典排序</li><li>2、全部参数值，把原始值按字符串进行拼接，并在最后加上app_secret密钥</li><li>3、对第2步结果，拼接密钥后，进行MD5加密</li><li>4、对第3步结果，转成大写，得到sign签名（32位）</li></ul>'),
        // 'uid' => array('name' => 'uid', 'type' => 'int', 'default' => 0, 'desc' => ''),
        // 'accessToken' => array('name' => 'access_token', 'default' => '', 'desc' => '访问令牌，保留使用但不需要在文档上展示', 'is_doc_hide' => true),
        
        'encryptData' => array('name' => 'encrypt_data', 'desc' => '客户端加密的接口，格式是：RSA公钥加密(base64编码(JSON原始数据))。开启后，同时支持原来普通的参数传递方式。'),
    ),
```

### 客户端RSA公钥加密传输API接口参数

将rsa公钥文件：./config/phalapi_pro_rsa.pub提供给受信任的客户端开发人员，其中待加密的encrypt_data参数格式是：  

encrypt_data = RSA公钥加密(base64编码(JSON原始数据))  

也就是：
 + 第1步、把全部的原始参数通过JSON格式封装
 + 第2步、对第1步结果进行base64编码
 + 第3步、使用RSA公钥进行加密
 + 第4步、得到encrypt_data

#### PHP 示例
以下是PHP作为客户端编写的代码示例：  
```php
    // 公钥加密-由客户端完成
    $public_key = openssl_pkey_get_public(file_get_contents('./phalapi_pro_rsa.pub'));
    openssl_public_encrypt('{"name":"phalapi pro"}', $crypted, $public_key);
    $_REQUEST['encrypt_data'] = base64_encode($crypted);
    // K07LIy/V+cfZqfHgZpIPnmdkwlkFbTkyRXVXx2JfQrF3YFAIsFcHnC9TjGTezzyup2f0V24nYH71Uf3oXVIqz/X9wgPXW0AGAbJw4kDOIq9Jao5L0mG7t5FV/2DLzJ14qO6fvANv6e/Hy2pFBcKvHnQ8uRJ/wyAV+RpUAa21wCY6zzuo9OhS89NPZg4B4CUORR8SIuIqWTUlXHB0woFIfRiO/AKCGltc9oDkyzJFYVvgI0LwijkQUV9RoruCEx6EvmZY7OVLB5+AXwfnfFKKtCw3jucqHyclzXwCQoif8FXN1NzCNpYvwj7DbzqU/WzRgxnPgXQSyjbCDlw19BrWoQ==
```
以上，原始的参数是：```{"name":"phalapi pro"}```，最后加密后的结果如上。  

> 温馨提示：客户端可以把需要加密的参数放到encrypt_data，同时服务端接口也会继续支持原来原始参数的传递。两者重复时以加密的参数为准。  

以下是各后端语言实现 RSA 公钥加密参数解密的示例代码，用于接收客户端传递的 `encrypt_data` 参数并解密。

#### Java 示例

```java
import javax.crypto.Cipher;
import java.security.KeyFactory;
import java.security.PrivateKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.util.Base64;

public class ParameterDecryptor {
    
    // RSA私钥（从文件读取或配置）
    private static final String PRIVATE_KEY = 
        "-----BEGIN PRIVATE KEY-----\n" +
        "xxxxxxxx\n" +
        "-----END PRIVATE KEY-----";
    
    public static String decryptEncryptData(String encryptData) throws Exception {
        // Step 1: Base64 解码
        byte[] encryptedBytes = Base64.getDecoder().decode(encryptData);
        
        // Step 2: RSA 私钥解密
        String privateKeyStr = PRIVATE_KEY
            .replace("-----BEGIN PRIVATE KEY-----", "")
            .replace("-----END PRIVATE KEY-----", "")
            .replaceAll("\\s", "");
        
        byte[] keyBytes = Base64.getDecoder().decode(privateKeyStr);
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        PrivateKey privateKey = keyFactory.generatePrivate(keySpec);
        
        Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        byte[] decryptedBytes = cipher.doFinal(encryptedBytes);
        
        // Step 3: 得到 JSON 原始数据
        String jsonData = new String(decryptedBytes, "UTF-8");
        return jsonData;
    }
    
    // 使用示例
    public static void main(String[] args) {
        String encryptData = request.getParameter("encrypt_data");
        if (encryptData != null && !encryptData.isEmpty()) {
            try {
                String jsonParams = decryptEncryptData(encryptData);
                // 解析 JSON 获取实际参数
                // JSONObject params = new JSONObject(jsonParams);
            } catch (Exception e) {
                // 解密失败处理
            }
        }
    }
}
```

#### Python 示例

```python
import base64
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5

# RSA私钥（从文件读取或配置）
PRIVATE_KEY = """-----BEGIN PRIVATE KEY-----
xxxxxxxxx
-----END PRIVATE KEY-----"""

def decrypt_encrypt_data(encrypt_data):
    # Step 1: Base64 解码
    encrypted_bytes = base64.b64decode(encrypt_data)
    
    # Step 2: RSA 私钥解密
    key = RSA.import_key(PRIVATE_KEY)
    cipher = PKCS1_v1_5.new(key)
    decrypted_bytes = cipher.decrypt(encrypted_bytes, None)
    
    if decrypted_bytes is None:
        raise Exception("RSA decryption failed")
    
    # Step 3: 得到 JSON 原始数据
    json_data = decrypted_bytes.decode('utf-8')
    return json_data

# 使用示例（Django/Flask框架）
from flask import request

@app.route('/api/endpoint', methods=['POST'])
def handle_request():
    encrypt_data = request.args.get('encrypt_data') or request.form.get('encrypt_data')
    
    if encrypt_data:
        try:
            json_params = decrypt_encrypt_data(encrypt_data)
            # 解析 JSON 获取实际参数
            import json
            params = json.loads(json_params)
            # 使用 params 继续处理...
        except Exception as e:
            return {'error': 'Decryption failed'}, 400
    
    # 正常参数处理...
    return {'result': 'success'}
```

## Node.js 示例

```javascript
const crypto = require('crypto');

// RSA私钥（从文件读取或配置）
const PRIVATE_KEY = `-----BEGIN PRIVATE KEY-----
xxxxxxxxx
-----END PRIVATE KEY-----`;

function decryptEncryptData(encryptData) {
    // Step 1: Base64 解码
    const encryptedBuffer = Buffer.from(encryptData, 'base64');
    
    // Step 2: RSA 私钥解密
    const decryptedBuffer = crypto.privateDecrypt(
        {
            key: PRIVATE_KEY,
            padding: crypto.constants.RSA_PKCS1_PADDING
        },
        encryptedBuffer
    );
    
    // Step 3: 得到 JSON 原始数据
    const jsonData = decryptedBuffer.toString('utf-8');
    return jsonData;
}

// 使用示例（Express框架）
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.get('/api/endpoint', (req, res) => {
    const encryptData = req.query.encrypt_data || req.body.encrypt_data;
    
    if (encryptData) {
        try {
            const jsonParams = decryptEncryptData(encryptData);
            const params = JSON.parse(jsonParams);
            // 使用 params 继续处理...
            res.json({ result: 'success', data: params });
        } catch (error) {
            res.status(400).json({ error: 'Decryption failed' });
        }
    } else {
        // 正常参数处理...
    }
});
```

## Golang 示例

```go
package main

import (
    "crypto/rand"
    "crypto/rsa"
    "crypto/x509"
    "encoding/base64"
    "encoding/json"
    "encoding/pem"
    "errors"
    "fmt"
    "net/http"
)

// RSA私钥（从文件读取或配置）
const privateKeyPEM = `-----BEGIN PRIVATE KEY-----
xxxxxxxxx
-----END PRIVATE KEY-----`

// 获取私钥
func getPrivateKey() (*rsa.PrivateKey, error) {
    block, _ := pem.Decode([]byte(privateKeyPEM))
    if block == nil {
        return nil, errors.New("failed to decode PEM block")
    }
    
    privateKey, err := x509.ParsePKCS8PrivateKey(block.Bytes)
    if err != nil {
        return nil, err
    }
    
    rsaPrivateKey, ok := privateKey.(*rsa.PrivateKey)
    if !ok {
        return nil, errors.New("not RSA private key")
    }
    
    return rsaPrivateKey, nil
}

// 解密 encrypt_data
func decryptEncryptData(encryptData string) (string, error) {
    // Step 1: Base64 解码
    encryptedBytes, err := base64.StdEncoding.DecodeString(encryptData)
    if err != nil {
        return "", err
    }
    
    // Step 2: RSA 私钥解密
    privateKey, err := getPrivateKey()
    if err != nil {
        return "", err
    }
    
    decryptedBytes, err := rsa.DecryptPKCS1v15(rand.Reader, privateKey, encryptedBytes)
    if err != nil {
        return "", err
    }
    
    // Step 3: 得到 JSON 原始数据
    return string(decryptedBytes), nil
}

// HTTP处理器示例
func apiHandler(w http.ResponseWriter, r *http.Request) {
    encryptData := r.URL.Query().Get("encrypt_data")
    if encryptData == "" {
        encryptData = r.FormValue("encrypt_data")
    }
    
    if encryptData != "" {
        jsonParams, err := decryptEncryptData(encryptData)
        if err != nil {
            w.WriteHeader(http.StatusBadRequest)
            json.NewEncoder(w).Encode(map[string]string{"error": "Decryption failed"})
            return
        }
        
        // 解析 JSON 获取实际参数
        var params map[string]interface{}
        if err := json.Unmarshal([]byte(jsonParams), &params); err != nil {
            w.WriteHeader(http.StatusBadRequest)
            json.NewEncoder(w).Encode(map[string]string{"error": "Invalid JSON"})
            return
        }
        
        // 使用 params 继续处理...
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(map[string]interface{}{
            "result": "success",
            "data":   params,
        })
        return
    }
    
    // 正常参数处理...
}

func main() {
    http.HandleFunc("/api/endpoint", apiHandler)
    http.ListenAndServe(":8080", nil)
}
```

#### 注意事项

- RSA 加密有长度限制（通常 117 字节），大数据量建议使用混合加密（RSA + AES）
- 私钥务必妥善保管，不要提交到版本控制系统
- 生产环境建议从配置中心或环境变量读取私钥
- 解密失败时应有降级方案（如记录日志、返回友好错误信息）

### 服务端RSA私钥解密API接口参数

此部分由后端API接口自动完成。不影响原有的接口参数传递方式和接口参数规则配置。   

## composer的使用

PhalApi专业版和PhalApi 2.x开源一样，都是使用了composer，如果对PHP composer不熟悉，可以查看[Composer 中文网 / Packagist 中国全量镜像](https://www.phpcomposer.com/)，简单来说，composer是 PHP 用来管理依赖（dependency）关系的工具。  

由于国外镜像通常会很慢，本地使用时可以切换到中国镜像。执行以下命令：  
```
$ composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

## 扩展阅读
更多内容请参考[PhalApi 2.x 开发文档](http://docs.phalapi.net/#/)。


