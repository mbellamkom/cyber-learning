_Note: This module was first shown in the July KC7 Threat Hunting workshop._

Based on an actual case, an independent video game studio received a message from a partner company, licensing their proprietary anti-cheat system, about a high amount of recon traffic from a certain IP address. The video game studio, Convoy Street Interactive, has 5 key systems related to their games. 

1. HustleShield - an in-house anti-cheat system licensed by several other game studios
2. HeatMapVision - an internal tool used to visualize player behavior across maps and scenarios
3. GreasyTender (GT) - the in-game currency used in Convoy Street Interactive's games, tied to real-world monetary value. 
4. Limited-edition skins - cosmetic items that players can unlock or trade
5. DevNet Sync - private sync and telemetry infrastructure used to test unreleased features

# Part 1 - Detection

A partner company sends Convoy Street Interactive a message about a high amount of recon traffic coming from the IP address `185.210.94.2` and suggested that the company check their logs. 

Running an initial `.show tables` query gives us all of the tables in Convoy Street Interactive's system. 

![Pasted image 20250720092229.png](../Pasted%20image%2020250720092229.png)

NetworkFlow and DeviceInfo aren't likely to be tables we need, so we can set those aside for now. Since, we're looking for traffic originating from `185.210.94.2`, we should look in the InboundNetworkEvents table. 

Running a take 10 query on the InboundNetworkEvents table 

```
InboundNetworkEvents
|take 10
```

gives us the following fields

![Pasted image 20250719124153.png](../Pasted%20image%2020250719124153.png)

Since we want to search by the IP address, we should use the `src_ip` field. 

Using the following query:

```
InboundNetworkEvents
| where src_ip == "185.210.94.2"
```

we find that there are 24 records of the IP. 

![Pasted image 20250720092843.png](../Pasted%20image%2020250720092843.png)

It looks like someone has been using this IP to search for Greasy Tender transaction logs, high value player accounts, and telemetry schema. Most of the searches are related to Greasy Tender, Convoy Street Interactive's in-game currency, or account credentials. With this, we can confirm that there was recon traffic on Convoy Street Interactive's end as well. 

Since they were looking for credentials, they may have tried to log in. Looking at the tables list from the `.show tables` query, AuthenticationEvents looks like a good table to check. 

Running 

```
AuthenticationEvents
|take 10
```

pulls up the following headers

![Pasted image 20250719125653.png](../Pasted%20image%2020250719125653.png)

`take 10` is a query that pulls up 10 random rows of a table. It's a good query to run when you're not sure what information a table contains or what the table headers are. 

Since we are looking at authentication events from the IP `185.210.94.2`, we will be searching by the `src_ip` again. 

Query:
```
AuthenticationEvents
|where src_ip == "185.210.94.2"
```

results in a blank table. 

![Pasted image 20250720094740.png](../Pasted%20image%2020250720094740.png)

It doesn't look like they tried logging in with that IP address at all. That doesn't mean they don't have any usable credentials however, they just haven't tried to log in with them. Since the recon traffic relates to the company's in-game currency, however, we should keep looking. 

Another thing we can check with the IP is the DNS. 

In our list of tables, there is a table called PassiveDNS. Passive DNS is a technique used to log DNS data as they occur. Unlike cached DNS data it's not used to answer user queries, but is instead stored for historical and security purposes. So looking at the PassiveDNS table should give us an idea of where the IP is coming from. 

Running the `take 10` query gives us the following headers

![Pasted image 20250720103500.png](../Pasted%20image%2020250720103500.png)

Interestingly, this table doesn't use `src_ip` like the other tables and uses `ip` instead. 

So our IP search query changes to
```
PassiveDns
|where ip == "185.210.94.2"
```

*Note: Table names are case sensitive in KQL, so PassiveDNS will return an error.*

and pulls up 1 result. 

![Pasted image 20250720104105.png](../Pasted%20image%2020250720104105.png)

In context of the searches done by the `185.210.94.2`, this domain looks suspicious. Let's run a search on the domain name and see if we get any other IPs.  Multiple IPs can point to a single domain. This is commonly used for larger domains such as google.com to balance server load. However, in this context, it could be a potential sign of distributed attacker infrastructure since this is inbound traffic. 

Searching for the domain

```
PassiveDns
| where domain == "telemetryfinancegaming.net"
```

pulls up 2 other IPs associated with it. 

![Pasted image 20250720105740.png](../Pasted%20image%2020250720105740.png)

Searching for those other IPs gives us 4 distinct domains. 

Query:
```
PassiveDns
| where ip == "185.210.94.4" or ip == "185.210.94.4" or ip == "185.210.94.3"
```

![Pasted image 20250719131408.png](../Pasted%20image%2020250719131408.png)

Interestingly, one of those domains `.cn`  is a country-specific domain. 

# Part 2 - Context
The findings get passed over to your intel team. This case doesn't specifically state what information would be passed over, but you typically want to give your intel team the who, the what, the when, and the where. In our case, the information would be something like the following:

