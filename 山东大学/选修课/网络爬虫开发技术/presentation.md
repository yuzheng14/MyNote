# 山东大学（威海）教务处工作通知监控

## 环境

- Linux python3.6.8
- Windows python3.8.2

以上环境均可运行

## 目标URL

[https://jwc.wh.sdu.edu.cn/gztz.htm（山东大学威海教务处工作通知）](https://jwc.wh.sdu.edu.cn/gztz.htm)及其各个具体通知页面

(crawler.Crawler)  
```python
class Crawler:
    __url__ = 'https://jwc.wh.sdu.edu.cn/gztz.htm'
    （省略）
    def crawl_content(self, number):
        '''
        爬取教务处工作通知网站的前number条通知
        '''
        urls = []
        for node in self.__nodes__[:number]:
            href=node.attrib['href']
            urls.append(f'https://jwc.wh.sdu.edu.cn/{href}')
            （省略）
```

## 处理网页

1. `工作通知页面`：集合所有通知
2. `具体通知页面`：通知的具体内容

## 元素路径

### 工作通知页面

//*[@id="content"]/div[2]/div[2]/div[1]/ul 的所有子节点

```python
class Crawler:
    （省略）
    def crawl(self):
        '''
        爬取教务处工作通知网站的源码
        '''
        （省略）
        self.__data__ = self.__selector__.xpath(
            '//*[@id="content"]/div[2]/div[2]/div[1]/ul') #定位通知所在的无序列表
        self.__children__ = self.__data__[0].getchildren() # 获取子节点
        self.__nodes__ = []
        for child in self.__children__:
            self.__nodes__.append(child.getchildren()[0]) # 取子节点中的第一个子节点，即通知名称及超链接所在节点
        （省略）
```

### 具体通知页面

//*[@id="content"]/div[2]/div[2]/form 中的html代码

``` python
class Crawler:
    （省略）
    def __crawl_all_content__(self,urls):
        '''
        根据urls爬取所有页面的正文内容并返回
        '''
        （省略）
        for url in urls:
            （省略）
            try:
                data=etree.HTML(request.urlopen(url).read().decode()).xpath('//*[@id="content"]/div[2]/div[2]/form') # 通知具体内容
                （省略）
```

## 主要技术

### 模块

- time
- json
- etree
- urllib
- smtplib
- email

#### time

(log.py)  
```python
import time
```

(main.py)  
```python
import time
```

#### json

(main.py)  
```python
import json
```

(send_email.py)  
```python
import json
```

(util.py)
```python
import json
```

#### etree

(crawler.py)  
``` python
from lxml import etree
```

(send_email.py)  
``` python
from lxml import etree
```

#### urllib

(crawler.py)  
```python
from urllib import request
```

(send_email.py)  
```python
from urllib import request
```

#### smtplib

(send_email.py)  
```python
import smtplib
```

#### email

(send_email.py)  
```python
import email
from email import message
from email.mime import text
from email.mime.text import MIMEText
```

### 主要技术

- json解析生成技术
- 面向对象技术
- 对象封装技术
- 根据配置文件运行技术
- shell脚本或者windows批处理文件执行技术

#### json解析生成技术

利用json文件充当配置文件，进行解析，同时为了不了解json的用户提供`util.py`来生成`config.json`

(main.py)   
```python
（省略）
while(True):
    （省略）
    with open('config.json','r') as f:
        data=json.load(f)
    time.sleep(data['interval']) # 读取配置文件以进行特定时间的休眠
```

(send_email.py)  
```python
def send_email(content, titles):
    # 读取config.json配置发信者信息及收信者信息
    with open('config.json', 'r') as f:
        data = json.load(f)
    mail_host = data['host']
    mail_user = data['user']
    mail_password = data['password']
    sender = data['sender']
    receivers = data['receivers']
    （省略）
```

#### 面向对象技术

将爬虫封装成对象，配合封装技术，开箱即用，无须关心内部实现

(crawler.py)  
```python
class Crawler:
    （省略）
```

#### 对象封装技术

将使用者需要的方法开放，不需要使用的方法及属性全部私有化，进行封装，提高易用性及移植性，并减小外界对对象内部属性的干扰，提高对象鲁棒性

```python
class Crawler:
    __url__ = 'https://jwc.wh.sdu.edu.cn/gztz.htm'
    def crawl(self):
        self.__html = request.urlopen(self.__url__)
        self.__html_content__ = self.__html.read().decode()
        self.__selector__ = etree.HTML(self.__html_content__)
        self.__data__ = self.__selector__.xpath(
            '//*[@id="content"]/div[2]/div[2]/div[1]/ul') #定位通知所在的无序列表
        self.__children__ = self.__data__[0].getchildren() # 获取子节点
        self.__nodes__ = []
        for child in self.__children__:
            self.__nodes__.append(child.getchildren()[0]) # 取子节点中的第一个子节点，即通知名称及超链接所在节点
```

#### 根据配置文件运行技术

程序根据`config.json`文件的配置进行运行，提高非编程人员易用性

(main.py)   
```python
（省略）
while(True):
    （省略）
    with open('config.json','r') as f:
        data=json.load(f)
    time.sleep(data['interval']) # 读取配置文件以进行特定时间的休眠
```

(send_email.py)  
```python
def send_email(content, titles):
    # 读取config.json配置发信者信息及收信者信息
    with open('config.json', 'r') as f:
        data = json.load(f)
    mail_host = data['host']
    mail_user = data['user']
    mail_password = data['password']
    sender = data['sender']
    receivers = data['receivers']
    （省略）
```

#### shell脚本或者windows批处理文件执行技术

- Linux下提供`run.sh`与`run_log.sh`两个shell脚本文件来运行系统
- Windows下提供`run.bat`批处理文件来运行系统

提高系统开箱即用性

### 爬取网页

如无不可处理异常，程序将持续运行

自运行开始，教务处有多少通知爬取（n+1）个网页

## 反爬

该网站无反爬措施，无需反爬

（UA标识都不用）

## 存在问题及展望

### 问题

1. 部分通知需要内网才能访问，目前尚未掌握Linux系统连接内网方面知识，无法获取需要内网才能访问的通知的内容，但仍可通过在内网环境内通知url来访问该通知
2. 模块拆分仍不够细致，可维护性仍可提高，可继续解耦合。
3. 未知Error，尝试try-catch无法处理
3. 未设置配置文件不正确时异常处理
4. Linux服务器运行时因不可处理异常停止运行时无法立即通知程序异常中断需重启
5. 未对具体通知中附件下载链接相对地址处理成绝对地址，无法通过直接点击附件链接进行下载，但可通过邮件内容中添加的通知url访问通知页面进行附件下载

### 展望

1. 使用多线程技术，除主线程外，增加3个线程
2. 主线程用于等待任务执行，任务执行时将具体任务交给线程1，继续休眠
3. 线程1用于爬取需要爬取的具体通知的url，放入url队列
4. 线程2用于监控url队列，当url队列不为空时爬取url队列中的具体通知的内容，并处理好封装好放入邮件队列
5. 线程3用于监控邮件队列，当邮件队列不为空时发送邮件队列中的邮件
6. 改进项目文件树组织
7. 使用Java或Kotlib重构，使用mirai框架直接将教务处通知通知到班群

## 爬取数据展示

因本系统目的在于及时将教务处通知通知到个人，故采用邮件发送的方式，无持久化文本数据，故采用截图形式展示爬取数据

![系统启动](presentation.assets/系统启动.JPG)

![正常通知](presentation.assets/正常通知.JPG)

![内网通知](presentation.assets/内网通知.JPG)