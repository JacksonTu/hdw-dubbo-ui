### hdw-dubbo-ui
- hdw-dubbo-ui基于vue、element-ui构建开发，实现后台管理前端功能，提供一套更优的前端解决方案
- 前后端分离，通过token进行数据交互，可独立部署
- 主题定制，通过sass变量统一一站式定制
- 动态菜单，通过菜单管理统一管理访问路由
- 数据切换，通过mock配置对接口数据／mock模拟数据进行切换
- 发布时，可动态配置CDN静态资源／切换新旧版本
- 演示地址：http://locahost:8090 (账号密码：admin/123456)

#### 项目地址

 平台  | hdw-dubbo（后端） |hdw-dubbo-ui（前端）
---|---|---
GitHub | [https://github.com/JacksonTu/hdw-dubbo](https://github.com/JacksonTu/hdw-dubbo) |[https://github.com/JacksonTu/hdw-dubbo-ui](https://github.com/JacksonTu/hdw-dubbo-ui)
Gitee  | [https://gitee.com/tumao2/hdw-dubbo](https://gitee.com/tumao2/hdw-dubbo) |[https://gitee.com/tumao2/hdw-dubbo-ui](https://gitee.com/tumao2/hdw-dubbo-ui)


#### 前端启动
    1. 安装淘宝NPM镜像
    ```bush
    npm install -g cnpm --registry=https://registry.npm.taobao.org
    npm config set registry  https://registry.npm.taobao.org
    ```
    2. 运行
    ```bush
        cnpm install
        npm run dev
    ```
    访问 http://localhost:8090

#### 打包部署
 ```bush
 npm run build  --正式环境
 npm run build -test --测试环境
 ```

#### 系统预览
<table>
  <tr>
     <td><img src="docs/images/1.png"/></td>
     <td><img src="docs/images/2.png"/></td>
  </tr>
  <tr>
     <td><img src="docs/images/3.png"/></td>
     <td><img src="docs/images/4.png"/></td>
  </tr>
  <tr>
     <td><img src="docs/images/5.png"/></td>
     <td><img src="docs/images/6.png"/></td>
  </tr>
  <tr>
    <td><img src="docs/images/7.png"/></td>
    <td><img src="docs/images/8.png"/></td>
  </tr>
</table>

#### 技术交流
加入QQ群和大家一起交流吹水：

![qq](docs/images/QQ.png)

#### 更新日志
+ v3.1.0
 + 代码优化
