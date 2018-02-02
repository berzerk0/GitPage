[Main Page](../index.md)
<br>

# Probable Password Trend Analysis

![Logo](https://raw.githubusercontent.com/berzerk0/Probable-Wordlists/master/ProbableWordlistLogo.png)

<br>

### Updated: 31 Jan 2018

<br>

## Overview

This document will show *why* you need a unique, secure password. It then will describe what the *least* unique passwords have in common, and why they are catnip for password cracking software. Finally, I'll show you how to create a more unique password that buck the trends while still being memorable.


In order to get a better idea of the trends of secure and insecure passwords, I ran a portion of my [Probable Wordlists](https://github.com/berzerk0/Probable-Wordlists) through [DigiNinja's Pipal](https://digi.ninja/projects/pipal.php) analysis tool.


If you'd like to get right to the findings, scroll right to the __Results__ section.


__Caveat:__ <br>
Following this guide will make it harder for an attacker to compromise your password via brute force. However, there may be no way to become 100% secure.
A person dedicated to determining a password has many tricks at their disposal. Sophisticated threat actors will use a combination of methodologies in this process. A password that is extremely resistant to brute forcing may still be stolen via social engineering or malware. Always exercise caution.


## Background

### What Can A Password List Tell *Me*?

A giant list of passwords *sounds* useful, but what can I *do* with it?

You could search through it for your passwords. If they're found, you rush to change them. If they aren't, you think *"Phew, that's a relief."* In both cases, you are likely to simply move on and stop thinking about password security.

You might share the list with a few friends, or you might revisit the list the next time you come up with a new password, but it's entirely possible that you don't continue to give your passwords much thought.
We have a lot to worry about, and not spending days worrying about password security is hardly a personal failure.


It's possible that the only time syou think about how secure or unique your passwords are, or even consider password security as a whole, is when you hear about leaks. A headline will appear talking about how a record-breaking breach resulted passwords blasted all over the internet. Experts appear and tell us that foreign governments and gangs are after our information and that the sky is falling. You might get concerned for an afternoon, but what else can be done? We know that identity theft happens, but that's what the protections at our banks are for - right? They've got it all under control, of course! Surely, the sky isn't really falling - why not leave this problem to "the cybersecurity people?"


The truth of the matter is that "the cybersecurity people" are working hard to make the internet a safer place. Security is a complex problem, however, and everyone can play a role. User participation in cybersecurity measures need not be intense or even difficult. We can find parallels in other aspect of our lives. Allow me to create an overly dramatic analogy.


### A Quick Trip To The Gym Ends With Shame On Your Family

Consider a gym locker. For this analogy, our locker is a simple box with a loop to thread your lock into. It doesn't have a built in combination dial or key you pick up from the front desk. After doing your research, you've purchased a brand new, top-of-the line combination lock. It really is a marvel of modern locksmithing. There is no such thing as an impenetrable lock, but this one is pretty close. You have reinforced titanium parts that can't be cut without industrial-strength equipment, a bulletproof mechanism that can't be defeated without hours of tedious work, and plenty of other, easier targets organized into neat rows nearby. Your lock seems pretty secure, and you feel comfortable that the solid-gold pocketwatch you promised your grandfather on his deathbed to never leave at home is going to be there when you finish your workout.


What would your kind, wise, old grandfather think if he found out someone had managed to get through this carefully crafted security system, seizing the priceless family heirloom because his grandchild had used a combination of `1234`?


Here, the "security people" are locksmiths. They did everything they could to create a secure system for you to protect your belongings. Measures had been taken against picks, shims, jimmying, prying, hammers, and scores of other methods of defeating their security system, only for all of them to be rendered useless by an owner who didn't want to take an extra moment to come up with an uncommon combination. An owner who now, after returning to an empty locker, needs to walk home in the rain without their jacket, pray someone is at home to open the door for them, and then prepare to call grandma and tell her that a piece of the family's soul is probably sitting in a pawn shop window.


### Actionable, Data-Supported Advice

I did say the analogy was going to be overly dramatic, but I hope the implications are clear. __The best security systems can be defeated if you are careless.__ You don't have to become a certified cybersecurity expert, but a little bit of effort can make a *huge* difference here. Use the right password generation techniques and it will be *astronomically* harder for anyone to brute force your password.


 Armed with [DigiNinja's Pipal](https://github.com/digininja/pipal), I performed an analysis on the list of the Top 32 Million Most Common Passwords. This isn't the largest list in the repository, but it is sizable enough to encompass notable trends. Pipal provided valuable data to interpret and chart, These findings provide actionable, data-supported information that is useful to everyone, even if your passwords weren't on the list.


Passwords included in this analysis appeared at least 10 times in the [Probable Wordlists](https://github.com/berzerk0/Probable-Wordlists) __Rev 1.2__ Files of the Probable-Worl


## Results

Advice based on data from each analysis are included on the charts themselves.

<br>
![Password Length Trends](https://i.imgur.com/KAYUHj3.png)
<br>


<br>
![Password Character Use Trends](https://i.imgur.com/Y5adW1d.png)
<br>


<br>
![Password Character Order Trends](https://i.imgur.com/ei66PJO.png)
<br>




### The Power of Hashcat and Other Cracking Tools

No password is uncrackable, period. <br>

Modern password cracking tools are very powerful.
However, a password that is strong, unique and bucks common trends will be the hardest to guess via brute force masking and dictionary attacks.

The logic here is simple: if your password isn't on an attacker's wordlist, it cannot be guessed via dictionary attack. If your password doesn't match the common patterns it will take a brute force attack much longer to reach it.

Let's look at an example: `password17`
<br>

If a password cracker is brute forcing passwords using a pattern of eight lowercase letters followed by 2 digits then our password is going to be guessed. A simple brute force attack starts at `aaaaaaaa00` and covers all possibilities until it gets to `zzzzzzzz99`. Along the way, it will land on `password17`. This might not happen quickly, but it will happen.


 Software like Hashcat has the ability to use hybrid attacks that combine a dictionary and a brute force mask. If this dictionary + mask attack was using a list of the most common passwords and appending a 2 digit number on to the end, then `password17` doesn't stand a chance! At the number 2 most common password, it would take just 117 guesses *(all 2 digit numbers appended to the No. 1 most common password + 17 guesses using password)* to hit the bull's eye. Keep in mind that this does not mean a cracker is manually typing in `password00`, `password01`, `password02`, until they get a hit. An algorithm, often accelerated by a GPU performs this process in milliseconds.


 The dictionary word + numbers format is very common, and guessing password candidates that match this pattern may be one of the first strategies a cracker might employ. The ability to append numbers at the end of words is included in the Hashcat default "rule" set. Rules create mutations of wordlist items that are used in a dictionary attack, and will create candidates based on strategies like swapping every letter `S` for a `$` and other common substitutions.


This mutation functionality may render passwords that may *appear* to be secure due to character choice and character order easily guessable. `password` can be mutated into something like `P@$$w0rd` - which some password "strength" algorithms might rate as strong. It's got capital and lowercase letters, special characters and numbers! How couldn't it be secure?! Well, easily.


The l33tspeak phenomenon has been around for some time now, and it doesn't take a Nobel prize winner to guess that `s` can be swapped for `$`. Theoretically, almost \@ny $3nt3nc3 c@n b3 \\/\\/r1tt3n 1n 1337 and still be legible. If you can read that, then you can figure out how to write a a rule to create mutations yourself. (If you couldn't read it, the l33t section reads *"any sentence can be written in leet"*)


Mutations of dictionary words and other common passwords are easy to guess, and shouldn't form your entire password.


<br>

### Make Your Password Buck the Trends With These Tips:

Let's walk through the process of taking a weak password, based on a dictionary word, and altering based on the advice based on the data from this analysis.
When we are done, it will be statistically much harder to guess. However, __don't use our final product as your password!__ Here it is published online!

Starting password: `noodle`

* About 90% of passwords used SOLELY lowercase letters and numbers. 47% used lowercase letters alone.

`noodle17`

* Special Characters appeared in fewer than 4% of the passwords
* Uppercase Letters appeared in fewer than 1%!
* 35% of passwords solely used letters followed by numbers
* Use special characters and capital letters! Hit that shift key!

`Noodle-17`

Just 0.02% of passwords on the list began with special characters.

Character order matters!

`-Noodle17`

* More than 91% of passwords had lengths between 8 and 14 characters. 23% of the passwords used exactly 8 characters.
* About 40% of passwords ended in a number
* Make 13 or 14 your minimum password length and you'll be like the rare 9%
* Pad your password with special characters and relevant words using capital letters to make it longer and even harder to guess

`-Noodle-17-SOUP-`

* The most "natural" way to include a capital letter is to capitalize the first letter of the dictionary word. Don't let that be your only capital! If it's natural for you, it would be natural for the crackers as well.


We began with started with `noodle`, and ended up with `-Noodle-17-SOUP-`, which includes  lowercase letters, numbers, capital letters (and not only at the beginning) and "friendly" special characters. Not all systems allow for things like spaces, stars, and ampersands to be part of a password. This results in the very infrequent use of these characters in passwords. This is due to the handling of the password systems themselves, which might use some special characters for control.



`-Noodle-17-SOUP` is much harder to crack than `noodle`, and is even harder to crack than `n00dl3$0UP` which might be mutated out of dictionary words.


<br>

## Thanks for reading!

<br>
[Main Page](../index.md)
