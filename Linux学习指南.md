# Linux学习指南

## 前言

其实，单纯做前端、后端的开发调试，用 Windows 电脑完全够用。那为什么我们还要专门写一份 Linux 指南呢？

因为一旦涉及**部署**——比如 Docker 环境、云服务器——基本就是 Linux 的“主场”了。这一章，我会带你入门 Linux 操作系统，**不需要你成为运维大神，只要能看懂命令行、会基本操作就够了**。

***

## 第一章：Linux简介

### 1.1 什么是Linux？

Linux是一个操作系统，跟Windows、macOS一样。但是它更牛X的地方在于：

- **开源免费**：谁都可以用，谁都可以改
- **超级稳定**：很多服务器都用它，一年不关机都没问题
- **无处不在**：你手机安卓系统底层就是Linux，抖音、微信、百度的服务器也是Linux
- **程序员必备**：不会Linux都不好意思说自己是搞技术的

协会的Docker容器、云服务器，用的都是Linux环境。

### 1.2 为什么要学Linux？

可能你会问："我平时用Windows不也挺好吗？"

好问题！学Linux的原因很简单：

1. **Docker运行环境**：Docker容器在Linux上运行最原生、最稳定
2. **云服务器**：阿里云、腾讯云的服务器基本都是Linux
3. **部署上线**：项目上线的时候，绝大多数都是部署在Linux服务器上
4. **逼格满满**：敲命令行多帅啊（bushi

### 1.3 常见的Linux发行版

Linux不是只有一个"Linux"，它有很多"发行版"——就像手机有华为、小米、OPPO一样。下面列举几个最常见的：

**新手友好型：**

- **Ubuntu**：最流行的Linux之一，界面漂亮，文档丰富，推荐！
- **Linux Mint**：更简单，适合从Windows过渡

**服务器专用型：**

- **CentOS / Rocky Linux / AlmaLinux**：企业服务器常用，稳定
- **Debian**：以稳定著称，很多云服务器镜像用这个
- **Alpine**：非常轻量，Docker镜像很多基于这个

**我们协会怎么选？**

- Docker容器里：推荐用Ubuntu或者Alpine
- 云服务器：推荐Ubuntu或者CentOS/Rocky Linux

***

## 第二章：Linux基本命令

### 2.1 目录操作

Linux的目录结构跟Windows不太一样。没有C盘D盘，而是从"/"（根目录）开始。

```bash
# 查看当前所在目录
pwd

# 列出当前目录的文件
ls

# 列出详细信息（包括隐藏文件）
ls -la

# 切换目录
cd /home          # 绝对路径
cd ./folder      # 相对路径
cd ..            # 返回上一级
cd ~             # 回到用户主目录

# 创建目录
mkdir myfolder

# 删除目录
rm -rf myfolder   # 递归强制删除（慎用！）
```

### 2.2 文件操作

```bash
# 查看文件内容
cat filename          # 一次性显示全部
head -n 10 filename   # 查看前10行
tail -n 10 filename   # 查看后10行
more filename         # 分页查看（空格翻页）
less filename         # 更强大的分页查看

# 创建文件
touch filename        # 创建空文件
echo "内容" > file   # 写入内容（覆盖）
echo "内容" >> file  # 追加内容

# 复制/移动/删除
cp source dest        # 复制
mv oldname newname    # 移动/重命名
rm filename          # 删除文件
```

### 2.3 文本编辑器

在Linux上编辑文件是家常便饭，推荐学一个命令行编辑器：

**vim（推荐）**

```bash
# 安装vim
apt install vim    # Ubuntu/Debian
yum install vim    # CentOS

# 打开文件
vim filename

# 常用操作：
# i - 进入编辑模式
# Esc - 退出编辑模式
# :w - 保存
# :q - 退出
# :wq - 保存并退出
# :q! - 强制退出不保存
# /keyword - 搜索关键词
```

**nano（更简单）**

```bash
# 安装
apt install nano

# 打开
nano filename

# 底部有快捷键提示，Ctrl+O保存，Ctrl+X退出
```

### 2.4 权限管理

Linux的权限系统很重要，每个文件都有"谁可以读/写/执行"的设置。

```bash
# 查看权限
ls -l filename
# 结果类似：-rwxr-xr-x 1 user group 1234 Jan 1 12:00 filename
# 解释：-文件类型 所有者权限 所属组权限 其他人权限

# 修改权限
chmod 755 filename    # 数字方式
chmod +x filename    # 添加执行权限
chmod -x filename    # 移除执行权限

# 修改所有者
chown user:group filename
```

### 2.5 进程管理

