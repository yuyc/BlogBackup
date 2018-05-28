---
title: redis主从部署
date: 2018-04-28 21:54:52
tags: 
    - redis
    - 主从配置
---

# redis主从部署
安装就不说了，详见《redis3.X安装》
## 基础配置
创建`6379`,`6380`两个文件夹，分别拷贝一份`redis.conf`,`redis-server`
## 主从配置
```
#我们看一下redis的从服务器配置
[root@centos6 6380]# grep -E -v "(^$|^#)" redis.conf
#是否开启后台运行模式
daemonize yes
#redis的pid文件位置
pidfile /var/run/redis.pid
#redis的运行端口
port 6380

tcp-backlog 511
timeout 0
tcp-keepalive 0
loglevel notice
logfile ""
databases 16

#save 900 1 意思是900秒内有1次更改就执行rdb拷贝
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir ./
#slaveof master服务器的ip地址及端口，这里我们master跟slave同机
slaveof 192.168.59.128 6379
#masterauth master的认证密码
masterauth foot
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
#requirepass redis服务的认证密码
requirepass foot

appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
[root@centos6 6380]#

```
主从使用的就`slaveof`,`masterauth`这两项基础配置，其他可根据需要进行填写。
至于master的配置就不贴了，默认的配置即可，主要是`requirepass`设置下认证密码。
## 启动服务
```
分别启动master、slave
#redis-server redis.conf
查看集群信息
#redis-cli
>info repcliation

```
## 测试主从
```
[root@centos6 6380]# /usr/local/src/redis-3.0.7/src/redis-cli
127.0.0.1:6379> auth foot
OK
127.0.0.1:6379> set key3 3
OK
127.0.0.1:6379> get key3
"3"
127.0.0.1:6379> exit
[root@centos6 6380]# /usr/local/src/redis-3.0.7/src/redis-cli -h 127.0.0.1 -p 6380
127.0.0.1:6380> auth foot
OK
127.0.0.1:6380> get key3
"3"
127.0.0.1:6380>

```
## 完成