- Who: IP/Domain doing the recon traffic
- What: the log of the recon traffic itself
- When: Any and all timestamps related to the IP or recon traffic
- Where: location where the information was found, which in this case is the InboundNetworkEvents and PassiveDNS tables

Your intel team comes back with 3 threat actor groups (Black Typhoon, Velvet Panda, and Granite Typhoon) that could potentially be the one looking at our network. All three are based in China (.cn is the country-level domain for China), however only Black Typhoon focuses on the gaming, finance, and technology sector...3 areas that Convoy Street Interactive is involved in, so Black Typhoon is most likely the threat actor group behind this. 

_Note: At the end of the workshop, we're told that the case was based off a real case involving Black Typhoon._

Black Typhoon's techniques, tactics, and procedures (TTP) typically involve spearphishing, credential dumping, and using living-off-the-land binaries. Spearphishing is a phishing attack targeted at a specific person and credential dumping is when a threat actor steals your credentials and uses them for other malicious activities like moving into another system on the same network. Living-off-the-land binaries is a technique attackers use binaries to mimic native system binaries. This allows them to bypass detection.

Living-off-the-land sounds like a later stage technique, so we can set that one aside for now. Credential dumping could be something to look for, but the threat actor needs to find them first. At this point, we should focus on looking for the initial compromise. Phishing is a common initial compromise technique and since Black Typhoon, typically uses spearphishing, we should look in our Email table for any suspicious activity.

Phishing attacks often involve malicious links, so we can search the Email table for the domains we found earlier. 

```
Email
| where links contains "telemetry-finance-gaming.net"
or links contains "gaming-telemetry-finance.cn"
or links contains "telemetryfinancegaming.net"
or links contains "gaming-telemetry-finance.com"
or links contains "gamingtelemetryfinance.cn"
```

It looks like 4 employees received emails containing at least one of these links. 

![Pasted image 20250724111639.png](../Pasted%20image%2020250724111639.png)

Those 4 employees also received attachments, one of which was called `GreasyTender_Audit_Report.pdf`, which sounds like a file you'd expect to receive if you worked in finance. There was also another file sent to the employees called `GT_TelemetrySync.log`.

The next step is to check whether any more employees got those files. Running the following query, pulls up 10 records. 

```
Email
| where attachments contains "GreasyTender_Audit_Report.pdf" or attachments contains "GT_TelemetrySync.log.pdf"
```
![Pasted image 20250728144123.png](../Pasted%20image%2020250728144123.png)
Note that the email verdict here is clean. Since we suspect spearphishing, the emails are most likely not clean, so this is probably a false negative.  A possible reason for a false negative could be that the email security wasn't configured properly and marked anything with a non-executable attachment as clean. The links in the email could have also been time-delayed, so when email security scans them, they lead to a legitimate website. However, when an actual user clicks on them they're given the malicious website. This type of attack, called a [SiteCloak](https://emailsecurity.checkpoint.com/blog/microsoft-safelinks-under-attack) attack can bypass things like Microsoft's Safe Link's feature. These types of attacks are difficult to prevent, but one possible way is to scan the links twice, once when the email comes in and once when the user clicks on the link. 

However, KC7's modules take place in a single dataset, so there's no telling what actually occurred in this scenario. However, it is good to be aware of the ways email security can be bypassed. Now, back to the investigation. 

We note that the timestamp of the first email was `3/8/2024, 4:39:47 PM`. This is useful for narrowing down our search and establishing a timeframe as we can exclude anything from before that date and time. 

The first employee targeted by these emails was Peter and we know 10 people in total were targeted by these emails. So, what do they have in common aside from working for the same company? Let's look in the Employees table. 

As always, let's run a take 10 query to find our fields. 

```
Employees
|take 10
```

![Pasted image 20250729111326.png](../Pasted%20image%2020250729111326.png)

We have a lot of headers here, but the most likely way to figure out what all of the employees had in common is by looking at the role and, possibly, hire-date columns. However, role seems more likely since the recon traffic was finance related. 

Since we have all 10 employee email addresses, we can search by the email_addr field.

```
Employees
| where email_addr == "wes_mantooth@convoyinteractive.com"
or email_addr == "sherman_hughes@convoyinteractive.com"
or email_addr == "robert_mitchell@convoyinteractive.com"
or email_addr == "peter_orourke@convoyinteractive.com"
or email_addr == "jerry_dougal@convoyinteractive.com"
or email_addr == "ellen_stevens@convoyinteractive.com"
or email_addr == "dorothy_gearheart@convoyinteractive.com"
or email_addr == "david_moreau@convoyinteractive.com"
or email_addr == "daryl_martin@convoyinteractive.com"
or email_addr == "cathy_brooks@convoyinteractive.com"
```
![Pasted image 20250729112328.png](../Pasted%20image%2020250729112328.png)
All of these roles look finance related, so it makes sense why they were targeted. Now we need to figure out if any of them interacted with any part of the email. 

Checking the OutboundNetworkEvents table shows us that several of them clicked the link in the email and entered in their login information.

```
OutboundNetworkEvents
| where url contains "gamingtelemetryfinance"
```

![Pasted image 20250729133221.png](../Pasted%20image%2020250729133221.png)

