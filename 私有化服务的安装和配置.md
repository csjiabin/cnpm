## cnpm 的安装
1. 下载
    - git clone https://github.com/cnpm/cnpmjs.org.git $HOME/cnpmjs.org
    - cd $HOME/cnpmjs.org

2. 添加配置信息
建议在 config 目录下新建 config.js 文件来配置 cnpm，而不是直接修改 config 目录下的 index.js，因为 index.js 会读取 config 目录下的 config.js，如果存在就会覆盖默认配置。
```
// config.js
    module.exports = {
      debug: false,
      database: {
        db: 'npm',     // 数据库名，默认为cnpmjs_test
        host: '127.0.0.1',    // 服务器地址
        port: 3306,           // 端口
        username: 'root',     // 用户名
        password: '      ',   // 密码
        dialect: 'mysql'      // 使用mysql，默认为sqlite, 还支持postgres,mariadb,暂时不支持oracle
      },
      syncModel: 'exist'         // 同步已存在的模块, 默认为none，即不同步, 还有个选项为all，同步所有模块
    };
```
> 配置字段含义在 index.js 内都有详细注释。

注意：重要提示如果搭建的 cnpm 希望被其他电脑访问，一定要将 index.js 中的
>  bindingHost: '127.0.0.1', // only binding on 127.0.0.1 for local access

这行注释。

cnpm 默认的两个访问端口是：

    1. 7001是 registry 端口，对应 registryPort 配置项
    2. 7002是 web 端口，对应 webPort 配置项
这两项都可以通过修改 config.js 文件来配置，配置设定完成后就启动服务：
```
npm install
npm start // 启动cnpm服务
```
> 启动成功:

![image](http://upload-images.jianshu.io/upload_images/1258730-9c680b5c1c909f0a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


3.验证：
>1. 访问 http://localhost:7001/

![image](http://upload-images.jianshu.io/upload_images/1258730-ca326998e7443b88.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 2.访问 http://localhost:7002/

![image](http://upload-images.jianshu.io/upload_images/1258730-cff80d6d7939d8be.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
## cnpm 的使用
最简单的方式就是下载 npm 包时修改下载依赖的目标地址。
```
 npm install vue-cli -g --registry=http://127.0.0.1:7001 // IP为你搭建服务器的地址，注意不要忘了端口号
```
如果，觉得每次在 npm 后面加 --registry=http://192.168.80.130:7001 很麻烦，也可以通过

```
  npm config set registry http://127.0.0.1:7001
```
设置默认的资源地址。

---

## 发布私有 npm 包

在发布 npm 包之前，需要先将原先的 config/config.js 中添加一些配置属性：

```
enablePrivate: true, // 只有管理员可以发布 npm 包，默认为 false，即任何人都可以发布包
admins: {
    admin: 'test@mycompany.com' // 管理员权限
},
scopes: ['@mycompany'], // 私有包必须依附于 scope 下
```
> 重新启动 cnpm 来应用这些配置。

然后，新建一个简单的项目来测试发布，在 package.json 文件中加入代码：

```
"name": "@mycompany/test", // 包名，之前必须加入 scope 名
"version": "1.0.1"           // 版本信息
```
一切准备就绪就可以发布了。

```
npm login --registry=http://127.0.0.1:7001 // publish 之前需要登录用户
    Username: admin // 管理员名
    Password: whateveryoulike
    Email: test@mycompany.com // 一定要填管理员账户邮箱地址，不然无法发布，因为设置了 enablePrivate: true

    npm publish --registry=http://127.0.0.1:7001
```
看到以下信息就发布成功了。

> +@mycompany/test@1.0.1

当然，其他机器也已经能通过

```
npm install @mycompany/test -registry=http://127.0.0.1:7001
```
来安装私有包了。

与此同时，还可以访问 http://127.0.0.1:7002/package/@mycompany/test 来查看发布包的详细信息。
![image](http://upload-images.jianshu.io/upload_images/1258730-5ce58442db5040d3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


