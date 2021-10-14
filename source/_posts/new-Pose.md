---
title: ubuntu下忘记MySQL密码
date: 2021-10-12 13:52:48
tags: "mysql"
cover: 
---
## ubuntu

忘记密码以后可以用debain-sys-maint账户找回
```
sudo cat /etc/mysql/debian.cnf
```

找到debian-sys-maint 的密码

```sql
mysql -u debian-sys-maint -p
```

刚才的密码
```sql
use mysql;
```

## 8.0
```sql
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '新密码注意包含大小写数字特殊字符不然修改不过';
```

## 5.0

```sql
update mysql.user set authentication_string=password('123456') where user='root' and Host ='localhost';
```
查询条件可在这个表里找

```sql
select user,host,plugin from mysql.user;
```
```sql
flush privileges;
```
OK!