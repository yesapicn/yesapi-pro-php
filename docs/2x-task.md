# 接口计划任务


## 配置计划任务

安装完成后，在```pp_task_progress```表可以看到全部的计划任务。   

通过管理后台或者后台接口或者数据库```pp_task_progress```表，可以配置和维护计划任务。  

## 执行计划任务

完成上述的计划任务配置后，有两种方式可以执行计划任务。一种是常规的定时执行，另一种是人工通过MQ触发执行。

需要注意的是，
 + 1）通过crontab定时任务执行时，会存在一定的延时（1分钟左右，视具体任务执行时间而定），不能严格保证按配置的时间点执行
 + 2）通过MQ触发执行时，默认采用的是数据库MQ实现，如果需要支持高并发，需要先切换到Redis MQ（在./config/di.php把MQ改为```\PhalApi\Task\MQ\RedisMQ```）

### 定时执行计划任务

通过配置，可以定时执行计划任务。  

在Linux系统上，通过```crontab -e```修改添加计划任务：
```bash
# PhalApi专业版计划任务
*/1 * * * * php /path/to/phalapi_pro/bin/run_task.php > /dev/null
```

请把上面的 ```/path/to/phalapi_pro``` 路径，换成你服务器本地的对应路径。

如果需要手动执行或进行测试，可以进入专业版根目录，然后运行脚本命令：  
```
$ php ./bin/run_task.php
```

你也可以把计划任务的执行日志保存下来，记得定期查看和清理。  
```bash
# PhalApi专业版计划任务-保存执行日志
*/1 * * * * php /path/to/phalapi_pro/bin/run_task.php >> /path/to/phalapi_pro/runtime/crontab.log 2>&1
```

### 通过MQ手动执行计划任务

通过MQ，可以在API接口中（即同步请求中），把耗时严重或者执行资源过多的请求，改为异步的后台执行。实现代码是：  
```php
$taskDomain = new \Base\Domain\Task();
$taskDomain->push('Task.Requests.DemoTask', array('days' => 30)); // 传递给计划任务的接口参数
```

默认MQ是采用数据库存储，添加到MQ后，系统将会在1分钟内消费。

## 如何编写计划任务接口

编写计划任务的接口，和普通的前台接口类似，区别在于：  

 - 1、计划任务用于后台运行，通过调度器采用CLI方式执行
 - 2、计划任务没有上下文信息，如果需要指定uid或者app_key，请手动添加接口参数
 - 3、计划任务接口，推荐统一放置在```Task.*.*```里面
 
以下是一个计划任务示例代码：  
```php
<?php
namespace Task\Api;

use Task\Common\Api;

/**
 * 请求日志
 */
class Requests extends Api {
    
    public function getRules() {
        $rules = parent::getRules();
        
        $rules['demoTask'] = array(
            'days' => array('name' => 'days', 'default' => 30, 'type' => 'int', 'min' => 1, 'desc' => '参数示例'),
        );

        
        return $rules;
    }

    /**
     * 接口任务示例
     * @desc 接口计划任务示例
     */
    public function demoTask() {
        return array('desc' => '仅供演示和测试，参数示例 days = ' . $this->days);
    }
}
 ```
 
 ## 扩展：直接执行接口命令

如果需要立即执行某个接口计划任务，可以使用以下命令，进入项目根目录后，执行 ```php ./bin/run_api.php``` 命令：  
```bash
$ php ./bin/run_api.php 
Usage: ./bin/run_api.php [options] [operands]
Options:
  -s, --service <arg>     接口服务
  -h, --help              查看帮助信息


Error: 缺少service参数
```

根据使用提示，你可以再通过 ```-s```命令参数指定需要执行哪个接口。例如：  
```bash
$ php ./bin/run_api.php -s Task.Requests.DemoTask
{"ret":400,"data":{},"msg":"非法请求：缺少必要参数access_token"}
```

此时，你可以追加 ```-h``` 参数查看接口帮助，即查看接口需要哪些参数。例如：    

```bash
$ php ./bin/run_api.php -s Task.Requests.DemoTask -h
Usage: ./bin/run_api.php [options] [operands]
Options:
  -s, --service <arg>     接口服务
  -h, --help              查看帮助信息
  --access_token <arg>    访问令牌，计划任务专用的静态访问令牌。
  --days [<arg>]          参数示例
```

最后，手动添加更多必要的接口参数，例如：  
```bash
$ php ./bin/run_api.php -s Task.Requests.DemoTask --access_token 123 --days 7
{"ret":200,"data":{"desc":"仅供演示和测试，参数示例 days = 7"},"msg":""}
```

 ## 扩展：另一种直接、准确执行接口任务的方式

 基于上面手动执行接口的命令，你可以单独使用调试好的命令方式，结合crontab配置更加直接、准确执行的接口任务。例如：  

 ```bash
 # 指定执行接口任务
 */1 * * * * php /path/to/phalapi_pro/bin/run_api.php -s Task.Requests.DemoTask --access_token 123 --days 7 > /dev/null
 ```

 这种方式更直接，但不好通过管理后台的网站页面进行可视化管理，需要为每个接口任务单独进行配置。  

