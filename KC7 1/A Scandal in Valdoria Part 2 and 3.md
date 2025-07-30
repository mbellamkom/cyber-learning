
# Incident Overview:

Shortly, before Valdoria's next mayoral election, an unauthorized article was published, without any editorial review, by the Valdorian Times.

# Response
Editorial director, Nene Leaks, immediately contacted a cyber incident responder to determine how the unauthorized article was published. 

## Information Obtained
According to Ms. Leaks, an article about the candidates was approved, but it was not the one that got published. Ms. Leaks also suggested talking to the Newspaper Printer, Clark Kent, who is the last person to review articles before they are published. 

In the conversation with Mr. Kent, he stated that he simply printed the article "like  he always does". He mentioned that he thinks this article was emailed to him by an editorial intern. 

![](<./assets/A Scandal in Valdoria Part 2 and 3/Employees take 10 1.png>)Running a quick 
```
Employees
|take 10
```
query to see what fields the table has, gives us the name of the Editorial Intern, Ronnie McLovin. However, just to check if there are multiple editorial interns, the Employees table is queried for all editorial interns. 

```
Employees 
|where role == "Editorial Intern"
```
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250408152551.png>)
Ms.McLovin is currently the only intern for the newspaper. 

In a conversation with Ms. McLovin, she stated that while she was assigned to the OpEd piece about the candidates, she had never sent it to Mr. Kent as she had overslept. However, Mr. Kent was very clear that email had come from Ms. McLovin as he had received the email on January 31, 2024.

 While Ms. McLovin was insistent that she didn't sent Mr. Kent an email, running a query in the email table for emails received by Mr. Kent's email address turned up an email sent from her email address on January 31st. 
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250324111854.png>)

Email Subject is *URGENT: Final OpEd Draft Edits (Please publish the following article in tomorrow's paper))* There is one extra ) at the end, possibly a typo. 

Document name is *OpEdFinal_to_print.docx* and the verdict was marked as Clean. 

Overall, the email looks legitimate, but Ms. McLovin is insistent she didn't send it, so further investigation is needed. 

Talked to a Senior Editor, Sonia Gose, on a visit to the newspaper office and was shown this email that she thought was suspicious. Ms. Gose stated that she didn't remember if she clicked on the link. 

![](<./assets/A Scandal in Valdoria Part 2 and 3/Email from Sonia.png>)
The url contained in this email begins with https://promotionrecruit.com.

Organizations track what their employees are doing, so if Ms. Gose clicked on the link, it will be logged somewhere. Running the `.show tables` command discovered in [A Scandal in Valdoria Part 1](<./A Scandal in Valdoria Part 1.md>), brings up a list of all the tables in the database. 
![](<./assets/A Scandal in Valdoria Part 2 and 3/show tables.png>)

OutboundNetworkEvents looks like the correct table. 

A quick
```
OutboundNetworkEvents
| take 10
```
shows that the table uses `src_ip` instead of a name. 
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250328172900.png>)
So to continue, it is necessary to obtain Ms. Gose's email address. 
Running the following query:
```
Employees 
| where name has "Sonia"
```
brings up the following information
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250328173253.png>)
Ms. Gose's IP is `10.10.0.3`. 

To find out if Ms. Gose clicked on the link in the email, the following query was run:
```
OutboundNetworkEvents
| where src_ip == "10.10.0.3"
| where url contains "promotionrecruit"
```

![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250408110528.png>)

Ms. Gose did indeed click the link. The full url is https://promotionrecruit.com/published/Valdorian_Times_Editorial_Offer_Letter.docx so a file was downloaded onto Ms. Gose' s computer. The next step is to find the file on Ms. Gose's computer. 

Running the following query in the Employees table

```
Employees 
| where name has "Gose"
```
pulls Ms. Gose's employee information. To see the hostname, the size of the other fields was reduced. 

![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250408135445.png>)

The hostname of Ms. Gose's machine is UL0M-MACHINE. 

Referring back to the list of tables to see which is the appropriate to check brings up FileCreationEvents. ![](<./assets/A Scandal in Valdoria Part 2 and 3/show tables.png>)Since the name refers to when files were created, this is the most appropriate table to check. 

Looking at the FileCreationEvents table confirms that the search can be done by using hostname. 

