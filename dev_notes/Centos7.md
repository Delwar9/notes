...menustart

 - [Centos7](#8db8d64778c367a62ad2b609fd6c2095)
     - [mono](#654db8a14a5f633b9ba85ec92dc51f7c)
     - [firewall](#36e5371ad91c9d2d09e9d7c0e76055db)
     - [Mysql](#9edb3c572b56b91542af659480518681)
     - [查看 TIME_WAIT](#6dfa7a8202801ab86e07a577ba4da59f)
     - [TIME_WAIT 优化](#0d0306c1df74d541e32e7dcbaaf0fe69)
     - [linux 系统监控](#65395697d2dab77dd22f054b888fb803)
     - [ab test](#107aedca6bab06cabee6aac093e48464)

...menuend


<h2 id="8db8d64778c367a62ad2b609fd6c2095"></h2>

# Centos7




<h2 id="654db8a14a5f633b9ba85ec92dc51f7c"></h2>

## mono

```bash
yum install yum-utils
rpm --import "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"
yum-config-manager --add-repo http://download.mono-project.com/repo/centos7/

yum -y install mono-complete  
# mono-core mono-devel ? I forgot the exact command ...
```

 - build c# project 

```bash
xbuild /p:Configuration=Release xxx.sln
```

<h2 id="36e5371ad91c9d2d09e9d7c0e76055db"></h2>

## firewall 

open a port :

```bash
# add ssh port as permanent opened port
firewall-cmd --zone=public --add-port=22/tcp --permanent
```


Then, you can reload rules to be sure that everything is ok

```bash
firewall-cmd --reload

firewall-cmd --zone=public --list-ports
```

remove a port

```bash
firewall-cmd --zone=public --remove-port=8091/tcp --permanent
```

option:  when you app launched , you can check whether the ports are correctly listening 

```bash
netstat -nltp
```

<h2 id="9edb3c572b56b91542af659480518681"></h2>

## Mysql

```
yum install mariadb-server mariadb  mariadb-devel

systemctl start mariadb  #启动MariaDB
systemctl stop mariadb  #停止MariaDB
systemctl restart mariadb  #重启MariaDB
systemctl enable mariadb  #设置开机启动
```

To make Mysql case sensitive,  find and modify `lower_case_table_names` to 1:

```
vi /etc/my.cnf

[mysqld]
lower_case_table_names=2
```

Delete anonymous user:

```
mysql> use mysql 
mysql> delete from user where user=''; 
mysql> FLUSH PRIVILEGES;
```

Create a database account, username and password is “kbe”:

```
mysql> grant all privileges on *.* to kbe@'%' identified by 'kbe';
mysql> grant select,insert,update,delete,create,drop on *.* to kbe@'%' identified by 'kbe';
mysql> FLUSH PRIVILEGES;
```


<h2 id="6dfa7a8202801ab86e07a577ba4da59f"></h2>

## 查看 TIME_WAIT

```
$ netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
ESTABLISHED 20002
TIME_WAIT 1
```

<h2 id="0d0306c1df74d541e32e7dcbaaf0fe69"></h2>

## TIME_WAIT 优化

```
# vi /etc/sysctl.conf

net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1

# 然后执行 /sbin/sysctl -p 让参数生效。
```

<h2 id="65395697d2dab77dd22f054b888fb803"></h2>

## linux 系统监控

```
$ yum install -y dstat
$ dstat 
----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
usr sys idl wai hiq siq| read  writ| recv  send|  in   out | int   csw 
  0   0 100   0   0   0|   0     0 | 431B  346B|   0     0 | 159   181 
  0   0 100   0   0   0|   0     0 | 371B  826B|   0     0 | 200   211 
```

 - To display information provided by vmstat,
    - Process stats
    - Memory stats

```
$ dstat --vmstat
```

 - to monitor a single program that is using the most CPU and consuming the most amount of memory.

```
$ dstat -c --top-cpu -dn --top-mem
```

 - you can also store the output of dstat in a .csv 
    - Here, we are displaying the time, cpu, mem, system load stats with a one second delay between 5 updates (counts).

```
$ dstat --time --cpu --mem --load --output report.csv 1 5
```


<h2 id="107aedca6bab06cabee6aac093e48464"></h2>

## ab test 

 - install ab

```
yum -y install httpd-tools 
```

 - ab test

```
$ echo "{ \"channel\": \"official\"}" > post.json
$ ab -k -n 500000 -c 20000 -T "application/json" -p post.json  -H "userID: debugUserID" -H "Authorization: 7eb0f0a9798af24a883f4859db88a634"  http://10.192.8.17:9000/announcement
```


