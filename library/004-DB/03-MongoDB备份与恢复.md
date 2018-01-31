# MongoDB备份与恢复
## 1. 备份 
采用mongodump工具

常用的选项如下：

```
mongodump -h host -d dbname -o directory
mongodump -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 -o 文件存在路径，如果想导出所有数据库，可以去掉-d
```


```
mongodump -h 127.0.0.1 -o /data/backup/ 
-h：MongDB所在服务器地址，如：127.0.0.1，也可以指定端口号：127.0.0.1:27017
-d：需要备份的数据库名称，如：db_test
-o：备份的数据存放位置，如：~\dump，当然该目录需要提前建立，在备份完成后，系统自动在dump目录下建立一个db_test目录，这个目录里面存放该数据库实例的备份数据。
```

## 2. 恢复
采用mongorestore工具。
mongorestore是Mongodb从备份中恢复数据的工具，它主要用来获取mongodump的输出结果，并将备份的数据插入到运行的Mongodb中。
mongorestore -h host -d dbname --directoryperdb dbdirectory
Eg:  # mongorestore --host=10.0.0.25 --port=27017 --db ztjy --dir=ztjy/
-h：MongoDB所在服务器地址
-d：需要恢复的数据库名称，如：db_test，当然这个名称可以不同于备份的时候，比如new_db
--directoryperdb：备份数据文件所在位置，如：~\dump\db_test（这里之所以要加db_test子目录，从mongoretore的help中的--directoryperdb，可以读出“每一个db在一个单独的目录”。）


