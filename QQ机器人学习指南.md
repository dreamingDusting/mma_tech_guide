# QQ机器人开发学习指南

## 前言

嗨！欢迎来到QQ机器人开发的行列！有没有觉得QQ群里那个叫"小数"的机器人很智能？想知道它是怎么实现的吗？这篇指南就教你从零开始写QQ机器人插件！

咱们协会用的是**AstrBot**框架，这是一个Python编写的QQ机器人框架。相比其他框架，AstrBot更容易上手，文档也比较完善，特别适合新手。

这篇指南会先带你了解机器人是怎么工作的，然后手把手教你写插件。学完这篇，你也能写出酷炫的机器人功能！

***

## 第一章：初识AstrBot

### 1.1 什么是AstrBot？

AstrBot是一个开源的一站式Agentic个人和群聊助手框架。它不仅支持QQ，还支持Telegram、企业微信、飞书、钉钉、Slack等数十款主流即时通讯软件。此外还内置了类似OpenWebUI的轻量化ChatUI，为个人、开发者和团队打造可靠、可扩展的对话式智能基础设施。

AstrBot的特点：

- **多平台支持**：支持QQ、Telegram、企业微信、飞书、钉钉、Slack等18+主流IM平台
- **AI能力集成**：支持各种AI模型接入，可以接入Dify、Coze、阿里云百炼应用等
- **插件系统完善**：灵活的插件机制，易于扩展
- **Python编写**：对新手友好
- **开源免费**：受AGPL-v3开源许可证保护

无论是个人AI伙伴、智能客服、自动化助手，还是企业知识库，AstrBot都能在你的即时通讯软件平台的工作流中快速构建AI应用。

### 1.2 机器人的工作原理

在说代码之前，先来了解一下机器人是怎么"跑"起来的：

```
用户发送消息 → QQ服务器 → CQHTTP/OneBot客户端 → AstrBot框架 → 你的插件 → 处理业务逻辑 → 返回响应 → AstrBot → 客户端 → QQ服务器 → 用户收到消息
```

简单来说就是：

1. 用户在群里发消息
2. 消息通过一系列"中间商"传到你的插件
3. 你的插件处理消息，返回结果
4. 结果再通过"中间商"发回给用户

### 1.3 安装和环境准备

**前置要求：**

- Python 3.8+
- 一个QQ小号（做机器人账号）

**安装AstrBot（和NapCat一起部署）：**

```bash
mkdir astrbot

cd astrbot

wget https://raw.githubusercontent.com/NapNeko/NapCat-Docker/main/compose/astrbot.yml

sudo docker compose -f astrbot.yml up -d
```

启动后访问 <http://localhost:8080> 就能看到管理界面了。

当然，如果你只是想学习写插件，可以直接看我们协会的代码，看完你就能自己写了，不一定要自己搭一个机器人。

***

## 第二章：Python基础

因为AstrBot是用Python写的，所以我们得先学点Python。不用学太深，了解基础就行。

### 2.1 变量和数据类型

```python
# 字符串
name = "小数"
greeting = '你好！'

# 数字
age = 18
price = 19.99

# 布尔值
is_student = True
is_admin = False

# 列表（类似数组）
hobbies = ["coding", "games", "music"]

# 字典（类似对象）
user = {
    "name": "张三",
    "age": 20,
    "qq": 123456789
}

# 元组（不可变的列表）
position = (10, 20)
```

### 2.2 条件判断

```python
age = 18

if age >= 18:
    print("成年了")
elif age >= 12:
    print("青少年")
else:
    print("小朋友")

# 三元运算符
status = "成年" if age >= 18 else "未成年"
```

### 2.3 循环

```python
# for循环
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

# 遍历列表
hobbies = ["coding", "games", "music"]
for hobby in hobbies:
    print(hobby)

# while循环
count = 0
while count < 5:
    print(count)
    count += 1
```

### 2.4 函数

```python
# 简单函数
def say_hello(name):
    return f"你好，{name}！"

# 带默认参数的函数
def greet(name, greeting="你好"):
    return f"{greeting}，{name}！"

# 调用
print(say_hello("小明"))  # 你好，小明！
print(greet("小红", "早上好"))  # 早上好，小红！
```

