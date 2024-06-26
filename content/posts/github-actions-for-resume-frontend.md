+++
title = "Setting up GitHub Actions to Deploy AWS S3 Static Website"
date = 2024-05-02T23:04:00-04:00
lastmod = 2024-05-03T12:46:22-04:00
draft = false
author = "Bryan Clark"
tags = ["AWS", "Cloud Resume Challenge"]
keywords = ["AWS", "Cloud Resume Challenge", "deploy S3 static website", "GitHub Actions"]
description = "A tutorial on how to use GitHub actions to securely deploy a static website to an S3 bucket and invalidate CloudFront CDN cache."
+++

## Overview {#overview}

Let's discuss setting up some CI/CD actions to automatically deploy our HTML resume to our S3 site every time we make a new commit. This is supposed to be completed later on in the Cloud Resume Challenge, however, I think it's silly not to implement this right away. I don't want to have to manually do anything when I can automate it. It's going to make it easier, save time, and reduce mistakes.

We're going to use GitHub actions to deploy our code to the AWS S3 bucket and then invalidate the CloudFront CDN cache every time a new commit is pushed to the `main` branch of our HTML resume repo. We're also going to set up an IAM role to allow GitHub to only access the S3 bucket and CloudFront in our AWS environment.


## Create Code Repository {#create-code-repository}

OK, if you weren't already commiting your code to a repo, why not? Seriously, everything should be in git (except sensitive info of course). Make a repo now if you haven't already.


## Create IAM Role {#create-iam-role}

Luckily for us, AWS has an excellent article[^fn:1] describing how to create a secure IAM role for GitHub actions. Go into your IAM console and set up OpenID Connect as an identity provider. We'll be using the GitHub/AWS specific URLs which is `https://token.actions.githubusercontent.com` for the `Provider URL` and `sts.amazonaws.com` for the `Audience`.

Next we'll create the role that will use the identity provider we just created. `Web indentity` should already be selected, be sure to pick `sts.amazonaws.com` as the Audience again. For permissions we need to enable full access for S3 and CloudFront so that the action can add/delete files in the S3 bucket and also invalidate the CloudFront cache.

Go ahead and finish creating the role. We'll need the role's ARN in the next step, so make sure you capture that.


## Create GitHub Action {#create-github-action}


### About GitHub Actions {#about-github-actions}

Now we're going to create the GitHub action workflow to deploy our site automagically on each push to the `main` branch. I would recommend reviewing the Learn GitHub Actions[^fn:2] and also the Using workflows section[^fn:3] if you aren't familiar with GitHub Actions.


### Set up Secrets! {#set-up-secrets}

First, let's set up our secrets. There are 2 items we'll add. The first being the ARN for the IAM role we just created, the second is the CloudFront Distribution ID. Go into your repo settings and choose "Secrets and Variables" and then create a secret named `AWS_ROLE_ARN` and one named `AWS_CF_DIST_ID` or whatever you want to call them. Add the appropriate values and then load up your project in your IDE.


### Create our workflow {#create-our-workflow}

Now we'll go ahead and create our workflow. Create the following file `.github/workflows/main.yml` and open it in your IDE. I'll break down what we're adding and why.


#### Define when to run the action {#define-when-to-run-the-action}

```yaml
name: push cloud resume to s3 website
on:
  push:
    branches:
      - main
```

This should be fairly self explanatory. Whenever there is a push to our main branch in this repo, the action is triggered and will run the jobs we specify in this file.


#### Set permissions for AWS JWT {#set-permissions-for-aws-jwt}

```yaml
permissions:
  id-token: write
  contents: read

```

This is required for our AWS credential action later on. Check out the documentation here[^fn:4].


#### Create job and steps to run {#create-job-and-steps-to-run}

```yaml
jobs:
deploy:
  runs-on: ubuntu-latest
  steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
        aws-region: us-east-1
```

OK, now we're getting into our jobs. This job is named `deploy` and it is going to run on an Ubuntu VM. It will check out this repo and then configure our AWS credentials so we have access to our environment. `role-to-assume` is going to use our IAM role that we created and the value will be pulled from the secret that we saved in this repo's settings. Adjust the `aws-region` to the appropriate value for your environment.

```yaml
- name: Deploy to S3
  run: |
    aws s3 sync . s3://cloud-resume-aa123 --delete
```

Now that we have configured our credentials we can access our AWS services. First we'll sync our files to S3 using the `aws-cli`. We're going to use the `sync` command[^fn:5]. Be sure to update the s3 bucket name to your appropriate value.

```yaml
- name: Invalidate CloudFront cache
  run: |
    aws cloudfront create-invalidation --distribution-id ${{ secrets.AWS_CF_DIST_ID }} --paths "/*"
```

Finally, we'll need to invalidate our CloudFront cache. If we don't do this our updates aren't going to be seen when visiting our website. We'll use the `aws-cli` tool again using the CloudFront `create-invalidation` command[^fn:6].


### Putting it all together {#putting-it-all-together}

Now that we have our workflow defined we can go ahead and commit it to our repo and the build should kick off. This will give us an opportunity to troubleshoot anything that may be misconfigured. We'll be able to see the build and logs in the `Actions` tab in the repo.

Here is the full `.github/workflows/main.yml` file for reference:

```yaml
name: push cloud resume to s3 website
on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Deploy to S3
        run: |
          aws s3 sync . s3://cloud-resume-aa123 --delete

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation --distribution-id ${{ secrets.AWS_CF_DIST_ID }} --paths "/*"

```


## Recap {#recap}

Now we have successfully set up CD for our HTML resume using GitHub actions. We've successfully configured an minimal IAM role to ensure only the minimal access necessary to services to deploy the site. No more manually uploading files to our S3 bucket and invalidating our CDN cache. Great work 🎉


## References {#references}

I would also like to mention an excellent post that I came across by Rebecca Murillo found [here](https://rebeca.murillo.link/en/blog/cicd-deploy-static-website-to-aws-s3/).

[^fn:1]: <https://aws.amazon.com/fr/blogs/security/use-iam-roles-to-connect-github-actions-to-actions-in-aws/>
[^fn:2]: <https://docs.github.com/en/actions/learn-github-actions>
[^fn:3]: <https://docs.github.com/en/actions/using-workflows>
[^fn:4]: <https://github.com/aws-actions/configure-aws-credentials/tree/v4>
[^fn:5]: <https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html>
[^fn:6]: <https://docs.aws.amazon.com/cli/latest/reference/cloudfront/create-invalidation.html>
