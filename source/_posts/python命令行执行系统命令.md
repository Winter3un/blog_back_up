---
title: python命令行执行系统命令
date: 2016-06-08 15:40:11
tags:
 - pwn
 - python
---

### 背景

来源于ISCC2016的一题pwn题目，nc连上去后发现是一个python shell，当时思路错了，死脑筋的认为这一定是一个python反序列化漏洞，然后各种搜集资料，ORZ。
好吧，这其实是一个导入shell模块，命令执行拿flag的姿势。

### 导入模块

python中导入模块有以下几种方式

1. `import xxx`
2. `from xxx import *`
3. `__import__('xxx')`
**程序把前两种都禁用了，而第三种却没有禁用。**

### shell模块

python中shell模块有以下几种
1. ` os.system() os.popen()`
2. `commands.getstatusoutput() commands.getoutput() `
3. ` commands.getstatus()`
4. `subprocess.call(command, shell=True) subprocess.Popen(command, shell=True)`
5. `pty.spawn()`

### 附上pwn脚本

```python
#!/usr/bin/env python2
# -*- coding:utf-8 -*-

def banner():
    print "============================================="
    print "   Simple calculator implemented by python   "
    print "============================================="
    return

def getexp():
    return raw_input(">>> ")

def _hook_import_(name, *args, **kwargs):
    module_blacklist = ['os', 'sys', 'time', 'bdb', 'bsddb', 'cgi', 
            'CGIHTTPServer', 'cgitb', 'compileall', 'ctypes', 'dircache',
            'doctest', 'dumbdbm', 'filecmp', 'fileinput', 'ftplib', 'gzip',
            'getopt', 'getpass', 'gettext', 'httplib', 'importlib', 'imputil', 
            'linecache', 'macpath', 'mailbox', 'mailcap', 'mhlib', 'mimetools',
            'mimetypes', 'modulefinder', 'multiprocessing', 'netrc', 'new',
            'optparse', 'pdb', 'pipes', 'pkgutil', 'platform', 'popen2', 'poplib',
            'posix', 'posixfile', 'profile', 'pstats', 'pty', 'py_compile',
            'pyclbr', 'pydoc', 'rexec', 'runpy', 'shlex', 'shutil', 'SimpleHTTPServer',
            'SimpleXMLRPCServer', 'site', 'smtpd', 'socket', 'SocketServer',
            'subprocess', 'sysconfig', 'tabnanny', 'tarfile', 'telnetlib',
            'tempfile', 'Tix', 'trace', 'turtle', 'urllib', 'urllib2',
            'user', 'uu', 'webbrowser', 'whichdb', 'zipfile', 'zipimport']
    for forbid in module_blacklist:
        if name == forbid:        # don't let user import these modules
            raise RuntimeError('No you can\' import {0}!!!'.format(forbid))
    return __import__(name, *args, **kwargs)    # normal modules can be imported

def sandbox_filter(command):
    blacklist = ['exec', 'sh', '__getitem__', '__setitem__', '=', 'open', 'read', 'sys', ';', 'os']
    for forbid in blacklist:
        if forbid in command:   return 0
    return 1

def sandbox_exec(command):      # sandbox user input
    result = 0
    __sandboxed_builtins__ = dict(__builtins__.__dict__)
    __sandboxed_builtins__['__import__'] = _hook_import_    # hook import
    del __sandboxed_builtins__['open']
    _global = {
        '__builtins__' : __sandboxed_builtins__
    }
    if sandbox_filter(command) == 0:
        print 'Malicious user input detected!!!'
        exit(0)
    command = 'result = ' + command
    try:
        exec command in _global     # do calculate in a sandboxed environment
    except Exception, e:
        print e
        return 0
    result = _global['result']  # extract the result
    return result

banner()
while 1:
    command = getexp()
    print sandbox_exec(command)
```