### 2.5 类和对象

Python是面向对象的语言，虽然写插件不一定要用类，但看协会的代码需要懂一些：

```python
# 定义类
class Student:
    def __init__(self, name, age):
        self.name = name  # 属性
        self.age = age
    
    def study(self):  # 方法
        print(f"{self.name}正在学习")
    
    def introduce(self):
        print(f"我叫{self.name}，今年{self.age}岁")

# 创建对象
student = Student("张三", 20)
student.study()
student.introduce()
```

### 2.6 常用数据结构操作

```python
# 列表操作
numbers = [1, 2, 3, 4, 5]
numbers.append(6)  # 添加
numbers.pop()      # 删除最后一个
print(numbers[0])   # 访问第一个元素

# 字典操作
user = {"name": "张三", "age": 20}
print(user["name"])  # 获取值
user["qq"] = 123456  # 添加键值对
user.pop("age")      # 删除

# 字符串操作
msg = "Hello World"
print(msg.upper())   # 转大写
print(msg.lower())   # 转小写
print(msg.split(" "))  # 分割
print(",".join(["a", "b", "c"]))  # 拼接
```

***

## 第三章：AstrBot插件开发

### 3.1 插件的基本结构

一个AstrBot插件通常包含以下文件：

```
my_plugin/
├── __init__.py      # 插件入口
├── main.py          # 主要逻辑
├── metadata.yaml    # 插件信息
└── config.json      # 配置文件（可选）
```

来看看咱们协会的插件结构：

```
plugins/
├── astrbot_plugin_mma_chat/       # 聊天插件
│   ├── main.py
│   ├── chat.py
│   ├── database.py
│   ├── model.py
│   └── metadata.yaml
└── astrbot_plugin_mma_secretariat/  # 秘书处插件
    ├── main.py
    ├── secretariat.py
    ├── task_database.py
    └── metadata.yaml
```

### 3.2 第一个插件——Hello World

让我们来写一个最简单的插件！

**1. metadata.yaml（插件信息）：**

```yaml
name: hello_plugin
author: 你的名字
description: 一个简单的打招呼插件
version: 1.0.0
```

**2. main.py（主逻辑）：**

```python
from astrbot.api.event import filter, AstrMessageEvent
from astrbot.api.star import Context, Star, register

@register("hello_plugin", "作者名", "插件描述", "1.0.0")
class HelloPlugin(Star):
    def __init__(self, context: Context):
        super().__init__(context)
    
    # 监听群消息
    @filter.event_message_type(filter.EventMessageType.GROUP_MESSAGE)
    async def on_group_message(self, event: AstrMessageEvent):
        # 获取发送者的QQ号
        user_id = event.get_sender_id()
        
        # 获取消息内容
        message = event.message_obj.raw_message
        
        # 如果有人说了"你好"
        if "你好" in message:
            yield event.plain_result(f"你好呀！我是机器人！")
```

**3. 插件装饰器详解：**

AstrBot用装饰器来定义命令，常用的有：

```python
# 监听所有群消息
@filter.event_message_type(filter.EventMessageType.GROUP_MESSAGE)

# 监听私聊消息
@filter.event_message_type(filter.EventMessageType.PRIVATE_MESSAGE)

# 定义命令（触发关键词）
@filter.command("hello", alias={"你好", "hello"})
async def hello_command(self, event: AstrMessageEvent):
    yield event.plain_result("你好！")
```

### 3.3 命令系统详解

协会的插件用了很多命令装饰器，来看看用法：

```python
# 基本命令
@filter.command("hello", alias={"你好"}, priority=1)
async def hello(self, event: AstrMessageEvent):
    """这是帮助文档"""
    yield event.plain_result("你好！我是小数")

# 带参数的命令
@filter.command("add", priority=1)
async def add_numbers(self, event: AstrMessageEvent, a: int, b: int):
    result = a + b
    yield event.plain_result(f"{a} + {b} = {result}")
```

使用示例：

```
/hello
你好
/add 3 5
```

来看看咱们协会的实际用法（秘书处插件）：