![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250408140335.png>)
Running the following query for Ms.Gose's hostname 
```
FileCreationEvents
|where hostname == "UL0M-MACHINE"
```
shows that Ms. Gose downloaded the file on 01/05. 
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250408144718.png>)
Upon taking a closer look at the table, it appears there was a file named hacktivist_manifesto.ps1 downloaded 28 seconds after the initial file. Therefore, it can be concluded that the initial download, downloaded this additional file onto Ms. Gose's computer.  

A quick google search for .ps1 brings up a Microsoft page on [PowerShell scripts](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scripts?view=powershell-7.5). It's very unlikely for an offer letter to need a PowerShell script, so this file is malicious. 

_Note: Since this is a beginner module, the powershell script is already given and no forensic analysis is needed._

This is the given powershell script.
![](<./assets/A Scandal in Valdoria Part 2 and 3/valdoria-hacktivist-ps1.jpg>)
Going back to the list of all tables in the database helps determine what table to check for more information about the PowerShell script. The ProcessEvents table sounds as if it logs events on the network or system and since running a script can be considered an event, this is the table to look at. 

Running the 
```
ProcessEvents
|take 10
```
query, shows that the table has hostname as a table. 
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250409145339.png>)
Some of the table columns are shrunk to save space. 

Ms. Gose's hostname is UL0M-MACHINE, so running the following query
```
ProcessEvents
|where hostname =="UL0M-MACHINE"
```
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250409145641.png>)

There are a lot of events on Ms. Gose's machine, so the query needs to be narrowed down. 
```
ProcessEvents
|where hostname == "UL0M-MACHINE"
|where process_name has "powershell"
```
This query narrows down the search to any process that contains powershell in the name. 
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250409145944.png>)
~~Expanding the process_commandline column shows that there was one powershell event related to the hacktivist_manifesto file. ~~
_Note: This turned out to be the wrong answer and after trying a few more variations of "one", I went to the KC7 discord for help. It was there that I was informed that this required a different query that used the file name "hacktivist_manifesto.ps1" mentioned in Question 18. Since the file was listed in process_commandline, I continued with the investigation after running the following query_

```
ProcessEvents
|where hostname == "UL0M-MACHINE"
| where process_commandline contains "hacktivist_manifesto"
```
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250409163148.png>)
Now it is clearer which events are related to the execution of this particular file. 

There were 3 events related to this file, an execution policy bypass, a scheduled task, and WINWORD.exe.

Looking at the full command of the scheduled task 
```
schtasks /create /sc hourly /mo 5 /tn "Hacktivist Manifesto" /tr "powershell.exe -ExecutionPolicy Bypass -File C:\ProgramData\hacktivist_manifesto.ps1"
```

shows that the task is most likely scheduling the execution policy bypass. Since the manifesto mentioned plink execution, the next step is to search Ms. Gose's machine for plink. 

Running the 
```
ProcessEvents
| where parent_process_name contains "plink"
```

didn't turn up any results, but running the 

```
ProcessEvents
| where process_commandline contains "plink"
```
query turned up multiple. 
![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250410122320.png>)
Looking at the full process_commandline turns up multiple ips. 

![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250410123024.png>)

Modifying the search query to 

```
ProcessEvents
| where process_commandline contains "plink"
| where hostname == "UL0M-MACHINE"
```
turns up only one ip. 

![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250410123205.png>)

$had0w is presumably the username and the given password is thruthW!llS3tUfree.
Plink was run at `2024-01-06 02:39:35.0000`. To find out what happened after the execution of plink, the timestamp needs to be larger than the given one for plink. 

Running the query 
```
ProcessEvents
| where timestamp >= datetime(2024-01-06 02:39:35.0000)
| where hostname == "UL0M-MACHINE"
```

![](<./assets/A Scandal in Valdoria Part 2 and 3/Pasted image 20250410124602.png>)
The Spotify events on this table are most likely from Ms. Gose, but presumably Ms. Gose wouldn't have a reason to use discovery commands such as `whoami` and `ipconfig`, so these are most likely from the attacker. The `arp -a`, `tasklist /svc`, and `net view` commands are most likely from the attacker as well. 

However, none of these alone show how the article got published. 