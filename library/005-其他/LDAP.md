
# 开始：
> 系统环境：centos7.3
> ldap版本  2.4.23

### 安装ldap以及相关依赖包
```
yum install -y openldap openldap-clients openldap-servers migrationtools
```
### 配置数据库文件

```
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown ldap. /var/lib/ldap/DB_CONFIG
```

### 启动和加入开机自启动

```
systemctl start slapd &&
systemctl enable slapd
```
### 设置openldap管理密码

重复输入两次明文密码后（我是：Jn25fT!ME@n6o），我得到一个加密后密码（后面会用到）：{SSHA}DwsbE/WBGL4U6Kt27NeVJNZI94MR4wFp

#### 下面具体看情况：

```
cp /usr/share/openldap-servers/slapd.conf.obsolete slapd.conf
```

#### 以下是slaod.conf，需要改的我拿黑体标出来，由于我是卸载了一遍又装的这里文件是空的，下边这个文件是我拼凑出来，如果又文件只需要修改就好，没有也不要害怕。

- **include /etc/openldap/schema/core.schema**
- **include /etc/openldap/schema/cosine.schema**
- **include /etc/openldap/schema/inetorgperson.schema**
- **include /etc/openldap/schema/nis.schema**


pidfile /var/run/openldap/slapd.pid
argsfile /var/run/openldap/slapd.args


- database	bdb
- suffix		**"dc=kmf,dc=com"**
- rootdn		**"dn=kmfadmin,dc=kmf,dc=com"**

- rootpw		**{SSHA}DwsbE/WBGL4U6Kt27NeVJNZI94MR4wFp**

directory	/var/lib/ldap         #这里是数据库存放位置根据自己的需要更改对应上边的第一步配置数据库文件

index cn,sn,uid,displayName pres,sub,eq
index uidNumber,gidNumber eq
index objectClass pres,eq
index default sub

loglevel 296
cachesize 1000
checkpoint 2018 10
### 验证配置是否成功
- slaptest -u
- slaptest -f /etc/openldap/slapd.conf  #检测这个配置文件是否正确

