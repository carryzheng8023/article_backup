1. 备份yum源文件
```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```
2. 下载阿里云yum源文件
```shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```
3. 清空本地缓存
```shell
yum clean all
```
4. 本地缓存服务器上的软件包信息
```shell
yum makecache
```
