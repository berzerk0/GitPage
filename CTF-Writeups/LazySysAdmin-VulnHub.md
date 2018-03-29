[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To Guides/HowTo-index.html) <br>

# CTF Writeup:
# LazySysAdmin on VulnHub
<br>
![a0](https://i.imgur.com/iFDpFwb.jpg)
<br>
## 8 November 2017


A fun box from Vulnhub, written by Togie McDogie.
You can find it [here](https://www.vulnhub.com/entry/lazysysadmin-1,205/) at https://www.vulnhub.com/entry/lazysysadmin-1,205/


## Introduction

LazySysAdmin was a fun little box that reminds us to keep it simple.
This write up assumes the reader has beginner knowledge of pentesting.

It also assumes the reader is using Kali, but all the tools are standard in distros like BlackArch as well.
I ran it on my native Kali host machine using VirtualBox; on a host-only network.

Before beginning, I added its IP to my hosts file for convenience.

[Here](https://pastebin.com/raw/briMUhtu) is a very quick summary __*that is entirely composed of spoilers*__ -

It doesn't show my methodology or process.
It will tell you how to get the root flag, but you won't learn much.
It's also encrypted (weakly) - but if you can root the box using just the spoiler, you'll know how to decode it.






## 1. Initial Scans
As always, we start with `nmap`

*  `nmap -sV -T4 lazysysadmin.ctf`

<br>
![A1](https://i.imgur.com/FiyPXpn.png)
<br>

 Alright, we have a website, so let's launch our regular HTTP battery.
 After that we will investigate SMB, MYSQL, and IRC - if we need to.
 If we figure out the credentials, we can SSH in as well.

 Run `dirsearch`, `nikto`, `nmap` w/ vuln scan, and manually browse the website. Run these scans in parallel.

 I like to run `dirsearch` twice - a quick scan without specifying a wordlist to find common things, then a deeper dive.
 Even my quick can uses a large number of extensions - I usually don't run it recursively until I find an interesting starting point,
 or get stuck and think I need to keep digging.

*  `dirsearch -u lazysysadmin.ctf -e sh,txt,php,html,htm,asp,aspx,js,xml,log,json,jpg,jpeg,png,gif,doc,pdf,mpg,mp3,zip,tar.gz,tar`

<br>
![A2](https://i.imgur.com/tjDk2in.png)
<br>

*  `nmap -A -O -T4 --script=vuln lazysysadmin.ctf`

<br>
![A3](https://i.imgur.com/WV6qfti.png)
<br>
![A4](https://i.imgur.com/XHKjJFc.png)
<br>

<br>

 * `nikto -h http://lazysysadmin.ctf`

<br>
![A5](https://i.imgur.com/WiAW82Q.png)
<br>


In the browser, I took a peek at the page sources for anything interesting. However, won't find much there.

<br>
![A6](https://i.imgur.com/d05dbp8.png)
<br>


 I like steganography (there is some in this guide) - so I read *"The answer is within you"* as *"Check this image for stego."* <br>
 Nothing came up with `steghide`, `strings` or stegsolve; so I checked my scan outputs.

 The first thing I checked was `robots.txt` - something that the scanners might not be able to do for me.
 It didn't contain anything too interesting... unlike my `nmap` vulnscan, which dumped out a ton of useful info.



## 2. Investigating the Website

* `/wordpress`/ suggests we have a very fertile ground for planting an attack. Wordpress admin access = shell.
* `/phpmyadmin/` suggests there is a database ready to plunder.
* `/info.php` gives us Kernel, hostname and OS information immediately.

 I dove right in with
 * `wpscan -u lazysysadmin.ctf/wordpress/ -e u,v,p`

 and browsed it myself while it ran.

<br>
![A8](https://i.imgur.com/y3FdRcQ.png)
<br>


`wpscan` found a bunch of potential vulnerabilities - but I like to start simply.
The default username is in use, so maybe the default password will be as well.
The only blog post is just *my name is togie* 50 times - so maybe thats the password?

<br>
![A7](https://i.imgur.com/Qzbpp70.png)
<br>

I tried `admin:admin`, `admin:togie`, `admin:mynameistogie`, and `admin:password` as well as a few capitalized versions.
No hits. I think it might be time to look at the other services and see if there is anything there.





### 3. Investigating the Other Services

SMB might hold some interesting information, and I know that it is running Linux, so I used `enum4linux`!

* `enum4linux lazysysadmin.ctf`

<br>
![A9](https://i.imgur.com/mbp5PxY.png)
<br>

Right off the bat, `[+] Server lazysysadmin.ctf allows sessions using username '', password ''` looks very promising.


It is shortly followed by `//lazysysadmin.ctf/share$	Mapping: OK, Listing: OK`


We connect with a simple

* `smbclient //lazysysadmin.ctf/share$`

and just press enter when asked for the password.

<br>
![A10](https://i.imgur.com/ffMwrLT.png)
<br>

What looks interesting or new... hmm.
`todolist.txt`, `deets.txt`... and ooh! A Wordpress directory! That might have our website setup in it!

* `cd wordpress`
* `ls`

<br>
![A11](https://i.imgur.com/CzDqWaI.png)
<br>

`wp-config.php` has given me passwords, or at least hashes, in the past. Let's check it out.

* `get wp-config.php`
* `exit`
* `less wp-config.php`

<br>
![A12](https://i.imgur.com/NyhJkrk.png)
<br>

*Why, hello there...*

Let's try this password at `/phpmyadmin`, and maybe we can drop a shell using a `select ... into outfile`!

Visit `/phpmyadmin` and try logging in with `Admin:TogieMYSQL12345^^` - and get db access!



## 4. Plundering the SQL Database

Alright, let's see what goodies we can find.

<br>
![A13](https://i.imgur.com/LOLAntO.png)
<br>

wp_users looks appetizing, let's view it.

<br>
![A14](https://i.imgur.com/JtNDX0O.png)
<br>

Hmm... maybe not.
I tried logging out and back in with `root:TogieMYSQL12345^^` - but to no avail.

There is the option to try manually inputting SQL commands, so let's try that.

First thing I want to see is if I can write a file - that could allow me to drop a shell script and gain access.
I try

* `select 'hello' into outfile 'hello.txt'`

<br>
![A15](https://i.imgur.com/XG0ry11.png)
<br>

Hmm, this user has pretty limited privileges.  Might be worth trying to view the db contents using SQL commands, however.

* `select * from wp_users from wordpress`

was forbidden, so I tried the more specific

* `show columns from wp_users from wordpress`

<br>
![A16](https://i.imgur.com/3D9gBmG.png)
<br>

`user_pass` looks good to me, and some of the other fields aren’t bad either. I made sure the wordpress database was selected in phpmyadmin, then ran

* `select user_login, user_pass, user_nicename, user_email from wp_users`

<br>
![A17](https://i.imgur.com/OvAtVBq.png)
<br>

Great, a hash! Maybe we can crack it. <br>
I wonder if it’s the same as the MySQL password…

Wait. <br>

We can try just try logging in with that password to the wordpress site.
Should have tried that first!

<br>
![A18](https://i.imgur.com/aJmC9LI.png)
<br>



## 5. Wordpress Admin Access

It’s a gift to be simple…

Let’s go ahead and drop a php shell into one of the plugins.

I’ve messed this process up before – so I always make a copy of the original text before trying any alterations. This also allows us to revert the plugin back to normal if we give ourselves another way in later.

From `/usr/share/webshells/php/` I grab what I like to call the *"monkey shell"* and make a copy in my pentest directory.

I know the plugin code already includes <?php and ?> flags, so I chop those off and edit in the correct IP and port information.

<br>
![A19](https://i.imgur.com/Fs3apdi.png)
<br>

Then, we just paste it in at the end of our plugin code.

<br>
![A20](https://i.imgur.com/GiASGXe.png)
<br>

Now we can start our listener with
* `nc -lnvp PORTNUMBER`

and hit *"Update File.""* My shell didn’t pop immediately, but after I clicked *"Installed Plugins"* again in order to make sure the plugin was active.

<br>
![A21](https://i.imgur.com/3PjSoOB.png)
<br>

We've got a shell, and it is time to enumerate!




### 6. Enumerating With `www-data`

Enumerating a system from the inside can sometimes feel like looking for a needle in a haystack. To be honest, if I lost a needle in a haystack, I’d just go look for another needle. But if I didn’t have another needle, I’d try to use a magnet. That can’t help us here, however.


The first thing I tried was looking in the `/home` folder for goodies left by users.

<br>
![A22](https://i.imgur.com/tz9pCNy.png)
<br>

Nothing here – I usually go right for the bash history to see what the user had been working on, but there isn’t one here.

Might as well upgrade my shell, too. Let’s see if we can run Python, then we might be able to run the python pty.

* `python -V`

returns a version - the box runs python. Let's get a TTY.

* `python -c ‘import pty;pty.spawn(“/bin/bash”)`

<br>
![A23](https://i.imgur.com/2WW8u9w.png)
<br>

After taking a quick peek at `/etc/passwd` to see all the usernames, I decided to see if I saw all there was to see on the website.

* `cd /var/www/`
* `ls`
* `cd html`
* `ls`

<br>
![A24](https://i.imgur.com/Htosvmy.png)
<br>

Oh look, the files from the SMB share are here too. Hmm, I got so excited by the wordpress directory that I didn’t even give these a thorough look.

* `cat todolist.txt`
* `cat deets.txt`

<br>
![A25](https://i.imgur.com/AE0A4kd.png)
<br>

Well, well, well. Togie did NOT seem to make it so users can’t see the web root – considering we could see into this directory via SMB. This suggests that togie did NOT remember to change the password either…

Could this have been laying here the whole time?

* `su togie`

Use password `12345`

<br>
![A26](https://i.imgur.com/AE0A4kd.png)
<br>



### 7. Call Me Togie!

This `togie` user seems to be the LazySysAdmin in question. I wonder if they can run anything as superuser.


* `sudo -l`

<br>
![A26](https://i.imgur.com/Lco9hyd.png)
<br>

Well, it looks like Togie *can* run some things as superuser.<br>
In fact, Togie can run *every* thing as superuser.

* `sudo su`


### 8. Did I say Togie? Sorry, my name is Oot. Richard Oot.

For presentation’s sake, I logged into SSH – but this is entirely optional and leads another entry in a log file. While doing the CTF myself, I continued to use the reverse shell.

Seeing that little `#` brings a smile to my face.

* `cd ~`
* `ls`
* `cat proof.txt`

<br>
![A27](https://i.imgur.com/EkirdBa.png)
<br>



## Lessons Learned
* Try using passwords in multiple locations, password reuse is rampant.
* If you have access to a group of files, READ THEM. At least grep for "pass\[word\]"
* Don't leave your web root in a publicly accessible SMB.
* Don't leave your root password lying around! Or any other for that matter!



## Togie's Questions:


__What could you have done to speed up the enumeration process?__

 I dove pretty deeply into HTTP first out of habit, instead of starting with a shallow wade-through of all possibilities.
In this case, it paid to do a breadth-first enumeration process instead of a depth-first process.

__Are there any obvious things that you missed, which you shouldn't have missed?__

 I didn't look at all the files I had access to on the SMB. I found one lead and then just chased it.
 Had I taken the time to look at all the files, I would have saved a lot of time.

__Did you learn anything interesting?__ and __What have you added to your enumeration process to prevent you from wasting time?__

The answer to both of these questions is related to breadth-vs-depth searching. I didn't even realize that I was diving deeply instead of lightly browsing everything first.




## Thanks Togie McDogie!
Good Luck on your next OSCP!

<br>
[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To Guides/HowTo-index.html) <br>
󠁈󠁔󠁂󠁻󠁴󠁲󠀱󠁴󠁨󠀳󠁭󠀱󠁵󠀵󠁟󠀱󠀴󠀹󠀹󠁽
