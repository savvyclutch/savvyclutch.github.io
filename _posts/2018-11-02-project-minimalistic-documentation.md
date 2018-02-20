---
layout: post
title: Project documentation 
description: "Minimalistic but useful project documentation."
author: bogdan
modified: 2014-12-24
tags: [documentation, agile, process, qa]
categories: [Process]
image:
    feature: posts/project_documentation/title.jpg
---

We don’t like writing documentation. What we do like is having good documentation for the tools we use. Which is a little bit controversial, isn’t it? Actually, this means that most likely we are too lazy to produce docs,
  at the same time we understand that it is necessary for any good project. Here are a few tips on how to create a “minimalistic but useful” documentation from a guy who does not necessarily enjoy the process but gets it done.

 <!-- more -->
 
## THE PROBLEM
It’s hard to determine level of detalization for docs.  Too much detail - and documentation becomes outdated in no time. Too little detail - and it is useless. So what should be covered, and what shouldn't? It appears we can figure it out with just five straightforward questions.

## “WHAT”, “WHY”, “WHO”, “HOW” AND “HOW TO”
### WHAT

First of all, write down what the subject is.  It should be clear and concise, so anyone who sees it for the first time is able to understand what it is all about. For example: *“Configuration.py is a library for configuration management in python apps.”* 

### WHY

Next thing to cover is why this tool/project exists, what problem you are trying to solve and how your team/company uses it. This is a good place to describe why this tool was created in the first place and what's different compared to the alternatives (if there are any). This part is very important and often skipped, leaving beginners perplexed and bemused by the tool's existence and wondering why they cannot use the alternative "X" instead. For example:

*“React makes it painless to create interactive UIs. Design simple views for each state in your application, and React will efficiently update and render just the right components when your data changes. Declarative views make your code more predictable, simpler to understand, and easier to debug.”*

### WHO
 
This part is important for the companies, as both contributors and users should know who to call if something unexpected happens. For example: *“Maintained by [blablateam] blablateam@company.com, Slack channel: blablateamchat. Tickets and bug reports are welcome here: blablaboard”*

### HOW 
“How it works”. This part is extremely important for new contributors and could save hours of code digging and help avoid silly questions. It’s often hard to understand the general workflow used in the code or the architectural idea behind it along with the used third-party resources (cache/db/configuration etc.). This does not change often, therefore documentation will not become obsolete too soon. Ideally, workflow should be presented as a diagram; graphic representation is way more expressive than text and will instantly help you understand how it works. Although, text is fine too - it should be covered if deployment or testing process does not appear to be straightforward. For example:  
*“Migration tool workflow:*  
*On run load configuration from “configfolder” and validate it*  
*Check connection to DB*  
*Run DB backup job using Jenkins API: url*  
*Apply migration scripts from “folder”*  
*During execution tool will publish run metrics to Statsd (by default expected on localhost). Metrics descriptions could be find here: url.”*      

It looks very simple, and straightforward **if you already know how it works**. But if you are a new developer, this simple overview could save you loads of time and energy by helping you understand how it works in general, before you face hundreds of implementation issues in code. 

## HOW TO

Last but not least should be the section describing how to use the tool, test its code and contribute to it. It is best if this part contains examples.


Hopefully it helps!
