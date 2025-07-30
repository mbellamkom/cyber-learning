Context: Game company making games and game support systems. 
Key systems:
- **HustleShield**: a proprietary anti-cheat engine built in-house, now licensed by several other studios to protect competitive multiplayer environments
- **HeatmapVision**: an internal tool for analyzing player behavior across maps and scenarios, often used to guide design decisions
- **Greasy Tender (GT)**: the in-game currency at the core of Hamburger Hustle’s economy, with real-world value tied to virtual transactions
- **Limited-Edition Skins**: rare cosmetic items that players can unlock or trade, some of which have gained collector status
- **DevNet Sync**: the company’s private sync and telemetry infrastructure used to test unreleased features across distributed teams

Message received from partner licensing HeatShield about suspicious activity coming from IP 185.210.94.2.

Information about IP's should be in the Inbound Network Events table. Using 

```
InboundNetworkEvents
|take 10
```
we can see what fields the table contains. 

![](<./assets/Threat Intel CTF based on APT 41 Notes/Pasted image 20250719124153.png>)
src_ip is the one we need. 

Using the query

```
InboundNetworkEvents
| where src_ip == "185.210.94.2"
```
we find that there are 24 records of the IP.

Now, we're looking for how many successful login attempts there were from 185.210.94.2.

```
AuthenticationEvents
|take 10 
```

![](<./assets/Threat Intel CTF based on APT 41 Notes/Pasted image 20250719125653.png>)
and running 

```
AuthenticationEvents
|where result == "Successful"
|where src_ip == "185.210.94.2"
```
doesn't give us anything
![](<./assets/Threat Intel CTF based on APT 41 Notes/Pasted image 20250719125808.png>)

We're mapping this IP to a domain by using the PassiveDNS table

Running the following query, shows us that the IP maps to telemetryfinancegaming.net

```
PassiveDns
| where ip == "185.210.94.2"
```

Looking up the domain in the PassiveDNS table shows us 3 distinct IPs that all resolve to the same domain.
Running a query on those IP's
```
PassiveDns
| where ip == "185.210.94.4" or ip == "185.210.94.4" or ip == "185.210.94.3"
```
gives us 4 distinct domains 
![](<./assets/Threat Intel CTF based on APT 41 Notes/Pasted image 20250719131408.png>)

Intel gives us 3 well known groups to look out for. 

### 🧑‍💻 Brass Typhoon

- Targets gaming, finance, and technology
- Known for dual-purpose ops: espionage and financially motivated attacks
- Common TTPs include spearphishing, credential dumping, and use of living-off-the-land binaries
- Frequently uses spoofed business-related infrastructure

### 🐼 Velvet Panda

- Typically targets NGOs, government, and think tanks
- Known for spearphishing with lures tied to current events
- Uses PlugX and similar remote access tools
- Sometimes uses infrastructure with .cn domains and generic names

### 🐉 Granite Typhoon

- Historical focus on espionage and intellectual property theft
- Associated with watering hole attacks and backdoored websites
- Uses web shells, DLL sideloading, and spoofed corporate domains
- Targeted a wide range of industries, including tech and defense

Knowing what techniques these threat actors use means that we have a place to start. We can check the Email log to see if any emails contained links to these suspicious domains. 

Query used:
```
Email
| where links contains "telemetry-finance-gaming.net"
or links contains "gaming-telemetry-finance.cn"
or links contains "telemetryfinancegaming.net"
or links contains "gaming-telemetry-finance.com"
or links contains "gamingtelemetryfinance.cn"
```
![](<./assets/Threat Intel CTF based on APT 41 Notes/Pasted image 20250719133828.png>)
It looks like 4 employees received these links. 

These 4 employees also received suspicious files attached to the email. Running a query for these files

```
Email
| where attachments contains "GreasyTender_Audit_Report.pdf" or attachments contains "GT_TelemetrySync.log.pdf"
```
shows us that 10 employees in total received these files. 

Out of all the 10  employees that received these files, there were only 3 distinct roles targeted: Monetization & Economy Designer, Fraud Prevention Specialist, and Game Currency Analyst

![](<./assets/Threat Intel CTF based on APT 41 Notes/Pasted image 20250719135538.png>)

The 10 employees that had the attachments on their machine are the same employees that received the file. 

Part 3 Question 2 - persistence technique: scheduled tasks
```
ProcessEvents
|where process_commandline contains "schtasks"
|where timestamp between(datetime(2024-03-09 09:12:47)..datetime(2024-03-21))
```