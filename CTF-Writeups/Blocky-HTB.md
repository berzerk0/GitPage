[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To-Guides/HowTo-index.html) <br>


# CTF Writeup:
# Blocky on HackTheBox

<br>
![a0-logo](https://i.imgur.com/Y0d03Mo.png)
<br>

## 9 December 2017





## Introduction


Blocky is a fun beginner's box that was the second or third CTF I ever attempted. At that time, I had booted up Kali and knew that a couple tools existed, but had very few strategies, context or experience. All the boxes I had solved so far had used default passwords or simply were CVE-2017-0144 insta-rooted with Metasploit.


This box will begin to give you a good understanding of the power of the tools at your disposal. By stumbling around trying out tools that I saw were part of the Kali suite, I began to figure the important concepts and found myself learning very quickly.


If you have never tried a CTF before, this box would be a nice place to start - assuming you can get past the HackTheBox Invite process.

This write up assumes that the reader is using Kali, but any pentesting distro such as [BlackArch](https://blackarch.org/) will work. The tools come with a stock Kali installation, unless otherwise mentioned.


## 1. Initial Scans

All HackTheBox CTFs are black-box. All we have is an IP. IPs should be scanned with `nmap`.

`nmap -sV -T4 10.10.10.37`

* The `-sV` flag attempts to tell us the software used on each port found
* The `-T4` flag tells `nmap` to use more CPU threads, and thus run faster

<br>
![a1-nmap](https://i.imgur.com/Hm6cYkB.png)
<br>

`nmap` finds 21, 22, and 80. These are the default ports for FTP, SSH and HTTP.  It also finds 8192, but we can come back to that later if we stall. I like to start with HTTP.

Kali categorizes most of the HTTP tools under "Web Application Analysis," so I took a peek and tried to get most of them working. Since starting out, I found that the most useful tools to start with on web page CTF's are `nikto` and `dirsearch`.

`nikto -h http://10.10.10.37 -output nikto_blocky.txt`


Our `-h` flag tells nikto the location of our target, and the `-output` flag creates an easily revisited log of the results.

<br>
![a2-nikto](https://i.imgur.com/XHOA47W.png)
<br>

My `nikto` scan didn't actually finish until after my dirsearch, and ended up not telling me anything `dirsearch` didn't tell me already. This will not always be the case, so I ofte run both.


`dirsearch` is a web directory bruteforcer implemented in python3. Alternatively, you can use `dirb` or `dirbuster`, which come with Kali, or another program such as `gobuster`.

I cloned it using

`git clone https://github.com/maurosoria/dirsearch /opt/dirsearch`

and ran it with

`python3 /opt/dirsearch/dirsearch -u "http://10.10.10.37" -e sh,txt,php,html,htm,zip,tar.gz,tar --plain-text-report=dirsearch_europa_http_quick -t 25`

* `-u` points to our URL starting point
* `-e` lists some common file extensions to look for. Most of the time `php,html,txt` is enough
* `--plain-text-report=` specifies that we want to save the results to  file
* `-t 25` allocates `dirbuster` more CPU threads

<br>
![a3](https://i.imgur.com/HReNAk5.png)
<br>


We find a whole bunch of paths that look interesting. The fact that we have directories that begin with `wp-...` is a good place to start. This means our site runs on Wordpress, which is associated with quite a few vulnerabilities.  The WPScan tool will help us find any that are present on this site.


`wpscan -u http://10.10.10.37 -e uvp | tee wpscan_block.txt`

<br>
![A4](https://i.imgur.com/XDfpv8b.png)
<br>

The `-e uvp` flag enumerates users and looks for vulnerable plugins, and `| tee ...` saves the output to a file we can come back to later.


WPScan finds many, many vulnerabilities we can dive into, but lets keep enumerating to make sure we get the full lay of the land.

## 2. Using our Reconnoitered Information to Dig Deeper

Let's review what we have so far:
* We've found the username *notch*
* We've found 2 places to log in on the site, *phpmyadmin* and *wp-login*
* A 3rd login exists at  SSH.


If we get phpmyadmin database access, we may be able to find some credentials to login somewhere else, or drop in some kind of reverse shell. If we get Wordpress admin access, we can spawn a reverse shell pretty easily.

Maybe the site has some more useful information. It looks like a simple Wordpress blog with only one post. Let's read it.

<br>
![A5-eyeballsite](https://i.imgur.com/ORSRzbV.png)
<br>

*"...developing a wiki system... and a core plugin..."* <br>
Perhaps some of that is already deployed to the site. Let's revisit our `dirsearch` results.

We have both the `/wp-content/plugins` folder, where `wpscan` found the akismet plugin, and a separate `/plugins` directory.

This is atypical for a Wordpress site, and somewhat suspicious. Why would it have both?


<br>
![A6-plugins](https://i.imgur.com/cpRuBMY.png)
<br>


Two `.jar`s are here for us to download. <br>
The *Core* plugin was mentioned in the blog post, so let's start there.


`wget http://10.10.10.37/plugins/files/BlockyCore.jar` <br>
`unzip BlockyCore.jar -d blockycore`

<br>
![A7-DLblockycore](https://i.imgur.com/OPSss30.png)
<br>

 `cd blockycore` <br>
 `ls -laR *`

<br>
![A8-blockycorecomps](https://i.imgur.com/HvhHJVx.png)
<br>

I don't know too much about Java, but `Blockycore.class` may have something for us.
Can we just read it and find something interesting?


`cat com/myfirstplugin/BlockyCore.class`

<br>
![A9 - cat](https://i.imgur.com/WtGUamo.png)
<br>

This file appears to have been compiled and has some lines we can't read. We can isolate the human-readable parts with the `strings` command.


`strings com/myfirstplugin/BlockyCore.class`

<br>
![A10-strings](https://i.imgur.com/sIfty8D.png)
<br>

There are some strings that look useful, but it's hard to to tell without more context.

I did some online searching and come across the `javap` command, which can be use to disassemble Java classes.
I believe I had to to install it using `apt-get`


`javap -c com/myfirstplugin/BlockyCore.class | tee  ~/CTF/blocky/blockycore_disassembled.txt`

<br>
![A11-javap](https://i.imgur.com/FKLeNA1.png)
<br>

That's the kind of detail I like to see! <br>
`sqluser` and `sqlpass` are VERY promising. These credentials will likely get us into the phpmyadmin database, and might even be re-used elsewhere.


## 3. Gaining Webserver Access


When I first attempted this box, I got stuck here for a few hours. I knew I was missing something very simple, but couldn't find my way through all the information I had at my disposal. The pieces were too disorganized to fit together.

What I should have done was get things clearly laid out by taking another inventory of all that had been found.

So far, we have:

Usernames:
* *notch* - from the WP site
* *root* - exists by default for SSH and phpmyadmin, seen in plugin code
* *admin* - exists by default on Wordpress


Passwords:
* *8YsqfCTnvxAUeduzjNSXe22* - from the plugin code

and

Login Areas:
* Wordpress
* phpmyadmin
* ssh


Let's try `root:8YsqfCTnvxAUeduzjNSXe22` as phpmyadmin credentials.

<br>
![A12- phpmyadmin log in](https://i.imgur.com/VrqfIZt.png)
<br>

Great, we're in the database. Odds are we are going to find hashes here that will lead to the Wordpress password. Before we go trying to finding and cracking them, however, let's keep it simple. We should first try logging in with our credentials somewhere else.


I'm feeling optimistic; I say we go right for the big fish - root via SSH.

`ssh root@10.10.10.37`

and give the password when asked.

<br>
![A13-rootsshfail](https://i.imgur.com/PGuwdxH.png)
<br>

Didn't hurt to try, as unlikely as it was. Since we only have a few usernames, and only one password candidate, it is feasible to simply try them all. Since SSH access is easier to work with than a webshell, let's try to get in here before trying `wp-admin` access.

`ssh notch@10.10.10.37`

<br>
![A14-notchsshwin](https://i.imgur.com/wDjcZOo.png)
<br>

We're in!


## 4. Capturing the User Flag


`ls /home` <br>
`ls -la /home/notch` <br>
`cat /home/notch/user.txt`


A little prior knowledge tells me that Notch is the inventor and boss of the Minecraft universe - Could this mean he is the admin of this system? Let's find out what we can run as superuser.

<br>
![A15-sudo_L](https://i.imgur.com/CQOQ3YB.png)
<br>

Yup, he's an admin. This could be all we need to know to gain root access.

`sudo su`

![A16 - rootshell](https://i.imgur.com/vc1DDda.png)

We have a root shell, and it feels good


## 5. Call Me Markus


From here, it is as trying to read the root flag.

`ls /root` <BR>
`cat /root/root.txt`

Success! Grab the root flag and let's clean up.


__NOTE:__ These are some pretty simple cleanup commands meant to cover our tracks a little bit, but only a little bit. A trained admin would notice that these files have been altered, so look at these commands as the beginning of your tracks-covering career and not as a MIB-Style mind wipe.


`echo '' > /home/notch/.bash_history` <br>
`echo '' > /var/log/auth.log ` <br>
`echo '' > /var/www/admin/logs/access.log` <br>
`echo '' > /root/.bash_history` <br>


Exit the root shell with


`history -c  && kill -9 $$`


Repeat this command as notch, and you're all done.


## Post-Mortem

This box was an easy target because of insecure setup, not any vulnerable applications that we had to exploit.

__Information Disclosure via Hard Coded Credentials__ <br>

*Blocky-Core.jar* disclosed `sqlpass`

Information doesn't get much more sensitive than a superuser password, and ours was found practically in the clear in a publicly accessible file.

__Credential Reuse__ <br>

`sqlpass` was the same as the user password for `notch`

There was no good reason why `notch`'s user password needed to be the same as the `sqlpass`. 


Thanks to HackTheBox and Arrexel!

## Thanks for reading!

<br>
[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To-Guides/HowTo-index.html) <br>