```python
# main.py
@filter.command("create_task", alias={'创建任务'}, priority=1)
async def create_task(self, event: AstrMessageEvent, content: str, deadline: str, department: str):
    raw_message = event.message_obj.raw_message
    user_id = raw_message.user_id
    
    # 验证时间格式
    is_valid, deadline = datetime_validator.validate_and_convert(deadline)
    if not is_valid:
        yield event.plain_result("时间格式错误")
        return
    
    # 验证部门
    if not secretariat.is_valid_department(department):
        yield event.plain_result("部门格式错误")
        return
    
    # 创建任务
    create_task_result = secretariat.create_task(user_id, content, deadline, department)
    if create_task_result['success']:
        task_id = create_task_result['task_id']
        get_task_by_id_result = secretariat.get_task_by_id(user_id, task_id)
        if get_task_by_id_result['success']:
            yield event.plain_result(f"创建任务成功\n{get_task_by_id_result['task']}")
    else:
        yield event.plain_result(f"{create_task_result['message']}")
```

使用方式：

```
/创建任务 完成活动策划 2026-12-31T23:59:00 组织部
```

### 3.4 消息处理

**1. 获取消息内容：**

```python
@filter.event_message_type(filter.EventMessageType.GROUP_MESSAGE)
async def on_message(self, event: AstrMessageEvent):
    raw_message = event.message_obj.raw_message  # 原始消息字符串
    user_id = event.get_sender_id()  # 发送者QQ号
    group_id = event.get_group_id()  # 群号
```

**2. 发送回复：**

```python
# 纯文本回复
yield event.plain_result("这是回复内容")

# 使用MessageChain（可以发图片、at人等）
from astrbot.api.message_components import Comp

chain = [
    Comp.At(qq=123456789),  # @某人
    Comp.Plain("欢迎新朋友！")
]
yield event.chain_result(chain)

# 发送图片
chain = [
    Comp.Image.fromURL("https://example.com/image.jpg"),  # 从URL
    Comp.Image.fromFileSystem("path/to/image.jpg")  # 从本地文件
]
yield event.chain_result(chain)
```

来看看咱们协会的用法：

```python
# astrbot_plugin_mma_chat/main.py
@filter.event_message_type(filter.EventMessageType.PRIVATE_MESSAGE)
async def on_private_message(self, event: AstrMessageEvent):
    raw_message = event.message_obj.raw_message
    if raw_message.message != None:
        if self.is_command(raw_message.raw_message):
            return
        
        logger.info(raw_message)
        yield event.plain_result("让小数窝想一想捏")
        
        # 调用聊天函数
        reply = chat.at_chat(raw_message)
        
        # 构建回复消息链
        chain = [
            Comp.Plain(reply)
        ]
        yield event.chain_result(chain)
        database.chat_history_assistant_add(reply, raw_message)
```

### 3.5 权限管理

协会的插件有权限控制，看看怎么实现的：

```python
# 权限白名单
ROOT_WHITELIST = [xxx,yyy,...]  # 超级管理员
ADMIN_WHITELIST = [xxx,yyy,zzz,...]  # 管理员

@staticmethod
def _check_root_permission(qq_number: int) -> bool:
    return qq_number in secretariat.ROOT_WHITELIST

@staticmethod
def _check_permission(qq_number: int) -> bool:
    return qq_number in secretariat.ADMIN_WHITELIST
```

然后在命令里使用：

```python
@filter.command("create_task", alias={'创建任务'}, priority=1)
async def create_task(self, event: AstrMessageEvent, ...):
    raw_message = event.message_obj.raw_message
    user_id = raw_message.user_id
    
    # 检查权限
    if not secretariat._check_permission(user_id):
        yield event.plain_result("权限不足，只有管理员可以创建任务")
        return
    
    # 继续处理...
```

***

## 第四章：数据库存储

有些数据需要持久化保存，比如聊天记录、任务列表等，这就需要用到数据库。

### 4.1 SQLite简介

AstrBot默认使用SQLite，这是一个轻量级的数据库，不需要单独安装服务器，直接用文件存储，特别适合小型项目。

协会的秘书处插件用的是SQLite：

