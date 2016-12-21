---
title: BDD tests for Hadoop with Cucumber. Part II
description: "Tutorial how to create behaviour tests for Hadoop application with Cucumber. Part II: tests"
layout: post
author: bogdan
tags: [bog data, java, hadoop, qa]
modified: 2016-12-21
image:
    feature: posts/bdd_test_hadoop/title.jpg
    
---

In [previous part](http://www.savvyclutch.com/BDD-tests-for-Hadoop-with-Cucumber-part-I/) we have created the application to process the data and docker container to isolate testing environment.
   
In this part we will create tests using [Cucumber for Java](https://cucumber.io/docs/reference/jvm) and [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin).

Acceptance tests, usually, check the normal behaviour of application - we want to be sure that the app do what is designed to do.  
Gherkin is the great way to create acceptance tests - because it allows to concentrate on what we are going to do instead of how we do it, 
and, as a side effect, we get document features and scenarios in human-readable format which is very helpful, especially for new team members.

<!-- more -->

Our application should sort the data to the different folders regarding to caller countries, so let’s check it. 

To do that we need to create folders for cucumber ‘features’:

`src -> test -> resources -> features -> callstream`

Lets create a new file `folder_structure.feature` for the feature inside `callstream` folder.

At this point we don’t check the output data files format, only the folders structure.  
To clarify that we should create understandable and clear feature description in terms of Gherkin:

{% highlight gherkin %}
Feature: CallStream normal data folder structure
  Check is CallStream job creates the correct file/directory structure
  for the collected callstream data
{% endhighlight %}

Now we can write first ‘happy case’ scenario:

{% highlight gherkin %}
  Scenario: processing data from one user from UA
  
{% endhighlight %}

Assume we have an input data file with one caller from Ukraine:

{% highlight gherkin %}
    Given a file with theme calls_log containing the following lines
      | country |
      | UA      |
{% endhighlight %}

I don’t include other data fields, because we will use default ones. 
Then we should run the processing job. 

{% highlight gherkin %}
    When I run the CallStream job
{% endhighlight %}

And check that job create the correct folder structure:

{% highlight gherkin %}
    Then the following directory structure should be created
      | UA/ |
{% endhighlight %}

And, of course, it should contain file with processed data:

{% highlight gherkin %}
    And folder 'UA/' should contain following files
      | part-00000-00000 |
{% endhighlight %}

And, in summary:

{% highlight gherkin %}
Feature: CallStream normal data folder structure
  Check is CallStream job creates the correct file/directory structure
  for the collected callstream data
  
  Scenario: processing data from one user from UA
    Given a file with theme calls_log containing the following lines
      | country |
      | UA      |
    When I run the CallStream job
    Then the following directory structure should be created
      | UA/ |
    And folder 'UA/' should contain following files
      | part-00000-00000 |
{% endhighlight %}

It’s important to wrote these steps before implementation, so we will not limit our mind with details of our tests implementation. 
After creation of scenario, we should create implementation of each step of the scenario. 
Implementation for steps lives in src -> java -> com -> steps folder. 
For our callstream feature we should create `callstream` folder  and add `FolderStructureSteps.java` into it. 
Implementation of steps looks like that (I will not provide whole class here, it’s too large, and it’s not necessary to provide it here. 
You can take a look in [article repository](https://github.com/savvyclutch/bdd_test_hadoop) for more)  :

{% highlight java %}
public class FolderStructureSteps {

    private IOPath ioPath;

    public FolderStructureSteps(IOPath ioPath) {
        this.ioPath = ioPath;
    }

    @Given("^a file with theme (calls_log) containing the following (lines)$")
    public void aFileContainingTheFollowingLines(String themeName, String dataPoint, DataTable table) throws Throwable {

        List<Map<String,String>> test_lines = table.asMaps(String.class, String.class);

        Chunk lines = getTemplate(themeName + "#" + dataPoint);
        lines.set(dataPoint, test_lines);

        this.ioPath.input_path = writeDataToFile("data.dat", lines.toString().getBytes());
    }
...
{% endhighlight %}

So, it is pretty simple - we match step string to the java regular expression in give/when/then annotations and wrote step implementation. 
For data fixture generation we use simple template generator  `Chunk Templates`. 
It uses `themes` folder for templates, so we habe to create folder in `themes` in test resources and add `calls_log.chtml` template with content:

{% highlight django %}
{% raw %}
{#lines}
{% loop in $lines as $line_parameters%}{% include #single_line_snippet %}{% endloop %}
{#}

{#single_line_snippet}
{$line_parameters.id:0000},{$line_parameters.country:UA},{$line_parameters.time:1433998201},{$line_parameters.duration:60000}
{#}
{% endraw %}
{% endhighlight %}

As you can see we use default values for fields, which keep our scenarios in Gherkin clean and readable.  

Now we have a basic test structure for acceptance tests and everyone can extend the test suite and include output content checks or other important things. 
 
Full code of the project you can find here: [https://github.com/savvyclutch/bdd_test_hadoop](https://github.com/savvyclutch/bdd_test_hadoop)


