---
layout: post
title: Continuous deployment to AWS ECS from CircleCI
description: "Tutorial, how to configure AWS ECS cluster, CircleCI and manage secrets during deployment"
tags: [docker, aws ecs, circleci, devops]
author: bogdan
image:
    feature: feature.jpg
    credit: dargadgetz
    creditlink: http://www.dargadgetz.com/ios-8-abstract-wallpaper-pack-for-iphone-5s-5c-and-ipod-touch-retina/
---

Intro here

We have some kind of website with dockerised environment and we want to configure automatic zero-time deployment on push to the our github repository `master` branch.

![Deploy Process]({{ site.url }}/images/posts/aws_deploy/deploy_process.png)

What we a going to use to do that:

1. CircleCI as build server
2. Github as code repository
3. Amazon EC2 Container Service (ECS) as production environment and our deployment target
4. Amazon S3 bucket to keep our secret keys used by website

<!-- more -->

# CircleCI

It's great build service integrated with Github. Very easy configurable by yml file - just put `circle.yml` into your repository, configure dependencies, test and deploy sections and that's it. 

# Amazon EC2 Container Service

![AWS ECS]({{ site.url }}/images/posts/aws_deploy/awsecs_cluster.png)
{: .image-right}

AWS ECS organise cluster from multiple Amazon machines (AWS EC2 instances). 
To run container on cluster from your docker image you should define `Task` - which is just resources definition for your container instance, like which docker image to use, how much memory to allocate for container etc.
`Tasks` runs on cluster instances, and which `Tasks` and how many `Tasks` should be are defined in `Service`.
Also,`Service` settings allow to configure which ELB (Amazon load balancer) to use for current cluster and keep the rules for Auto Scaling - in case you want to change number of containers dynamically. 

To deploy new version of your docker image you should create a new `Task Definition` with new image tag inside, register it in `Service` and run `Service` update  after that to populate new `Task`. 

On `Service` update, new `Task` will be run on any free reserved EC2 instance, and, after that, old `Task` will be stopped.  
Unfortunately, if you want to have smooth updating without maintenance period, you can't just run `Task` with new image version on the same machine because it use the same port as previous one, so to do that you should shot down the old container before run of new one.
Which means you should keep one EC2 instance on cluster free, for deployment purposes. 

![AWS ECS]({{ site.url }}/images/posts/aws_deploy/cluster_update.png)

# Amazon EC2 Container Registry (ECR) 

It's a docker registry like Dockerhub, but managed by Amazon and have more options on access management. We will use it because of speed and security reasons. 

# S3 bucket

One of the challenges when deploying production applications using Docker containers is deciding how to handle run-time configuration and secrets (database credentials, certs, etc.). 
We are going to use S3 bucket with the special access rules based on AWS Identity and Access Management (IAM) role to handle this.
 

# What we are going to do

So, our detailed deployment process will be look like this:

![Detailed Process]({{ site.url }}/images/posts/aws_deploy/process.png)

And implementation steps:

1. Create repository for docker images
2. Create task template for ECS service
3. Configure secrets bucket
4. Configure docker image
5. Create cluster
6. Configure cluster Service
7. Configure CircleCI

## Create repository for docker images

Go to the `Amazon EC2 container service` -> `Repositories` in your AWS Console. Click `Create repository` and enter repository name.
I suggest to use more specific name, like `projectname-server`. Save `Repository URL` and follow the instructions. 

##  Create task template for ECS service

Now we need to create template for `Task Definitions`. We will create new `Task Definition` after build new docker image using this template.

`ecs-task-template.json`

{% highlight json %}
{% raw %}
 {
     "family": "<TASK FAMILY>",
     "containerDefinitions": [
         {
             "image": "<Repository URL>:%IMAGE_TAG%",
             "name": "<TASK NAME>",
             "cpu": 384,
             "memory": 512,
             "essential": true,
             "portMappings": [
                 {
                     "containerPort": 80,
                     "hostPort": 80,
                     "protocol": "tcp"
                 }
             ]
         }
     ]
 }
{% endraw %}
{% endhighlight %}

Replace `<Repository URL>` with docker repository url from the previous step. `<TASK FAMILY>` we will replace later with the correct `Task` family option. 
You can choose  any `<TASK NAME>` you want (`website`, for example).  

On deployment our deploy script will replace `%IMAGE_TAG%` to the real one and produce new `Task Definition` for `Service`. 
`memory`, `portMappings` and `cpu` options are self-explained, but make sure that you allocate enough memory for container.

## Configure secrets bucket

## Configure docker image

## Create cluster

## Configure cluster Service

## Configure CircleCI

# Literature

1. [How to Manage Secrets for Amazon EC2 Container Service–Based Applications by Using Amazon S3 and Docker](https://blogs.aws.amazon.com/security/post/Tx2B3QUWAA7KOU/How-to-Manage-Secrets-for-Amazon-EC2-Container-Service-Based-Applications-by-Usi)
2. [Set up a build pipeline with Jenkins and Amazon ECS](https://blogs.aws.amazon.com/application-management/post/Tx32RHFZHXY6ME1/Set-up-a-build-pipeline-with-Jenkins-and-Amazon-ECS)
3. [Amazon EC2 Container Service Docs](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)