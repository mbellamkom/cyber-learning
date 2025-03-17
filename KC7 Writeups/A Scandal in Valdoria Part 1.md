# Overall Goal
Figure out how an unapproved news article got published to the front page of the Valdorian Times without editorial approval.

Level: Beginner

# Part 1 Writeup 

This part, consisting of questins 1-10, is just an introduction to Kusto Query Language (KQL). It's worth going through them if you've never used it before. I'm going through them since I've never used Kusto Query Language before. Since, this is a beginner module, all of the queries are given to you. As I went through the questions, I got sidetracked by what KQL could do instead of actually answering the questions. 

## Question 1:
Easy, read the setup and respond to the question to get started. Note: General responses aren't case sensitive.

## Question 2:
Tl;dr this is a question to get you used to KQL syntax and running queries.

This uses [Azure Data Explorer](https://dataexplorer.azure.com/publicfreecluster) (ADX) which is free to try if you have a Microsoft account (who doesn't?). ADX uses [KQL](https://learn.microsoft.com/en-us/kusto/query/?view=microsoft-fabric), which seems to be specific to Microsoft right now. The only database language I've used of and used is [Structured Query Language ](https://en.wikipedia.org/wiki/SQL?oldformat=true)(SQL). I've used SQL early in my career, when I was doing software support, so I don't think KQL should be too different. Whether, I remember how to use SQL is another matter though. 😅

At this stage KC7 gives you the queries you need and you can just copy/paste them into the environment they provide. I like typing them because it helps me learn them better. 

![image](https://github.com/user-attachments/assets/d132267a-8b4e-4bf3-b3ce-796035de3f98)

I didn't get anything when running this query, but this was the example query given and not the one you were actually supposed to run. Always read through the instructions first before beginning anything.

```
Employees 
| take 10 
```
is the actual query you're supposed to run and running it, populates the table.

![image](https://github.com/user-attachments/assets/e5ec0751-71be-4a72-bfcd-87468d19f6d7)

Once you get this table, you can answer the question and move on. 

## Question 3:

tl;dr: Run the given query to find out how many employees work at the newspaper.

In this question, you are asked to find out how many employees are in the company. Since this is a beginner's level, KC7 gives you the query. 

```
Employees
| count
```

The expected result is:

![image](https://github.com/user-attachments/assets/9f6717fe-3107-4f5c-99d7-f27cf378abcb)

However, I was curious to know if count could be used to narrow down the number of records there were for a certain role such as "IT Specialist", so I tried the following syntax:

```
Employees
| count
| where role == "IT Specialist"
```
This did not work and I got an error:

![image](https://github.com/user-attachments/assets/3051633a-3125-4f63-98e8-8a857e1c3157)

I didn't look up this error because I figured I'd try reversing the order of role and count first. 

```
Employees 
| where role == "IT Specialist"
| count
```
This worked, returning a result of 5. 

![image](https://github.com/user-attachments/assets/89e3fb12-b6bf-4955-b2f4-84ad6c255fd9)

## Question 4:

tl;dr: Run the given query to find out who is the Editorial Director of the newspaper.

This question narrows down the focus to one single role "Editorial Director". 
Again, KC7 gives you the query, but in case they didn't, it's pretty easy to figure out. 

```
Employees
| where role == "Editorial Director"
```
This gives you the name of the Editorial Director, along with some additional information. 

I was curious to know if the double quotes were necessary or if single quotes could be used, so I ran the query again using single quotes. 

![image](https://github.com/user-attachments/assets/7b2c4762-ce51-4086-90be-b0ab05d7de84)

It worked. I don't know why you'd want to use single quotes, but if you do, KQL works with them. 

## Question 5:

Apparently, this is the question that shows you how to combine a ```where``` statement with ```count```. I jumped a little ahead in Question 3. 😄

Here, we're looking for how many emails the editorial director received. This time, we're searching the email table instead of the employee table, so the table name at the beginning of the code block changes. 

![image](https://github.com/user-attachments/assets/77db506a-3aa0-42dd-91fd-ac7706af1717)

## Question 6:

Follow the instructions and you'll get your answer. 

## Question 7:

Now, this question requires some investigative work. The question is "How many distinct websites did Lois Lane visit" and you're given an example of a query for a table called OutboundNetworkEvents. 

```
OutboundNetworkEvents
| where src_ip == "Lois Lane IP"
| <operator> <field>
| <operator>
```
Here, we're missing a key piece of information, Lois Lane's IP. 

Running a `take 10` query on OutboundNetworkEvents, gives us the following table headers:

![image](https://github.com/user-attachments/assets/6385e2c2-16a0-4058-81ee-0f4925ba164a)

None of these headers help us determine Lois Lane's IP address. But we know the database consists of 2 other tables, Email and Employees. It's unlikely that the Email table will give us the IP, so let's take a look at the Employee's table. 

We can either run the `take 10` query again or just look at our [prior screenshot of the Employee table](https://github.com/user-attachments/assets/d132267a-8b4e-4bf3-b3ce-796035de3f98) to determine the table headers. 

The Employees table lists both names and IP addresses, so we can query the table for Lois Lane's record. 

![image](https://github.com/user-attachments/assets/df8ff968-6873-4b41-94a2-e3997602a2f7)

Now, we have Lois Lane's IP and we can run the query with her IP. 

```
OutboundNetworkEvents
| where src_ip == "10.10.0.22"
```
And we get a list of every website Lois Lane visited. 

![image](https://github.com/user-attachments/assets/f11a361b-600e-4fda-8043-ad1ed5f8bd8d)

KC7 has made this easy, so every url is distinct. However, in reality people will visit multiple webpages/websites multiple times. So, how do we filter out duplicate website/webpage names?

To answer that, I tried searching for "How to remove duplicate domain names using KQL" which didn't turn up many useful results, though I did find out that there is a [Kibana Query Language ](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)which also uses the KQL acronym. 

That's not what I was looking for, so I specified Kusto Query Language in the search. That search led me to a [stack overflow page ](https://stackoverflow.com/questions/78384416/how-to-remove-duplicates-in-kusto-query-language)about the [distinct operator.](https://learn.microsoft.com/en-us/kusto/query/distinct-operator?view=microsoft-fabric) This operator can be used to filter out duplicates of whatever is specified such as a column header. 

Running that query gave me 5,139 records consisting of just urls. 

![image](https://github.com/user-attachments/assets/4c29422d-e082-4114-88c0-a74491dbdbed)

There is no other identifying information in this table. Thankfully, I am not the only on who has had this question as running a search for "how to filter by distinct while still keeping all table headers kusto query" pulls up a [stack overflow page](https://stackoverflow.com/questions/67077241/obtaining-all-values-from-a-table-but-with-distinct-runid) with the exact same question. Unfortunately, the answer is that you must include all headers in your query. 

But, if KC7's database had duplicate web addresses for Lois Lane, we could filter them out with this query:
```
OutboundNetworkEvents
|where src_ip == "10.10.0.22"
|distinct url
```
which would give us non-duplicate urls. In this case, running this query on KC7's database brings up the exact same list as before minus the rest of the columns. 

![image](https://github.com/user-attachments/assets/822f1125-7df6-477e-a029-2f22a3a6df2d)