```python
# task_database.py
import sqlite3
from .task import Task

class TaskDatabase:
    def __init__(self, db_path="database/task_database.db"):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        """初始化数据库表"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS tasks (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                content TEXT NOT NULL,
                deadline TEXT NOT NULL,
                department TEXT NOT NULL,
                status TEXT DEFAULT 'pending',
                response_received INTEGER DEFAULT 0,
                created_time TEXT,
                updated_time TEXT,
                completion_time TEXT
            )
        ''')
        
        conn.commit()
        conn.close()
    
    def create_task(self, content: str, deadline: str, department: str):
        """创建新任务"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        from datetime import datetime
        created_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        
        cursor.execute(
            'INSERT INTO tasks (content, deadline, department, created_time) VALUES (?, ?, ?, ?)',
            (content, deadline, department, created_time)
        )
        
        task_id = cursor.lastrowid
        conn.commit()
        conn.close()
        
        return task_id
    
    def get_all_tasks(self):
        """获取所有任务"""
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row
        cursor = conn.cursor()
        
        cursor.execute('SELECT * FROM tasks ORDER BY created_time DESC')
        rows = cursor.fetchall()
        conn.close()
        
        return [Task(dict(row)) for row in rows]
    
    def get_task_by_id(self, task_id):
        """根据ID获取任务"""
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row
        cursor = conn.cursor()
        
        cursor.execute('SELECT * FROM tasks WHERE id = ?', (task_id,))
        row = cursor.fetchone()
        conn.close()
        
        return Task(dict(row)) if row else None
    
    def update_task(self, task_id, content=None, deadline=None, department=None):
        """更新任务"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        from datetime import datetime
        updated_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        
        updates = []
        params = []
        
        if content:
            updates.append("content = ?")
            params.append(content)
        if deadline:
            updates.append("deadline = ?")
            params.append(deadline)
        if department:
            updates.append("department = ?")
            params.append(department)
        
        updates.append("updated_time = ?")
        params.append(updated_time)
        
        params.append(task_id)
        
        cursor.execute(f"UPDATE tasks SET {', '.join(updates)} WHERE id = ?", params)
        
        conn.commit()
        success = cursor.rowcount > 0
        conn.close()
        
        return success
    
    def complete_task(self, task_id):
        """完成任务"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        from datetime import datetime
        completion_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        
        cursor.execute(
            'UPDATE tasks SET status = ?, completion_time = ? WHERE id = ?',
            ('completed', completion_time, task_id)
        )
        
        conn.commit()
        success = cursor.rowcount > 0
        conn.close()
        
        return success
    
    def delete_task(self, task_id):
        """删除任务"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute('DELETE FROM tasks WHERE id = ?', (task_id,))
        
        conn.commit()
        success = cursor.rowcount > 0
        conn.close()
        
        return success
    
    def get_statistics(self):
        """获取统计数据"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        # 总任务数
        cursor.execute('SELECT COUNT(*) FROM tasks')
        total = cursor.fetchone()[0]
        
        # 进行中
        cursor.execute("SELECT COUNT(*) FROM tasks WHERE status = 'pending'")
        pending = cursor.fetchone()[0]
        
        # 已完成
        cursor.execute("SELECT COUNT(*) FROM tasks WHERE status = 'completed'")
        completed = cursor.fetchone()[0]
        
        # 按部门统计
        cursor.execute('SELECT department, COUNT(*) FROM tasks GROUP BY department')
        by_department = {row[0]: row[1] for row in cursor.fetchall()}
        
        conn.close()
        
        return {
            "total": total,
            "pending": pending,
            "completed": completed,
            "by_department": by_department
        }
```

### 4.2 基础SQL语句

即使用了Python的数据库封装，了解SQL还是很有用的：

```python
# 插入数据
cursor.execute('INSERT INTO table (col1, col2) VALUES (?, ?)', (val1, val2))

# 查询数据
cursor.execute('SELECT * FROM table WHERE condition = ?', (value,))
rows = cursor.fetchall()  # 获取所有
row = cursor.fetchone()   # 获取一条

# 更新数据
cursor.execute('UPDATE table SET col = ? WHERE id = ?', (new_val, id))

# 删除数据
cursor.execute('DELETE FROM table WHERE id = ?', (id,))

# 提交修改
conn.commit()

# 关闭连接
conn.close()
```

