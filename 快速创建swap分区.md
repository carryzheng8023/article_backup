### 一、分配文件预留空间（32G）
```shell
sudo fallocate -l 32G /swapfile
```

### 二、修改swapfile文件权限
```shell
sudo chmod 600 /swapfile
```

### 三、将swapfile初始化为交换文件
```shell
sudo mkswap /swapfile
```

### 四、启用交换文件
```
sudo swapon /swapfile
```

### 五、配置自动挂载交换分区文件
```shell
sudo echo "/swapfile none swap sw 0 0" >> /etc/fstab
```
### 六、查看内存详情
```shell
free -m
```
### 七、卸载交换分区
```shell
sudo swapoff /swapfile
```
