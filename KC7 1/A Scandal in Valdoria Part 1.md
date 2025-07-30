
# Goal

Figure out how an unapproved news article got published to the front page of the Valdorian Times without editorial approval. 

# Walkthrough

Note: Questions 1-11 are just an introduction to KQL. It's worth going through them if you've never used it before. Question 12 is just a "congrats you've reached the end of this part" question. 

## Question 1: 
Easy, read the setup and respond to the question to get started. Note: General responses aren't case sensitive. 

## Question 2:

Tl;dr this is a question to get you used to KQL syntax and running queries.

This uses [Azure Data Explorer](https://dataexplorer.azure.com/publicfreecluster) (ADX) which is free to try if you have a Microsoft account (who doesn't?). ADX uses [Kusto Query Language (KQL)](https://learn.microsoft.com/en-us/kusto/query/?view=microsoft-fabric), which seems to be specific to Microsoft right now. The only database language I've used of and used is [Structured Query Language ](https://en.wikipedia.org/wiki/SQL?oldformat=true)(SQL). I've used SQL early in my career, when I was doing software support, so I don't think KQL should be too different. Whether, I remember how to use SQL is another matter. 😅

At this stage KC7 gives you the queries you need and you can just copy/paste them into the environment they provide. I like typing them because it helps me learn them better. 

![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317120307.png>)

I didn't get anything when running this query, but this was the example query given and not the one you were actually supposed to run. Always read through the instructions first before beginning anything.


```
Employees 
| take 10 
```

is the actual query you're supposed to run and running it, populates the table.

![](<./assets/A Scandal in Valdoria Part 1/Employees take 10 1.png>)

Once you get this table, you can answer the question and move on. 

## Question 3:

tl;dr: Run the given query to find out how many employees work at the newspaper.

In this question, you are asked to find out how many employees are in the company. Since this is a beginner's level, KC7 gives you the query. 

```
Employees
| count
```

The expected result is:
![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317124255.png>)

However, I was curious to know if count could be used to narrow down the number of records there were for a certain role such as "IT Specialist", so I tried the following syntax:

```
Employees
| count
| where role == "IT Specialist"
```

This did not work and I got an error:
![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317124937.png>)

I didn't look up this error because I figured I'd try reversing the order of role and count first. 

```
Employees 
| where role == "IT Specialist"
| count
```
This worked, returning a result of 5. 
![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317125358.png>)


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
![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317132142.png>)

It worked. I don't know why you'd want to use single quotes, but if you do, KQL works with them. 

## Question 5:

Apparently, this is the question that shows you how to combine a ```where``` statement with ```count```. I jumped a little ahead in Question 3. 😄

Here, we're looking for how many emails the editorial director received. This time, we're searching the email table instead of the employee table, so the table name at the beginning of the code block changes. 

![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317133015.png>)

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

![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317135538.png>)

None of these headers help us determine Lois Lane's IP address. But we know the database consists of 2 other tables, Email and Employees. It's unlikely that the Email table will give us the IP, so let's take a look at the Employee's table. 

We can either run the `take 10` query again or just look at our [prior screenshots](<./assets/A Scandal in Valdoria Part 1/Employees take 10 1.png>) to determine the table headers. 

The Employees table lists both names and IP addresses, so we can query the table for Lois Lane's record. 

![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317140520.png>)

Now, we have Lois Lane's IP and we can run the query with her IP. 

```
OutboundNetworkEvents
| where src_ip == "10.10.0.22"
```


And we get a list of every website Lois Lane visited. 

![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317140953.png>)

KC7 has made this easy, so every url is distinct. However, in reality people will visit multiple webpages/websites multiple times. So, how do we filter out duplicate website/webpage names?

To answer that, I tried searching for "How to remove duplicate domain names using KQL" which didn't turn up many useful results, though I did find out that there is a [Kibana Query Language ](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)which also uses the KQL acronym. 

That's not what I was looking for, so I specified Kusto Query Language in the search. That search led me to a [stack overflow page ](https://stackoverflow.com/questions/78384416/how-to-remove-duplicates-in-kusto-query-language)about the [distinct operator.](https://learn.microsoft.com/en-us/kusto/query/distinct-operator?view=microsoft-fabric) This operator can be used to filter out duplicates of whatever is specified such as a column header. 

Running that query gave me 5,139 records consisting of just urls. 

![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317160607.png>)

There is no other identifying information in this table. Thankfully, I am not the only on who has had this question as running a search for "how to filter by distinct while still keeping all table headers kusto query" pulls up a [stack overflow page](https://stackoverflow.com/questions/67077241/obtaining-all-values-from-a-table-but-with-distinct-runid) with the exact same question. Unfortunately, the answer is that you must include all headers in your query. 

But, if KC7's database had duplicate web addresses for Lois Lane, we could filter them out with this query:

```
OutboundNetworkEvents
|where src_ip == "10.10.0.22"
|distinct url
```
which would give us non-duplicate urls. In this case, running this query on KC7's database brings up the exact same list as before minus the rest of the columns. 

**![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250317161606.png>)**

## Question 8:

Again, follow the prompt and you get the answer. 

However, it was around this question that I started wondering exactly how many tables were in this database, so I searched for "how to get a list of all tables in a database using kql". That gave me a[ documentation page ](https://learn.microsoft.com/en-us/kusto/management/show-tables-command?view=microsoft-fabric)about the `.show tables` command. 

Running that gives us a nice little list of every table in the database. 
![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250318153137.png>)
The SecurityAlerts table looks interesting doesn't it?

Running 
```
SecurityAlerts
| take 10
```
gives us this
![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250318153322.png>)
I don't know about you, but that seems to be high number of people who have suspicious files on their devices, especially with the report of a suspicious email. This will probably be important later, so we're just make a note of it and move on to Question 9.

## Question 9
Distinct, which I learned about after finishing question 7, shows up in this one and running the given prompt only results in one IP. 

## Question 10
Tl'dr: Follow the prompt and you'll get the answer. 

This is the question where I realized that I actually knew what `let mary_ips = ` meant without having to google "let statements in KQL"....of course I did that anyway, just to confirm that [these statements](https://learn.microsoft.com/en-us/kusto/query/let-statement?view=microsoft-fabric) are KQL's way of defining variables. 

## Question 11 

This question builds on question 10, allowing you to create and use your created variable in another statement. Again, follow the prompt. _However_, if you're like me, you might be tempted to leave off the () since none of the previous queries used them. KQL needs them and leaving them off will result in an error. 

![](<./assets/A Scandal in Valdoria Part 1/Pasted image 20250319180631.png>)

Here, I didn't know what to name the variable, so I just used the prompt default. 

But, adding the () back in, will make the code run just fine and you'll get the answer.

Then, answer question 12 (the answer is given in the prompt) and you can move on to Part 2, the actual investigation! 
