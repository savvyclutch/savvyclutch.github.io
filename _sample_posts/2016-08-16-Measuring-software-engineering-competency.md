---
title: Measuring software engineering competency
layout: post
description: "Measuring... something that's neither easy nor simple. But let's try!"
author: bogdan, sergii
permalink: /measuring-software-engineering-competency/
source-id: 1OWEfPgEhJI5Lb4Eq5Llcj7UH2R1PC7ONGYjRcLc61oQ
published: true
---
![image alt text]({{ site.url }}/public/0sGIlVkcMs5sDfGFgISOA_img_0.jpg)

MEASURING SOFTWARE ENGINEERING COMPETENCY

by Bogdan Frankovskyi and Sergii Naumenko

# OVERVIEW & PURPOSE

Why do we want to measure our competency? Well... because we want to improve. And we have to! The world is rapidly changing: tools, technologies, paradigms, concepts, processes, competitors, crises. We must run to stay in business. We must deliver the product, maintain the quality, meet the budget...
But competency and quality measuring is tough! We don't have rules to assess them. [Joel Spolsky](http://www.joelonsoftware.com/) tried to build a simple checklist for a team to rate its quality in his famous [The Joel Test](http://www.joelonsoftware.com/articles/fog0000000043.html), but it was sixteen years ago. So let's stand on the shoulders of giants, and revise it from just the test to a set of rules that can guide us for the better software engineering competency.  
 

# HOW TO USE IT


The list of rules is splitted in two parts: *Obligations* and *Merits*. Each rule is a Yes/No question. "Yes" answer to an "Obligation" gives you 0 points, “No” answer gives you -1 point. “Yes” answer to a “Merit” rulegives you 1 point, “No” answer gives you 0 points. You've got the idea, right? Having merits is good, not fulfilling obligations is bad.
The resulting sum of points is your, let's say, Engineering Competency Score when negative number means that either your competency is questionable or is going to degrade over the time. 0 score means that you follow the professional discipline and capable of producing good quality product. Positive number says that you increase your competency and grow as professional.
Let's go.

# OBLIGATIONS

1. Do you have Continuous Integration?

2. Do you have one-step deployment?

3. Do you use tasks and errors management system(s)?

4. Do you have daily status check-in meetings?

5. Do you have short iterational process?

6. Do you have unit tests with at least 50% (and growing) coverage?

7. Do you have automated end-to-end tests in DSL?

8. Do you have an Acceptance Person in your team?

9. Do you do code reviews?

10. Do you have team with 2 pizzas size max?

11. Do you have less than 2 pizzas meetings?

12. Do you have a primary communication channel?

13. Do you have a library?

14. Do you stick to the "Boy Scout Rule"?

15. Do you have a person that advocates rules from above?

# MERITS

1. Do you contribute to Open Source?

2. Do you adopt something new?

3. Do you share your expertise with your peers?

4. Do you share your expertise with the world?

5. Do you ready at least 1 book in 6 months?

Now let's elaborate a little bit on the items from this list.

## Do you have Continuous Integration?

No CI - no automated tests - no quality - bad job. You may have tests but without CI they are useless, because sooner or later someone will forget/postpone fixing a broken test, push code without any test coverage, etc. A system that runs the tests regularly, notifies about errors, shows test coverage is a must.

## Do you have Automatic Deployment?

Importance of a simple, robust, and safe deployment process is one of the most underestimated thing in software development. It may looks we can get away with not having automated deployment just for now and do it later. But when the later becomes today we have a mess of ad-hoc scripts, checklists, and notes about things that have to be done manually. This mess causes mistakes, very often, on production. 

## Do you use tasks and errors management system?

We need to track what's going on on the project and guarantee that defects or tasks won't be forgotten or overlooked. It doesn’t matter what it is - whiteboard with stickers on it, online tools or something else, but we have to use something to keep things organised and the whole process more transparent for everybody. 

## Do you have daily status check-in meetings?

Communication is important, keep everybody in the loop but do not bore them with details. 

## Do you have short iterational process?

Short process, like weeks or two, keeps team focused on a small and achievable goals.

## Do you have unit tests?

We don't know the better way to make sure that code does what it's supposed to, then to have another code that runs it and check results.
 

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

