# redis 集群部署

###    本redis集群基于docker部署.三主三从.

##### 创建一个集群配置最小模板

   touch redis-cluster.tmpl

   vi redis-cluster.tmpl

 ##节点端口

port ${PORT}

 ##cluster集群模式

cluster-enabled yes

 ##集群配置名

cluster-config-file nodes-${port}.conf

##超时时间  

cluster-node-timeout 5000

 ##实际为各节点网卡分配ip  先用上网关ip代替
cluster-announce-ip 172.18.0.1

  ##节点映射端口 

cluster-announce-port ${PORT} 

 ##节点总线端

cluster-announce-bus-port 1${PORT} 

 ##持久化模式

appendonly yes

##### 创建redis-cluster文件夹.用来存放6个redis实例配置与数据

mkdir redis-cluster

执行

for port in `seq 7010 7015`; do   mkdir -p ./${port}/conf   && PORT=${port} envsubst < ./redis-cluster.tmpl > ./${port}/conf/redis.conf   && mkdir -p ./${port}/data; done

![1547691443658](C:\Users\zk\AppData\Local\Temp\1547691443658.png)



最后目录结构为上图.(aof与rdb为redis持久化数据, 刚创建目录时没有,nodes.conf为创建redis集群时自动创建,也没有)

##### docker创建网桥,编写docker-compose文件.

执行

​	docker network create redis-net



vi redis-compose.yml



version: "3"
services:
    redis1:
        container_name: redis-7010
        image: redis
        ports:
            - 7010:7010
            - 17010:17010
        volumes:
            - /home/zk/redis-cluster/7010/conf/redis.conf:/etc/redis/redis.conf
            - /home/zk/redis-cluster/7010/data:/data
        networks:
         default:
            ipv4_address: 172.18.0.10
        command: redis-server /etc/redis/redis.conf
    redis2:
        container_name: redis-7011
        image: redis
        ports:
            - 7011:7011
            - 17011:17011
        volumes:
            - /home/zk/redis-cluster/7011/conf/redis.conf:/etc/redis/redis.conf
            - /home/zk/redis-cluster/7011/data:/data
        networks:
         default:
            ipv4_address: 172.18.0.11
        command: redis-server /etc/redis/redis.conf
    redis3:
        container_name: redis-7012
        image: redis
        ports:
            - 7012:7012
            - 17012:17012
        volumes:
            - /home/zk/redis-cluster/7012/conf/redis.conf:/etc/redis/redis.conf
            - /home/zk/redis-cluster/7012/data:/data
        networks:
         default:
            ipv4_address: 172.18.0.12
        command: redis-server /etc/redis/redis.conf
    redis4:
        container_name: redis-7013
        image: redis
        ports:
            - 7013:7013
            - 17013:17013
        volumes:
            - /home/zk/redis-cluster/7013/conf/redis.conf:/etc/redis/redis.conf
            - /home/zk/redis-cluster/7013/data:/data
        networks:
         default:
            ipv4_address: 172.18.0.13
        command: redis-server /etc/redis/redis.conf
    redis5:
        container_name: redis-7014
        image: redis
        ports:
            - 7014:7014
            - 17014:17014
        volumes:
            - /home/zk/redis-cluster/7014/conf/redis.conf:/etc/redis/redis.conf
            - /home/zk/redis-cluster/7014/data:/data
        networks:
         default:
            ipv4_address: 172.18.0.14
        command: redis-server /etc/redis/redis.conf
    redis6:
        container_name: redis-7015
        image: redis
        ports:
            - 7015:7015
            - 17015:17015
        volumes:
            - /home/zk/redis-cluster/7015/conf/redis.conf:/etc/redis/redis.conf
            - /home/zk/redis-cluster/7015/data:/data
        networks:
         default:
            ipv4_address: 172.18.0.15
        command: redis-server /etc/redis/redis.conf
networks:
  default:
    external:
      name: redis-net



通过docker network inspect redis-net 查看网关ip 并且更改redis-compose.yml文件与各个文件下redis.conf

##### 启动redis实例并且集群化

docker-compose -f redis-compose.yml up -d

docker exec -it redis-7010 redis-cli -p 7010

![1547692248581](C:\Users\zk\AppData\Local\Temp\1547692248581.png)

cluster create 172.18.0.10:7010 172.18.0.11:7011 172.18.0.12:7012 172.18.0.13:7013 172.18.0.14:7014 172.18.0.15:7015 --cluster-replicas 1

集群化完成.