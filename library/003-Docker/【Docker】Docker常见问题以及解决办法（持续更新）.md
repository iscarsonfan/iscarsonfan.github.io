# 【Docker】Docker常见问题以及解决办法（持续更新）

1. centos7容器内无法使用systemctl命令

Error：Failed to get D-Bus connection: Operation not permitted

容器内报错原因是因为centos7 dbus-daemon未启动，所以systemctl并不是不能使用，需要将将CMD或者entrypoint设置为/usr/sbin/init，docker容器会自动将dbus等服务启动起来

**例：**

```
docker run  --privileged -ti -d  -p 82:80 -v /Users/fanchuncheng/Documents/szy-server:/data/web ztjy:v2  /usr/sbin/init
```

