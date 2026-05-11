# 连接其他数据库

如果项目原有已经有数据库，或者在使用Pro开发过程中需要连接其他数据库，可按本教程进行开发。

1. 适配Oracle,需要PDO支持oci数据库驱动
2. 适配PostgreSQL和OpenGauss,需要PDO支持pgsql数据库驱动

## 第1步：添加其他数据库配置
修改数据库配置文件```./config/db_plus.php```，修改里面的数据库连接配置。例如：  
```php
    'servers' => array(
        'db_master' => array(                       // 服务器标记
            'type'      => 'mysql',                 // TODO: 数据库类型，暂时只支持：mysql, sqlserver
            'host'      => '127.0.0.1',             // TODO: 数据库域名
            'name'      => 'phalapi_pro',           // TODO: 数据库名字
            'user'      => 'root',                  // TODO: 数据库用户名
            'password'  => '',	                    // TODO: 数据库密码
            'port'      => 3306,                    // 数据库端口
            'charset'   => 'utf8mb4',                  // 数据库字符集 utf8mb4或utf8
            'pdo_attr_string'   => false,           // 数据库查询结果统一使用字符串，true是，false否
            'driver_options' => array(              // PDO初始化时的连接选项配置
                // 若需要更多配置，请参考官方文档：https://www.php.net/manual/zh/pdo.constants.php
            ),
        ),
    ),
```
> 关于数据库的配置，可参考数据库连接文档：http://docs.phalapi.net/#/v2.0/database-connect

## 第2步：在di服务中注册新数据库NotORM
请修改DI注册文件```./config/di.php```，去掉以下代码的注释，注册并开启```$di->notorm_plus```服务。  
```php
// 连接其他数据库（或项目原有的数据库，需要时去掉注释）
// $di->notorm_plus = new NotORMDatabase($di->config->get('dbs_plus'), $di->config->get('sys.notorm_debug'));
```

## 第3步：实现Model基类，切换数据库
此步骤，默认已经实现，可参考代码：```./src/base/Model/PlusBase.php```。如果需要连接第3个数据库或更多，可将此文件复制一份，作相应修改。

## 第4步：开发Model子类，实现你的业务逻辑
在完成前面配置后，最后一步，便可以开始开发你的业务逻辑，连接或操作其他数据库。此时，新加的Model子类，如果是操作新增的数据库，则需要继承于```Base\Model\PlusBase```基类。例如：  
```php
<?php
namespace App\Model\Plus;

use Base\Model\PlusBase;

class Comment extends PlusBase {
    // 实现具体的数据库操作
}
```

> 温馨提示：为了将不同数据库的Model代码文件分开管理，推荐对于不同的数据库，在./src/app/Model目录下创建相应的子目录，分开管理和维护。例如上面放置在./src/app/Model/Plus目录。  

更多帮助说明，请参考开源版[支持任意多个不同数据库](http://docs.phalapi.net/#/v2.0/database-other?id=%e5%a4%8d%e6%9d%82%e6%96%b9%e6%a1%88%ef%bc%9a%e6%94%af%e6%8c%81%e4%bb%bb%e6%84%8f%e5%a4%9a%e4%b8%aa%e4%b8%8d%e5%90%8c%e6%95%b0%e6%8d%ae%e5%ba%93)。  
