# 重要项目配置
考虑到为了方便项目进行配置，同时兼顾系统性能，将一些常用、需要进行个性但不频繁调整的配置，统一放置在./config/app.php配置文件中统一维护。  

其中，project为项目配置，你可能会在你的项目中看到类似以下的配置内容：  
```php
    
    /**
     * 项目配置
     */
    'project' => array(
        // 项目名称
        'name' => '果果云',
        // logo链接
        'logo' => '/logo.png',
        // 顶端图标
        'ico' => '/favicon.ico',
        // 查看文档的密码，为空时不需要密码
        'doc_view_code' => '',
        // 总开关，是否允许会员注册，true允许，false不允许
        'is_member_register' => true,
        // 总开关，是否允许开放者注册，true允许，false不允许
        'is_dev_register' => true,
        // 用户等级及映射，下标对应member表的member_level字段，建议按权限从低到高配置
        // 0,100,101,200,201为系统自带等级，不宜更改。可扩展追加
        'member_level_map' => array(
            // 0~99区间表示会员
            0 => array(
                'name' => '普通会员',
                'is_register' => true, // 是否允许注册
            ),
            
            // 100~199区间表示开发者
            100 => array(
                'name' => '个人开发者',
                'is_register' => true, // 是否允许开放平台注册
                'app_limit' => 0, // 开发者角色每日接口调用次数限制，为0时表示无限制（优先使用应用的app_limit）
            ),
            101 => array(
                'name' => '企业开发者',
                'is_register' => true, // 是否允许开放平台注册
                'app_limit' => 0, // 开发者角色每日接口调用次数限制，为0时表示无限制（优先使用应用的app_limit）
            ),
            
            // 200~255区间表示内部管理员
            200 => array(
                'name' => '普通管理员',
                'is_register' => false, // 是否允许注册
            ),
            255 => array(
                'name' => '超级管理员',
                'is_register' => false, // 是否允许注册
            ),
        ),
        // 每个开发者最多可以创建的应用数量上限
        'dev_max_app_num' => 10,
        // 计划任务专用的静态访问令牌，可以自行修改，通过HTTP请求时需要使用，CLI执行时不需要
        'task_access_token' => 'erVDwgv98Ls',
        // 默认应用接口每日接口次数上限，0表示没有限制
        'default_daily_app_limit' => 100000,
        // 默认接口权限，没有任何配置时应用对于接口调用的默认权限，推荐设置为false
        'default_app_api_rigths_is_allow' => false,
        // 开放接口的命名空间，配置后可提供接口权限分配，可配置多个
        'open_api_namespaces' => array('App'),
        // 默认语言，简体中文，可选：zh_cn/zh_tw/fr/de/en，或自行扩展
        'default_language' => 'zh_cn',
        // 语言列表
        'language_list' => array(
            'zh_cn' => '简体中文',
            'en' => 'English',
        ),
        // 验证码
        'captcha' => array(
            'switch' => array(
                'admin_login_captcha' => true, // 管理后台登录页面验证码开关
                'platform_login_catpcha' => true, // 开放平台登录页面验证码开关
                'platform_register_captcha' => true, // 开放平台注册页面验证码开关
            ),
            'options' => array(
                'charset' => 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789', // 验证码字符
                'length' => 4, // 验证码长度（默认长度为4，范围4~5）
                'height' => 50, // 验证码显示高度（默认40，单位像素）
                'width' => 200, // 验证码显示宽度（默认300，单位像素）
                'is_case_sensitive' => false, // 是否区分大小写
            ),
        ),
        // IP白名单，为空时不限制IP，只针对App开放接口
        'allow_ips' => array(
             // '127.0.0.1', // 每一组一个IP地址
        ),
    ),
```

下面分别介绍一些重要的项目配置。  

## 项目名称
### 配置项：name

用于平台站点展示的项目名称，修改只影响展示和文案的显示，不影响功能使用。  

```php
        // 项目名称
        'name' => '果果云',
```

## 项目Logo和图标

### 配置项：logo和ico

```php
        // logo链接
        'logo' => '/logo.png',
        // 顶端图标
        'ico' => '/favicon.ico',
```

## 接口文档查看密码
### 配置项：doc_view_code
查看文档的密码，为空时不需要密码，可防止外部非相关的人员查看接口文档。 
```php
        // 查看文档的密码，为空时不需要密码
        'doc_view_code' => '',
```

## 是否允许会员注册
### 配置项：is_member_register
总开关，是否允许会员注册，true允许，false不允许。关闭后，顾客将不能进行注册。   
```php
        // 总开关，是否允许会员注册，true允许，false不允许
        'is_member_register' => true,
```

## 每日接口请求次数限制
### 配置项：app_limit  
角色级别的每日掊次数限制，优先使用应用配置。应用未限制时，再使用此配置。
```php
        'app_limit' => 0, // 开发者角色每日接口调用次数限制，为0时表示无限制（优先使用应用的app_limit）

```

## 是否允许开放者注册
### 配置项：is_dev_register
总开关，是否允许开放者注册，true允许，false不允许。关闭后，开发者将不能在开放平台上进行注册。但可以通过管理后台进行添加。
```php
        // 总开关，是否允许开放者注册，true允许，false不允许
        'is_dev_register' => true,
```

## 用户等级及映射
### 配置项：member_level_map
用户等级及映射，下标对应member表的member_level字段，建议按权限从低到高配置。0,100,101,200,201为系统自带等级，不宜更改。可扩展追加。

