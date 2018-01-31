# docker搭建sentry


## 安装 python-pip

```
yum -y install python-pip
```

## 安装 docker-compose

**注意：**docker-compose必须为1.6版本以上

```
curl -L https://github.com/docker/compose/releases/download/1.9.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
## 获取sentry

```
cd /data/
git clone https://github.com/getsentry/onpremise.git
```

```
mkdir  -p {sentry,postgres}
```


```
docker-compose run --rm web config generate-secret-key
```

