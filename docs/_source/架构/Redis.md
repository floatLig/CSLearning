# Redis

## 1. Redis的安装

> 官方文档：<https://redis.io/download>

安装、配置、测试等步骤：

```bash
wget http://download.redis.io/releases/redis-5.0.8.tar.gz

tar xzf redis-5.0.8.tar.gz 

# 将 redis放到/user/local/redis的位置上
mv redis-5.0.8.tar.gz /usr/local/redis   

# make让多核CPU同时make
make -j 2  

# 将编译好的添加到系统的可执行目录里面
make install  

# 查看redis的路径
whereis redis 

# 修改redis的配置： bind 0.0.0.0, 允许后台运行 daemonize yes 密码 requirepass root等
vim redis.conf 

# 以指定配置文件打开
redis-server ./redis.conf 

# 查看进程
ps -ef | grep redis 

# 客户端 进行set get
redis-cli 
    shutdown save # 重启

ps -ef | grep redis

redis-server ./redis/conf

redis-cli

cd utils/
./install_server.sh # 设置为系统进程
    6379
    /usr/local/redis/redis.conf  # 放在同一个文件夹下面，方便管理
    /usr/local/redis/redis.log
    /usr/local/redis/data

chkconfig --list redis  

# 检查服务有没有开启，2 3 4 5应该是要打开的
chkconfig --list | grep redis 

systemctl status redis_6379

systemctl stop redis_6379

systemctl start redis_6379

ps -ef | grep redis

vim /etc/init.d/redis_6379 
    EXEC
    CLIEXEC等
```