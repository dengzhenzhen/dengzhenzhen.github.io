---
layout:     post
title:      "用smtplib和email模块发送邮件"
subtitle:   "顺便了解email的组成"
date:       2018-07-31
author:     dengzhenzhen
header-img: img/post-bg-EmailLogos.png
catalog: true
tags:
    - smtplib
    - email
    - 邮件
---

## smtplib模块

通过python内置的smtplib模块可以很方便地发送邮件

有时想从vps上把文件下到本地，用scp挺麻烦的，可以直接用vim写一段python代码把文件发到自己邮箱

首先是导入smtplib

### 1.从最简单的例子了解smtplib


```python
import smtplib
```

一个最简单的例子：

先配置账号信息,填写自己的邮箱账号密码：


```python
from_addr = '374894000@qq.com'
password = '**************'
```

使用QQ邮箱的话，应该用**授权码**来代替邮箱密码

授权码在**邮箱主页-设置-账户-SMTP服务**中，有一个**生成授权码**选项

![ ](https://s1.ax1x.com/2018/07/31/PdBdL6.png)




```python
to_addr = ['3210946881@qq.com']
server = smtplib.SMTP('smtp.qq.com')  #指定smtp服务器,一般就是 "smtp.邮箱域名"
server.login(from_addr, password)  #登陆邮箱
server.sendmail(from_addr , to_addr, 'mail text')   #发送邮件，to_addr可以是个列表
server.quit()  
```




    (221, b'Bye')



出现状态码221就说明邮件已经成功发送了

不过刚才是最简单的一种情况，从收件箱中可以看出收到的邮件什么也没有。这是因为email是有格式的，而我们刚才发的只是几个随便写的字符串

### 2.为邮件添加正文

我们可以用email模块来辅助我们完善邮件的格式，从而发送一封"正常"的邮件


```python
from email.mime.text import MIMEText
from email.header import Header
from email.mime.multipart import MIMEMultipart
```

先看一下MIMEText


```python
msg = MIMEText('this is an email text!')
```


```python
#发送时把这个邮件作为串有格式的字符串发送出去，用as_string方法
msg.as_string()
```




    'Content-Type: text/plain; charset="us-ascii"\nMIME-Version: 1.0\nContent-Transfer-Encoding: 7bit\n\nthis is an email text!'




```python
server = smtplib.SMTP('smtp.qq.com')
server.login(from_addr, password)
server.sendmail(from_addr , to_addr, msg.as_string()) 
server.quit()  
```




    (221, b'Bye')



![aaa](https://s1.ax1x.com/2018/07/31/PdBUQ1.png)
这时候可以看到，收到的邮件有正文了，但是其他的什么都没有


### 3.添加header信息

用**Header**和**MIMEMultipart**可以很方便地解决这个问题


```python
msg = MIMEMultipart()
msg['From'] = Header('发件人')
msg['To'] = Header('收件人')
msg['Subject'] = Header('邮件标题')
msg.attach(MIMEText('邮件正文'))
```

这时邮件变成这样了:


```python
print(msg.as_string())
```

    Content-Type: multipart/mixed; boundary="===============6879720765695580357=="
    MIME-Version: 1.0
    From: =?utf-8?b?5Y+R5Lu25Lq6?=
    To: =?utf-8?b?5pS25Lu25Lq6?=
    Subject: =?utf-8?b?6YKu5Lu25qCH6aKY?=
    
    --===============6879720765695580357==
    Content-Type: text/plain; charset="utf-8"
    MIME-Version: 1.0
    Content-Transfer-Encoding: base64
    
    6YKu5Lu25q2j5paH
    
    --===============6879720765695580357==--
    
    

再发送一次看变成什么样


```python
server = smtplib.SMTP('smtp.qq.com')
server.login(from_addr, password)
server.sendmail(from_addr , to_addr, msg.as_string()) 
server.quit()  
```




    (221, b'Bye')



这次收到的邮件是这样的

![有header的邮件](https://s1.ax1x.com/2018/07/31/PdBasx.png)

### 4.添加附件

用attach方法还能给邮件添加附件


```python
attachment = MIMEText(open('hello.txt', 'rb').read(), 'base64', 'utf-8')
attachment['Content-Type'] = 'application/octet-stream'                     #表明这是个二进制文件
attachment['Content-Disposition'] = 'attachment;filename="hello.txt"'      #表明这是个附件
msg.attach(attachment)
```

邮件变成了这样:


```python
print(msg.as_string())
```

    Content-Type: multipart/mixed; boundary="===============6879720765695580357=="
    MIME-Version: 1.0
    From: =?utf-8?b?5Y+R5Lu25Lq6?=
    To: =?utf-8?b?5pS25Lu25Lq6?=
    Subject: =?utf-8?b?6YKu5Lu25qCH6aKY?=
    
    --===============6879720765695580357==
    Content-Type: text/plain; charset="utf-8"
    MIME-Version: 1.0
    Content-Transfer-Encoding: base64
    
    6YKu5Lu25q2j5paH
    
    --===============6879720765695580357==
    Content-Type: text/base64; charset="utf-8"
    MIME-Version: 1.0
    Content-Transfer-Encoding: base64
    Content-Type: application/octet-stream
    Content-Disposition: attachment;filename="hello.txt"
    
    MTIzNDU2NDY0
    
    --===============6879720765695580357==--
    
    

再次发送


```python
server = smtplib.SMTP('smtp.qq.com')
server.login(from_addr, password)
server.sendmail(from_addr , to_addr, msg.as_string()) 
server.quit()  
```




    (221, b'Bye')



发送成功，有正文有附件

![附件邮件](https://s1.ax1x.com/2018/07/31/PdBtzR.png)

## 总结

最开始学习这个是因为Kesci上没有提供下载数据集的方法，只能通过发邮件把数据集发到自己邮箱然后再从邮箱里下载

在学习发邮件的过程中稍微了解了email是怎么构成的，也算是有所收获
