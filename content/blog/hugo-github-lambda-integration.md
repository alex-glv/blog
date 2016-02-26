+++
date = "2016-02-26"
draft = false
title = "Hugo on Amazon AWS Lambda integrated with GitHub"
description = "Automate your Hugo website generation with on Amazon AWS Lambda with GitHub Webhooks"

+++

# The Problem

I love static website generators. I think it's a great iteration over the old approach before dynamic websites became a de-facto standard.

The most compelling argument towards using static websites is "minimum-hosting" approach.
The only thing you need is a web server and a domain name.

You still have the problem of continuos delivery. If you are anything like me, you don't want to keep rebuilding and deploying your website manually.

Make changes, test locally, push, make a sandwich and reflect on eternity of consciousness in the Universe. The changes are live. Ah, beautiful.

# Inspiration

My approach was inspired by [this blogpost](bezdelev.com/post/hugo-aws-lambda-static-website/).
I though, huh, this is cool. Then I set it up and quickly realised its shortcoming: manual S3 website upload.

I decided to take it a step further and integrate the process with GitHub.

# Prerequisites & process

Some familiarity with AWS is required.

You have configured all the required components on AWS to publish your website:

 - S3 bucket with the source files
 - Route 53 for DNS
 - CloudFront for SSL and load balancing.
 
This guide will only address publishing your static content through Hugo compiler from GitHub to S3 bucket.

My AWS configuration is very similar described in the post above. 

There are some changes, however: 

 - I don't have the input.<website> bucket, the changes are pulled off GitHub
 - I have configured AWS SNS with GitHub WebHooks. Each commit is trigering SNS which in turn is configured as AWS Lambda event source.

# Introducing GitHub-Hugo-Lambda integration project

[Github Hugo Lambda](https://github.com/alex-glv/github-hugo-lambda) project should do most of the configuration for you.
It will initialise Node dependencies, create Lambda code package and upload it on Amazon AWS Lambda.
There don't seem to be a way to create Event source mapping from SNS to Lambda through AWS Cli so you have to do it manually.
You have to create GitHub.com webhook service set-up to fire Amazon SNS event.

Start by checking out the project:

```git clone https://github.com/alex-glv/github-hugo-lambda```

Create config files from provided samples:

```
cp {sample,}.config.mk
cp {sample,}.config.json
cp {sample,}.rolepolicyS3.json
```

Edit the config files, this should be straightforward

Then, run:

```make createsns```

This will create an SNS topic.
Now, navigate to GitHub.com project Settings -> Webhooks & services -> Add Service -> Amazon SNS.
Fill in the required values and submit. This should be it for configuring GitHub.
We need to create Roles and Policies to allow Lambda access to our S3 bucket.

Run ```make createrole``` to create the role and necessary policies.

Now run:

```make initnodedeps && make build && make deploy```

This will pull NodeJS dependencies, zip the files and push it to Amazon Lambda.
After this has been completed, go to Lambda configuration page, add Event source and choose the SNS entry you created earlier.

At this point, everything should be set.

NOTE: p7zip is a must for creating the archive. It seems that de-facto zip archiver on Unix does not retain executable flags that are required to run hugo, because it's a compiled executable.

You can rune the test by clicking "Test service" from the Github webhooks page and it will send the test payload.

# Conclusion

Hopefully this post and project help you to streamline creation of simple static websites.
Configured once, deployment will be fully automated and you can concentrate on creating value rather than worrying how to publish your website changes.



