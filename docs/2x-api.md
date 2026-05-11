# 接口分类

API接口主要分为四大类，分别是：  

 + App开放接口：提供给开发者客户端应用使用的API接口，可以进行接口权限分配和管理。  
 + Platform开放平台接口：提供给Platform开放平台调用的接口，需要开发者账号或者管理员账号。  
 + Admin管理后台接口：提供给Admin管理后台使用的接口，需要管理员权限。  
 + Task计划任务接口：提供给后台调用的定时任务，不需要权限，但需要校验静态的令牌。  
 
> 温馨提示：为避免外部查看内部接口，需要登录管理员后，才能在在线接口文档查看Platform开放平台接口、Admin管理后台接口和Task计划任务接口。  

## 接口对比

为方便对比不同系列的接口，可参考以下表格。  

对比项|App开放接口|Platform开放平台接口|Admin管理后台接口|Task计划任务接口
---|---|---|---|---|---
调用方|开发者客户端应用|Platform开放平台前端|Admin管理后台前端|crontab计划任务
权限要求|应用需要具备接口权限|开发者账号或管理员账号|管理员账号|不限
访问入口|```/api/app.php```|```/api/platform.php```|```/api/admin.php```|```/api/task.php```，或通过```./bin/run_task.php```执行
是否统计接口次数|统计，超出后限制调用|不统计|不统计|不统计
接口基类|App\Common\Api|Platform\Common\Api|Admin\Common\Api|PhalApi\Api

## App开放接口

App开放接口，对应的命名空间为```App```，目录路径是：```./src/app```，即接口服务名称统一以```App.```为前缀，例如：```App.Auth.ApplyToken```、```App.Config.GetConfig```、```App.User.Register```等。  

通过在线接口文档，可以看到全部的前台接口。  

当需要新加开放接口API给开发者时，可以在这里添加。具体接口开发请见后面的技API接口开发文档。  

> 客户端调用的入口为：http://你的域名/api/app.php  

## Platform开放平台接口

提供给Platform开放平台调用的接口，对应的命名空间为```Platform```，目录路径是：```./src/platform```。  

当需要增加开放平台的功能时，可以在此追加需要的接口。  

> 客户端调用的入口为：http://你的域名/api/platform.php  

## Admin管理后台接口

管理后台接口，对应的命名空间为```Admin```，目录路径是：```./src/admin```，即接口服务名称统一以```Admin.```为前缀，例如：```Admin.File.GetList```、```Admin.Index.GetDataFlow```等。  

通过在线接口文档，可以看到全部的后台接口。  

当需要增加管理后台的功能时，可以在此追加需要的接口。  

> 客户端调用的入口为：http://你的域名/api/admin.php  

## Task计划任务接口

提供给计划任务调用的接口，对应的命名空间为```Task```，目录路径是：```./src/task```。  

静态令牌，在./config/app.php的task_access_token配置，可自行修改。  
```php
        // 计划任务专用的静态访问令牌，可以自行修改，通过HTTP请求时需要使用，CLI执行时不需要
        'task_access_token' => 'erVDwgv98Ls',
```

当需要增加计划任务时，可以在此追加需要的接口。  

> 客户端调用的入口为：http://你的域名/api/task.php  