#### 到下面这一步具体看自己服务器，我觉得下边这个可以执行
#### 之前到这里我是报错了。因为这个坑实在太多我已经忘记有多少错了，这个是因为我本身服务器目录下边就没有任何文件，所以以后再安装假如删除也是无所谓的。
- 删除之前配置文件生成的信息
- rm -rf /etc/openldap/slapd.d/* #这个现在明白了，是每次修改slapd.conf都需要从新生成这个目录下的文件。
- laptest -f /etc/openldap/slapd.conf -F /etc/openldap/slapd.d #这个就是生成命令。
 
57763ec6 bdb_monitor_db_open: monitoring disabled; configure monitor database to enable  
config file testing succeeded 

### 授权
- chown -R ldap. /etc/openldap/slapd.d/
### 查看端口
- netstat -anlpt

### 新建临时配置目录：
- mkdir /root/my_ldif ; cd /root/my_ldif
### 插件初始的入口文件
vim basedomain.ldif

>dn: dc=kmf,dc=com
>objectClass: top
>objectClass: dcObject
>objectclass: organization
>o: kmf com
>dc: kmf

>dn: cn=kmfadmin,dc=kmf,dc=com
>objectClass: organizationalRole
>cn: kmfadmin
>description: Directory Manager

>dn: ou=People,dc=kmf,dc=com
>objectClass: organizationalUnit
>ou: People

>dn: ou=Group,dc=kmf,dc=com
>objectClass: organizationalUnit
>ou: Group

### 导入配置文件到ldap
- ldapadd -x -W -D "cn=kmfadmin,dc=kmf,dc=com" -f /root/my_ldif/basedomain.ldif
- 这里补充一下：有时候在导入你需要添加的东西是，比如这个文件的第一个帐号在数据库中已经存在了，下边的第二个用户是你希望新添加的，但hi这时候会卡住说：  already exists   解决：可以加-c参数强制加入，ldapadd -x -c -W -D。或者是把存在的帐号先注视掉。
### 查看是否添加成功
- [root@bj-jumpserver my_ldif]# ldapsearch -LLL -W -x -D "cn=kmfadmin,dc=kmf,dc=com" -H ldap://localhost -b "dc=kmf,dc=com"
- Enter LDAP Password: 
dn: dc=kmf,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
o: kmf com
dc: kmf

dn: cn=kmfadmin,dc=kmf,dc=com
objectClass: organizationalRole
cn: kmfadmin
description: Directory Manager

dn: ou=People,dc=kmf,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=kmf,dc=com
objectClass: organizationalUnit
ou: Group

### 这里说一下怎么导入批量用户进ldap
### 1.创建需要添加的用户
- useradd ldapuser01
- useradd ldapuser02
### 设置密码
- passwd ldapuser01
- passwd ldapuser02

### 3.截取所有用户和组信息到指定的文件中
- grep ":10[0-9][0-9]" /etc/passwd > /root/passwd       #用户
- grep ":10[0-9][0-9]" /etc/group > /root/my_ldif/group  #截取想导入到ldap的组信息
### 4. 生成文件
- 先参考[这个文档](http://www.cnblogs.com/lemon-le/p/6266921.html)对照着下方来做。
- cd  /usr/share/migrationtools
- vim migrate_common.ph  #修改下边这三个位置的值
- $DEFAULT_MAIL_DOMAIN = "**ixmsoft.com**";
- $DEFAULT_BASE = "**dc=ixmsoft,dc=com**";
- $EXTENDED_SCHEMA = **1**;
### 5.开始生成ldif文档
- /usr/share/migrationtools/migrate_base.pl >/root/my_ldif/base.ldif  #生成数据库基础模版
- 对于下边这个有一些补充。我在步骤3生成的是对于后来自己添加的用户的一个截取然后导入的模版
- /usr/share/migrationtools/migrate_passwd.pl /root/my_ldif/passwd /root/my_ldif/users.ldif  #用户模版
- /usr/share/migrationtools/migrate_group.pl /root/my_ldif/group /root/my_ldif/group.ldif
### 6.导入ldif文件到ldap
- ldapadd -x -W -D cn=kmfadmin,dc=kmf,dc=com -f /root/my_ldif/users.ldif #把用户导入进去
- ldapadd -x -W -D cn=kmfadmin,dc=kmf,dc=com -f /root/my_ldif/group.ldif #把组导入进去
### 7.查看添加的用户ludapuser02
- [root@bj-jumpserver my_ldif]# **ldapsearch -LLL -W -x -D "cn=kmfadmin,dc=kmf,dc=com" -H ldap://localhost -b "dc=kmf,dc=com" "(uid=ldapuser02)"**

Enter LDAP Password: 

dn: uid=ldapuser02,ou=People,dc=kmf,dc=com
uid: ldapuser02
cn: ldapuser02
sn: ldapuser02
mail: ldapuser02@kmf.com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JDdMb09CMzc3JHAuSEc5YS9TMElqWDRTR1g5YXpuLmxTQUE0TGd
 OQUNqOHhDcEVuTWZYaXhaSnhaamRvNlBLTk1YZkc5WWdhZGRuS2ZDeFBsUXlCd04wN2tjNWRTdHQw
shadowLastChange: 17373
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1003
gidNumber: 1003
homeDirectory: /home/ldapuser02

### 到此ldap算是搭建完毕，下面开始进行ldap-web的管理界面
- 需要nginx和php
> 这里对于nginx和apache个人根据公司环境，但是用apache会相对简单。
> php5.6以上
> 参考官方文档[这里](https://www.ldap-account-manager.org/static/doc/manual-onePage/index.html#a_install)
> 如果用nginx需要安装tar.bz2的源码吧
### 开始安装
- 首先下载安装包需要版本的安装包
- wget http://prdownloads.sourceforge.net/lam/ldap-account-manager-6.0.tar.bz2?download
- 重命名： mv ldap-account-manager-6.0.tar.bz2?download ldap-account-manager-6.0.tar.bz2
- 解压： tar xjf ldap-account-manager-6.0.tar.bz2
- 安装文件-手动复制
将文件复制到Web服务器的访问目录范围，列入我是放到/opt/www/root/
然后修改权限，修改成nginx的用户对此文件可读写和执行权限
然后访问在界面上进行相关配置参考[官方文档](https://www.ldap-account-manager.org/static/doc/manual-onePage/index.html#a_install)

## LDAP客户端设置
#### LDAP客户端通过与服务端关联起来，就可以使用服务端的系统账号登录系统，通过useradd 添加用户是在ldap里是没有显示的，ldap添加用户，在/etc/passwd里也是没有显示的，ldap添加用户后，就可以通过su - user，登录到创建的用户，ssh也可以登录


