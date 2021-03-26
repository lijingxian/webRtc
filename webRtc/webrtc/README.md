# webrtc


#### 项目结构

```bash

├── README.md                       项目介绍
├── package.json                        安装、启动文件，npm包配置文件，里面定义了项目的npm脚本，依赖包等信息
├── .eslintrc.js                        代码检测规则定制
├── babel.config                        babel编译插件配置
├── index.html                          入口页面
│
├── public                          纯静态资源，不会被wabpack构建
│   ├── index.html                      入口页面
│   ├── widget.html                     画板页面
│   ├── widget.js                       画板资源
│   └── favicon                         图标
│
│
├── src                             源码目录
│   ├── main.js                         入口js文件
│   ├── app.vue                         根组件
│   ├── components                      公共组件目录
│   │   └── Rtc.vue
│   ├── assets                          资源目录，这里的资源会被wabpack构建
│   │   └── img
│   │       └── logo.png
│   ├── router                          前端路由
│   │   └── index.js
│   ├── views                           页面目录
│   │   ├── student.vue   │   
        └── teacher.vue


#### 环境搭建

一、在搭建vue的开发环境之前，一定一定要先下载node.js，，vue的运行是要依赖于node的npm的管理工具来实现，node可以在官网或者中文网里面下载，根据自己的电脑选择是32还是64 ，网址：http://nodejs.cn；

二、下载好node之后，打开docs管理工具，先看看node安装成功了没有，输入 node -v ，回车，会输出node的版本号，

由于在国内使用npm是非常慢的，所以在这里我们推荐使用淘宝npm镜像，使用

淘宝的cnpm命令管理工具可以代替默认的npm管理工具：$ npm install -g cnpm --registry=https://registry.npm.taobao.org；

三、设置环境变量（非常重要）

四、打开项目，修改package.json的 serve地址为本地开发地址

五、打开Rtc.vue文件，修改socketURL地址为本地开发地址

六、打开视频服务项目RTCMultiConnection-Server-master，安装依赖npm install,然后启动npm run start

七、打开项目，启动命令npm run serve,启动项目。