***

## 第五章：实战——看协会的插件代码

### 5.1 聊天插件解析

协会的聊天插件（astrbot\_plugin\_mma\_chat）可以实现智能回复功能。来看看主要逻辑：

**main.py（入口）：**

```python
from astrbot.api.event import filter, AstrMessageEvent, MessageEventResult
from astrbot.api.star import Context, Star, register
import astrbot.api.message_components as Comp
from astrbot.api import logger

from .chat import *

@register("astrbot_plugin_mma_chat", "dreamDust", "数学建模协会小数自由聊天模块", "1.0.0")
class MMAChatPlugin(Star):
    def __init__(self, context: Context):
        super().__init__(context)
        # 定义命令列表（用于判断是否是命令）
        self.commands_list = [
            "create_task", "创建任务",
            "send_task", "发送任务",
            # ... 更多命令
        ]
    
    # 监听私聊消息
    @filter.event_message_type(filter.EventMessageType.PRIVATE_MESSAGE)
    async def on_private_message(self, event: AstrMessageEvent):
        raw_message = event.message_obj.raw_message
        if raw_message.message != None:
            # 如果是命令就不回复
            if self.is_command(raw_message.raw_message):
                return
            
            # 调用聊天函数
            reply = chat.at_chat(raw_message)
            
            # 构建回复
            chain = [Comp.Plain(reply)]
            yield event.chain_result(chain)
            
            # 保存聊天记录
            database.chat_history_assistant_add(reply, raw_message)
    
    # 判断是否是命令
    def is_command(self, input_str):
        if not input_str or not isinstance(input_str, str):
            return False
        
        input_str = input_str.strip()
        
        # 去掉斜杠
        if input_str.startswith('/'):
            input_str = input_str[1:]
        
        # 检查是否匹配命令列表
        for command in self.commands_list:
            if (input_str == command or input_str.startswith(command + ' ')):
                return True
        
        return False
```

### 5.2 秘书处插件解析

秘书处插件是协会最复杂的插件，实现了任务管理功能：

**主要功能：**

1. 创建任务：管理员创建任务，分配给部门
2. 发送任务：通知相关部门
3. 更新任务：修改任务内容
4. 删除任务：删除任务
5. 完结任务：标记任务完成
6. 查看任务：查看任务列表
7. 任务统计：查看统计数据

**命令示例：**

```
# 创建任务
/创建任务 完成活动策划 2026-12-31T23:59:00 组织部

# 查看任务列表
/查看任务 进行中

# 完结任务
/完结任务 1

# 部门回复收到
/收到 1
```

### 5.3 插件配置系统

AstrBot支持在管理页面配置插件参数：

**配置文件（\_conf\_schema.json）：**

```json
{
    "type": "object",
    "properties": {
        "notice_group_id": {
            "type": "string",
            "title": "通知群ID",
            "description": "任务通知发送的群号"
        },
        "secretariat_group_id": {
            "type": "string",
            "title": "秘书处群ID"
        },
        "web_port": {
            "type": "number",
            "title": "Web服务端口",
            "default": 8081
        },
        "department_contacts": {
            "type": "object",
            "title": "部门联系方式",
            "properties": {
                "技术部": {"type": "string"},
                "宣传部": {"type": "string"}
            }
        }
    }
}
```

在代码中获取配置：

```python
class MMASecretariatPlugin(Star):
    def __init__(self, context: Context, config: AstrBotConfig):
        super().__init__(context)
        self.config = config
        secretariat.NOTICE_GROUP_ID = self.config['notice_group_id']
        secretariat.SECRETARIAT_GROUP_ID = self.config['secretariat_group_id']
        
        for department in self.config['department_contacts'].keys():
            secretariat.DEPARTMENT_CONTACTS[department] = self.config['department_contacts'][department]
```

***

## 第六章：进阶功能

### 6.1 异步编程

AstrBot是异步框架，学会异步编程能让插件性能更好：

