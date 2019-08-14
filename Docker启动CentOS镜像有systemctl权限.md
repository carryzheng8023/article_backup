```shell
docker run -d -e "container=docker" \
-p hostPort1:containerPort1 -p hostPort2:containerPort2 \
--privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup \
--name containerName imageId /usr/sbin/init
```
