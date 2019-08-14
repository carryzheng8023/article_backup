## nifi执行shell



#### 1.新建ExecuteProcess Processor

![](http://pic.carryzheng.xin/zx_md/20190715161501.png)



#### 2.填写配置信息

![](http://pic.carryzheng.xin/zx_md/20190715161617.png)

> 其中：
>
> * **Command：** shell脚本的绝对路径



#### 3. 创建shell脚本(nifi所在节点)

```shell
echo "/usr/bin/date > /time_list.txt" > /test-nifi.sh
chmod a+x /test-nifi.sh
```

