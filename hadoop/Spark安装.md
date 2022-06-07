```
export JAVA_HOME=/opt/module/jdk1
export SPARK_MASTER_HOST=bigdata111
export SPARK_MASTER_PORT=7077

export HADOOP_HOME=/opt/hadoop-2.10.2
export HADOOP_CONF_DIR=/opt/hadoop-2.10.2/etc/
下面的可以不写，默认
export SPARK_WORKER_CORES=1
export SPARK_WORKER_MEMORY=1024m
————————————————
版权声明：本文为CSDN博主「SuperBigData~」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/Jackson_mvp/article/details/103960470
```

配置从机
```
hadoop01
hadoop02
hadoop03
```



查看8080