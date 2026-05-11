# 如何开发Admin管理后台

## 运行环境
项目运行适用，  
 + Nodejs版本: v14.15.3  
 + Vue版本: v2.5.10  
  
若运行时提示node-sass等相关版本问题，请先检查本地Nodejs版本是否匹配。

## 管理后台源代码

PhalApi Pro 管理后台，是基于[iView-admin2.0](https://lison16.github.io/iview-admin-doc/#/icons)开发的，源代码位置```pro_admin```目录。

目录结构如下：  

```
.
├── config  开发相关配置
├── public  打包所需静态资源
└── src
    ├── api  AJAX请求
    └── assets  项目静态资源
        ├── icons  自定义图标资源
        └── images  图片资源
    ├── components  业务组件
    ├── config  项目运行配置
    ├── directive  自定义指令
    ├── libs  封装工具函数
    ├── locale  多语言文件
    ├── mock  mock模拟数据
    ├── router  路由配置
    ├── store  Vuex配置
    ├── view  页面文件
    └── tests  测试相关
```    

修改根目录下的vue.config.js文件中的接口测试域名，更换成您本地的测试域名或正式域名。  
```js
  // 添加如下配置
  devServer: {
    proxy: {
      '/api': {
        // TODO：更换成您本地的测试域名或正式域名
        target: 'http://pro-test.api.yesapi.cn/',
        changeOrigin: true
      }
    }
  }
```

在本地首次运行管理后台时，需要进行npm安装，进入```pro_admin```目录后，执行命令：  
```
$ npm install
```

安装完毕后，再运行命令：  
```
$ npm run dev
```

然后，浏览器会自动打开管理后台，例如：  
![](./img/yesyesapi_20200426164442_898e22552eb1fbbbb82cf118b20eac3e.png)

## 需要使用的接口

管理后台需要用到的接口，建议统一放置在Admin后台API系列中，进行统一的管理员身份和权限的验证。  

## 发布

管理后台开发完成后，进行打包：  
```
$ npm run build
```

打包后会生成dist目录，其目录结构如下：  

```
./dist
├── css  # 样式
├── favicon.ico  # 图标
├── fonts # 字段
├── img # 图片
├── index.html # 首页文件
└── js # js代码
```

最后把dist目录里面全部的文件复制替换到```public/admin/```，更新到生产环境即可。  

## 如何自定义国际化语言包
1.本项目全局注入了vue-i18n，切换语言原理很简单，在```/src/locale```目录下操作即可  

2.在```/src/locale/lang```目录下创建需要的语言包js文件，例如zh-TW.js。  
   在该文件中以键值对的形式配置每个需要翻译的变量值。  
   注意：每添加一个需要翻译的变量key都要在该目录下的其他语言包配置相应的翻译值  

3.配置```/src/locale/index.js```文件  
```javascript
   //引入自定义的语言包
   import customZhCn from './lang/zh-CN'
   import customZhTw from './lang/zh-TW'
   import customEnUs from './lang/en-US'
   //引入iview语言包
   import zhCnLocale from 'iview/src/locale/lang/zh-CN'
   import enUsLocale from 'iview/src/locale/lang/en-US'
   import zhTwLocale from 'iview/src/locale/lang/zh-TW'

   // 引进语言包
   // vue-i18n 6.x+写法
   const messages = {
     'zh-CN': Object.assign(zhCnLocale, customZhCn),
     'zh-TW': Object.assign(zhTwLocale, customZhTw),
     'en-US': Object.assign(enUsLocale, customEnUs)
   }
   export const i18n = new VueI18n({
     locale: lang,
     messages
   })
   
   // 配置项目中要调用的语言包，用于界面选择语言中下拉选项调用
   export const localList = {
     'zh-CN': '中文简体',
     'en-US': 'English'
   }
```

完成。

## 前端页面的鉴权
1.admin权限配置方法: 运行项目 >> 使用超级管理员登录后台管理admin >> 在权限管理目录中进行权限管理  
    ① 权限分配的页面根据前端代码中```/src/router```路由表的路径path进行一一对应, 在操作列表中的每个操作代表一个页面的权限,
   点击**添加新操作**可添加一个新路径的权限控制  
    ② 对每个页面的权限分配还可以通过分配给*用户类型*或者*特定用户ID*进行权限分配  
    ③ 对子级别路由的path进行分配可控制子级别页面的权限  
     对父级别路由的path进行分配可控制父级别页面的权限以及在页面左侧目录中的展示情况  
     父子路由的权限配置相互独立  

2.鉴权原理:  
   admin和platform项目中vueRouter的路由分为```/src/router/router.js```中的基础页面以及```/src/router/aysnRouters.js```中的异步路由。  
   异步路由aysnRouters的加载需要前端调接口获得相应账号的权限数组permission (permission与aysnRouters进行匹配后通过addRouter方法加载到项目路由表中)  
   二次开发可对基础路由和异步路由进行修改, 添加新异步路由后可使用超管账号对新路径进行**添加新操作**  

3.在该项目vue组件开发中判断运行时路径是否有访问权限的方法   
   ```/src/libs/util.js```中的*hasPermission*方法  
   开发中可能遇到以下场景: 设置一个button, 点击跳转到某个需要权限的页面, button的显示隐藏与是否有权限相关  
   例如Platform项目中的```/src/view/account/accountInfo.vue```(具体可参见项目源码)  

```html
<!-- /src/view/account/accountInfo.vue -->
<template>
  <div class="content-layer accountInfo min-h">
    <h3>{{$t('developer_info')}}</h3>
    <div class="content">
      <header class="header">
        <img :src="avatar">
        <span class="username">{{userProfile.username }}</span>
        <!-- 示例: 通过/account/accountSettings权限情况, 判断是否显示组件 -->
        <!-- 将权限状态保存在showAccess后 绑定组件判断显示隐藏-->
        <Button v-if="showAccess" @click="setInfo">{{$t('developer_info_alter')}} >></Button>
    </header>
      <Table
      :columns="columns1"
      :data="data1"
      :show-header=false
      width=660
      >
      </Table>
    </div>

  </div>
</template>
```


```javascript
// /src/view/account/accountInfo.vue
// hasPermission方法判断是否有访问该路径的权限
import { getToken, hasPermission } from '@/libs/util'
export default {
  name: 'accountInfo',
  computed: {
    // 用计算属性保存hasPermission的状态值,用于组件v-if判断
    showAccess () {
      return hasPermission('/account/accountSettings')
    }
  }
}

// 绑定完成。
```
