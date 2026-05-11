# MQ异步队列

在API接口同步请求过程中，不适合处理耗时过长、或者一直轮询的工作。此时，可以结合MQ异步队列任务进行后台处理。  

## MQ异步队列服务 - Gearman
关于异步队列服务有很多种，这里PhalApi选择使用了Gearman，它的特点是：开源、使用简单、支持多客户端开发语言。  

 + Gearmana官网：http://gearman.org/  
 + Gearman下载：http://gearman.org/download/  
 + 安装PHP Gearman扩展：http://gearman.org/download/#php  

## 安装和启动Gearman服务
例如，在RHEL/Fedora/Linux/CentOS系统，可以执行：  
```bash
yum install gearmand
```

如果是Debain/Ubuntu，可以执行：  
```bash
apt-get install gearman-job-server
```

源代码编码安装的方式：  
```bash
tar xzf gearmand-X.Y.tar.gz
cd gearmand-X.Y
./configure
make
make install
```

其他操作系统，可以参考[Getting Started with Gearman](http://gearman.org/getting-started/) 进行安装。  

在服务端本地安装好Gearman服务后，启动Gearman服务命令：
```bash
$ gearmand -d
```

再检查一下是否正常运行：  
```bash
$ ps -ef | grep gearman
gearmand  1149     1  0 Jan06 ?        00:11:56 /usr/sbin/gearmand -d
```

## MQ配置

修改 ./config/sys.php 配置文件，填写对应的Gearman服务IP和端口。默认端口是

```php
    // MQ服务 - Gearman
    'gearman' => array(
        'ip' => '127.0.0.1',   //消息服务器
        'port' => 4730         //消息服务器端口
    ),
```

接着，在PhalApi框架中，注册Gearman客户端的DI服务，修改 ./config/di.php 文件，取消PHP代码注销：  
```php
// gearman异步队列
// 如何使用？请参考本地文档：/wiki/#/2x-mq
$di->gearmanClient = new \Base\Common\GearmanClient();
```

## MQ脚本测试

默认情况下，可以通过以下两个脚本，先进行快速测试。  

进入PhalApi根目录，先手动启动服务端脚本：  
```bash
$ php ./bin/mq/phalapi_pro_gearman_mq_server.php
Starting
Waiting for job...
```
可以看到，服务启动正常，正在等待接收新数据。  

接着，启动客户端测试脚本：  
```bash
$ php ./bin/mq/phalapi_pro_gearman_mq_example.php
```

这时候，你会看到在前面的服务端脚本会有显示和反应：  
```bash
$ php ./bin/mq/phalapi_pro_gearman_mq_server.php
Starting
Waiting for job...
Domain领域层，已接收到新数据：app_key = APP_KEY，data = {"title":"test\u6d4b\u8bd5\u6570\u636e"}
```

运行正常后，可以继续配置以下内容。  

## 后台启动脚本和守护进程

手动单次测试完成后，就可以在服务器正式启动MQ服务。  

进入根目录，启动MQ服务脚本：  
```bash
$ chmod +x ./bin/mq/phalapi_pro_gearman_mq_server.sh
$ ./bin/mq/phalapi_pro_gearman_mq_server.sh
ok
$ ps -ef | grep mq_server
apps     22537     1  0 15:55 pts/4    00:00:00 php /path/to/phalapi-pro/bin/mq/phalapi_pro_gearman_mq_server.php
apps     22538     1  0 15:55 pts/4    00:00:00 php /path/to/phalapi-pro/bin/mq/phalapi_pro_gearman_mq_server.php
apps     22539     1  0 15:55 pts/4    00:00:00 php /path/to/phalapi-pro/bin/mq/phalapi_pro_gearman_mq_server.php
apps     22550 16696  0 15:55 pts/4    00:00:00 grep --color=auto mq_server
```
从上往下，分别是添加执行权限、启动脚本和查看是否正常后台运行。可以看到，默认情况下，运行了3个消耗进程。  

在随后的系统升级和发布时，也可以手动执行以下命令，对MQ服务进行重启：  
```bash
$ chmod +x ./bin/mq/phalapi_pro_gearman_mq_server_restart.s
$ ./bin/mq/phalapi_pro_gearman_mq_server_restart.sh
restart ...
ok
$ ps -ef | grep mq_server
apps     22963     1  0 15:57 pts/4    00:00:00 php /path/to/phalapi-pro/bin/mq/phalapi_pro_gearman_mq_server.php
apps     22964     1  0 15:57 pts/4    00:00:00 php /path/to/phalapi-pro/bin/mq/phalapi_pro_gearman_mq_server.php
apps     22965     1  0 15:57 pts/4    00:00:00 php /path/to/phalapi-pro/bin/mq/phalapi_pro_gearman_mq_server.php
apps     22976 16696  0 15:57 pts/4    00:00:00 grep --color=auto mq_server
```
以上命令，依次是：添加执行权限、重启、确保重启成功。可以看到进程ID是刷新的了。  

随后，为了保障MQ异步退出能继续恢复，可以在crontab添加每一分钟的守护进程。  

```bash
$ crontab -e
# phalapi_pro_gearman_mq_server守护进程
*/1 * * * * /path/to/phalapi-pro/bin/mq/phalapi_pro_gearman_mq_server.sh
```

## 如何开发？
当你需要添加一个新的主题时，可以先修改 ./bin/mq/phalapi_pro_gearman_mq_server.php，复制并修改以下代码片段，添加新的主题，例如： 
```php
// 示例：接口请求后的更多异步处理
$gmworker->addFunction("api_request", "gearman_client_api_request");


// 示例：接口请求后的更多异步处理
function gearman_client_api_request($job)
{
    // 获取提交的日志数据
    $workload= $job->workload();
    $workload = json_decode($workload, true);

    $appKey = $workload['appKey'];
    $data = $workload['data'];

    \PhalApi\DI()->logger->debug('已接收到新的MQ队列数据#' . $finishTimes , ['app_key' => $workload['appKey'], 'data'=>$data]);

    if (empty($appKey) || empty($data)) {
        echo "empty app_app or data ...\n";
        return false;
    }

    try {
        // TODO：开始你的业务处理（特别是耗时的操作）……
        $domain = new \Base\Domain\MQ();
        $domain->afterApiRequestMQ($appKey, $data);

    } catch (Exception $ex) {
        echo "Failed to deal with data: ".$ex->getMessage(), json_encode($data), PHP_EOL;
        throw $ex;
    }

    // Return what we want to send back to the client.
    return true;
}

``` 

接着，在前台或客户端或API接口请求时，将队列数据压入，例如：  
```php
$gearmanCient = \PhalApi\DI()->gearmanClient;

$theme = 'api_request'; // 主题。必须对应
$appKey = 'APP_KEY';
$data = array('title' => 'test测试数据');

$gearmanCient->pushMQ($theme, $appKey, $data);
```

当你熟悉时，用一行代码即可：  
```php
\PhalApi\DI()->gearmanClient->pushMQ('api_request', 'APP_KEY', array('title' => 'test测试数据'));
```

最后，就是根据需要，在Domain层编写你的业务逻辑。参考 ./src/base/Domain/MQ.php 文件：  
```php
<?php
namespace Base\Domain;

/**
 * MQ异步队列的领域层处理
 * @author dogstar 20221019
 */
class MQ {

	// 示例：接口后的更多处理逻辑
	public function afterApiRequestMQ($appKey, $data) {
		echo 'Domain领域层，已接收到新数据：app_key = ' . $appKey . '，data = ' . json_encode($data) . PHP_EOL;
		// 继续你的业务处理……
	}
}
```

至此，快速开发和使用已介绍完毕。  

## 如何查看MQ日记？  
为了和API接口日记分开，MQ脚本本身的日志存放在目录：runtime/mq/phalapi_pro_gearman_mq_server.log；可以自己定时清理。  
如果是PHP业务层的日记，则存放在runtime/mq/目录下，例如：  

```bash
$ less ./runtime/mq/log/202210/20221019.log 
2022-10-19 15:26:29|DEBUG|已接收到新的MQ队列数据|{"app_key":"APP_KEY","data":{"title":"test测试数据"}}
2022-10-19 15:27:07|DEBUG|已接收到新的MQ队列数据|{"app_key":"APP_KEY","data":{"title":"test测试数据"}}
2022-10-19 15:29:42|DEBUG|已接收到新的MQ队列数据|{"app_key":"APP_KEY","data":{"title":"test测试数据"}}
```

## 相关代码文件位置  

本次涉及的文件和配置有：  
```
# 配置
./config/sys.php

# PHP代码
./src/base/Domain/MQ.php

# 脚本
$ tree ./bin/mq 
./bin/mq
├── phalapi_pro_gearman_mq_example.php
├── phalapi_pro_gearman_mq_server.php
├── phalapi_pro_gearman_mq_server.sh
└── phalapi_pro_gearman_mq_server_restart.sh
```

其他使用，可参考PhalApi开源接口框架和Gearman官方文档，进行完整的开发。  



