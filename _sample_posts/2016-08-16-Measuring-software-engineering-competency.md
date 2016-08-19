---
title: Measuring software engineering competency
layout: post
author: [sergii, bogdan]
permalink: /measuring-software-engineering-competency/
source-id: 1OWEfPgEhJI5Lb4Eq5Llcj7UH2R1PC7ONGYjRcLc61oQ
published: true
---
![image alt text]({{ site.url }}/public/0sGIlVkcMs5sDfGFgISOA_img_0.jpg)

MEASURING SOFTWARE ENGINEERING COMPETENCY

by Bogdan Frankovskyi and Sergii Naumenko

# OVERVIEW & PURPOSE

As is well known, "if you can't measure it, you can’t ~~measure~~ manage it." Of course, it’s mostly advice, than a rule, but there's a certain common sense of measuring. At least we should try measure to see the progress. Measuring competency and quality is not simple, because there are no specific rules, or attributes. But there are always some things that have to be done to keep quality on a high level in the long run.  

Sixteen years ago [Joel Spolsky](http://www.joelonsoftware.com/) tried to build ф simple checklist for a team to rate its quality  in his [The Joel Test](http://www.joelonsoftware.com/articles/fog0000000043.html). Much time has passed since the publication of this test, so we decide to review it, and try to use it to improve our quality and work efficiency. 

This document is an attempt to revised  and extend it from just a test to a set of rules that allow to measure and improve our engineering competency. 

# HOW TO USE IT

The list of rules is splitted in two parts: must have and good to have. Each rule is a Yes/No question. "Yes" answer to a “Must” rule gives you 0 points, “no” answer gives you -1 point. “Yes” answer to a “Good” rule gives you 1 point, “no” answer gives you 0 points. The resulting sum of points is your Engineering Competency Score when negative number means that either your competency is questionable or is going to degrade over the time.  0 score means that you follow professional discipline and capable of producing good quality product. Positive number says that you increase your competency and grow as professional.

# MUST RULES

1. Do you have Continuous Integration?

2. Do you have one-step deployment?

3. Do you use tasks and errors management system(s)?

4. Do you have daily status check-in meetings?

5. Do you have short iterational process?

6. Do you have unit tests with at least 50% coverage?

7. Do you have automated end-to-end tests in DSL?

8. Do you have an Acceptance Person?

9. Do you do code reviews?

10. Do you have team with 2 pizzas size max?

11. Do you have less than 2 pizzas meetings?

12. Do you have a primary communication channel?

13. Do you have a library?

14. Do you stick to the "Boy Scout Rule"?

15. Do you have a person that advocates rules from above?

# GOOD RULES

1. Do you contribute to Open Source?

2. Do you adopt something new?

3. Do you share your expertise with your peers?

4. Do you share your expertise with the world?

5. Do you ready at least 1 book in 6 months?

Now let's elaborate a little bit on the items from this list.

## Do you have Continuous Integration?

CI allow to measure the code health by ran the unit tests on any commit, show the numbers of tests and code coverage, automatically build it and add possibility to extend to automatic deployment. All of this things help developers to be more confident in project quality and code lifecycle stability because everyone can see build results, what was changed and when. 

## Do you have one-step deployment?

Complex, complicated deployment process, with any manual steps make this process dangerous - just because deployment usually means moving things to the production stage and people make mistakes. On production this mistakes can be very expensive.  That's why deployment should be simple and safe and that’s why it should be fully automatic and  include manual actions as little as possible.

## Do you use tasks and errors management system(s)?

This, obviously, need to track what's going on on project and guarantee you will not forget about defects or tasks which should be done. Doesn’t matter what it will be - Kanban board, online tools or something else, but you have to use something to keep yourself more organised and process more transparent for any team members. 

## Do you have daily status check-in meetings?

Everyone in team should know what's going on in project to avoid miscommunication issues and distractions.

## Do you have short iterational process?

Short process, like weeks or two, keeps team focused on a small goals  which is always good for a quality.  

## Do you have unit tests with at least 50% coverage?

Unit testing is important for a code quality because it helps developers to build more flexible code design and update/refactor the code without fear to break existing functionality. It's not necessary to get crazy and always keep code coverage 100% but code should be covered by unit tests at least in half.  

## Do you have automated end-to-end tests in DSL?

## Do you have an Acceptance Person?

## Do you do code reviews?

## Do you have team with 2 pizzas size max?

## Do you have less than 2 pizzas meetings?

## Do you have a primary communication channel?

## Do you have a library?

Library does not guaranty that team members will read the books from it, but it should be available if they want to start. When people can talk use the same terminology and with the similar background of knowledge it makes discussion more productive.  

## Do you stick to the "Boy Scout Rule"?

Leave things better than you found it - a good advice to keep project healthy. This does not mean you should change everything you touch, but if you see small issue, that can be easily fixed or improved -  do it.

## Do you have a person that advocates rules from above?

