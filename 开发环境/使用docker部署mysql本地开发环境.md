本文介绍如何使用docker在本地搭建一个简单的mysql开发环境。

# 准备工作
使用本文用例需要预先安装好docker，windows用户直接下载windows版本的docker客户端即可，使用linux容器或windows容器都不影响结果。

# 启动mysql
运行命令：
```bash
$ docker run -p 3308:3306 -v /home/dev-enviroment/:/var/lib/mysql -d -e MYSQL_ROOT_PASSWORD=1234 mysql
```
这句命令的意思是，使用docker启动一个名为mysql的容器，如果本地没有这个容器，则从远程仓库中寻找，容器启动后，将容器内的3306端口映射到宿主机的3308端口，并将容器内的`/var/lib/mysql`目录映射到宿主机的`/home/dev0enviroment`目录，`-d`表示在后台运行，容器启动后仅返回容器id，`-e`表示配置环境变量

注意：windows用户使用cmd, powershell或者gitbash等不用的命令行工具时，需要输入的路径可能不同，视终端和容器类型而定，可以尝试`d:/dev`或者`/d/dev`等形式。

# 配置远程访问
对于容器内服务来说，宿主机是一台远程主机，需要修改配置以支持数据库连接。
```bash
$ docker exec -it 01873d bash # 以交互模式进入容器，并运行bash
# windows用户需要以管理员身份运行，如果在cmd中无法进入，可尝试使用git bash等工具

# 进入mysql命令行
$ mysql -uroot -p1234
```
```sql
-- 允许远程连接
GRANT ALL ON *.* TO 'root'@'%';
-- 更新权限配置
flush privileges;
-- 修改加密模式（navicat等不支持默认模式）
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '1234';
flush privileges;
```

现在打开navicat等工具，连接`localhost:3308`即可访问数据库了。

如果docker被关闭，只需要运行以下命令就可以恢复数据库服务：
```bash
$ docker ps -a
# 从列出的容器列表中找出mysql的容器id
$ docker restart id
```
