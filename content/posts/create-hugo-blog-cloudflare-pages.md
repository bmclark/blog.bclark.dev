+++
title = "How to create a Hugo blog that deploys to Cloudflare Pages"
date = "2023-08-06T12:13:05-04:00"
author = "Bryan Clark"
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

I finally decided to just get a dev blog going quickly within a git repo that is automatically deployed when I push a new post/update. This was so easy to setup I can't believe I didn't do it sooner. I'll eventually move it over to a more "devopsy" pipeline on my own servers, but this is so easy to just get started writing articles I don't think anybody should wait to follow these steps.

There is an extra step if you're using Gitea instead of GitHub or GitLab to host your repo. It's not very difficult, but Cloudflare Pages only interacts with GitHub and GitLab at the moment. This requires us to mirror our Gitea repo to GitHub or GitLab and I've included those steps.

## Install Hugo / create new site

First, follow the steps for your OS to install hugo[^1].

Then run the commands to create your site. Substitute placeholders for your specific case:

```bash
BLOG_URL="blog.example.com"
hugo new site $BLOG_URL
cd $BLOG_URL
```

## Setup Git Repo

I use Gitea[^2] as my personal git server and then mirror certain repos to GitHub. First we'll create a repo (but don't initialize it) in Gitea or GitHub or GitLab. We'll name the repo whatever the blog's URL will be: `blog.example.com`. Once it's created we'll follow the steps to initialize the repo.

```bash
REPO_URL="git@github.com:example/blog.example.com"
touch README
touch LICENSE
git init
git remote add $REPO_URL
git add .
git commit -m "initial commit"
git push
```

## Mirror Gitea repo to GitHub

If you aren't using Gitea you can [skip to the next step](#update-hugo-config-and-theme). These steps describe how to mirror to GitHub, but it's fairly similar on GitLab and the process is documented on the same Gitea documenation page.

If you are using Gitea, follow these steps[^3] in order to be able to push to GitHub or GitLab. This means creating a repo on GitHub with the same name we used for our Gitea repo. Again, remember to not initialize the repo, only create it. Then creating a Personal Access Token[^4] by going into Settings > Developer Settings > Personal Access Tokens > Tokens (Classic) > Generate New Token (Classic). Use the repo name for the Note and set an expiration date. Ensure that `public_repo` box is checked for the scope. Then copy the PAT that is provided in the next screen.

Then, in your Gitea repo, add a `Git Remote Repository URL` with the GitHub repo's URL. For Authorization use your username and the PAT from GitHub for your password. Then select `Sync when new commits are pushed` and select `Add Push Mirror` to save the config. You'll need to manually force a push by selecting `Synchronize Now`. Now you should see your initial commit mirrored in your GitHub repo.

## Update Hugo config and theme

Next we'll go ahead and add our theme. I'm just using the basic one mentioned in the Cloudflare Pages walkthrough[^5], `terminal`. To add the theme run these commands:

```bash
git submodule add https://github.com/panr/hugo-theme-terminal.git themes/terminal
git submodule update --init --recursive
```

Next we'll update the configuration[^6]. I took the base configuration example from my theme and since it was in `toml` format and I wanted it in `yaml` I had ChatGPT[^7] rewrite it in `yaml` and then I updated it with my values.

```yaml
# Base URL
baseurl: "/"

# Language code
languageCode: "en-us"

# Add it only if you keep the theme in the `themes` directory.
# Remove it if you use the theme as a remote Hugo Module.
theme: "terminal"

# Pagination limit
paginate: 5

params:
  # Dir name of your main content (default is `content/posts`).
  # The list of set content will show up on your index page (baseurl).
  contentTypeName: "posts"

  # Theme color: "orange", "blue", "red", "green", "pink"
  themeColor: "orange"

  # If you set this to 0, only submenu trigger will be visible
  showMenuItems: 2

  # Show selector to switch language
  showLanguageSelector: false

  # Set theme to full screen width
  fullWidthTheme: false

  # Center theme with default width
  centerTheme: true

  # If your resource directory contains an image called `cover.(jpg|png|webp)`,
  # then the file will be used as a cover automatically.
  # With this option you don't have to put the `cover` param in a front-matter.
  autoCover: true

  # Set post to show the last updated
  # If you use git, you can set `enableGitInfo` to `true` and then post will automatically get the last updated
  showLastUpdated: false

twitter:
  # Set Twitter handles for Twitter cards
  # Do not include @
  creator: ""
  site: ""

languages:
  en:
    params:
      languageName: "English"
      title: "Terminal"
      subtitle: "A simple, retro theme for Hugo"
      owner: ""
      keywords: ""
      copyright: ""
      menuMore: "Show more"
      readMore: "Read more"
      readOtherPosts: "Read other posts"
      newerPosts: "Newer posts"
      olderPosts: "Older posts"
      missingContentMessage: "Page not found..."
      missingBackButtonLabel: "Back to home page"
      logo:
      logoText: "Terminal"
      logoHomeLink: "/"
    menu:
      main:
        - identifier: "about"
          name: "About"
          url: "/about"
        - identifier: "showcase"
          name: "Showcase"
          url: "/showcase"

```

## Configure Cloudflare Pages

Now we'll set up the Cloudflare portion. Following along with the official guide[^5], we'll navigate from the Cloudflare Dashboard to Workers & Pages > Create application > Pages > Connect to Git. Authorize the GitHub/Cloudflare connection and select the blog repo. Select `Hugo` as the `Framework Preset` and your build commands will be filled out for you. Be sure to add the `HUGO_VERSION` environment variable set to a recent version, `0.116.1` at the time of this article.

Select Custom Domain and configure the pages to your custom domain already registered with Cloudflare `blog.example.com`. Cloudflare automatically configures everything to point to the Cloudflare Pages hugo build subdomain.

## Push first article

Finally, we'll create a `Hello World` blog post.

```bash
hugo new posts/hello-world.md
echo "Hello world!" >> content/posts/hello-world.md
git add content/posts/hello-world.md
git commit -m "initial hello-world post"
git push
```

We should now be able to navigate to `blog.example.com` and see our first post within seconds. We can also take a look at the builds and watch them while they are in progress in our Cloudflare account.

Next, you can continue on to adding comments to the blog: [Add Comments to a Hugo Blog Using Utterances]({{< ref "add-utterances-hugo-blog" >}} "Add Comments to a Hugo Blog")

## References

[^1]: [Hugo Installation](https://gohugo.io/installation/)
[^2]: [Gitea](https://gitea.com)
[^3]: [Gitea Repo Mirror Setup](https://docs.gitea.com/usage/repo-mirror#pushing-to-a-remote-repository)
[^4]: [Github Personal Access Token Setup](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
[^5]: [Official Cloudflare Pages Hugo Guide](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site/)
[^6]: [Hugo Config Getting Started](https://gohugo.io/getting-started/configuration/)
[^7]: [ChatGPT](https://chat.openai.com)
