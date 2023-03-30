# VScode 远程开发

## 环境搭建

### window

1. 安装VScode 
2. 安装Remote Development插件。 在VS Code中打开Extensions View，搜索“ Remote Development ”，然后安装。
3. 连接到远程服务器。 按Ctrl + Shift + P，打开命令面板，并搜索“Remote-SSH：Connect to Host”。 输入您的用户名和主机IP地址，然后输入密码。
4. VSCode将连接到您的远程服务器。 在弹出的菜单中，选择“Open Folder”来打开服务器上的文件夹。您现在可以像在本地计算机上一样编辑文件，保存更改并在服务器上编译代码。

注：如果是多人使用同一个机子进行开发，需要保持 VScode 版本一致。

### Linux

注：如果机子不能连接公网，那么需要手动安装 vs-code-server → [教程](https://zhuanlan.zhihu.com/p/294933020)

#### 安装 Git

```shell
# 直接下载
yum install -y git
```



#### 安装 Node

略



#### 安装 Docker 

https://www.runoob.com/docker/centos-docker-install.html



#### 安装 Docker-compose

https://www.runoob.com/docker/docker-compose.html

快速初始化开发环境

```yml
version: '2'

services:
  mysql:
    image: mysql:5.7.36
    hostname: mysql
    restart: always
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root123456
      MYSQL_DATABASE: platform_db
    ports:
      - "3306:3306"
    # 将数据文件和日志文件分别挂载到了本地的目录下，便于数据的持久化存储和管理。
    volumes:
      - /data/platform_db/mysql/data:/var/lib/mysql
      - /data/platform_db/mysql/config/my.cnf:/etc/my.cnf
      - /data/platform_db/mysql/init/:/docker-entrypoint-initdb.d/

  mongodb:
    image: mongo:4.4.6
    container_name: mongodb
    restart: always
    environment:
      MONGO_INITDB_DATABASE: platform_db
    # 将数据文件和日志文件分别挂载到了本地的目录下，便于数据的持久化存储和管理。
    volumes:
      - /data/platform_db/mongodb/data:/data/db
      - /data/platform_db/mongodb/logs:/data/logs
    ports:
      - "27017:27017"

  redis:
    image: redis:5.0.9
    container_name: redis
    command: redis-server /usr/local/etc/redis/redis.conf
    restart: always
    environment:
      REDIS_PASSWORD: redis123456
    ports:
      - "6379:6379"
    # 将数据文件和日志文件分别挂载到了本地的目录下，便于数据的持久化存储和管理。
    volumes:
      - /data/platform_db/redis/data:/data
      - /data/platform_db/redis/redis.conf:/usr/local/etc/redis/redis.conf
```



#### MySQL 服务

https://www.runoob.com/docker/docker-install-mysql.html

```sh
# 拉取
docker pull mysql:5.7.36
# 运行
docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7.36
```



#### MongoDB 服务

https://www.runoob.com/docker/docker-install-mongodb.html

```shell
# 拉取
docker pull mongo:4.4.6
# 运行
docker run -itd --name mongo -p 27017:27017 mongo:4.4.6 --auth
# 初始化用户

```



#### Redis 服务

https://www.runoob.com/docker/docker-install-redis.html

```sh
# 拉取
docker pull redis:5.0.9
# 运行
docker run -itd --name redis-test -p 6379:6379 redis:5.0.9
```



#### 分配账号

```shell
# 添加子账号
adduser username -p password

# 给子账号加权限
 usermod -aG wheel [username]
 sudo chown -R [username]:[username] /home/xmt/
```



