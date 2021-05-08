redis 环境搭建

**1.环境配置**

Redis是c开发的,因此安装redis需要c语言的编译环境,即需要安装gcc
  是否安装gcc: gcc -v
  如果没有gcc,则需要在线安装
  yum install gcc-c++

解压redis压缩包

tar zxf redis-3.0.0.tar.gz

进入目录查看是否存在Makefile文件,存在则直接make编译redis源码

安装编译后的redis代码到指定目录,一般存放于/usr/local下的redis目录,指令如下

make install PREFIX=/usr/local/redis

进入 /usr/local/redis/bin
至此,可以启动redis了,默认启动模式为前端启动,指令如下 
./redis-server

后端启动
前端启动的话,如果客户端关闭,redis服务也会停掉,所以需要改成后台启动redis.
具体做法分为两步 -> 第一步:将redis解压文件里面的redis.conf文件复制到当前目录,指令如下

cp ~/redis-3.0.0/redis.conf .
1
第二步:修改redis.conf文件,将daemonize no -> daemonize yes,这样便将启动方式修改为后台启动了

vim redis.conf
————————————————
启动redis -> 后台启动

./redis-server redis.conf

 查看redis是否在运行,指令如下

ps aux|grep redis

1
2.13 打开redis连接

./redis-cli

集群环境：
在usr/local目录下新建redis-cluster目录，用于存放集群节点

把redis目录下的bin目录下的所有文件复制到/usr/local/redis-cluster/redis01目录下，不用担心这里没有redis01目录，会自动创建的。操作命令如下（注意当前所在路径）：

cp -r redis/bin/ redis-cluster/redis01


删除rm -rf dump.rdb

修改端口号为7001,默认是6379

将cluster-enabled yes 的注释打开

复制5份(cp -r redis01/ redis02)  redis01-redis05 分别修改为7002-7006
批量启动 start_all.sh  
修改启动权限chmod +x start-all.sh

cd redis01
./redis-server redis.conf
cd ..
cd redis02
./redis-server redis.conf
cd ..
cd redis03
./redis-server redis.conf
cd ..
cd redis04
./redis-server redis.conf
cd ..
cd redis05
./redis-server redis.conf
cd ..
cd redis06
./redis-server redis.conf
cd ..

ok，至此6个redis节点启动成功，接下来正式开启搭建集群，以上都是准备条件

按照yum install ruby
gem install redis-3.0.0.gem

接下来需要把这个ruby脚本工具复制到usr/local/redis-cluster目录下。那么这个ruby脚本工具在哪里呢？之前提到过，在redis解压文件的源代码里，即redis/src目录下的redis-trib.rb文件。

 将该ruby工具（redis-trib.rb）复制到redis-cluster目录下，指令如下：

cp redis-trib.rb /usr/local/redis-cluster

使用  ./redis-trib.rb create --replicas 1 9.134.64.243:7001 9.134.64.243:7002 9.134.64.243:7003 9.134.64.243:7004 9.134.64.243:7005 9.134.64.243:7006
创建集群 选择yes

最后连接集群节点，连接任意一个即可：

redis01/redis-cli -p 7001 -c 

最后，加上两条redis集群基本命令：
1.查看当前集群信息
cluster info 1

2.查看集群里有多少个节点
cluster nodes


