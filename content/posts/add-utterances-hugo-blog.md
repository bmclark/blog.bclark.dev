+++
title = "Add Comments to a Hugo Blog Using Utterances"
date = "2023-08-07T16:20:43-04:00"
author = "Bryan Clark"
authorTwitter = "" #do not include @
cover = ""
tags = ["hugo", "utterances", "github", "how-to"]
keywords = ["hugo blog comments", "hugo utterances setup"]
description = "This tutorial discusses how to add Utterances to a hugo blog in order to have comments."
showFullContent = false
readingTime = true
hideComments = false
color = "" #color from the theme settings
+++

It's trivial to add comments to a Hugo blog. I believe you can use Disqus, but for a simple dev blog I just want something lightweight and easy to use.

Enter [Utteranc.es](https://utteranc.es).

Utterances uses GitHub issues as a commenting system for your blog. This does mean that a GitHub account is required in order to set Utterances up or for readers to comment. I think that it is a safe assumption that most readers of dev/tech blogs have or should have a GitHub account so this doesn't really present a problem in my mind to requiring a GitHub account to comment.

## How to set up Utterances

1. sign into GitHub and make sure the repo you're going to use is public
1. add the [utterances app](https://github.com/apps/utterances) to your repo
1. fill out the [config wizard](https://utteranc.es/index#configuration) with the relevant info
1. copy the 8 line config the wizard provided based on your input
1. paste that info into `layouts/partials/comments.html` in your hugo dir (your theme may have additional config parameters to turn on comments)
1. commit your changes to git, rebuild your hugo blog, and boom, you'll have comments on your blog
