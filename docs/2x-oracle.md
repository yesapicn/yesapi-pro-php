# 连接Oracle数据库

## 安装 PHP PDO_OCI 扩展
从你的PHP源码包中找到pdo_oci 扩展文件夹，通常路径 ```php-x.x.x/ext/pdo_oci```。  

如果你没有源码包，请从官网下载你的PHP对应版本的源码。  

```bash
# 进入安装包目录
cd pdo_oci/

# 生成安装配置文件
phpize

# 配置
# 如有需要，请提供你的 php-config 位置，使用 --with-php-config 参数
./configure --with-pdo-oci=instantclient,/Users/splitter/instantclient

# 执行编译安装(可能需要root权限)
make && make install

```

安装完成后将 extension=pdo_oci.so 添加至 php.ini 文件的最后一行，重启你的 Apache 服务或者php-fpm。  

检查安装是否成功 php -m | grep PDO_OCI  


## 配置Oracle数据库

修改文件：./config/dbs_oracle.php，配置自己的Oracle数据库IP、账号密码等。 

## 注册Oracle数据库服务

修改文件：./config/di.php，去掉这行注释。  
```php
// Oracle连接
$di->notorm_dbs_oracle = new \App\Common\OracleDB($di->config->get('dbs_oracle'));
```

## 创建你的Model子类

为你的每张数据库表，创建对应的Model子类，并继承于```Base\Model\OracleModelBase```父类，例如：  

为 week 表添加文件：src/base/Model/Week.php，写入：  

```php
<?php
namespace Base\Model;

class Week extends OracleModelBase {
    protected function getTableName($id) {
        return 'WEEK';
    }
}
```

## 使用和操作数据库

剩下的开发，请参考NotORM的数据库操作接口。  

