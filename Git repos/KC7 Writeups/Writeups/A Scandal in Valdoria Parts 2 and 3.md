_This is written in the style of part investigative writeup, part blog post and covers parts 2 and 3 of ![KC7's](https://kc7cyber.com/) a Scandal in Valdoria. This is a beginner module meant to introduce the playere to investigative thinking. As I am doing these while going through the module, part 4 will come at a later date._

# Incident Overview:

Shortly, before Valdoria's next mayoral election, an unauthorized article was published, without any editorial review, by the Valdorian Times.

# Response
Editorial director, Nene Leaks, immediately contacted a cyber incident responder to determine how the unauthorized article was published. 

## Information Obtained
_Note: Many of the columns have been shrunk to show the relevant evidence._

According to Ms. Leaks, an article about the candidates was approved, but it was not the one that got published. Ms. Leaks also suggested talking to the Newspaper Printer, Clark Kent, who is the last person to review articles before they are published. 

In the conversation with Mr. Kent, he stated that he simply printed the article "like  he always does". He mentioned that he thinks this article was emailed to him by an editorial intern.

Running a quick 
```
Employees
|take 10
```
query to see what fields the table has, gives us the name of the Editorial Intern, Ronnie McLovin. 

![Employees take 10](https://github.com/user-attachments/assets/c1dd9405-b651-4091-99fd-a9f3aeb64548)


However, just to check if there are multiple editorial interns, the Employees table is queried for all editorial interns. 

```
Employees 
|where role == "Editorial Intern"
```
![editorial intern](https://github.com/user-attachments/assets/3e023c79-9ddb-462d-b48b-643117c6d2ed)
Ms.McLovin is currently the only intern for the newspaper. 

In a conversation with Ms. McLovin, she stated that while she was assigned to the OpEd piece about the candidates, she had never sent it to Mr. Kent as she had overslept. However, Mr. Kent was very clear that email had come from Ms. McLovin as he had received the email on January 31, 2024.

While Ms. McLovin was insistent that she didn't sent Mr. Kent an email, running a query in the email table for emails received by Mr. Kent's email address turned up an email sent from her email address on January 31st. 
![Emails sent to Kent](https://github.com/user-attachments/assets/ac67c631-b6a3-4024-9b08-c9aab3681d47)

Email Subject is `URGENT: Final OpEd Draft Edits (Please publish the following article in tomorrow's paper))` There is one extra ) at the end, possibly a typo. 

Document name is `OpEdFinal_to_print.docx` and the verdict was marked as `Clean`. 

Overall, the email looks legitimate, but Ms. McLovin is insistent she didn't send it, so further investigation is needed. 

During a conversation with Senior Editor Sonia Gose, on a visit to the newspaper office, Ms. Gose presented an email she thought suspicious. When asked, Ms. Gose stated that she didn't remember if she clicked on the link. 

![Email from Sonia](https://github.com/user-attachments/assets/436ac274-7107-44b9-9b68-bbed63d5e52d)

The url contained in this email begins with `https://promotionrecruit.com`.

Organizations track what their employees are doing, so if Ms. Gose clicked on the link, it will be logged somewhere. Running the `.show tables` command discovered in ![A Scandal in Valdoria Part 1](https://github.com/mbellamkom/cyber-learning/blob/main/KC7%20Writeups/A%20Scandal%20in%20Valdoria%20Part%201.md), brings up a list of all the tables in the database. 

![show tables](https://github.com/user-attachments/assets/c3dd7f8b-2dca-4a3e-be7b-1fb6b06ff848)

OutboundNetworkEvents looks like the correct table. 

A quick
```
OutboundNetworkEvents
| take 10
```
shows that the table uses `src_ip` instead of a name. 

![OutBoundNetworkEvents](https://github.com/user-attachments/assets/74e40637-90ba-4a73-9c17-170ffcd63fd4)

So to continue, it is necessary to obtain Ms. Gose's ip address. 

Running the following query in the Employee's table
```
Employees 
| where name has "Sonia"
```
brings up the following information

![Sonia's IP](https://github.com/user-attachments/assets/ef14ffef-c938-4ae4-bd87-11e6b8d589b4)

Ms. Gose's IP is `10.10.0.3`. 

To find out if Ms. Gose clicked on the link in the email, the following query was run
```
OutboundNetworkEvents
| where src_ip == "10.10.0.3"
| where url contains "promotionrecruit"
```

![Link Clicked](https://github.com/user-attachments/assets/7e12bc69-8307-4f2e-af29-52dc3f7750ae)

Ms. Gose did indeed click the link. The full url is `https://promotionrecruit.com/published/Valdorian_Times_Editorial_Offer_Letter.docx` so a file was downloaded onto Ms. Gose' s computer. The next step is to find the file on Ms. Gose's computer. 

Running the following query in the Employees table

```
Employees 
| where name has "Gose"
```
pulls Ms. Gose's employee information. To see the hostname, the size of the other fields was reduced. 

![Gose Hostname](https://github.com/user-attachments/assets/5b9a3e2b-e2fa-457e-bbdc-8714cb2c39bd)

_Note: In hindsight, I should have noted this down when finding Ms. Gose's ip address._

The hostname of Ms. Gose's machine is `UL0M-MACHINE`. 

Referring back to the list of tables shown earlier in this document to see which is the appropriate to check brings up FileCreationEvents. 

Since the name refers to when files were created, this is the most appropriate table to check. 

Looking at the FileCreationEvents table confirms that the search can be done by using hostname. 

![FileCreationEvents](https://github.com/user-attachments/assets/3e0360cf-d99c-4e8c-9d63-1882341c423e)

Running the following query for Ms.Gose's hostname 
```
FileCreationEvents
|where hostname == "UL0M-MACHINE"
```
shows that Ms. Gose downloaded the file on 01/05. 

![downloadfile](https://github.com/user-attachments/assets/26dcae56-b2f8-46b0-8deb-e778419fbc1a)


Upon taking a closer look at the table, it appears there was a file named hacktivist_manifesto.ps1 downloaded 28 seconds after the initial file. Therefore, it can be concluded that the initial download, downloaded this additional file onto Ms. Gose's computer.  

A quick google search for .ps1 brings up a Microsoft page on [PowerShell scripts](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scripts?view=powershell-7.5). It's very unlikely for an offer letter to need a PowerShell script, so this file is malicious. 

_Note: Since this is a beginner module, the powershell script is already given and no forensic analysis is needed._

This is the given powershell script.

![valdoria-hacktivist-ps1](https://github.com/user-attachments/assets/5b5afd7f-5e11-4967-a308-99ee25fefead)

Going back to the list of all tables in the database helps determine what table to check for more information about the PowerShell script. The ProcessEvents table sounds logs events on the network or system and since running a script is an event, this is the table to look at. 

Running the 
```
ProcessEvents
|take 10
```
query, shows that the table has hostname as a column. 

![ProcessEventstake10](https://github.com/user-attachments/assets/3664578e-16ef-4bc3-b9a0-d805a039e0b2)

Ms. Gose's hostname is `UL0M-MACHINE`, so running the following query pulls up all of the events for Ms. Gose's machine.
```
ProcessEvents
|where hostname =="UL0M-MACHINE"
```
![ProcessEventsSonia](https://github.com/user-attachments/assets/7f1afc4c-bfad-4692-a92e-a49a4f507251)


There are a lot of events on Ms. Gose's machine, so the query needs to be narrowed down. 
```
ProcessEvents
|where hostname == "UL0M-MACHINE"
|where process_name has "powershell"
```
This query narrows down the search to any process that contains powershell in the name. 

![Powershellprocess](https://github.com/user-attachments/assets/3e186871-f39c-463d-a7ae-9cfbf7555fce)


Expanding the process_commandline column shows that there was one powershell event related to the hacktivist_manifesto file. 

_Note: This turned out to be the wrong answer and after trying a few more variations of "one", I went to the KC7 discord for help. It was there that I was informed that this required a different query that used the file name "hacktivist_manifesto.ps1" mentioned in Question 18. Since the file was listed in process_commandline, I continued with the investigation after running the following query_

```
ProcessEvents
|where hostname == "UL0M-MACHINE"
| where process_commandline contains "hacktivist_manifesto"
```
![Hacktivistmanifestorelatedevents](https://github.com/user-attachments/assets/b9ede166-3abb-42b0-8776-73b3449d01ff)

Now it is clearer which events are related to the execution of this particular file. 

There were 3 events related to this file, an execution policy bypass, a scheduled task, and WINWORD.exe.

Looking at the full command of the scheduled task 
```
schtasks /create /sc hourly /mo 5 /tn "Hacktivist Manifesto" /tr "powershell.exe -ExecutionPolicy Bypass -File C:\ProgramData\hacktivist_manifesto.ps1"
```

shows that the task is most likely scheduling the execution policy bypass. Since the manifesto mentioned plink execution, the next step is to search Ms. Gose's machine for plink. 

Running the following query
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

![plinkexe](https://github.com/user-attachments/assets/dd2b49a3-e359-45b8-9234-15d38737b048)

Looking at the full process_commandline turns up multiple ips. 

![plinkips](https://github.com/user-attachments/assets/486b645a-eaa9-4ac4-a8d5-7a6d5c623d9a)


Modifying the search query to narrow down the ip's for which one was used on Ms. Gose's machine

```
ProcessEvents
| where process_commandline contains "plink"
| where hostname == "UL0M-MACHINE"
```
turns up only one ip. 

![soniaplinkip](https://github.com/user-attachments/assets/fb9ee8ac-af2d-4376-a372-697424b863cc)


`$had0w` is presumably the username and the given password is `thruthW!llS3tUfree`.
Plink was run at `2024-01-06 02:39:35.0000`. To find out what happened after the execution of plink, the timestamp needs to be larger than the given one for plink. 

Running the following query pulls up all the events that occured on Ms. Gose's machine during and after the execution of plink.  

```
ProcessEvents
| where timestamp >= datetime(2024-01-06 02:39:35.0000)
| where hostname == "UL0M-MACHINE"
```
![eventsafterplinkexecuted](https://github.com/user-attachments/assets/ed608adc-b84f-4266-82d3-3571b5626bd7)

The Spotify events on this table are most likely from Ms. Gose, but presumably Ms. Gose wouldn't have a reason to use discovery commands such as `whoami` and `ipconfig`, so these are most likely from the attacker. The `arp -a`, `tasklist /svc`, and `net view` commands are most likely from the attacker as well. 

However, none of these alone show how the article got published so further investigation is needed. 
