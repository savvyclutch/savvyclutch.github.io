---
layout: post
title: Continuous deployment to AWS ECS from CircleCI with secrets keys management
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

Our deployment process will be look like this


1. Create task template for ECS service
2. Configure secrets bucket
3. Configure docker image
4. Configure CircleCI

##  Create task template for ECS service

# Literature

1. [How to Manage Secrets for Amazon EC2 Container Service–Based Applications by Using Amazon S3 and Docker](https://blogs.aws.amazon.com/security/post/Tx2B3QUWAA7KOU/How-to-Manage-Secrets-for-Amazon-EC2-Container-Service-Based-Applications-by-Usi)
2. [Set up a build pipeline with Jenkins and Amazon ECS](https://blogs.aws.amazon.com/application-management/post/Tx32RHFZHXY6ME1/Set-up-a-build-pipeline-with-Jenkins-and-Amazon-ECS)
3. [Amazon EC2 Container Service Docs](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)