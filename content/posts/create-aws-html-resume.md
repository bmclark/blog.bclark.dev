---
title: "Create an HTML Resume and Serve it on AWS"
date: 2024-05-01T17:01:04-04:00
author: "Bryan Clark"
authorTwitter: "" #do not include @
cover: ""
tags: ["AWS", "CloudFront", "cloud-resume-challenge", "how-to"]
keywords: ["Cloud Resume Challenge", "AWS", "tutorial", "html resume", "Github Copilot", "CloudFront CDN"]
description: "This tutorial discusses how to create an HTML resume and deploy it as a static site via AWS with a custom SSL certificate."
showFullContent: false
readingTime: true
hideComments: false
color: "" #color from the theme settings
toc: "true"
---

This is the first part in a multipart series detailing my actions going through the [Cloud Resume Challenge](https://cloudresumechallenge.dev/docs/the-challenge/aws/)[^1]. This article is going to go over steps 1 through 6 of the Cloud Resume Challenge. The steps are as follows:

1. Get your AWS Cloud Practitioner cert.
1. Create an HTML based resume
1. Style the HTML with CSS
1. Upload as a static website on S3
1. Set up HTTPS via CloudFront
1. Set up DNS

## Get your AWS Cloud Practitioner cert
OK, this should be easy enough. This cert is set up for even non technical people to learn about AWS/cloud technologies and how they can be best implemented in your business. Take a course, take notes, do some practice exams, take your test. I will say this was probably one of the worst certification courses I've ever taken regarding trick questions, so pay attention.

## Create an HTML based resume
I am not a front end developer by any means. I just wanted to get something done quick and dirty to get it up there and will come back and fix it up later. I took my resume and pasted it into VSCode and had Github Copilot turn it into an HTML resume. That gave me a good scaffolding to work from and then I just tweaked from there. I created some tables and also a "banner" div for my contact info.

`Prompt: generate a basic resume scaffold in html with blank info`
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>My Resume</title>
	<style>
		/* Add your CSS styles here */
	</style>
</head>
<body>
	<header>
		<h1>My Resume</h1>
	</header>
	<main>
		<section>
			<h2>Contact Information</h2>
			<ul>
				<li>Name: </li>
				<li>Email: </li>
				<li>Phone: </li>
				<li>Address: </li>
			</ul>
		</section>
		<section>
			<h2>Education</h2>
			<ul>
				<li>Degree: </li>
				<li>University: </li>
				<li>Year: </li>
			</ul>
		</section>
		<section>
			<h2>Experience</h2>
			<ul>
				<li>Company: </li>
				<li>Position: </li>
				<li>Duration: </li>
			</ul>
		</section>
		<section>
			<h2>Skills</h2>
			<ul>
				<li>Skill 1</li>
				<li>Skill 2</li>
				<li>Skill 3</li>
			</ul>
		</section>
	</main>
</body>
</html>
```

## Style the resume with CSS
Again, not a fan of CSS so I had Copilot do some basic CSS and tweaked it from there. Eventually I will come back to this and use bootstrap or tailwind to make it look nice, but today the focus is on getting things done and this site needs to go live! 🚀 it!

`Prompt: generate a pretty css config for the html resume`
```css
body {
	font-family: Arial, sans-serif;
	background-color: #f2f2f2;
}

header {
	background-color: #333;
	color: #fff;
	padding: 20px;
}

h1 {
	margin: 0;
}

section {
	margin-bottom: 20px;
}

h2 {
	color: #333;
	margin-bottom: 10px;
}

ul {
	list-style-type: none;
	padding: 0;
}

li {
	margin-bottom: 5px;
}

li::before {
	content: "\2022";
	color: #333;
	display: inline-block;
	width: 1em;
	margin-left: -1em;
}
```

## Upload the resume as a static website on S3
Ok, now the real work begins (sorry front end devs 😝). Log in to your AWS console (you aren't using your root IAM user, are you?) and select S3. Now we're going to create a bucket. Enter your bucket name (we'll use "cloud-resume-aa123" as the example name for this tutorial). We're going to leave all settings as default except for the "Block all public access" portion. Obviously we *want* the public to access our website so uncheck all of that.

![S3 Bucket Access](/s3-bucket-access.png)

Once you have created the bucket, go ahead and click on it and we're going to upload objects. Upload your index.html and style.css or whatever files you've created. Now we're going to go to properties and turn on S3 static website hosting. Specify the "index" page for your site (index.html, resume.html... whatever your resume html filename is) and go ahead and enable hosting. S3 will give you a website address to access your bucket.

Now you might think we're done, but if you do I bet you didn't pass your AWS Cloud Practitioner exam, did you? We need to set a policy for access![^2] Go ahead and input the below code in your bucket policy under the permissions tab. Be sure to change the resource to your bucket's arn.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cloud-resume-aa123/*"
        }
    ]
}
```

Now you should be able to access your resource via the static web address. [Here](http://cloud-resume-aa123.s3-website-us-east-1.amazonaws.com/) is our example.

## Set up HTTPS via CloudFront
OK, that's great and all, but it isn't secure. Doesn't ***everything*** need to be served over HTTPS on the Internet these days? Not really, and certainly not a basic HTML resume, but we're trying to showcase that we know what we're doing and in reality we'll be deploying all sorts of applications that ***do*** need to be served securely. So let's go ahead and serve our site over HTTPS. We'll do this through CloudFront and ACM (AWS Certificate Manager).

Create a CloudFront distribution and select your S3 bucket. Ensure that you are using the website endpoint (AWS will probably prompt you to remind you). Select "Do not enable security protections" under Web App Firewall. Then scroll down to settings. Add your domain (and any applicable subdomains such as www) to the Alternate Domain Name field. Then look for Custom SSL Certificate. Select "Request Certificate" which will take you to ACM and allow you to request an SSL cert for your domain.

![CloudFront distribution custom domain and SSL](/custom-domain-ssl-certificate.png)

Follow the prompts for ACM to create an SSL cert for your domain. I would recommend DNS validation[^3] which is pretty simple. ACM will give you a record name and value which you will enter as a CNAME entry in your DNS provider for your domain. Once it propagates and is validated your certificate will be set up.

Now, back on the CloudFront distribution setup you can select the refresh icon beside the custom SSL certificates and you should be able to select your domain from the dropdown. Now you can create the distribution and CloudFront will begin deploying it. You'll have a new URL that you can access which will be something like this: [d2sfswoikqyxcb.cloudfront.net](https://d2sfswoikqyxcb.cloudfront.net). If you visit that address and see that you have an HTTPS connection and can view your resume you're good to move to the next step.

## Set up DNS

Last but not least, you'll want to take your weird CloudFront URL and add it to your DNS provider as a CNAME record for your domain. The name will be your root domain and then the value will be your CloudFront URL. This will send all traffic to your route domain to the CloudFront distribution. If you properly validated your certificate via DNS in the last section you'll have a working SSL cert for your domain verified by Amazon.

## Wrap Up

So what did we learn today? Well, it's pretty simple to host a static site via an S3 bucket on Amazon, especially if there is no need for an SSL cert. Even still, it's trivial to add an SSL cert for a custom domain. It's seamless to host your static site via the AWS CloudFront CDN across the globe or in select continents.

### What's next?
Looking forward, obviously cleaning up the HTML and making the css look prettier with something like tailwind or bootstrap will be pretty important. Really though, I would much rather get some CI/CD set up so whenever I push a change to my git repo for the site it is propagated out throughout the chain. I would also like to set all of this up via terraform rather than manually configuring everything. IaC FTW 💪💪🏆🥇

## References
[^1]: [Cloud Resume Challenge](https://cloudresumechallenge.dev/docs/the-challenge/aws/)
[^2]: [AWS: Setting permissions for website access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteAccessPermissionsReqd.html)
[^3]: [AWS: DNS validation](https://docs.aws.amazon.com/acm/latest/userguide/dns-validation.html)