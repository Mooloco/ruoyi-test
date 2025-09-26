# 后端部署文档

本项目的 **backend** 文件夹存放后端相关文件。请将其克隆至您的 Linux 服务器，例如 `/opt` 目录。

---

## 1. 后端程序说明
- `ruoyi-admin.jar`：基于 Java 运行的后端程序。

---

## 2. 安装并检查 JDK（192.168.1.25）
后端依赖 Java 环境，最低要求为 **JDK 1.8**。

### 检查 JDK 版本
```bash
java -version
```

如果提示命令不存在，表示尚未安装 JDK。

### 安装 JDK（以 CentOS7 为例）
```bash
yum claean all && yum makecache
yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel -y
```

安装完成后再次执行：
```bash
java -version
```
若输出版本号 ≥ 1.8，则说明 JDK 环境已准备就绪。

---

## 3. 安装并配置 MySQL（192.168.1.22）
请在 `192.168.1.22` 上部署 **MySQL 5.7** 数据库。  
安装方法可参考 👉 [MySQL 安装教程](https://www.mooloco.com/?p=106)

---

## 4. 创建数据库并导入数据
1. 登录 MySQL：
   ```bash
   mysql -u root -p
   ```

2. 创建数据库：
   ```sql
   CREATE DATABASE ry_vue;
   ```

3. 导入数据：
   ```bash
   mysql -u root -p ry_vue < ry_20250522.sql
   mysql -u root -p ry_vue < quartz.sql
   ```

---

## 5. 授权数据库访问
在 MySQL 中执行以下命令，允许 `192.168.1.25` 的后端服务访问数据库：

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.1.25' IDENTIFIED BY '123456';
FLUSH PRIVILEGES;
```

---

## 6. 部署 Redis（192.168.1.116）

后端需要使用 Redis，请在 `192.168.1.116` 上安装并配置 Redis。

### 1. 安装流程

```bash
# 关闭防火墙与 SELinux
systemctl stop firewalld
setenforce 0

# 安装依赖
yum install -y gcc gcc-c++ make

# 下载并解压 Redis 源码包（以 5.0.7 为例）
tar zxvf redis-5.0.7.tar.gz -C /opt/
cd /opt/redis-5.0.7/

# 编译与安装
make && make PREFIX=/usr/local/redis install

# 执行安装脚本进行初始化配置
cd /opt/redis-5.0.7/utils
./install_server.sh
# 提示时请将 redis-server 路径设置为 /usr/local/redis/bin/redis-server
```

初始化完成后，会显示以下配置：

- **Port**: 默认端口 6379  
- **Config file**: `/etc/redis/6379.conf`  
- **Log file**: `/var/log/redis_6379.log`  
- **Data dir**: `/var/lib/redis/6379`  
- **Executable**: `/usr/local/redis/bin/redis-server`  
- **Cli Executable**: `/usr/local/bin/redis-cli`  

将可执行程序加入 PATH 方便调用：
```bash
ln -s /usr/local/redis/bin/* /usr/local/bin/
```

### 2. 服务管理

```bash
/etc/init.d/redis_6379 start      # 启动
/etc/init.d/redis_6379 stop       # 停止
/etc/init.d/redis_6379 restart    # 重启
/etc/init.d/redis_6379 status     # 查看状态
```

### 3. 配置文件调整

编辑 `/etc/redis/6379.conf`：

```conf
bind 127.0.0.1 192.168.1.116   # 允许远程访问
port 6379                      # 默认端口
daemonize yes                  # 守护进程模式
pidfile /var/run/redis_6379.pid
loglevel notice
logfile /var/log/redis_6379.log
# requirepass 123456           # 可选，设置访问密码
```

修改后重启服务：
```bash
/etc/init.d/redis_6379 restart
```


## 7. 启动后端服务（192.168.1.25）
进入 `ruoyi-admin.jar` 所在目录，运行：

```bash
java -jar ruoyi-admin.jar
```

如果控制台出现 **RuoYi 启动成功** 的提示，则说明后端部署完成 🎉。

---
