---
layout: post
title: Continuous deployment to AWS ECS from CircleCI
description: "Tutorial, how to configure AWS ECS cluster, CircleCI and manage secrets during deployment"
tags: [docker, aws ecs, circleci, devops]
author: bogdan
image:
    feature: posts/aws_deploy/title.jpg

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

Simple, heh?

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
I suggest to use more specific name, like `projectname-server`. Save `Repository URL`. 

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

Create bucket on S3 with name `ecs-secrets` and add following policy to the bucket (`Properties` -> `Permissions` -> `Edit bucket policy`)

{% highlight json %}
{% raw %}
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "DenyUnEncryptedObjectUploads",
			"Effect": "Deny",
			"Principal": "*",
			"Action": "s3:PutObject",
			"Resource": "arn:aws:s3:::ecs-secrets/*",
			"Condition": {
				"StringNotEquals": {
					"s3:x-amz-server-side-encryption": "AES256"
				}
			}
		},
		{
			"Sid": " DenyUnEncryptedInflightOperations",
			"Effect": "Deny",
			"Principal": "*",
			"Action": "s3:*",
			"Resource": "arn:aws:s3:::ecs-secrets/*",
			"Condition": {
				"Bool": {
					"aws:SecureTransport": "false"
				}
			}
		},
		{
			"Sid": "Access-from-specific-VPC-only",
			"Effect": "Allow",
			"Principal": "*",
			"Action": [
				"s3:GetObject",
				"s3:PutObject",
				"s3:ListBucket"
			],
			"Resource": [
				"arn:aws:s3:::ecs-secrets",
				"arn:aws:s3:::ecs-secrets/*"
			],
			"Condition": {
				"StringEquals": {
					"aws:sourceVpc": "<YOUR VPC ID>"
				}
			}
		}
	]
}
{% endraw %}
{% endhighlight %}


This policy allow to put only encrypted files inside, and allow to get files from specific VPC only.  
(Assume you already have configured VPC. If not - you can create new one on `Create cluster` step and use this new VPC for bucket policy).

