Note: OvertheWire requests that writeups don't contain credentails, so there are none in here. 

**If you haven't done this level, go do it and then come back!** 


[Level 0 -1 ](https://overthewire.org/wargames/bandit/bandit1.html)of Bandit is fairly simple. The filename, directory, and a list of suggested commands are given to you. 

I haven't used Linux since finishing my Cybersecurity Technology degree in 2022, but [find](https://manpages.ubuntu.com/manpages/noble/man1/find.1.html) seemed self-explanatory. I did have to go look up the[ cd command  ](https://manpages.ubuntu.com/manpages/noble/man1/cd.1posix.html) as I couldn't remember how to move directories. 

Then, I did  `cd readme` instead of `cd home`, so of course the message I got back was that the directory didn't exist. Then when I did `cd home`, the message I got was that there was no such file or directory.  

Then, I did `find "readme"` which just brought back readme, so clearly I was in the right directory all along.

So, I tried `cat readme` which concatenates files and prints the output. In this case, since there was no other file to concatenate readme with, it just displayed the contents of readme and I got the password.
