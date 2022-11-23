# Docker 手册

需要运行一个redis后台服务器；

**首先，**搜索docker-hub中包含redis的镜像名；

```shell
docker search redis
```

## **运行一个容器**

```shell
docker run <options> <image-name>
```

默认情况下，docker运行一个命令在前台，所以需要使用`-d`来进行指定；

```shell
docker run -d redis:3.2 # 指定redis版本号
```

## **寻找正在运行的容器**

`docker ps`

## **查看容器更多细节**

`docker inspect <friendly-name|container-id>`

## **查看容器打印的日志**

`docker logs <friendly-name|container-id>`

## **映射容器端口**

每个容器的运行都是隔离得（namespace,chroot, cgroup）；

` -p <host-port>:<container-port>`

`docker run -d --name redisHostPort -p 6379:6379 redis:latest`

> 默认情况下，host映射为全ip

如果只使用 `-p <container-port>` docker，则会映射动态的端口；

## 持久化存储

在重新创建的容器的时候，容器内部存储的数据会被删除；

docker的参数为： ` *-v <host-dir>:<container-dir>*`

`docker run -d --name redisMapped -v /opt/docker/data/redis:/data redis`

# 使用Dockerfile

Docker镜像可以基于Dockerfile的内容进行构建；

Dockerfile将说明如何部署应用程序的指令列表；

例如：

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

说明:

- 第一行定义我们的基础镜像；
- 第二行将拷贝当前文件夹的内容到容器内指定的位置；

## 执行build

build命令将执行dockerfile中的每一条指令；

`*docker build -t <build-directory>*`

-t ： tag

**相关命令**

FROM：指定基础镜像；

RUN：允许像在命令行下一样的执行命令；结果将永久的保存到镜像中；

COPY <src> <dest> 

**暴露端口**

Expose <port>, 只是一个声明；

**默认命令**

CMD 定义容器启动时要运行的默认命令；如果命令需要参数，建议使用数组的方式；



**Docker在Dockerfile中缓存执行一行的结果以供将来构建时使用；**

如果不想使用缓存构建，则使用--no-cache=true设置docker build 命令的一部分；



# 使用OnBuild进行优化构建

在命令前加上onbuild，当前命令并不执行，只有在被别的镜像作为基础镜像时才进行编译；





# 忽略文件

.dockerignore 中的文件不会被拷贝到容器中；

可以缩短构建时间；



# 数据容器

数据容器是唯一负责存储/管理数据的容器；

在docker ps 时，程序并未执行；

```shell
docker create -v /config --name dataContainer busybox	
```

## 拷贝文件

```shell
docker cp config.conf dataContainer:/config/
```

## 挂载卷

```shell
docker run --volumes-from dataContainer ubuntu ls /config
```

如果容器中/config存在，挂在卷则会覆盖；可以映射多个卷到一个容器；

## 导出、导入

```shell
docker export dataContainer > dataContainer.tar
docker import dataContainer.tar
```

# 创建link

当创建一个新容器时，想要链接另外一个容器，可以使用 

`--link <container-name|id>:<alias>`

alias为新容器内部中/etc/hosts的DNS命名；

# 容器网络

创建一个网络，能够允许网络内的容器能够相互发现；

## 创建

```shell
docker network create backend-network
```

## 链接

在运行容器时使用--net来将容器链接到网络上；

```shell
docker run -d --name=redis --net=backend-network redis
```

不想link, docker network能够像传统的网络一样，节点可以上线/下线；

另外，不想link那样会在新容器中包含其他容器的环境变量以及修改/etc/hosts文件；与之替代的，使用嵌入式的DNS服务；

当容器试图使用容器名称去连接其他的容器时，DNS服务奖返回容器正确的ip地址；例如；

```shell
docker run --net=backend-network alpine ping -c1 redis
```

**已经存在的容器如何连接网络**

```shell
docker network connect frontend-network redis
```

## 创建别名

```
docker network connect --alias db frontend-network redis
```

## 查看当前网络上容器

```shell
docker network inspect frontend-network
```

## 断开连接

```shell
docker network disconnect frontend-network redis
```

# 数据卷

数据卷允许你映射一个主机目录到共享数据的容器中；

在创建容器时，使用 `-v  <host>:<container>`进行操作；

例如：

```shell
docker run -v /docker/redis-data:/data --name r1 -d redis redis-server --appendonly yes
```

如何传输数据到容器中：

```shell
cat data | docker exec -i r1 redis-cli --pipe
```

解释： docker exec -i r1 指名容器

`redis-cli --pipe` 为redis命令；

同样的目录可以挂在到其他的容器上；

使用 `-volumes-from`来代替；

```shell
docker run --volumes-from r1 -it ubuntu ls /data
```

## 只读卷

挂载卷如不限制，将给与容器所有读和写权限；

使用`:ro`  来确保容器只读；



# 容器日志

当启动一个容器时，docker将追踪标准输出和标准错误输出

```shell
docker logs <container-id:name>
```

## 系统日志

默认情况下，docker的系统日志使用json-file格式输出到主机磁盘上；可能会导致磁盘被大文件占满；



# 配置重启策略

启动时，使用参数`--restart=on-failure:#`#为重试次数；

如果想一直重启，使用`--restart=always`



# 容器标签

## 单个标签

使用`-l  <key>=<value>`来对容器打标签；

## 多个标签

可以使用如下方式：

```shell
echo 'user=123461' >> labels && echo 'role=cache' >> labels
docker run --label-file=labels -d redis
```

Dockerfile中使用LABEL <key>=<value> /

可以再docker ps 查询时使用filter来查询一类标签；



## 查看docker指标

```
docker stats <container-id>|<name>
```



# 格式化查询

```
docker ps --format '{{.Names}} container is using {{.I
```

使用table:

```shell
docker ps --format 'table {{.Names}}\t{{.Image}}'
```

需要inspect中的信息：

```shell
docker ps -q | xargs docker inspect --format '{{ .Id }} - {{ .Name }} - {{ .NetworkSettings.IPAddress }}'
```