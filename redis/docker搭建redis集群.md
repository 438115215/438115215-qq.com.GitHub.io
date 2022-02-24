基于容器ip搭建
========

拉取镜像
----
```
docker  pull  redis:5.0.5
```
创建3个redis容器
-----------
```
docker create --name redis-node1 -v d/dataproject/redis/redis-cluster/node1:/data -p 6379:6379 redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-1.conf
​
docker create --name redis-node2 -v d/dataproject/redis/redis-cluster/node2:/data -p 6380:6379 redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-2.conf
​
docker create --name redis-node3 -v d/dataproject/redis/redis-cluster/node3:/data -p 6381:6379 redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-3.conf
```
--cluster-enabled yes 表示开启集群

--cluster-config-file filename 表示生成配置文件

使用docker inspect查看容器ip
```
docker inspect redis-node1
#172.17.0.2-node1
docker inspect redis-node2
#172.17.0.3-node2
docker inspect redis-node3
#172.17.0.4-node3
```
拿到 ip 信息后（每个人的ip信息可能不一样），接下来进入某一个容器进行组建集群：

# 这里以进入 node1 为例
```
docker exec -it redis-node1 /bin/bash
```

​
# 接着执行组建集群命令（请根据自己的ip信息进行拼接）
```
redis-cli --cluster create 172.17.0.2:6379  172.17.0.3:6379  172.17.0.4:6379 --cluster-replicas 0
```
基于host模式搭建
==========
```
docker create --name redis-node1 --net host -v d/dataproject/redis/redis-cluster/node1:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-1.conf --port 6379
​
docker create --name redis-node2 --net host -v d/dataproject/redis/redis-cluster/node2:/data redis:5.0.5 --cluster-enabled yes --cluster-config-file nodes-node-2.conf --port 6380
​
docker create --name redis-node3 --net host -v d/dataproject/redis/redis-cluster/node3:/data redis:5.0. --cluster-enabled yes --cluster-config-file nodes-node-3.conf --port 6381
```
跟之前创建命令不同，一是指定了 `--net` 网络类型为 `host`，二是这种情况下就不需要端口映射了，比如 `-p 6379:6379`，因为此时需要对外共享容器端口服务，所以只需要指定对外暴露的端口 `-p 6379`、`-p 6380` 等

启动容器并搭建集群
---------

# 启动命令

```
docker start redis-node1 redis-node2 redis-node3
```
# 进入某一个容器
```
docker exec -it redis-node1 /bin/bash
```


```
# 组建集群,192.168.29.194为当前物理机的ip地址
# 可能物理ip无法达成，可以试一试127.0.0.1
redis-cli --cluster create 127.0.0.1:6379  127.0.0.1:6380  127.0.0.1:6381 --cluster-replicas 0
```


参考文档：[https://www.cnblogs.com/niceyoo/p/13011626.html](https://www.cnblogs.com/niceyoo/p/13011626.html)