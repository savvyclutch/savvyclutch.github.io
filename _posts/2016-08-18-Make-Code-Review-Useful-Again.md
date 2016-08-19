---
title: Make Code Review Useful Again
description: "A few tips on how to organize code review process in a most effective and less distracting way. "
layout: post
author: bogdan
tags: [code review, agile, process]
image:
    feature: posts/code_review/title.jpg
    
---

Code Review is commonly used technique and it's definitely one of the must-have processes to keep high quality of code. 
But quite often it becomes a formal process or people experience different issues using it. 
So let’s take a look what is Code Review and what good and bad parts we can get using this technique. 

# WHY

Code Review is used for a different reasons regarding to process specifics - for example in open source development it is the only way to include contributed code from the third-party developers. But in a team (especially a small one) the must-have review is not necessary. But they still need it.  So why people use Code Review in one team:

1. Catch logic errors.

2. Catch missed parts (not updated documentation, missed tests etc.) 

3. Catch missed requirements. Sometimes developer can accidentally overlooked some of the implemented task requirements.

4. View from the outside. Someone from outside can see issues in code which developer can miss because of 'bogged down in code'

5. Check code quality.

6. Knowledge sharing. 

7. Education. Doing the Code Review developers educating each other.

8. Make sure implementation complies to coding styles, standards, etc.

<!-- more -->

# GOOD PARTS

Using Code Review on a regular basis keep developers knowledge synchronised between each other and make the members inside a team replaceable, more universal, which is good for performance and keep "bus factor" number for your project as high as possible. And it forces developers to use best practices in their work - just because someone will see it, and such lazy as I am can't just skip something because of laziness or distraction.  

# BAD PARTS

Unfortunately, Code Review takes time. In real life everyone is busy - so code reviews quickly became to a formal process, which making it pointless. This happens because of few reasons: 

1. Reviewers should understand task context, which is not easy. There can be a lot of context details - legacy code specifics, business requirements, environment specific details etc.

2. Reviewers should mentally switch from their own tasks.

3. Point out other people's mistakes psychologically unpleasant. Reviewers should be careful and polite to avoid aggressive-defensive reaction from other developers, but we're all human beings, aren't we? Sometimes it's not easy to explain your point of view in writing, and it takes a lot of time. 

4. Lack of time for task implementation. Usually code review happens after full implementation, in the middle of iteration, or, most likely, closer to the end of iteration. So, usually, there is no time for any not-trivial changes (which should be reviewed too). And it's not always obvious which changes have to be done immediately, and which can be postponed.

# LET'S MAKE IT BETTER

## Use checklists

Anyone participating in the code review should have a checklist for what should be checked or described. It should be short and simple. For example, to publish the review:

1. Check that task requirements are satisfied:

    1. List of requirements...

2. Cover code with unit tests

3. Update media files

4. Check that all new files are committed

5. Update README

6. Make sure that build is green

7. Write description for the review

## Prepare pull request/code for a review

**Do not fear.** Engineers born brave, so do not fear to show your work. Any advice or found issue will make you better, and make your colleagues to trust your opinion and your expertise.

**Always describe what you did, context, and how you did it in general**. Try to be short, but always explain non-oblivious choices. 

It's good to use automatic lint programs to check code conventions, so you will not have "use 2 spaces instead of 4"-like comments in review.

**Create more than one review for task (especially for tasks that are going to be done at the end of implementation)**. Create some MVP of the feature, if possible, and send it to the reviewer ASAP. This allows to see major issues not at the end of an iteration, and give you more time to find the better way to implement it.

**Ask for advice**. Two heads are better than one, right? If there is something wrong, or you stucked with the one solution but you feel that there should be something better - ask for the outside view. 

**Submit review at the end of day** - this will guarantee you do not interrupt someone in the middle of the process, and give them more time for review - which is good. Most likely, reviewers will do the review at the next day morning, which is also good for you - clear mind can see more. 

## Reviewers

**Do the review first in the morning**, when your mind is still clear.  This will give other developer more time to fix the code, and it's not necessary to break your mind flow.

**Do not offer solutions** at the first place. Show problems, but don't tell how to solve them, so developer will have chance to make his own decisions. People don’t like to follow orders, they enjoy to find their own way. And, your solution can be not so good as you think - you don’t have the full context, so there can be details you don’t know. Anyway, usually, people ask for advice. 

**Not necessarily always find something**. Sometimes things are good enough. Of course, there is always a room for improvements, but let's do things good, then do them better. 

 **Use 'presence' checklist**. Just to make sure that everything are there - unit tests, updated documentation, no requirements skipped, resources updated etc.

**Do review from general to details**. Try to understand what should be implemented and how it was done in general. This make your mind construct your own solution, so when you will see details, you will have right questions to ask. 

**Make sure you are really understand on what you are looking at**.  Ask author to help you with this understanding. Sometimes the best way to do this - review this code in pair with the author or other reviewer. 

**Use Fawler's 'code smells’ checklist**. Using this checklist helps to quickly see the issues in the code and make your review more concrete - It’s much easier to explain what wrong if everyone uses the same definition and understanding what, actually, is "wrong" and why. 



Code review is great technique that makes team competency grow, and grow fast, and it's very important to do it on a regular basis. 

