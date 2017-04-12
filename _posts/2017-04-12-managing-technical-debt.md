---
layout: post
title: Managing Technical Debt
description: "Quite obvious plan to manage a technical debt."
author: bogdan
modified: 2014-12-24
tags: [code review, agile, process, qa]
categories: [process]
image:
    feature: posts/tech_debt/title.jpg
---

# WHAT IS “TECHNICAL DEBT”?

Martin Fowler [has a great explanation](https://martinfowler.com/bliki/TechnicalDebt.html) of this term and the problem behind it.  Many startups and new projects use quick-and-dirty solutions to achieve their goals as soon as possible, which in some point makes project complicated, slow, hard to change or implement a new features and full of bugs. I want to talk about most usual issues and how manage most effectively from my point of view.
Project with the big technical debt leading to make developers fear for changes. Any change can have unexpected effect and produce more bugs. And there is only one way to fearless development - it’s automated tests, which such projects usually don’t have. So technical debt is mostly about luck of control of the project quality.  There is other common issues in the project with big TD, but this is the most important one, and, what the worst thing here, very underestimated.
 <!-- more -->
 
# TL;DR: Plan
1. **Create automatic backup for application users data.** We don’t want to corrupt user data.
2. **Start using [commit strategy and workflow](http://nvie.com/posts/a-successful-git-branching-model/)** and make sure that everybody in a team understand it. There should be clear process to publish changes.
3. **Create/integrate a tool for application requirements management.** In these days some frameworks already have it: ruby bundler, npm etc. Also, OS requirements for testing/production can be managed using containerized solutions like Docker
4. **Create/integrate a tool for configuration management.** Some frameworks also has this from-the-box, Rails, for example.
5. **Choose the testing tools**
6. **Create Acceptance Test infrastructure**, so everybody in a team can create and run high-level tests for application behaviour from user point of view. Usually, it's the most effective way to cover existing functionality with tests without spending too much time on it. Also, it's a good way to create documentation on project parts behaviour.
7. **Create unit/integration tests infrastructure** in your application, so everybody in a team will be able to create and run the tests locally
8. **Build Continuous Integration** to run tests on a regular bases and collect the metrics from your code.
9. **Build one-step deployment process**. [Automatic deployment from CI](http://www.savvyclutch.com/devops/continuous-deployment-to-aws-ecs-and-circle-ci/) is the perfect solution. 
10. **Use one code style**, and use tool on CI to check it.
11. **Start [doing code reviews](http://www.savvyclutch.com/process/Make-Code-Review-Useful-Again/)**
12. **Triage and move all technical issues/stories in you issue tracker application to the one list**, so you will be able to see whole picture of your real technical debt and have realistic plan for it. 
13. **Create template for new issues/bugs/ stories in the issue tracker.** Without good descriptions in your issues they became an absolutely useless. This is necessary bureaucracy to save developers time and focus. 
14. **Investigate most confusing and affective parts in application** and create a plan to how to make them better
15. **Start creating tests for all newly created code**, and insist on them on code reviews (at least acceptance tests should cover the issue).
16. **Follow the “The Boy Scouts Rule”**, create the tests for part you touch and refactor it. Try to spend on this at least 20% of time.  

# LR: Details
So, where is the technical debt most often accumulated? On a new project team usually:

1. Fear to make the changes
2. Has no completely understanding of the application domain logic 
3. Skip unit/functional/acceptance/performance tests, and testing infrastructure at all
4. Implement manual deployment strategy and infrastructure configuration
5. Do not track requirements
6. Configure Dev environment and onboarding manually
7. Has no backup automation for data
8. Has no data sampling for dev/QA purposes 
9. Has a lot of “code smells” in source code, design by committee with a lot of hidden requirements due to lack of refactoring
10. Has a lot of bugs and requests with the poor descriptions, divided between 3-4 different projects/boards/lists in the issue tracker 
11. Has almost no process or procedures how to dealing with issues/features

Let me describe each a little. 

From my experience, the most destructive, hard to fix and influencing the speed of development thing is infrastructure. This including: application configuration management, application requirements management, dev environment, deployment process, testing infrastructure (local and CI) and commitment process. Without any of this the efficiency fall dramastically with no chance to become better. 

Without configuration management it’s very hard to not introduce bugs on production because of difference between local machines and production machines and this can cause critical issues for application users. 

Without project requirements management impossible to create good deployment and onboarding process, so both can take a days or, sometimes, weeks. 

Without useful developer environment, developers will spend a lot of time trying to figure out is some particular issue are related to their local environment, or it’s a general issue; some thing can’t be run or tested on a local machine at all (for example delayed jobs, file storing in case when it’s configured to store on the third-party resources like S3, mail sending etc.); will be not able to reproduce bugs from other developers or production; it will be hard to implement configuration or new requirements without  breaking other team member processes; 

Without simple and understandable deployment process team will have a hard time on delivering the fixes and new features, and this impact a application users which is critical. Technical dept in this process usually very time-consuming risky and stressful.  

Without testing infrastructure team can’t create the tests. It’s not necessary to became crazy and move all resources to create tests for existing code, but developers have to has possibility to creating the tests when they can and run in a regular bases. Without tests developers can’t effectively refactor code or implement a new features without introducing new bugs, so they should do all manual testing which is very ineffective, time-consuming and bad for team morale. Having this infrastructure will give the opportunity to became project health better over the time. Without it this will likely never happen.

So, fixing the infrastructure issues is the key to manage the technical debt.  

Data backing - it a most important part to decrease the risk. There is no way to make a lot of changes without mistakes, so team should not fear that their mistakes will result in data loss or corruption, which is, probably, the worst thing that can happen. 
Managing the TD in code is more straightforward. Some solutions already covered by the best practices of used languages, tools and frameworks. To identify the problem and engineer the solution are not very difficult, and with the proper testing risk can be minimal. Refactoring, test creation and code reviews should be permanent part of the development process.  
One more thing, which, I believe,  is very important to successful reduce of TD. Technical debt does not grow overnight. Sometimes it takes a years before people start thinking about to do something with, and it’s usually because they didn’t saw the whole picture to the very end, and start looking at it when any change became taking weeks and bugs and complaints numbers growth exponentially. This often because of issues organization for tech team. Leads organize bugs/stories/requests/tasks independently into different lists/projects, which hide the real scope of work. To avoid this, team should collect all technical tasks, bugs and issues in the one list. It will be a big list, but it will show how are you really deal with existing Technical Debt and implement features stories in the same time. Keeping things in different lists/boards/projects only hide the real situation. Splitting them to the different project has sense only when the separate teams will work on them.  

So, to summarize all of that:

1. Make user data safe
2. Isolate and unify environment and configurations to avoid confusions
3. Add tests infrastructure
4. Make development more safe by adding high-level tests
5. Make delivery process as simple as possible
6. Cover changed code with tests
7. Make real debt visible to everyone
8. Use simple rules to make things clear  

Hope it helps!
