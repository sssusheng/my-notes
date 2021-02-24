docker
```json
// 查找镜像
docker search xxx:x.x // :后接版本 默认latest
// 下载镜像
docker pull xxx


// 创建容器并启动
docker run 
  --name xxx // 指定名称
  -p // 指定端口映射 主机：容器
  -e // 设置环境变量
  -m // 设置内存使用最大值
  -d // 后台运行容器，并返回容器ID
  -v // 绑定一个卷
  -it // 交互式运行容器并分配一个伪终端
// 创建容器不启动
docker create --name xxx

// 容器启动、停止、重启
docker start/stop/restart name|id
// 容器暂停、恢复
docker pause/unpause name|id
// 杀死容器、删除容器
docker kill name|id
docker rm 
  -f // 强制删除容器
  -v // 删除与容器相关的卷
  -l // 移除容器间的网络连接而非容器本身
  name|id
// 容器重命名
docker rename old_name new_name

// 查看运行中的容器
docker ps
  -a // 查看所有容器

// 查看已有镜像
docker images

// 从指定容器创建一个新的镜像
// commit创建的镜像，不包含原有容器的数据
docker commit 
  -a "sunke" // 作者
  -m "xxx" // 提交说明
  -p // 在创建的时候将容器暂停

/**
有些服务是需要修改配置文件的，常用的做法有：
1.使用docker cp 指令 把配置文件拷贝出来，修改完成后再替换回去
2.在docker run 创建容器时 用 -v 参数 把容器中配置文件的目录挂载到本地目录里
*/
```


MySQL
```json
// mysql

// 1.下载mysql镜像
docker pull mysql

// 2.初次启动mysql镜像（创建一个新容器并启动）
docker run 
  --name mysql-xxx // 容器名字
  -p 3306:3306 // 端口映射 主机：容器
  -e MYSQL_ROOT_PASSWORD=test // 账密
  -d 
  mysql // 使用的镜像名称

```

MongoDB
```json
// mongoDB

// 1.下载最新版mongo
docker pull mongo:latest

// 2.首次启动
docker run 
  -it
  -d
  --name mymongo
  -p 27017:27017
  mongo
  --auth // 启用密码权限认证

// 3.进入容器终端
docker exec 
  -it mymongo mongo admin
// 4.创建admin用户，并对该用户赋予admin数据库的userAdmin权限和所有数据库的读写权限
> db.createUser({ user: "admin", pwd: "123456", roles:[{ role:"userAdminAnyDatabase", db: "admin"},"readWriteAnyDatabase"]})
> db.auth("admin", "123456") // 对admin账号授权
// 5.切换到environment数据库（无论是否存在）
> use environment
// 6.在environment数据库下创建用户
> db.createUser({ user: "environment", pwd: "123456", roles:[{ role:"readWrite", db: "environment"}]})
```

PostgreSQL
```json
// 1.个人最常用的是9.6版本
docker pull postgres:9.6

// 2.启动并创建容器
docker run
  --name postgresql-9.6
  -e POSTGRES_PASSWORD=postgres // 设置默认密码
  -e ALLOW_IP_RANGE=0.0.0.0/0 // 允许所有ip访问
  -p 5432:5432
  -d // 后台运行
  postgres:9.6

// 进入容器终端
docker exec -it postgresql-9.6 bash
exit // 退出终端

// 3.配置远程访问许可 需要修改 pg_hba.conf 和 postgresql.conf 文件所在位置
/var/lib/postgresql/data/pg_hba.conf
/var/lib/postgresql/data/postgresql.conf

// 4.重启容器，还需要修改宿主机的防火墙配置，才能实现外部访问
systemctl status firewalld
// 防火墙未启动则启动
systemctl start firewalld
// 检查 firewall-cmd 运行状态是否正常
firewall-cmd --state
// 配置外部访问端口 permanent永久的
firewall-cmd --zone=public --add-port=5432/tcp --permanent
firewall-cmd reload
systemctl stop firewalld.service
systemctl start firewalld.service
```

nginx
```json
// 1.下载最新版镜像
docker pull nginx

// 2.需要启动一下，把配置文件拷贝出来后，再把这个临时容器删除，如果有相应的配置文件，这一步可以省略
docker run 
  --name nginx-temp
  -p 8080:80
  -d 
  nginx

// 拷贝配置文件到本地指定目录下
docker cp nginx-temp:/etc/nginx/nginx.conf /data/home/sunke/Tools/nginx/conf
docker cp nginx-temp:/etc/nginx/conf.d/default.conf /data/home/sunke/Tools/nginx/conf

// 把这个临时容器强制删除
docker rm -f nginx-temp

// 3.启动一个新容器，指定容器内nginx挂载的卷
docker run
  -it
  -d
  --name mynginx
  -p 9999:5000 // 5000是nginx容器监听的端口
  -p 9998:5001 // 如果有多个服务，可以配置多端口映射
  -v /data/home/sunke/Tools/nginx/html:/usr/share/nginx/html // 挂载资源
  -v /data/home/sunke/Tools/nginx/conf/nginx.conf:/etc/nginx/nginx.conf // 挂载配置文件
  -v /data/home/sunke/Tools/nginx/conf/default.conf:/etc/nginx/conf.d/default.conf // 挂载配置文件
  -v /data/home/sunke/Tools/nginx/logs:/var/log/nginx // 挂载日志文件
  nginx // 镜像名

// nginx的基础配置在nginx.conf文件里，代理配置在default.conf文件里
// 也可以不复制配置文件，不挂载资源，直接进入容器终端中修改
docker exec -it mynginx bash
// 退出容器终端
exit

// 4.修改nginx配置文件后重启nginx服务
docker restart mynginx
```