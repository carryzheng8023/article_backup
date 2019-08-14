1. 关闭docker
```shell
sudo service docker stop
```
2. 修改配置文件
```shell
sudo vim /var/lib/docker/containers/{containerId}/hostconfig.json
sudo vim /var/lib/docker/containers/{containerId}/config.v2.json
```
3. 启动docker
```shell
sudo service docker start
```
