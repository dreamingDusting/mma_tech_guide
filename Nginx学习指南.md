# Nginx学习指南

## 前言

Nginx（发音是"engine X"）是一个超级好用的Web服务器和反向代理服务器。说白了，它就是一个"中间人"——帮你转发请求、负载均衡、静态文件服务等等。

协会的官网就是用Nginx来托管的：
- 前端静态文件由Nginx提供服务
- API请求通过Nginx转发到后端
- 还可以用Nginx配置SSL证书，实现HTTPS

这篇文章，带你从零开始搞懂Nginx！

---

## 第一章：Nginx到底是啥？

### 1.1 Nginx能做什么？

Nginx主要有以下几个功能：

**1. Web服务器**
- 静态文件服务（HTML、CSS、JS、图片等）
- 搭建个人网站

**2. 反向代理**
- 把请求转发到后端服务器
- 隐藏真实服务器

**3. 负载均衡**
- 把请求分发到多个服务器
- 提高可用性和性能

**4. 缓存服务器**
- 缓存后端响应
- 减轻后端压力

### 1.2 Nginx vs Apache

| 特性 | Nginx | Apache |
|------|-------|--------|
| 性能 | 高（事件驱动） | 一般（进程/线程） |
| 内存占用 | 低 | 较高 |
| 配置文件 | 简洁 | 复杂 |
| 社区 | 活跃 | 非常活跃 |

Nginx特别适合高并发场景，这也是为什么很多大型网站都用Nginx。

---

## 第二章：安装Nginx

### 2.1 Windows安装

1. 去官网下载：https://nginx.org/en/download.html
2. 解压到某个目录
3. 双击nginx.exe运行
4. 访问 http://localhost 看看是否成功

**常用命令：**
```bash
# 启动
start nginx

# 停止
nginx -s stop

# 重新加载配置
nginx -s reload

# 退出
nginx -s quit
```

### 2.2 Linux安装（Ubuntu）

```bash
# 更新apt
sudo apt update

# 安装Nginx
sudo apt install nginx

# 启动
sudo systemctl start nginx

# 开机自启
sudo systemctl enable nginx

# 检查状态
sudo systemctl status nginx
```

### 2.3 Docker安装（最简单！）

```bash
# 启动Nginx
docker run -d -p 80:80 --name my-nginx nginx

# 挂载本地目录
docker run -d -p 80:80 -v /my/html:/usr/share/nginx/html --name my-nginx nginx
```

---

## 第三章：Nginx基本配置

### 3.1 配置文件位置

- **Ubuntu/Debian**: `/etc/nginx/nginx.conf`
- **CentOS/RHEL**: `/etc/nginx/nginx.conf`
- **Windows**: `安装目录/conf/nginx.conf`

### 3.2 最简单的配置

```nginx
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type application/octet-stream;

    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;

    # 静态文件服务
    server {
        listen 80;
        server_name localhost;

        # 根目录
        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
        }

        # 错误页面
        error_page 404 /404.html;
        error_page 500 502 503 504 /50x.html;
    }
}
```

### 3.3 核心指令解释

- `worker_processes`: 工作进程数，一般设为CPU核心数
- `worker_connections`: 每个进程最大连接数
- `server`: 虚拟主机配置
- `location`: URL路径匹配规则
- `listen`: 监听的端口
- `server_name`: 域名
- `root`: 文件根目录
- `index`: 默认首页文件

### 3.4 配置语法

```nginx
# 注释

# 指令以分号结尾
worker_processes 1;

# 块用大括号
events {
    worker_connections 1024;
}

# 字符串可以不用引号
server_name example.com;

# 路径建议加引号
root "/var/www/html";

# 正则表达式以~开头
location ~ \.php$ {
    # 处理PHP文件
}
```

---

## 第四章：实战配置

### 4.1 静态网站服务

假设你有一个静态网站，放在`/var/www/mma-web`：

```nginx
server {
    listen 80;
    server_name mma.example.com;

    # 网站根目录
    root /var/www/mma-web;
    index index.html index.htm;

    # 访问日志
    access_log /var/log/nginx/mma-access.log;
    error_log /var/log/nginx/mma-error.log;

    # 所有请求
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静态资源缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # 安全：禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
}
```

### 4.2 反向代理——前后端分离

这是最常用的场景！前端请求API时，Nginx把请求转发到后端：

```nginx
upstream backend {
    server 127.0.0.1:8080;
    # 如果有多个后端，可以加
    # server 127.0.0.1:8081;
}

server {
    listen 80;
    server_name mma.example.com;

    # 前端静态文件
    location / {
        root /var/www/mma-web;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # API请求转发
    location /api/ {
        proxy_pass http://backend/api/;
        
        # 设置代理请求头
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # 超时设置
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # 上传文件
    location /upload/ {
        proxy_pass http://backend/upload/;
        client_max_body_size 100m;
    }
}
```

### 4.3 负载均衡

把请求分发到多台服务器：

