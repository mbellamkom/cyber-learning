
## Scenario Background
You all work for a relatively small cryptocurrency business. This organization prides itself on making sure their transactions are safe and secure, handling millions of customer assts using their own proprietary trading algorithm. 

## Initial Alert
One day, your SIEM detects a DNS query for a TXT record. This query was made to a subdomain of [herokuapp.com](http://herokuapp.com), a legitimate platform-as-a-service, but one that's known to be used by threat actors. This DNS query originated from a powershell.exe process

## Attack Chain
1. Initial Compromise - Social Engineering
The attackers used the company's contact us form to send in an inquiry about the company's preparedness for an AI future (poses as a legitimate LLC). After the company initiated the conversation, they went back and forth with the threat actor for about 2 weeks on how the threat actor's company could help them implement AI. Then, at the end of the conversation, the threat actor proposed a partnership and sent over a _zip file_ containing relevant documents. The zip file consisted of _pdfs, doc files, and a lnk file_.

The employee downloads the file and opens the doc/pdf files. 

_Note: This current campaign used "AI Transformation" as a pretext, but attackers using this methodology could use any recent industry relevant trend._

2. C2 & Exfil/Persistence - DNS as C2; Malicious Service  (Persistence)
The powershell script the employee downloads, contains a malicious _lnk_ file which triggers a powershell loader that deploys _MixShell_ (powershell variant), which communicates with the attacker using DNS text record queries (sends data out as an exfil mechanism).

This link initially pointed to a subdomain of _herokuapp.com_, a legitimate Platform-as-a-Service org that allows you to host web applications. However, any legitimate hosting site can most likely be used for this purpose. 

TypeLib Hijacking that persists using explorer.exe as explorer is always trying to load some TypeLib libraries. 


3. Pivot and Escalate - Broadcast/Multicast Protocol Poisoning 
_Note: the writeup didn't really mention a pivot and escalation method, so this one I made up. Since the attacker was using mixshell, they might try to pivot and escalate using the same methodology._

Along with MixShell, the script also deployed Responder, a tool used to poison Broadcast/Mulitcast Protocols. The tool listens for broadcasts and tries to authenticate to the broadcast sender's machine if it hears a broadcast. This captures the password/password hash and can be used in a pass-the-hash attack to gain access to another machine. 


## Resources
Writeup  the scenario was based on - https://research.checkpoint.com/2025/zipline-phishing-campaign/
TypeLib Hijacking - https://cicada-8.medium.com/hijack-the-typelib-new-com-persistence-technique-32ae1d284661
Pass-the-Hash Attack - https://en.wikipedia.org/wiki/Pass_the_hash?oldformat=true