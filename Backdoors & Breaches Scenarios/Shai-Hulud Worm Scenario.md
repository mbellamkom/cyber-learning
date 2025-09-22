
_Note: This scenario was more heavily drafted with Gemini than usual since the attack chain wasn't quite clicking for me before brainstorming with the AI._
# Background

You're part of Red Cherry, a game studio about to launch a new multiplayer rpg. The selling point is the new procedural engine built on open source libraries such as voxel-kit. (_Note: This is apparently, a real library._)

Your lead developer, Joseph, is the maintainer of this library and one day you get an alert for an unusual github commit from them. Further investigation revealed that the IP and client didn't belong to anyone in the company.


# Attack Chain 

_Note: All of this is automated, so the actual threat actor isn't doing much here._

## Initial Compromise - Supply Chain Attack
_Note: Endpoint Protection Analysis, Endpoint Analysis, or Network Threat Hunting will turn this card over._

Joseph received a legitimate looking email from nmp (Javascript package manager) detailing changes to their MFA login options. The email requested developers to update to the latest version. He clicked the link, entered his creds, and downloaded the update. The update contained a malicious payload. 

The payload executes and immediately starts looking for .npmrc tokens (what nmp uses to authenticate), environmental variables and config files for cloud service AIP's and GitHub Personal Access Tokens. This happens on both the host and any continuous integration environments it's linked to by using tools like TruffleHog and querying metadata endpoints.

_Note: According to [Unit 42's writeup](https://unit42.paloaltonetworks.com/npm-supply-chain-attack/), this email may have been AI generated. Not a surprise, but it is more evidence that AI is becoming more involved in cyberattacks. Also, at this point, Joseph needs to get more security training. Red Cherry might want to review its training policies and frequency._
## Pivot and Escalate - Credentials Exposed in Environmental Variables

_Note: SIEM, Cloud Event Log Analysis, Cyber Deception, and User Entity Behavior Analytics will turn this card over. 

_Using SIEM:_ The SIEM flags a single process making an unusually high number of queries to config files like `.npmrc` and `~/.aws/config`. This is not Joseph's typically behavior. 

_Using Cloud Event Log Analysis:_ The log shows an API call, `sts:AssumeRole`, from an unidentified IP. The credentials belong to Joseph. 

_Using Cyber Deception:_ The attacker attempts to authenticate to one of your honeypot databases. 

_Note: if the players decide to go for isolation immediately after this and roll successfully, then the rest of the cards are cleared and they've won because they've stopped the rest of the chain from occurring. They've satisfied the win condition in this case.  

_Using User Entity Behavior and Analytics_: Investigation correlates the usual git activity to an API request to a cloud service that Joseph has never used before. Upon talking to Joseph, you realize he's never even heard of this service. 

## Exfil - Living off the Cloud & Snapshotting Resources
The SIEM reveals that the malicious commit was cloud credentials being published to a github that no one in the company owned. 

_Note: Living off the Cloud is the initial exfil. The successful SIEM (Cloud or non-cloud) roll will turn this card over first The Snapshotting Resources card can also be turned over using the Active Defense and Cyber Deception card, but the detections deck I'm using only has the Cyber Defense Card._

Further investigation reveals a high severity alert. Someone has used Joseph's github access to commit a database snapshot to an AWS account. No one in your company owns this account either. 

## Persistence - Malware Injection into Client Software
_Note: Endpoint Analysis, User and Entity Behavior Analysis, and Endpoint Security Protection Analysis will turn this card over as well._

Investigation of Joseph's code base reveals that his npm token was used to authenticate to the npm registry. The worm was automatically injected into his other packages which were then published to the registry allowing the worm to propagate. 




# Resources
- Unit 42 writeup - https://unit42.paloaltonetworks.com/npm-supply-chain-attack/
- Aikido writeup - https://www.aikido.dev/blog/s1ngularity-nx-attackers-strike-again
- Voxel-kit - https://dl.acm.org/doi/10.1145/3706598.3713948
- Official Backdoors & Breaches rules - https://www.blackhillsinfosec.com/tools/backdoorsandbreaches/