```python
import asyncio

# 定义异步函数
async def fetch_data():
    await asyncio.sleep(1)  # 模拟耗时操作
    return "数据"

# 调用异步函数
result = await fetch_data()
```

### 6.2 定时任务

可以实现定时执行的功能：

```python
import asyncio

@filter.event_message_type(filter.EventMessageType.GROUP_MESSAGE)
async def on_message(self, event: AstrMessageEvent):
    # 每次消息触发时检查
    pass

# 定时任务（需要自己实现或用第三方库）
```

### 6.3 HTTP请求

有时候需要调用其他API：

```python
import aiohttp

async def fetch_weather():
    async with aiohttp.ClientSession() as session:
        async with session.get('https://api.example.com/weather') as resp:
            data = await resp.json()
            return data
```

协会的秘书处插件用到了Web服务：

```python
from aiohttp import web

async def initialize(self):
    app = web.Application()
    
    # 添加中间件（处理跨域）
    async def cors_middleware(app, handler):
        async def middleware(request):
            if request.method == 'OPTIONS':
                response = web.Response()
                response.headers['Access-Control-Allow-Origin'] = '*'
                return response
            response = await handler(request)
            response.headers['Access-Control-Allow-Origin'] = '*'
            return response
        return middleware
    
    app.middlewares.append(cors_middleware)
    
    # 添加路由
    app.router.add_post('/webhook', self.secretariat_web.handle_request)
    
    # 启动Web服务
    self.web_runner = web.AppRunner(app)
    await self.web_runner.setup()
    site = web.TCPSite(self.web_runner, '0.0.0.0', 8081)
    await site.start()
```

***

## 第七章：实践练习

### 7.1 从简单功能开始

学完上面的知识，可以尝试写一些简单的功能：

**1. 欢迎新成员：**

```python
@filter.event_message_type(filter.EventMessageType.GROUP_MESSAGE)
async def on_member_join(self, event: AstrMessageEvent):
    # 新成员加入时发送欢迎消息
    yield event.plain_result("欢迎新朋友！")
```

**2. 每日提醒：**

```python
@filter.command("提醒", alias={"设置提醒"}, priority=1)
async def set_reminder(self, event: AstrMessageEvent, content: str):
    yield event.plain_result(f"提醒已设置：{content}")
```

**3. 查天气（调用API）：**

```python
@filter.command("天气", alias={"查天气"}, priority=1)
async def weather(self, event: AstrMessageEvent, city: str):
    # 调用天气API
    data = await fetch_weather(city)
    yield event.plain_result(f"{city}今天天气：{data['weather']}")
```

### 7.2 参考协会代码

协会的代码是很好的学习参考：

- **简单插件**：看 `astrbot_plugin_mma_chat`，主要是消息处理
- **复杂插件**：看 `astrbot_plugin_mma_secretariat`，有数据库、有权限控制、有Web服务

### 7.3 调试技巧

开发过程中难免会遇到问题，这里有些调试方法：

**1. 打印日志：**

```python
from astrbot.api import logger

logger.info("这是普通信息")
logger.debug("这是调试信息")
logger.warning("这是警告")
logger.error("这是错误")
```

**2. 打印变量：**

```python
yield event.plain_result(f"收到的消息: {raw_message}")
```

***

## 附录：推荐学习资源

### Python学习

- 官方文档：<https://docs.python.org/zh-cn/3/>
- 菜鸟教程：<https://www.runoob.com/python/>

### AstrBot

- 官方网站：<https://docs.astrbot.app/what-is-astrbot.html>
- 官方文档：<https://docs.astrbot.app/>
- GitHub：<https://github.com/AstrBotDevs/AstrBot>

### SQLite

- 菜鸟教程：<https://www.runoob.com/sqlite/>

***

## 结语

QQ机器人开发其实没那么难，主要就是：

1. 学点Python基础
2. 了解AstrBot的插件机制
3. 学会处理消息、发送回复
4. 有需要的话学学数据库

协会的插件代码是最好的学习资料，不懂的地方多看几遍，动手改一改、跑一跑，慢慢就懂了。

如果遇到问题，随时在群里问大家一起讨论！期待你也能写出有趣的插件！

加油！期待看到你的作品！
