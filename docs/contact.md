# 联系我们

QQ(dogstar)：376741929  
邮箱：huangchanzong@yesapi.cn  
官网：http://pro.yesapi.cn/  

欢迎扫码联系：  

![](./img/yesyesapi_20191231165055_0fc922e996e46664a6efb8510a2fa9de.png)  

# FAQ常见问题

## Q1、OpenAPI看不到任何接口？
请自查：  
 + 是否开启了Opcache，要关掉，否则无法读取注释。  
 + 根目录是否缺少composer.json文件，若缺少无法查看任何API文档。  
 + 新增的Api接口的PHP源代码，是否存在PHP语法错误。 
 + 若修改composer.json有新增的命名空间，需要执行：```composer dumpautoload```命令更新。 

## Q2、platform或admin前端本地运行提示：Error: Cannot find module 'eslint'  这个错误怎么解决？
在本地，删除node_modules文件夹删除然后用cnpm 重新安装项目依赖。  