To upload a new file to the bucket you can use following command with [aws cli](https://aws.amazon.com/cli/) (AWS command line tool):

`aws s3 cp website_secrets.txt s3://ecs-secrets/website_secrets.txt --sse`

Put your secret environment inside `website_secrets.txt`, for example:


{% highlight bash %}
SECRET_KEY_BASE=adsafasdfsafwfwefdsfsdacwaeewfdadsfasdfewceascadcasdcdsadceeas
DB_HOST=db_host_dress
DB_USER=dbuser
DB_PASSWORD=supersecretpass
{% endhighlight %}

Also, I suggest to enable logging for this bucket — just in case. 

## Configure docker image

Now we need to configure our docker image to load secrets from S3 into container environment on container start. We will use endpoint script for this.
It will load each line of the `website_secrets.txt` into container environment, so all environment variables will be accessible by webserver. 
Create following `secrets-endpoint.sh` inside your repository:

{% highlight bash %}
#!/bin/bash

# Load the S3 secrets file contents into the environment variables
eval $(aws s3 cp s3://ecs-secrets/website_secrets.txt - | sed 's/^/export /')

exec "$@"
{% endhighlight %}

Don't forget to add `execute` permissions to script.

{% highlight bash %}
$ chmod +x ./website_secrets.txt
{% endhighlight %} 

As you can see, this script use aws cli to download file with secrets, so before run it in container we should install aws cli into docker images. 
Change your `Dockerfile` and add following lines to install aws cli inside docker image (example for debian-based distro images, like Ubuntu):

{% highlight bash %}
...
RUN apt-get install -y python curl unzip && cd /tmp && \
    curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" \
    -o "awscli-bundle.zip" && \
    unzip awscli-bundle.zip && \
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws && \
    rm awscli-bundle.zip && rm -rf awscli-bundle
...
{% endhighlight %}

and, we need to put endpoint script to the image and run it right before `CMD` line in `Dockerfile`:

{% highlight bash %}
...
ADD ./secrets-entrypoint.sh /
ENTRYPOINT ["/secrets-entrypoint.sh"]
CMD <YOUR RUN OPTIONS HERE>
{% endhighlight %}

## Create cluster

For cluster creation I suggest to use [ecs cli](https://github.com/aws/amazon-ecs-cli). There is two reasons to use ecs cli insted of aws cli - first of all it mo simple to use.
The secуnd reason - ecs cli setup cluster through [Cloud Formation](https://aws.amazon.com/cloudformation/) which will manage cluster resources - EC2 instances, vpc, security roles etc.
So we don't need to do that manually. Additionally it allow to use docker-compose yml file for Task Definition creation. We are not going to use this feature, but it can be helpful in other scenarios.

So, install the ecs cli and create configuration file for it (on Linux it will be `~/.ecs/config`). Example config file:
{% highlight conf %}
[ecs]
cluster = <CLUSTER NAME>
aws_profile = default
region = us-east-1
aws_access_key_id =
aws_secret_access_key =
{% endhighlight %}

Choose your cluster name at that point. Usually I create the name by pattern `projectname-formation` to add information how this cluster resources are managed (`Cloud Formation` in this case).

To create cluster on existing VPC run the following command:

{% highlight bash %}
$ ecs-cli up --keypair <key here> --capability-iam --size 2 --vpc <VPC ID> --subnets subnet-<SUBNET 1 ID>,subnet-<SUBNET 2 ID>
{% endhighlight %}

Keypairs are stored on `AWS EC2` -> look to the left panel -> `Key pairs`. There you can create key pair to access to the EC2 instances.

If you still have no VPC, `ecs-cli up` without `--vpc` option will create a new one. 

You should add minimum 2 instances in to your cluster in order to have one machine free as deployment target. 

## Configure cluster Service

After cluster creation we should create `Service` in the cluster to manage our `Tasks`. Before do that, lets push latest docker image to the ECR repository and register a new Task for this image.

Login to the ECR:
{% highlight bash %}
$ $(aws ecr get-login --region us-east-1)
{% endhighlight %}

Build new image
{% highlight bash %}
$ docker build -t my-website .
{% endhighlight %}

Tag image with ECR tags
{% highlight bash %}
$ docker tag my-website <ECR Repository URL>:1
$ docker tag my-website <ECR Repository URL>:latest
{% endhighlight %}

Where `<ECR Repository URL>` - URL to the project docker repository (looks like `1234567890.dkr.ecr.us-east-1.amazonaws.com/my-website`)

Push images to ECR
{% highlight bash %}
$ docker push <ECR Repository URL>:1
$ docker push <ECR Repository URL>:latest
{% endhighlight %}

Now, let's create a new `Task Definition` and register it on ECS:

{% highlight bash %}
$ sed -e "s;%IMAGE_TAG%;1;g" ecs-task-template.json > my_website-1.json 
$ aws ecs register-task-definition --family <TASK FAMILY> --cli-input-json file://my_website-1.json
{% endhighlight %}

Where `<TASK FAMILY>` can be any name you want to group your `Tasks`. Ususally, I use `project_name`, like `my_great_website`.
 
Ok, now we can create `Service`. Go to the ECS -> your cluster -> `Service` tab -> click `Create` button.
 
Choose your `Task Definition`, add service name (`website`) and set 1 task in field `Number of tasks`.
`Service` should run container on one of the registered instances. 

Let's make sure that everything works. Go to the `Tasks` tab and click on value in `Container Instance` column for the active task.
You should be able to see `Public IP`. Try to open it in browser, and check is everything works as expected.

### Troubleshooting

If something goes wrong, you can check your container in instance. To do that, at first, you must allow SSH connection to the instance. 

* Go to instance on EC2 (you can do it from `Container Instance` page)
* `Description` -> `Security groups`
* Click on current security group
* Go to `Inbound`
* Click `Edit`
* Add rule for SSH

Connect to the instance:

{% highlight bash %}
$ ssh -i your_keypair.pem -o 'IdentitiesOnly yes' ec2-user@<INSTANCE IP>
{% endhighlight %}

where `<INSTANCE IP>` is public IP for EC2 instance with pur container. 

After that you can check containers on this instance:
{% highlight bash %}
$ docker ps -a
{% endhighlight %}

And see container logs:
{% highlight bash %}
$ docker logs -f <CONTAINER ID>
{% endhighlight %}

Also you can do regular docker stuff. 

Don't forget to remove SSH permission from instance after debugging!

## Configure CircleCI

If everything works fine, we can configure CircleCI to automatically deploy new version of docker image to the AWS ECS.
Add empty `circle.yml` to the project and connect project on service.

We need AWS credentials to allow CircleCI to push new images to the ECR and update ur ECS Service. I's highly recommended to create separate role for that on AWS IAM.
 
Add AWS credentials to the CircleCI project settings.

project settings -> `Permissions` section -> `AWS permissions` -> add AWS key and secret

`aws cli` expect default region for work, so we need to set environment variable with AWS region in build environment

Add default AWS region:

project settings -> `Build Settings` section -> `Environment Varibles` -> add `AWS_DEFAULT_REGION`  varible with you region (`us-east-1` in my case)

Now, configure `circle.yml` to build new image and push it to the ECR and run deployment script.  Example:

{% highlight yaml %}
machine:
  services:
    - docker

dependencies:
  override:
    - $(aws ecr get-login --region us-east-1)
    - docker build -t my-website .

test:
  override:
    ## put your test command here
    ## - docker run -e RAILS_ENV=test -it my-website rake test

deployment:
  hub:
    branch: master ## do deployment on commit to the master branch only 
    commands:
      - docker tag my-website <ECR Repository URL>:$CIRCLE_SHA1
      - docker tag my-website <ECR Repository URL>:latest
      - docker push <ECR Repository URL>:$CIRCLE_SHA
      - docker push <ECR Repository URL>:latest
      - ./deploy.sh
{% endhighlight %}

Now we need to creare `deploy.sh`. This script should:

1. Create new task definition for docker image with `$CIRCLE_SHA1` tag
2. Register it in cluster `Service`
3. Run `Service` update process

Example:

{% highlight bash %}
#!/bin/bash
SERVICE_NAME=<YOUR SERVICE NAME>
CLUSTER_NAME=<YOUR CLUSTER NAME>
BUILD_NUMBER=${CIRCLE_BUILD_NUM}
IMAGE_TAG=${CIRCLE_SHA1}
TASK_FAMILY=<YOUR TASK FAMILY>

# Create a new task definition for this build
sed -e "s;%IMAGE_TAG%;${IMAGE_TAG};g" ecs-task-template.json > my_website-${BUILD_NUMBER}.json
aws ecs register-task-definition --family ${TASK_FAMILY} --cli-input-json file://my_website-${BUILD_NUMBER}.json

# Update the service with the new task definition and desired count
TASK_REVISION=`aws ecs describe-task-definition --task-definition ${TASK_FAMILY} | egrep "revision" | tr "/" " " | awk '{print $2}' | sed 's/"$//'`
DESIRED_COUNT=`aws ecs describe-services --cluster ${CLUSTER_NAME} --services ${SERVICE_NAME} | egrep "desiredCount" | head -1 | tr "/" " " | awk '{print $2}' | sed 's/,$//'`
if [ ${DESIRED_COUNT} = "0" ]; then
    DESIRED_COUNT="1"
fi

aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --task-definition ${TASK_FAMILY}:${TASK_REVISION} --desired-count ${DESIRED_COUNT}
{% endhighlight %}

Don't forget to replace `<YOUR SERVICE NAME>` `<YOUR CLUSTER NAME>` `<YOUR TASK FAMILY>` with the correct values.

That's it. Now you have Continues Deployment to the Amazon EC2 Container Service. To more information, please check the Literature section.
Hopefully this will be helpfull for someone. 

# Literature

1. [How to Manage Secrets for Amazon EC2 Container Service–Based Applications by Using Amazon S3 and Docker](https://blogs.aws.amazon.com/security/post/Tx2B3QUWAA7KOU/How-to-Manage-Secrets-for-Amazon-EC2-Container-Service-Based-Applications-by-Usi)
2. [Set up a build pipeline with Jenkins and Amazon ECS](https://blogs.aws.amazon.com/application-management/post/Tx32RHFZHXY6ME1/Set-up-a-build-pipeline-with-Jenkins-and-Amazon-ECS)
3. [Amazon EC2 Container Service Docs](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)