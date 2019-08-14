## Activiti6.0安装部署

#### 一、下载Activiti6.0

```shell
wget https://github.com/Activiti/Activiti/releases/download/activiti-6.0.0/activiti-6.0.0.zip
```

#### 二、解压

```shell
unzip activiti-6.0.0.zip
```

#### 三、部署war包

1. 安装tomcat

```she
docker pull tomcat:8.5.43-jdk8
```

2. 启动tomcat

```shell
docker run -d --name tomcat01 -p8888:8080 tomcat:8.5.43-jdk8
```

3. 将war包复制到webapps下

```shell
docker cp activiti-admin.war containerId:/usr/local/tomcat/webapps/
docker cp activiti-app.war containerId:/usr/local/tomcat/webapps/
```

#### 四、访问web页面

1. 用户页面


> http://127.0.0.1:8888/activiti-app/
> 用户名：admin
> 密码：test
> ![](http://pic.carryzheng.xin/zx_md/20190724162150.png)
> ![](http://pic.carryzheng.xin/zx_md/20190724162447.png)

2. 管理员页面


> http://127.0.0.1:8888/activiti-admin/
> 用户名：admin
> 密码：admin
> ![](http://pic.carryzheng.xin/zx_md/20190724163610.png)
> ![](http://pic.carryzheng.xin/zx_md/20190724163743.png)