```php
        // 用户等级及映射，下标对应member表的member_level字段，建议按权限从低到高配置
        // 0,100,101,200,201为系统自带等级，不宜更改。可扩展追加
        'member_level_map' => array(
            // 0~99区间表示会员
            0 => array(
                'name' => '普通会员',
                'is_register' => true, // 是否允许注册
            ),
            
            // 100~199区间表示开发者
            100 => array(
                'name' => '个人开发者',
                'is_register' => true, // 是否允许开放平台注册
            ),
            101 => array(
                'name' => '企业开发者',
                'is_register' => true, // 是否允许开放平台注册
            ),
            
            // 200~255区间表示内部管理员
            200 => array(
                'name' => '普通管理员',
                'is_register' => false, // 是否允许注册
            ),
            255 => array(
                'name' => '超级管理员',
                'is_register' => false, // 是否允许注册
            ),
        ),
```

添加后，建议同步修改./src/base/Domain/UserType.php文件，定义账号类型常量，方便开发，避免使用魔法数字。  
```php
/**
 * 账号类型和等级
 * - 依赖于app.project.member_level_map配置
 */
class UserType {
    
    // 用户等级
    const MEMBER_LEVEL_USER = 0;        // 普通用户
    const MEMBER_LEVEL_PERSON_DEVELOPER = 100; // 个人开发者
    const MEMBER_LEVEL_COMPAY_DEVELOPER = 101; // 企业开发者
    const MEMBER_LEVEL_ADMIN = 200;       // 管理员
    const MEMBER_LEVEL_SUPER_ADMIN = 255; // 超级管理员
```

## 开发者应用最大数量
### 配置项：dev_max_app_num  
每个开发者最多可以创建的应用数量上限。管理后台添加应用不受数量限制。  
```php
        // 每个开发者最多可以创建的应用数量上限
        'dev_max_app_num' => 10,
```

## 默认应用接口每日接口次数上限
### 配置项：default_daily_app_limit
默认应用接口每日接口次数上限，0表示没有限制，不分接口。每个应用的接口次数限制可通过管理后台单独修改和配置。    

```php
        // 默认应用接口每日接口次数上限，0表示没有限制
        'default_daily_app_limit' => 100000,
```

## 默认接口权限
### 配置项：default_app_api_rigths_is_allow

```php
        // 默认接口权限，没有任何配置时应用对于接口调用的默认权限，推荐设置为false
        'default_app_api_rigths_is_allow' => false,
```

## 开放接口的命名空间
### 配置项：open_api_namespaces
```php
        // 开放接口的命名空间，配置后可提供接口权限分配，可配置多个
        'open_api_namespaces' => array('App'),
```
可以追加自己新增的接口命名空间。后面文档会有专门介绍。  

## 翻译语言

见default_language和language_list。  


## 验证码

见captcha。  


## 其他配置：JWT令牌
jwt对应access_token的配置，你可以修改```key```，但修改后原来全部授权的access_token都会全部生效，需要客户端重新申请新的令牌。  
你还可以修改```exp```令牌有效的时间，只对新分配的令牌有影响。言下之意，对已经申请的令牌无影响。  

```php
    /**
     * JWT令牌
     */
    'jwt' => array(
        'key' => '*#FD2F9DM~E*', // 用于加密的key（安装时自动生成，不能修改！）
        'exp' => 30 * 86400, // 令牌生成后多少秒内有效，可自行修改
        'enable_app_unique_token' => false, // 是否开启app_key的唯一access_token（开启后app_key只有最新的一个token可用）
        'enable_app_and_user_unique_token' => false, // 是否开启app_key + 会员的唯一access_token（开启后会员只能单点登录）
    ),
```

## 其他配置：用户
账号用户里面有一个配置，是用于进行密码加密的公共盐值。不要修改，会导致原有的全部账号密码不正确。  
强烈建议不要修改，除非清楚在做什么。  

```php
    /**
     * 用户
     */
    'member' => array(
        'salt' => '9DfnseJ%sD#', // 用于增强用户密码（安装时自动生成，不能修改！）
    ),
```

## 其他配置：上传配置
如果你需要单独为图片文件配置域名，可以修改```host```；如果需要切换上传的目录位置，可以修改```upload_folder_path```并确保有写入权限。  

```php
    /**
     * 上传配置
     */
    'upload' => [
        'host' => '', // 图片文件域名，为空时取当前域名，修改不影响原有的文件地址，可单独配置图片域名或CDN域名，格式：http://img.xxx.com（前面加协议，最后不用斜杠）
        'upload_folder_path' => 'uploads', // 上传目录，可修改，注意：1）目录存在且有写入权限；2）需要放置在public目录下，支持外部访问；
        
        // 如果需要存储到阿里云OSS，请修改以下配置（全部必填方可生效）。若OSS上传失败，将会降级使用本地存储
        // 如何开通阿里云开通OSS服务：https://help.aliyun.com/document_detail/31884.html?spm=a2c4g.11186623.2.15.d8f41c62os2XRV#task-njz-hf4-tdb
        // 开通后，创建bucket，并把权限设置为:公共读
        'aliyu_oss' => array(
            'accessKeyId' => '', // TODO：修改成你的配置
            'accessSecret' => '', // TODO
            'bucket' => '', // TODO Bucket名称
            'endpoint' => '', // TODO endpoint，例如：http://oss-cn-beijing.aliyuncs.com
        ),
     ],
```

## IP白名单配置
如果需要配置全局的IP白名单，这部分IP白名单只针对调用App命名空间的开放接口的客户端IP地址进行限制，可以修改以下配置。为空时不限制，每一组一个IP地址。  
```
        // IP白名单，为空时不限制IP，只针对App开放接口
        'allow_ips' => array(
             // '127.0.0.1', // 每一组一个IP地址
        ),
```
> 温暖提示：如果需要针对单个应用进行限制，可以进入Admin管理后台单独修改应用的IP白名单。  

