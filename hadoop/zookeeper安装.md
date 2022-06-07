1. 下载安装包

```java
wget https://dlcdn.apache.org/zookeeper/zookeeper-3.8.0/apache-zookeeper-3.8.0-bin.tar.gz --no-check-certificate

tar -zxvf apache-zookeeper-3.8.0-bin.tar.gz
mv apache-zookeeper-3.8.0-bin zookeeper
```

2. 新建数据文件夹

```java
   mkdir -p /usr/local/zookeeper/data
```

3.

```java
cp zoo_sample.cfg zoo.cfg
vi zoo.cfg

dataDir=/usr/local/zookeeper/data  //将数据文件夹改为自己刚刚创建的文件夹

//在文件夹的末尾增加自己服务的节点
server.1=hadoop01:2888:3888
server.2=hadoop02:2888:3888
server.3=hadoop03:2888:3888
```

4

```
scp -r zookeeper hadoop02:/opt
scp -r zookeeper hadoop03:/opt

echo 1 >  /usr/local/zookeeper/data/myid
echo 2 >  /usr/local/zookeeper/data/myid
echo 3 >  /usr/local/zookeeper/data/myid
```

5.

```
./bin/zkServer.sh start
```

6.

```
./bin/zkServer.sh status
```

