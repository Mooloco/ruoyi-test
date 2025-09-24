# Nginx 平滑升级指南

## 1. 导入升级包
```
# 将升级包导入 /opt
cd /opt
tar -zxvf nginx-1.22.0.tar.gz
```

## 2. 检查已安装模块
```
nginx -V
```

## 3. 配置新版本 Nginx
```
cd /opt/nginx-1.22.0/
./configure \
--prefix=/usr/local/nginx \
--user=nginx \
--group=nginx \
--with-http_stub_status_module \
--with-http_ssl_module \
--add-module=/opt/nginx-module-vts-0.2.4/
```

## 4. 编译源码
```
make
```

## 5. 备份现有 Nginx 二进制文件
```
mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
```

## 6. 替换为新版本二进制文件
```
cp /opt/nginx-1.22.0/objs/nginx /usr/local/nginx/sbin/
```

## 7. 平滑升级操作
```
# 发送平滑升级信号
kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`

# 关闭旧版本 Nginx 进程
kill -WINCH `cat /usr/local/nginx/logs/nginx.pid.oldbin`

# 重新加载配置
kill -HUP `cat /usr/local/nginx/logs/nginx.pid.oldbin`

# 终止旧版工作进程
kill -QUIT `cat /usr/local/nginx/logs/nginx.pid.oldbin`

# 重新加载新版本 Nginx 进程
kill -HUP `cat /usr/local/nginx/logs/nginx.pid`
```

## 8. 验证升级
```
nginx -V
```