```nginx
upstream backend {
    # 轮询（默认）
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
    
    # 权重分配
    # server 192.168.1.10:8080 weight=3;
    # server 192.168.1.11:8080 weight=1;
    
    # IP哈希（同一个IP访问同一台服务器）
    # ip_hash;
    
    # 最少连接
    # least_conn;
}

server {
    listen 80;
    server_name mma.example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 4.4 SSL/HTTPS配置

配置SSL证书，实现HTTPS：

```nginx
server {
    listen 443 ssl http2;
    server_name mma.example.com;

    # SSL证书
    ssl_certificate /etc/nginx/ssl/mma.example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/mma.example.com.key;

    # SSL配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # HSTS（强制使用HTTPS）
    add_header Strict-Transport-Security "max-age=31536000" always;

    # 前端
    location / {
        root /var/www/mma-web;
        index index.html;
    }

    # API
    location /api/ {
        proxy_pass http://127.0.0.1:8080/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# HTTP重定向到HTTPS
server {
    listen 80;
    server_name mma.example.com;
    return 301 https://$server_name$request_uri;
}
```

### 4.5 常用location匹配规则

```nginx
# 精确匹配
location = /api/user {
    # 只匹配 /api/user
}

# 前缀匹配（优先匹配）
location ^~ /api/ {
    # 匹配所有以/api/开头的路径
}

# 正则匹配（区分大小写）
location ~ /api/\d+ {
    # 匹配 /api/123 这样的路径
}

# 正则匹配（不区分大小写）
location ~* \.(jpg|jpeg|png)$ {
    # 匹配图片文件
}

# 普通前缀匹配
location /api/ {
    # 匹配所有/api/开头的路径
}
```

### 4.6 协会的实际配置

来看看协会网站的Nginx配置：

```nginx
server {
    listen 80;
    server_name mathmua.top www.mathmua.top;

    # 前端静态文件
    location / {
        root /usr/share/nginx/html/dist;
        index index.html;
        try_files $uri $uri/ /index.html;
        
        # SPA history模式需要
    }

    # API反向代理
    location /api/ {
        proxy_pass http://backend:8080/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 文件上传
    location /files/ {
        alias /uploads/;
        autoindex on;
    }

    # 静态资源
    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }
}
```

---

## 第五章：进阶技巧

### 5.1 Gzip压缩

开启Gzip压缩，减小传输体积，加快加载速度：

```nginx
http {
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;
    gzip_disable "MSIE [1-6]\.";
}
```

### 5.2 防盗链

防止其他网站盗用你的图片：

```nginx
location ~* \.(jpg|jpeg|png|gif)$ {
    valid_referers none blocked mma.example.com;
    if ($invalid_referer) {
        return 403;
    }
}
```

### 5.3 限流

防止恶意请求：

```nginx
# 基于IP的连接限流
limit_conn_zone $binary_remote_addr zone=addr:10m;

server {
    location /api/ {
        limit_conn addr 10;  # 每个IP最多10个连接
        limit_req zone=req_limit burst=20 nodelay;
    }
}

http {
    limit_req_zone $binary_remote_addr zone=req_limit:10m;
}
```

### 5.4 访问控制

```nginx
# 禁止某个IP访问
location / {
    deny 192.168.1.100;
    allow all;
}

# 只允许某个IP段访问
location /admin/ {
    allow 192.168.1.0/24;
    deny all;
}
```

### 5.5 日志分析

```bash
# 查看访问最多的IP
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head

# 查看访问最多的页面
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head

# 查看404错误
grep " 404 " /var/log/nginx/access.log

# 实时查看访问日志
tail -f /var/log/nginx/access.log
```

---

## 第六章：常见问题和调试

### 6.1 测试配置

每次修改配置后，先测试再重载：

```bash
# 测试配置是否有语法错误
nginx -t

# 重新加载配置
nginx -s reload

# 如果不生效，可以停止再启动
nginx -s stop
nginx
```

### 6.2 查看错误日志

```bash
# 查看错误日志
tail -f /var/log/nginx/error.log

# 查看具体某个server的错误
grep "mma.example.com" /var/log/nginx/error.log
```

### 6.3 常用调试命令

```bash
# 查看Nginx进程
ps aux | grep nginx

# 查看监听端口
netstat -tlnp | grep nginx

# 查看系统打开的文件数（定位"too many open files"）
lsof -p $(cat /var/run/nginx.pid) | wc -l
```

---

## 第七章：Docker中运行Nginx

### 7.1 快速启动

```bash
# 启动Nginx
docker run -d -p 80:80 --name nginx nginx

# 挂载本地配置
docker run -d -p 80:80 -v /my/nginx.conf:/etc/nginx/nginx.conf:ro --name my-nginx nginx

# 挂载网站目录
docker run -d -p 80:80 -v /my/html:/usr/share/nginx/html --name my-nginx nginx
```

### 7.2 协会的Docker部署

```yaml
# docker-compose.yaml
nginx:
    image: nginx:latest
    container_name: mma_nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/ssl:/etc/nginx/ssl
      - ./frontend/dist:/usr/share/nginx/html
      - ./uploads:/uploads
    depends_on:
      - backend
    networks:
      - mma-network
```

---

## 附录：常用命令汇总

```bash
# 启动
nginx

# 停止
nginx -s stop

# 优雅停止（处理完请求再停）
nginx -s quit

# 重新加载配置
nginx -s reload

# 测试配置
nginx -t

# 查看版本
nginx -v

# 查看详细版本信息
nginx -V
```

---

## 附录：推荐学习资源

### 官方文档
- Nginx官方文档：http://nginx.org/en/docs/
- 中文文档：https://www.nginx.cn/doc/

### 实践建议
1. 在本地用Docker跑Nginx玩一玩
2. 尝试配置协会网站的Nginx
3. 试着配置HTTPS

---

## 结语

Nginx是每个后端开发者都必须掌握的技术！学完这篇，你应该能够：

- 理解Nginx的工作原理
- 配置静态网站服务
- 配置反向代理
- 配置负载均衡
- 配置SSL/HTTPS
- 用Docker部署Nginx

动手试试吧，有问题随时问~
