# 接口调试

PhalApi专业版的调试继续保留[开源版的调试](http://docs.phalapi.net/#/v2.0/response-and-debug)，并加以升级，进行了新的调整。

## 开启调试模式

开启调试模式很简单，主要有两种方式：

 + **单次请求开启调试**：默认添加请求参数```&__debug__=1```
 + **全部请求开启调试**：把配置文件```./Config/sys.php```文件中的配置改成```'debug' => true,```

如果不允许手动单次开启调试，可以修改./config/di.php文件，把```$di->debug```调整为：  
```php
// 调试模式，仅允许由系统配置指定
$di->debug = $di->config->get('sys.debug');
```

开启调试模式后，将不会进行进行access_token严格的检测，即不会进行后置权限、状态等统一的Filter检测。如果要手动指定当前会员ID或者app_key，可以使用以下参数： 

 + 调试模式时，通过```&_uid=123```可指定本次登录的会员ID  
 + 调试模式时，通过```&_app_key=abc```可指定本次的app_key  
 
## 查看SQL语句
在runtime目录下，在每天运行的日志文件，通过匹配SQL关键字可以看到每次接口请求背后执行的SQL语句。  

例如，对于获取配置的接口```App.Config.GetConfig```，接口背后执行的SQL部分日志效果如下：  
```
$ tail -f ./runtime/log/202001/20200109.log | grep SQL
2020-01-09 11:27:08|SQL|[#2 - 45.49ms - SQL]/path/to/phalapi-pro/src/app/Domain/Config.php(55):    App\Model\Base::getBy()    pp_config    SELECT * FROM pp_config WHERE (config_name = 'config_year') LIMIT 1;|{"request":{"service":"App.Config.GetConfig","access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiI5ODMwOTk0MyIsImF1ZCI6InVzZXIiLCJzdWIiOiJ0ZXN0IiwidWlkIjozLCJpYXQiOjE1Nzg0NTU0ODksImV4cCI6MTU4MTA0NzQ4OX0.YSRAdAjRmCD1616rMsligyhf0qIOtMEEhwYtQBmMQuQ","config_name":"config_year"}}
``` 

## 查看错误日志
在runtime目录下，可以看到错误日志。  

例如，一些关于警告、提醒和错误的日志：
```
$ ll ./runtime/log/202004 
-rwxrwxrwx  1 _www     staff   2.7K  4  5 17:08 error_20200405.log
-rwxrwxrwx  1 _www     staff   1.4K  4  7 22:48 error_20200407.log
-rwxrwxrwx  1 _www     staff   662B  4  9 11:06 error_20200409.log
-rwxrwxrwx  1 _www     staff   576B  4  2 09:27 notice_20200402.log
-rwxrwxrwx  1 _www     staff   192B  4  4 22:34 notice_20200404.log
-rwxrwxrwx  1 _www     staff    78K  4  8 23:11 notice_20200408.log
-rwxrwxrwx  1 _www     staff   9.1K  4  9 14:59 notice_20200409.log
-rwxrwxrwx  1 dogstar  staff    92K  4  5 17:17 warning_20200405.log
-rwxrwxrwx  1 dogstar  staff   174K  4  8 23:11 warning_20200408.log
-rwxrwxrwx  1 _www     staff    19K  4  9 14:59 warning_20200409.log
```


