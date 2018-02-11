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

We don’t like to write documentation. But we do like good documentation for tools we use. Which is a little bit controversial, isn’t it? Actually, this mean that we are too lazy to write docs, but we understand that it’s necessary for any good project. Here is few tips  how to create a “minimalistic but useful” documentation  from guy that don’t like to write documentation too. Hope it will be helpful.

 <!-- more -->
 
## THE PROBLEM
It’s hard to determine level of detalization for docs. Too many details  and documentation will become outdated almost immediately. Too little detail - and it will be useless. So, what should be covered, and what is not? We can figure it out with a five right questions.

## “WHAT”, “WHY”, “WHO”, “HOW” AND “HOW TO”
### WHAT

First of all, write down what is this. It should be clear and short - so anyone who will see it at first, will understand what is all about.  For example: *“Configuration.py is a library for configuration management in python apps.”*. 

### WHY

Next, what should be covered - it’s why this tool/project exists, and what problem you are trying to solve. How your team/company use it.  This is a good place to describe why this tool were created at the first place and what difference it has in compare to the alternatives (if some). This part is very important and often skipped, leaving beginners in perplexity and wondering why it exists, and the tool "X" not used instead. For example:

*“React makes it painless to create interactive UIs. Design simple views for each state in your application, and React will efficiently update and render just the right components when your data changes. Declarative views make your code more predictable, simpler to understand, and easier to debug.”*

### WHO
 
This part is more important for companies. Contributors and users should know who to call if something.  For example: *“Maintained by [blablateam] blablateam@company.com, Slack channel: blablateamchat. Tickets and bug reports are welcome here: blablaboard”*

### HOW 
“How it works”. This part is extremely important for new contributes and could save hours of code digging and avoiding of silly questions. It’s often hard to understand the general workflow used in code, the architecture idea behind it, along with third-party resources used (cache/db/configuration etc.) - and this not changing often, so documentation will not become obsolete too quick. Ideally, workflow could be presented as diagram - just because graphic representation much more expressive than text, and will give immediate understanding how it works. But text is ok too. If deployment or testing process is not straightforward it should be covered too. For example:  
*“Migration tool workflow:*  
*On run load configuration from “configfolder” and validate it*  
*Check connection to DB*  
*Run DB backup job using Jenkins API: url*  
*Apply migration scripts from “folder”*  
*During execution tool will publish run metrics to Statsd (by default expected on localhost). Metrics descriptions could be find here: url.”*      

It looks very simple, and straightforward **if you already know how it works**. But for new developer this simple overview could save a lot of time, so he/she will understand how it works at general before faced hundreds implementation details in code. 

## HOW TO

Last but not least should be the section that describe how to use tool, test it’s code and contribute to it. It is best if this part contains examples.


Hope it helps!
