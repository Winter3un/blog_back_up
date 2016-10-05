---
title: C/S-WeChat
date: 2016-07-19 08:48:33
tags:
 - code
 - python
---


### 前记

某日战队的一个小伙伴提出来，可以制作一个基于socket的聊天室出来，用来应对比赛时的交流问题（嗯。。不是有微信和QQ么。。），不过想法很不错。
我也借着这次机会复习一下python的socket网络编程。

这些项目保存在我的github上，并且看我心情更新。

未解决问题：
1. 数据明文传输，未加密。
2. 未设置登录限制，可以匿名随意进入聊天室。
3. 易被DDOS
4. 未发现的问题

### client模块

1. 首先检测是否有更新，若有更新则下载最新的更新。
2. 异步接收远程服务器传递过来的数据，一旦接收到数据则puts到界面。
3. 接收用户输入的数据，当用户输入回车后将数据发送到远程服务器上

``` python
#!/usr/bin/env python
# encoding: utf-8


"""
@version: ??
@author: WinterSun
@contact: 511683586@qq.com
@site: http://blog.csdn.net/yuanyunfeng3
@software: PyCharm
@file: client.py
@time: 2016/7/8 0:43
"""

import socket,threading,select
import hashlib,sys,os

def upgrade():
    filepath =  sys.path[0]+os.sep+sys.argv[0]
    with open(filepath,'rb') as f:
        local_hash = hashlib.md5(f.read()).hexdigest()
    addr = ('123.207.98.208',55557)
    s = socket.socket()
    try:
        s.connect(addr)
        s.send(local_hash)
        inputs=  [s]
        rs,ws,es = select.select(inputs,[],[],1)
        for r in rs:
            data = r.recv(1024)
            if data  == '1':
                print "this is the lastest version!"
                inputs.remove(r)
                r.close()
                return False
            else:
                s.send('get file_size')
                size = s.recv(1024)
                s.send("get file")
                new_file = s.recv(int(size))
                with open(filepath,'wb') as f:
                    f.write(new_file)
                print "the client is upgraded,plz restart it!"
                return True
    except Exception,e:
        print e
        print "can't connect update_sever"
        return True
def read(s):
    inputs=  [s]
    while True:
        rs,ws,es = select.select(inputs,[],[],1)
        for r in rs:
            print r.recv(1024)
def write(s):
    while True:
        try:
            info = raw_input()
        except Exception,e:
            print e
            exit(0)
        try:
            s.send(info)
        except Exception,e:
            print e
            exit()
if __name__ == '__main__':
    if upgrade():
        exit(0)
    host = raw_input("plz input sever [ip:port] :")
    host,port = host.split(':')
    addr = (host,int(port))
    s = socket.socket()
    try:
        s.connect(addr)
    except:
        print "can't connect this sever!"
        exit(0)
    name = raw_input("plz input your name:")
    s.send(name)
    t1 = threading.Thread(target=read,args=(s,))
    t2 = threading.Thread(target=write,args=(s,))

    t1.start()
    t2.start()

```

### server模块

请设置port，默认为10087

1. 不断接收新的套接字。
2. 当有新的套接字进入时，将欢迎信息发送过去。
3. 第一次输入为用户名。

``` python
#!/usr/bin/env python
#-*-coding=utf8-*-


"""
@version: ??
@author: WinterSun
@contact: 511683586@qq.com
@site: http://blog.csdn.net/yuanyunfeng3
@software: PyCharm
@file: sever.py
@time: 2016/7/8 0:08
"""

import socket,select

host='0.0.0.0'
prot=10087
s= socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.bind((host,prot))
s.listen(5)
inputs = [s]
namedic = {}
namelist = []

def send(data,inputs):
    for x in inputs:
        if x != s:
            try:
                x.send(data)
            except Exception,e:
                print e
                if x in inputs:
                    inputs.remove(x)
                    x.close()
                send(namedic[x]+"\tleaving!\n",inputs)

if __name__ == '__main__':
    while True:
        rs,ws,es = select.select(inputs,[],[],1)
        for r in rs:
            if r is s:
                conn,addr = r.accept()
                print 'Connect by',addr
                while True:
                    try:
                        name = conn.recv(1024)
                        if len(name) > 30:
                            conn.send("your name is toooooo long!\n")
                            continue
                        else:
                            namedic[conn] = name
                            namelist.append(name)
                            inputs.append(conn)
                            data = "Welcome "+name
                            send(data,inputs)
                            break
                    except socket.error:
                        pass

            else:
                try:
                    data = r.recv(1024)
                    print namedic[r]+":"+data
                    if not data:
                        data =  "\tleaving!\n"
                        print namedic[r]+data
                        if r in inputs:
                            inputs.remove(r)
                            r.close()
                        send(namedic[r]+data,inputs)
                    else:
                        send(namedic[r]+":"+data,inputs)
                except :
                    data =  "\tleaving!\n"
                    print namedic[r]+data
                    if r in inputs:
                        inputs.remove(r)
                        r.close()
                    send(namedic[r]+data,inputs)
```

### upgrade模块

为了方便软件更新，特加入upgrade模块用于更新。（讲道理，这个其实是后门）

``` python

#!/usr/bin/env python
# encoding: utf-8
import hashlib,socket,select
host='0.0.0.0'
prot=55557
s= socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.bind((host,prot))
s.listen(5)
inputs = [s]
while True:
	rs,ws,es = select.select(inputs,[],[],1)
	for r in rs:
		if r is s:
			conn,addr = r.accept()
			print 'Connect by',addr
			try:
				local_hash = conn.recv(1024)
				with open('client.py','rb') as f:
					lastest_hash = hashlib.md5(f.read()).hexdigest()
				if local_hash == lastest_hash:
					conn.send('1')
					conn.close()
				else:
					conn.send('0')
					inputs.append(conn)
			except:
				pass
		else:
			try:
				data = r.recv(1024)
				with open('client.py','rb') as f:
					file = f.read()
				if data =="get file_size":
					print "send file_size"
					# print len(file),type(len(file))
					r.sendall(str(len(file)))
				if data =="get file":
					print "send file"
					r.sendall(file)
			except:
				pass
				
```