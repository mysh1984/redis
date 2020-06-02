## 一、参考来源

https://www.runoob.com/docker/docker-redis-cluster.html

https://www.cnblogs.com/kismetv/p/9609938.html



源码来源：

[https://github.com/mysh1984/redis/tree/master/redis集群容器化部署]: 



![企业微信截图_20200602162646.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gfe0wp9zfuj30fx0bgtek.jpg)



## 二、架构

采用三主三从三哨兵机制

| 容器名         | 容器IP         | 映射端口    | 功能   |
| -------------- | -------------- | ----------- | ------ |
| redis-master-1 | 172.16.238.11  | 6379:6379   | master |
| redis-master-2 | 172.16.238.12  | 6380:6379   | master |
| redis-master-3 | 172.16.238.13  | 6381:6379   | master |
| redis-slave-1  | 172.16.238.14  | 6382:6379   | slave  |
| redis-slave-2  | 172.16.238.15  | 6383:6379   | slave  |
| redis-slave-3  | 172.16.238.16  | 6384:6379   | slave  |
| sentinel-9001  | 172.16.238.240 | 26379:26379 | 哨兵   |
| sentinel-9002  | 172.16.238.241 | 26380:26379 | 哨兵   |
| sentinel-9003  | 172.16.238.242 | 26381:26379 | 哨兵   |







## 三、部署 

3.1、环境文件

```bash
# vi .env 
# 节点1端口配置
redis1_port=6379
redis1_ports=6379:6379
redis1_cluster=16379:16379

# 节点2端口配置
redis2_port=6380
redis2_ports=6380:6379
redis2_cluster=16380:16379

# 节点3端口配置
redis3_port=6381
redis3_ports=6381:6379
redis3_cluster=16381:16379

# 节点4端口配置
redis4_port=6382
redis4_ports=6382:6379
redis4_cluster=16382:16379

# 节点5端口配置
redis5_port=6383
redis5_ports=6383:6379
redis5_cluster=16383:16379

# 节点6端口配置
redis6_port=6384
redis6_ports=6384:6379
redis6_cluster=16384:16379
```



3.2、docker-compose文件

```bash
# cat docker-compose.yml   

# redis-cli -a redis --cluster create 172.16.238.11:6379 172.16.238.12:6379 172.16.238.13:6379 172.16.238.14:6379 172.16.238.15:6379 172.16.238.16:6379 --cluster-replicas 1
version: '3.3'
services:
  # 节点1
  redis1:
    # 启动之后的容器名称
    container_name: redis-master-1
    env_file: 
      # 环境配置
      - ./.env
    # 使用哪种镜像
    image: 'harbor.ubtrobot.com/database/redis:5.0.7'
    # 端口映射
    ports:
      - ${redis1_ports}
      - ${redis1_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.11
    volumes:
      # 容器的data映射到宿主机
      - ./node-${redis1_port}/data:/data
      # 加载配置文件
      - ./node-${redis1_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment: 
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点2
  redis2:
    container_name: redis-master-2
    env_file: 
      - ./.env
    # 使用哪种镜像
    image: 'harbor.ubtrobot.com/database/redis:5.0.7'
    # 端口映射
    ports:
      - ${redis2_ports}
      - ${redis2_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.12
    volumes:
      # 容器的data映射到宿主机
      - ./node-${redis2_port}/data:/data
      # 加载配置文件
      - ./node-${redis2_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件  
    command: redis-server /usr/local/etc/redis/redis.conf
    environment: 
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点3
  redis3:
    container_name: redis-master-3
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'harbor.ubtrobot.com/database/redis:5.0.7'
    # 端口映射
    ports:
      - ${redis3_ports}
      - ${redis3_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.13
    volumes:
      # 容器的data映射到宿主机
      - ./node-${redis3_port}/data:/data
      # 加载配置文件
      - ./node-${redis3_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点4
  redis4:
    container_name: redis-slave-1
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'harbor.ubtrobot.com/database/redis:5.0.7'
    # 端口映射
    ports:
      - ${redis4_ports}
      - ${redis4_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.14
    volumes:
      # 容器的data映射到宿主机
      - ./node-${redis4_port}/data:/data
      # 加载配置文件
      - ./node-${redis4_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点5
  redis5:
    container_name: redis-slave-2
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'harbor.ubtrobot.com/database/redis:5.0.7'
    # 端口映射
    ports:
      - ${redis5_ports}
      - ${redis5_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.15
    volumes:
      # 容器的data映射到宿主机
      - ./node-${redis5_port}/data:/data
      # 加载配置文件
      - ./node-${redis5_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  # 节点6
  redis6:
    container_name: redis-slave-3
    env_file:
      - ./.env
    # 使用哪种镜像
    image: 'harbor.ubtrobot.com/database/redis:5.0.7'
    # 端口映射
    ports:
      - ${redis6_ports}
      - ${redis6_cluster}
    networks:
        cluster-net:
          ipv4_address: 172.16.238.16
    volumes:
      # 容器的data映射到宿主机
      - ./node-${redis6_port}/data:/data
      # 加载配置文件
      - ./node-${redis6_port}/redis.conf:/usr/local/etc/redis/redis.conf
    # 启动redis的时候指定配置文件
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      # 设置时区为上海，否则时间会有问题
      - TZ=Asia/Shanghai
  sentinel-9001:
    container_name: sentinel-9001
    image: 'harbor.ubtrobot.com/database/redis:5.0.7'
    ports:
      - 26379:26379
    networks:
        cluster-net:
          ipv4_address: 172.16.238.240
    volumes:
      - ./sentinel-9001/data:/data
      - ./sentinel-9001/sentinel.conf:/usr/local/etc/redis/sentinel.conf
    command: redis-server /usr/local/etc/redis/sentinel.conf --sentinel

  sentinel-9002:
    container_name: sentinel-9002
    image: 'harbor.ubtrobot.com/database/redis:5.0.7'
    ports:
      - 26380:26379
    networks:
        cluster-net:
          ipv4_address: 172.16.238.241
    volumes:
      - ./sentinel-9002/data:/data
      - ./sentinel-9002/sentinel.conf:/usr/local/etc/redis/sentinel.conf
    command: redis-server /usr/local/etc/redis/sentinel.conf --sentinel

  sentinel-9003:
    container_name: sentinel-9003
    image: 'harbor.ubtrobot.com/database/redis:5.0.7'
    ports:
      - 26381:26379
    networks:
        cluster-net:
          ipv4_address: 172.16.238.242
    volumes:
      - ./sentinel-9003/data:/data
      - ./sentinel-9003/sentinel.conf:/usr/local/etc/redis/sentinel.conf
    command: redis-server /usr/local/etc/redis/sentinel.conf --sentinel
networks:
  # 创建集群网络，在容器之间通信
  cluster-net:
    ipam:
      config:
        - subnet: 172.16.238.0/24
```



3.3、启动

```bash
#docker-compose up -d --build
```

如果哨兵容器启动异常，将sentinel.conf文件属性改成777再重启



3.4、初始化集群

进入任一容器，执行

```bash
# docker exec -it redis-master-1 sh
#  redis-cli -a redis --cluster create 172.16.238.11:6379 172.16.238.12:6379 172.16.238.13:6379 172.16.238.14:6379 172.16.238.15:6379 172.16.238.16:6379 --cluster-replicas 1
```

![企业微信截图_20200602155709.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gfe021iyjsj30vl0oqtnl.jpg)





## 四、查看信息

4.1、查看节点信息

```bash
# redis-cli   
127.0.0.1:6379> auth redis
OK
127.0.0.1:6379> cluster nodes
42dfcebc238de4e5a053b917c90ca7c3066436f9 172.16.238.15:6379@16379 slave 1a24090644780d774642cb8ecfb44b77a8d16dd3 0 1591084697647 5 connected
ba3dee70f5bb467e23692c0358dad479b03a1ae2 172.16.238.12:6379@16379 master - 0 1591084696641 2 connected 5461-10922
9b38838b0700cac347910133e938798b14ce7938 172.16.238.16:6379@16379 slave ba3dee70f5bb467e23692c0358dad479b03a1ae2 0 1591084694625 6 connected
007ec8c6ab7c80d237a287c33854447967de3db9 172.16.238.14:6379@16379 slave 534450178b5a2a0cdd5964596125f9048c4ab83d 0 1591084696000 4 connected
534450178b5a2a0cdd5964596125f9048c4ab83d 172.16.238.13:6379@16379 master - 0 1591084697000 3 connected 10923-16383
1a24090644780d774642cb8ecfb44b77a8d16dd3 172.16.238.11:6379@16379 myself,master - 0 1591084695000 1 connected 0-5460
```





4.2、查看info

```bash
127.0.0.1:6379> info

```

![企业微信截图_20200602160658.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gfe0c3tv32j30mm0p6dps.jpg)



4.3、插入数据测试 

 往集群中某个redis中放入值，会计算key的 hash所属的槽，然后计算 槽是不是属于当前实例，如果不是返回error，并且返回正确节点地址和端口

```bash
127.0.0.1:6379> set aaa aaa
(error) MOVED 10439 172.16.238.12:6379

#在以上提示的节点上可以正确执行
# docker exec -it redis-master-2 sh
# redis-cli     
127.0.0.1:6379> auth redis
OK
127.0.0.1:6379> set aaa aaa
OK
127.0.0.1:6379> get aaa
"aaa"

#不同的键值对存在不同的节点上，keys * 只会返回当前节点的所有key
127.0.0.1:6379> keys *
1) "aaa"
```



4.4、模拟节点宕机

模拟master-01、master-02宕机

```bash
# docker stop redis-master-1
redis-master-1
# docker stop redis-master-2
redis-master-2



127.0.0.1:6379> cluster nodes
42dfcebc238de4e5a053b917c90ca7c3066436f9 172.16.238.15:6379@16379 myself,slave 1a24090644780d774642cb8ecfb44b77a8d16dd3 0 1591085999000 5 connected
9b38838b0700cac347910133e938798b14ce7938 172.16.238.16:6379@16379 slave ba3dee70f5bb467e23692c0358dad479b03a1ae2 0 1591086003743 6 connected
534450178b5a2a0cdd5964596125f9048c4ab83d 172.16.238.13:6379@16379 master - 0 1591086002000 3 connected 10923-16383
1a24090644780d774642cb8ecfb44b77a8d16dd3 172.16.238.11:6379@16379 master,fail? - 1591085722931 1591085720000 1 connected 0-5460
007ec8c6ab7c80d237a287c33854447967de3db9 172.16.238.14:6379@16379 slave 534450178b5a2a0cdd5964596125f9048c4ab83d 0 1591086003000 4 connected
ba3dee70f5bb467e23692c0358dad479b03a1ae2 172.16.238.12:6379@16379 master,fail? - 1591085724643 1591085722000 2 connected 5461-10922
```



在slave节点上可以看到对应的master节点状态down

![企业微信截图_20200602162138.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gfe0rcy4xaj30kp0p57di.jpg)



数据转移到对应的slave节点上

```bash
[root@file redis-sen]# docker exec -it redis-slave-2 sh
# redis-cli
127.0.0.1:6379> auth redis
OK
127.0.0.1:6379> keys *
1) "b"
127.0.0.1:6379> exit
# exit
[root@file redis-sen]# docker exec -it redis-slave-3 sh
# redis-cli
127.0.0.1:6379> auth redis
OK
127.0.0.1:6379> keys *
1) "aaa"
```





4.5、故障恢复

恢复故障节点

```bash
[root@file sentinel-9002]# docker start redis-master-1
[root@file sentinel-9002]# docker start redis-master-2
```



之前的master节点恢复master角色

```bash
[root@file sentinel-9002]# docker exec -it redis-master-1 sh
# redis-cli
127.0.0.1:6379> auth redis
OK
127.0.0.1:6379> keys *
1) "b"
127.0.0.1:6379> exit
# exit
[root@file sentinel-9002]# docker exec -it redis-master-2 sh
# redis-cli
127.0.0.1:6379> auth redis
OK
127.0.0.1:6379> keys *
1) "aaa"
```

![企业微信截图_20200602162603.png](http://ww1.sinaimg.cn/large/007Xg1efgy1gfe0vxczysj30ni0pjq40.jpg)