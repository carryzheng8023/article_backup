### 一、虚拟环境管理
1. 查看虚拟环境
```shell
conda info -e
```
2. 安装虚拟环境
```shell
conda create -n your_env_name [python_package] python=python_version
```
3. 删除虚拟环境
```shell
conda remove -n your_env_name --all
```

4. 激活虚拟环境
```shell
conda activate your_env_name
```
5. 退出虚拟环境
```shell
conda deactivate
```

### 二、用一个jupyter notebook管理anaconda多环境kernel

1. 在主环境root中安装nb_conda
```shell
conda install nb_conda
```
2. 切换到虚拟环境中，我这里的虚拟环境名是test_nb，然后在shell中输入activate切换到指定虚拟环境
```shell
conda activate test_nb
```
3. 在虚拟环境中使用conda安装ipykernel
```shell
conda install ipykernel
```

### 三、配置jupyter notebook远程访问
1. 生成配置文件
```shell
jupyter notebook --generate-config
```
2. 生成密码（使用ipython）
```python
In [1]: from notebook.auth import passwd
In [2]: passwd()
```
3. 修改配置文件
```properties
c.NotebookApp.ip='0.0.0.0' #设置访问notebook的ip，0.0.0.0表示所有IP，这里设置ip为都可访问
c.NotebookApp.notebook_dir = '/home/anaconda/notebook'#共享目录
c.NotebookApp.password = '' #填写刚刚生成的密文  
c.NotebookApp.open_browser = False # 禁止notebook启动时自动打开浏览器(在linux服务器一般都是ssh命令行访问，没有图形界面的。所以，启动也没啥用)  
c.NotebookApp.port =1234 #指定访问的端口，默认是8888 
c.NotebookApp.allow_remote_access = True
```
4. 后台运行
```shell
nohup jupyter notebook &
```

