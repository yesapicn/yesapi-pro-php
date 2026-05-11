# 3.9 新增接口目录教程

添加一个新的接口目录，可以先参考[如何增加一个顶级命名空间？](http://docs.phalapi.net/#/v2.0/autoload?id=%e5%a6%82%e4%bd%95%e5%a2%9e%e5%8a%a0%e4%b8%80%e4%b8%aa%e9%a1%b6%e7%ba%a7%e5%91%bd%e5%90%8d%e7%a9%ba%e9%97%b4%ef%bc%9f)。  

下面以添加Teacher接口命名空间为例（对应teacher目录）进行讲解，其他目录名称相应改成自己的即可。  

## 第1步，更新composer
修改 根目录的composer.json 文件，找到autoload里的psr-4，在前面添加一行：  
```
            "Teacher\\": "src/teacher",
```
前面是接口命名空间名称，后面是对应的文件目录，注意大小写。  

添加后完整的部分是：  
```
	"autoload" : {
		"files" : [
			"src/app/functions.php"
		],
		"psr-4" : {
            "Teacher\\": "src/teacher",
			"App\\" : "src/app",
			"Platform\\" : "src/platform",
            "Task\\": "src/task",
            "Base\\": "src/base",
            "Mall\\": "src/mall",
			"Admin\\" : "src/admin"
		}
	}
```

然后更新命名空间的映射关系，执行命令：  
```
composer dumpautoload
```

## 第2步，添加接口目录

根据上面的配置，以及PhalApi的规范，接口目录放置在src目录里，这里需要添加的接口目录是：```src/teacher```。  

![](./img/yesyesapi_20210624112006_a36e48f043f5ed3e0c3091f086375334.png)  

随后，可以继续添加 ```src/teacher/Api```目录，放置你的接口PHP代码文件。 

## 第3步，添加接口访问入口文件  

在接口大师，每个接口目录，即每个接口命名空间，都需要提供独立、配套、对应的外部访问入口。  

针对这里的Teacher，把原来的开发平台的入口文件  ./public/api/platform.php 复制一份给 ./public/api/teacher.php ，修改里面的filter过滤器进行重新注册。  

```php
// 切换你的过滤器
// $di->filter = new \Platform\Common\Filter();
$di->filter = new \Teacher\Common\Filter();
```

## 第4步，实现你的过滤器

添加```src/teacher/Common```目录，并新建```src/teacher/Common/Filter.php```文件，放置以下代码，并根据自己的需要调整逻辑。  

```php
<?php
namespace Teacher\Common;

use Base\Common\Filter as BaseFilter;
use PhalApi\Exception\BadRequestException;

class Filter extends BaseFilter {

    public function check() {
        parent::check();
        
        // 实现你自己的检测逻辑
    }
}
```

## 第5步，修改接口文档的路径

打开修改 ```./src/view/docs/api_desc_tpl.php```接口详情模板文件，添加接口命名空间和访问路径的映射。  

```php
$uriMap = array(
    'Teacher' => '/api/teacher.php', // 添加新映射
    'Admin' => '/api/admin.php',
    'App' => '/api/app.php',
    'Platform' => '/api/platform.php',
    'Task' => '/api/task.php',
);
```

就可以解决访问路径不对的问题，例如：  
![](./img/yesyesapi_20210624113257_7cb5e4e7d970318c077fcefe8479121d.png)  

此外，还可以修改 ./language/zh_cn/common.php 翻译文件，最后添加一行（注意双斜杠）：  
```php
'Teacher\\' => '教师平台',
```

解决接口列表页的名称显示问题。例如：  
![](./img/yesyesapi_20210624113515_ddb5e5d828c381a136abf1c51a355f9a.png)  

## 第6步，接口列表文档的显示控制

例如，有些新加的接口目录，只是内部使用的，那么可以隐藏接口列表的入口。  

如果想控制新接口目录在接口列表的显示，可以修改 ```./public/docs.php ```文件，找到以下代码，并在最后添加自己的接口命名空间，就可以隐藏。  

```
    // 对外排除的命名空间，如果需要增加项目，需要在此追加
    $globalExcludeNamespaces = array('Task', 'Platform', 'Admin', 'Teacher'); // 在最后追加
```

## 第7步，开始使用新目录

到这里，就可以在新目录添加和开发自己的接口。  

剩下的事情，接口大师会自动帮你完成，包括接口的权限以及分配等。   

