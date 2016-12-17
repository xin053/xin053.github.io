---
title: yagmail邮件发送库使用详解
date: 2016-12-17 16:26:07
categories:
- Python模块学习
tags:
- Python
- yagmail
---

## yagmail简介

使用python标准库进行邮件的处理比较复杂，所以产生了yagmail，但是yagmail目前只能用SMTP协议进行邮件发送，并不能读取邮件，也不支持其他的邮件相关协议，但是对于一般使用完全够了。

<img src="https://github.com/kootenpv/yagmail/raw/master/resources/icon.png" style="zoom:35%" />

<!-- more -->

## yagmail使用

首先是通过`yagmail.SMTP()`生成一个客户端，但是为了不将我们的密码暴露下脚本文件中，yagmail使用[keyring](https://github.com/jaraco/keyring/)模块将密码存放在系统keyring服务中。

关于keyring是什么，请看:[What does a Keyring do?](https://askubuntu.com/questions/32164/what-does-a-keyring-do)

官方文档中，

```python
yagmail.register('mygmailusername', 'mygmailpassword')
```

实际上是对`keyring.set_password('yagmail', 'mygmailusername', 'mygmailpassword')`的封装。

`SMTP()`方法会去用户主文件夹读取`.yagmail`文件，但是以上操作并不会生成这个文件，所以需要自己创建，并将自己的邮箱写入文件中。

例如，我测试过程中写入`.yagmail`文件中的内容为:

```
810620174@qq.com
```

而之前我已经通过`register()`方法将该邮箱的密码保存到了系统keyring中，所以接下来就可以初始化一个SMTP客户端

另外还需要注意的是，经过测试，163邮箱很容易将邮件识别为垃圾邮件，导致邮件发送错误，而qq邮箱需要关闭[邮件保护](https://aq.qq.com/cn2/safe_service/device_lock)，其他邮箱没有测试，这里推荐使用qq邮箱。

### 常用邮箱SMTP服务器地址和端口

```
sina.com: 
POP3服务器地址:pop3.sina.com.cn（端口：110） 
SMTP服务器地址:smtp.sina.com.cn（端口：25）   

sinaVIP： 
POP3服务器:pop3.vip.sina.com （端口：110） 
SMTP服务器:smtp.vip.sina.com （端口：25）  

sohu.com: 
POP3服务器地址:pop3.sohu.com（端口：110） 
SMTP服务器地址:smtp.sohu.com（端口：25）  

126邮箱： 
POP3服务器地址:pop.126.com（端口：110） 
SMTP服务器地址:smtp.126.com（端口：25）  

139邮箱： 
POP3服务器地址：POP.139.com（端口：110） 
SMTP服务器地址：SMTP.139.com(端口：25)  

163.com: 
POP3服务器地址:pop.163.com（端口：110） 
SMTP服务器地址:smtp.163.com（端口：25）  

QQ邮箱  
POP3服务器地址：pop.qq.com（端口：110） 
SMTP服务器地址：smtp.qq.com （端口：25）  

QQ企业邮箱 
POP3服务器地址：pop.exmail.qq.com （SSL启用 端口：995） 
SMTP服务器地址：smtp.exmail.qq.com（SSL启用 端口：587/465）

yahoo.com: 
POP3服务器地址:pop.mail.yahoo.com 
SMTP服务器地址:smtp.mail.yahoo.com  

yahoo.com.cn: 
POP3服务器地址:pop.mail.yahoo.com.cn（端口：995） 
SMTP服务器地址:smtp.mail.yahoo.com.cn（端口：587）  

HotMail 
POP3服务器地址：pop3.live.com （端口：995） 
SMTP服务器地址：smtp.live.com （端口：587） 

gmail(google.com) 
POP3服务器地址:pop.gmail.com（SSL启用 端口：995） 
SMTP服务器地址:smtp.gmail.com（SSL启用 端口：587）  

263.net: 
POP3服务器地址:pop3.263.net（端口：110） 
SMTP服务器地址:smtp.263.net（端口：25）  

263.net.cn: 
POP3服务器地址:pop.263.net.cn（端口：110） 
SMTP服务器地址:smtp.263.net.cn（端口：25） 

x263.net: 
POP3服务器地址:pop.x263.net（端口：110） 
SMTP服务器地址:smtp.x263.net（端口：25） 

21cn.com: 
POP3服务器地址:pop.21cn.com（端口：110） 
SMTP服务器地址:smtp.21cn.com（端口：25） 

Foxmail： 
POP3服务器地址:POP.foxmail.com（端口：110） 
SMTP服务器地址:SMTP.foxmail.com（端口：25）  

china.com: 
POP3服务器地址:pop.china.com（端口：110） 
SMTP服务器地址:smtp.china.com（端口：25） 

tom.com: 
POP3服务器地址:pop.tom.com（端口：110） 
SMTP服务器地址:smtp.tom.com（端口：25）  

etang.com: 
POP3服务器地址:pop.etang.com 
SMTP服务器地址:smtp.etang.com
```

`yagmail.SMTP()`默认使用的gmail的SMTP服务，所以我们如果使用qq邮箱，则使用如下代码初始化一个SMTP客户端

```python
yag = yagmail.SMTP('810620174@qq.com', host='smtp.qq.com', port='25')
```

紧接着就可以发送邮件了

```python
yag.send('13207130066.cool@163.com', '邮件主题', '这是邮件内容')
```

至此，便像`13207130066.cool@163.com`这个邮箱发送了一封邮件。

注意`send()`方法的定义:

```python
def send(self, to=None, subject=None, contents=None, attachments=None, cc=None, bcc=None,preview_only=False, validate_email=True, throw_invalid_exception=False, headers=None)
```

如果不指定`to`参数，则发送给自己,如果`to`参数是一个列表，则将该邮件发送给列表中的所有用户，`attachments`表示附件，该参数可以是列表，表示发送多个附件

对于`contents`参数，官方说明如下:

- If it is a dictionary it will assume the key is the content and the value is an alias (only for images currently!) e.g. {'/path/to/image.png' : 'MyPicture'}
- It will try to see if the content (string) can be read as a file locally, e.g. '/path/to/image.png'
- if impossible, it will check if the string is valid html e.g. `This is a big title`
- if not, it must be text. e.g. 'Hi Dorika!'

## 参考文档

- [yagmail官方文档](https://github.com/kootenpv/yagmail#no-more-password-and-username)
- [常用的邮箱服务器(SMTP、POP3)地址、端口](http://wenku.baidu.com/link?url=dzf8yMnLf6TwrW44kjjl364hD_qSkRsjtc3T9nUuxwjrzo6ohG-9RxJSES5YupoXuzYe2S4vYRCcTvCE8mwH_8EJEqZOslUxo_nxQmtqAXi)