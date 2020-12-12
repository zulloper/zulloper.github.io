---
layout: post
title: "Pwntools 사용법"
description: "-"
#featured_image: /assets/images/posts/2018/12/5.gif
date: 2020-08-09
category: Pwnable
tags: [CTF, Pwntools, Wargame]
comments: true
---
pwntoosl는 CTF Framwork 및 Exploit development  라이브러리로 Python을 통해 사용할 수 있다. 
Wargame이나 CTF를 풀때 원격의 서버에 Raw Data(Exploit 등을 위한 hex 데이터나 Byte)를 전송할 때 주로 사용한다. 원격 뿐만 아니라 로컬의 Process에도 사용이 가능하다.
pwntools 사용 시 보통 Python2 를 사용하지만, Python3도 지원이 가능하다.
문제풀이에 필요한 데이터를 받거나 보내기가 가능하며, Interactive Shell도 제공한다. Data Encodine/Decoding 등, 정말 많은 기능을 제공하는데 해당 포스팅에서는 기본적인 부분만 다룬다. 


## Table of Contents
* [Connection](#connection-tubes)
	* [Local Process](#local-process)
	* [Remote](#remote)
	* [Secure Shell](#ssh)
	* [Serial Port](#serial-port)
* [Basic IO](#basic-io)
	* [Receiving Data](#receiving-data)
	* [Sending Data](#sending-data)
* [Utility Functions](#utility-functions)
	* [Packing / Unpacking](#packingunpacking)
	* [Hex](#hex)
	* [Base64](#base64)
	* [Hashes](#hashes)
	* [URL Encoding](#url-encoding)

- - - -

## Connection (Tubes)
Exploit을 위해 로컬이나 원격에 데이터를 송/수신을 해야한다. 이 때 대상에 따라 아래와 같이 객체로 생성하여 데이터 송/수신이 가능하다.

### Local Process
로컬 프로세스에 연결하려면 `process()` 객체에 바이너리나 프로세스 명을 입력하여 생성한다.
```py
from pwn import *

p = process('./prcess_name')
p.recvline()
# 'Hello, world\n'
```

### Remote
`remote()` 를 통해 Host와 Port명을 입력하면 원격으로 연결이 가능하다.
```py
from pwn import *

io = remote('google.com', 80)
io.send('GET /\r\n\r\n')
io.recvline()
# 'HTTP/1.0 200 OK\r\n'
```

### SSH
ssh를 통해 프로세스에 접근할 수 있다. 아래와 같이  `ssh()` 통해 로그인 후 session을 통해 process에 접근이 가능하다.
```py
rom pwn import *

session = ssh('user', 'host', password='password')

sp = session.process('process')
sp.recvline()
# 'Hello, world!\n'
```

### Serial Port
시리얼 포트도 지원을 한다.
```py
from pwn import *

s = serialtube('/dev/ttyUSB0', baudrate=115200)
```


## Basic IO
### Receiving Data
* `recv(n)`  -  n만큼 데이터를 수신한다.
* `recvline()`  - 개행을 만날 때 까지 데이터를 수신한다.
* `recvuntil(data)` - 특정 데이터(문자열, 정수 등)을 만날 때 까지 데이터를 수신한다.
* `recvregex(pattern)` - 정규표현식에 해당 하는 패턴을 만날 때 까지 데이터를 수신한다.
* `recvrepeat(timeout)` - 타임아웃이 발생할 때 까지 데이터를 수신한다.
* `clear()` - 버퍼에 있는 데이터를 제거한다.

### Sending Data
* `send(data)` - 데이터를 송신한다. 수신 측에서 Read() 함수를 사용할 경우 주로 사용한다.
* `sendline(data)` - 데이터 끝에 개행(new line)을 포함시켜 데이터를 송신한다. 수신 측에서 scanf(), gets(), fget() 등을 사용할 때 사용한다. 

## Utility Functions
데이터를 변환하여 송/수신할 경우가 많기 때문에 아래와 같은 함수를 알고있으면 도움이 된다.
### Packing/Unpacking
데이터(정수)를 16/32/64 bit로 Packing/Unpacking 해주는 함수로 Hex 데이터를 전달할 때 용이하다.
endian이나 bits는 context에 정의가 가능하여 함수마다 매개변수로 지정하지 않고 환경변수처럼 사용이 가능하다.
```py
pack(1234) # '\xd2\x04\x00\x00'
unpack('\xd2\x04\x00\x00') # 1234

# 32bit
p32(1234) # '\xd2\x04\x00\x00'
u32('\xd2\x04\x00\x00') # 1234

# 64bit
p64(1234) # '\xd2\x04\x00\x00\x00\x00\x00\x00'
u64('\xd2\x04\x00\x00\x00\x00\x00\x00') # 1234

# Endian
p32(1234, endian='big') # '\x00\x00\x04\xd2'
u32('\x00\x00\x04\xd2', endian='big') # 1234
u32('\x00\x00\x04\xd2') # 3523477504 / 엔디안 설정을 잘못할 경우 잘못된 값 출력

hex(unpack('AAAA')) # '0x41414141'
```

### Hex
Hex Encoding과 Decoding 기능도 제공한다.
```py
enhex('hello')
# '68656c6c6f'
unhex('776f726c64')
# 'world'
```
### Base64
Base64 Encoding과 Decoding이 가능하다.
```py
b64e('hello') # 'aGVsbG8='
b64d('aGVsbG8=') # 'hello'
```

### Hashes
Hash같은 경우에 md5와 sha1등을 제공한다. 
```py
md5sumhex('hello') == '5d41402abc4b2a76b9719d911017c592'
# md5filehex()는 파일을 읽어서 해당 md5를 구한다.
md5filehex(File) == '5d41402abc4b2a76b9719d911017c592'
sha1sumhex('hello') == 'aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d'
```

### URL Encoding
```py
urlencode("Hello, World!") == '%48%65%6c%6c%6f%2c%20%57%6f%72%6c%64%21'
```

## 참고
Github Repository : [PWNTOOLS](https://github.com/Gallopsled/pwntools)

Document : [Docs - pwntools](http://docs.pwntools.com/en/stable/#)