```bash
# 查看进程
ps aux              # 查看所有进程
top                 # 实时查看系统资源占用

# 杀掉进程
kill pid            # 正常杀掉
kill -9 pid         # 强制杀掉

# 后台运行
command &           # 后台运行
Ctrl+Z              # 暂停当前进程
bg                  # 后台继续
fg                  # 回到前台
```

### 2.6 网络命令

```bash
# 查看IP
ip addr
ifconfig

# 测试网络连通
ping baidu.com

# 查看端口占用
netstat -tulpn
lsof -i :8080

# 下载文件
curl -O url
wget url
```

### 2.7 系统信息

```bash
# 查看系统版本
cat /etc/os-release

# 查看内存使用
free -h

# 查看磁盘使用
df -h

# 查看CPU信息
lscpu

# 查看运行时间
uptime
```

### 2.8 软件安装

```bash
# Ubuntu/Debian
apt update              # 更新软件源
apt install vim         # 安装软件
apt remove vim          # 卸载
apt upgrade             # 升级所有软件

# CentOS/Rocky Linux
yum update              # 更新
yum install vim         # 安装
yum remove vim          # 卸载
```

***

## 第三章：实用技巧

### 3.1 常用快捷键

```bash
Ctrl+C          # 取消当前命令
Ctrl+Z          # 暂停当前进程
Ctrl+L          # 清屏（跟clear一样）
Ctrl+A          # 移动到行首
Ctrl+E          # 移动到行尾
Ctrl+U          # 清除当前行
Tab             # 自动补全（多按两下看看有啥）
```

### 3.2 管道和重定向

这是Linux最强大的地方之一：

```bash
# 管道：把上一个命令的输出传给下一个
ls | grep keyword        # 列出包含keyword的文件
cat file | wc -l         # 统计文件行数

# 重定向：改变输入输出方向
command > file           # 输出重定向（覆盖）
command >> file          # 输出重定向（追加）
command 2> error.log    # 错误输出重定向
command > output.log 2>&1  # 标准输出和错误都保存
```

### 3.3 SSH远程连接

如果你的云服务器是Linux，那就要用SSH连接：

```bash
# 连接远程服务器
ssh username@ip地址
ssh -p 22 username@ip   # 指定端口

# 常用SSH工具：
# - Windows: Xshell, PuTTY, Termius
# - Mac: 直接用终端
# - VS Code: Remote-SSH插件

# 复制文件到服务器
scp localfile user@ip:/path/
scp -r folder user@ip:/path/
```

### 3.4 文件传输

```bash
# 本地上传/服务器下载
# 用的命令是 scp（上文有）或者 rz/sz

# 如果服务器装了 lrzsz：
rz                      # 从本地上传到服务器
sz filename            # 从服务器下载到本地
```

***

## 第四章：实战——在Linux上运行Docker

说了这么多，来个实际的！让我们在Linux上跑一个Docker：

```bash
# 1. 更新系统
apt update && apt upgrade -y

# 2. 安装Docker
apt install docker.io

# 3. 启动Docker
systemctl start docker
systemctl enable docker   # 开机自启

# 4. 拉取并运行MySQL
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -e MYSQL_DATABASE=mma \
  -p 3306:3306 \
  mysql:8.0

# 5. 查看运行状态
docker ps

# 6. 进入容器看看
docker exec -it mysql bash
```

简单吧！一行命令，MySQL就跑起来了\~

***

## 第五章：协会项目中的Linux

协会的很多服务都跑在Linux上：

- **Docker容器**：本质上就是运行在Linux上的
- **云服务器**：协会的官网就是部署在Linux服务器上

所以，学会Linux之后，你就可以：

- 自己搭建各种服务
- 部署协会的项目
- 玩转Docker和云服务器

***

## 附录：推荐学习资源

### 在线教程

- 菜鸟教程Linux：<https://www.runoob.com/linux/>
- Linux命令大全：<https://www.linuxcool.com/>

### 实践平台

- 阿里云ECS：第一年很便宜
- 腾讯云学生机：很便宜
- VirtualBox：在自己电脑上装Linux虚拟机

### 视频教程

- Bilibili搜索"Linux入门"、"鸟哥Linux"有很多优质教程

***

## 结语

Linux入门其实不难，关键是多动手、多敲命令。刚开始可能会觉得麻烦，但敲多了就熟练了。

建议的学习路径是：

1. 在自己电脑上装个Linux虚拟机（VirtualBox+Ubuntu）
2. 每天玩一玩，把常用命令练熟
3. 尝试在Linux上安装Docker，跑跑MySQL、Redis
4. 申请个云服务器，部署个项目试试

协会的服务器就是Linux，学好这个，以后部署项目、运维服务都不怕！

加油！🐧
