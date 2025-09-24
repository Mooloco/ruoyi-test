# Nginx 安装与配置指南

## 1. 安装 Nginx

### 1.1 确保环境干净
```
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
```

### 1.2 安装依赖
```
yum install -y gcc gcc-c++ make pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

### 1.3 创建用户
```
useradd -M -s /sbin/nologin nginx
```

### 1.4 编译安装
```
# 将 Nginx 压缩包导入 /opt 并解压
cd /opt
tar zxvf nginx-1.20.2.tar.gz
cd nginx-1.20.2/

# 配置编译选项
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module --with-http_ssl_module

# 编译安装
make && make install

# 创建软连接
ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/

# 检查配置文件
nginx -t

# 启动 Nginx
nginx
```

## 2. 添加 Nginx 服务

### 2.1 创建服务脚本
```
vim /etc/init.d/nginx
```
输入内容：
```
#!/bin/bash
# chkconfig: 35 99 20
# description: Nginx Service Control Script

COM="/usr/local/nginx/sbin/nginx"
PID="/usr/local/nginx/logs/nginx.pid"

case "$1" in
start)
    $COM
;;
stop)
    kill -s QUIT $(cat $PID)
;;
restart)
    $0 stop
    $0 start
;;
reload)
    kill -s HUP $(cat $PID)
;;
*)
    echo "Usage: $0 {start|stop|restart|reload}"
    exit 1
esac

exit 0
```

### 2.2 添加权限并启动服务
```
chmod +x /etc/init.d/nginx
chkconfig --add nginx
systemctl start nginx
```

### 2.3 测试
浏览器输入本机 IP 查看 Nginx 页面是否加载。

## 3. 配置页面
```
cd /usr/local/nginx/html
rm -rf index.html
unzip dist
```

## 4. Nginx 配置

### 4.1 编辑配置文件
```
vim /opt/nginx-1.20.2/conf/nginx.conf
```

### 4.2 修改 server 模块
```
server {
    server_name 服务器ip; # 如 1.15.67.191

    location /dist/ {
        root /usr/local/nginx/html;
        index index.html index.htm;
    }
}
```

### 4.3 重启 Nginx
```
systemctl restart nginx.service
```

### 4.4 测试
浏览器输入服务器 IP 验证页面是否加载。

如需平滑升级，请参考[upgrade.md](https://github.com/Mooloco/ruoyi-test/blob/main/ux/upgrade.md)
