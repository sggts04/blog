---
layout: post
title: "Deploying your static Jekyll Site: Netlify vs Github Actions vs Travis CI"
date: 2021-10-21T12:28:03+05:30
authors: ["Shreyas Gupta"]
categories: ["Github", "Jekyll", "Hosting/Deployment"]
description: When hosting your own static site/blog with Jekyll, you'll have many options on how to implement continuous deployment for it, here we take a look at three of the popular options - Netlify, Github Actions with Pages, and Travis CI with Pages
thumbnail: "https://www.aleksandrhovhannisyan.com/assets/images/posts/github-pages-vs-netlify/thumbnail-1280.jpeg"
image: "https://www.aleksandrhovhannisyan.com/assets/images/posts/github-pages-vs-netlify/thumbnail-1280.jpeg"
---

As I started to set up my blog, I was certain on using a Jekyll template given how simple and popular it was for such a usecase. However what I was not clear on was how exactly to setup the deployment for it. There are certain templates like [Jekyll Now](https://github.com/barryclark/jekyll-now) which work out of the box with Github Pages and don't require a deployment pipeline, however **most good templates** like [TeXt Theme](https://github.com/kitian616/jekyll-TeXt-theme) **require you to continously build and deploy the site**. This was also the case with the template I was considering using, which is the one you're viewing right now.

So the question arrived, how should I setup continuous deployment for my blog? There were a few recommended options
- Netlify Deployment
- Github Pages Deployment with Github Actions
- Github Pages Deployment with Travis CI

First thing you need to think about is which hosting service you want to use. I was pretty set on using Github Pages since most of my projects are hosted through it, however I later realized the deployment process for what I was looking to do is much much easier with Netlify. So lets look at the options you have and the effort you'll need for each one.

## Netlify

Netlify is quite literally the most straightforward tool for our usecase, the configuration file is the smallest and easiest to understand aswell. Here's the `netlify.toml` file that you'll need in your repository to have Netlify do a Jekyll deployment for you:

```toml
[build]
  command = "bundle exec jekyll build"
  publish = "_site"

[build.environment]
  NODE_VERSION = "14"
  NPM_VERSION = "7"
  JEKYLL_ENV = "production"
  NODE_ENV = "production"
```

Now just go to Netlify, connect your Github Account and start a project for your Jekyll Repo. Netlify will automatically pickup all the details using this config file and instantly begin with the continous deployment!

![](https://c.tenor.com/lNL7iBKHQuIAAAAC/that-all-snl.gif)

Just like Github Pages, you can specify a custom domain, setup HTTPS for free, yada yada yada. This blog is being deployed through Netlify aswell! I setup a `blog` CNAME in my GoDaddy DNS Management to point to the original Netlify URL, added `blog.shreyasgupta.in` as a custom domain in Netlify project settings, and it was all done! Netlify setup the HTTPS pretty much instantly aswell.

**Set on using Github Pages instead?** Don't worry, I got you.

## Github Pages: with Travis CI

Now Github Pages is pretty barebones and doesn't support such deployment pipelines out of the box. So we must integrate a CI/CD tool in the middle to take care of this. Popular options being Travis CI or Github's very own Github Actions. Lets take a look at **Travis CI** first.

> Travis CI is a hosted continuous integration service used to build and test software projects hosted on GitHub.

Its one of the most popular choices when it comes to setting up CI pipelines, and its free for public repositories for basic use.

As prerequisites:
- You'll have to [generate a Github Personal Access Token](https://github.com/settings/tokens/new) to give repo permissions to your Travis Pipeline.
- Make your dependencies Travis compatible through `bundle lock --add-platform x86_64-linux`
- Specify the ruby version for Travis to use through `echo "2.5.3" > .ruby-version`

Finally, for Travis CI, your `.travis.yml` configuration file would look something like this:

```yml
language: ruby

cache: bundler

script:
  - rm -rf _site
  - gem install jekyll bundler
  - bundle install
  - JEKYLL_ENV=production bundle exec jekyll build
  - find . -mindepth 1 -maxdepth 1 -not -name _site -not -name .git -exec rm -rf '{}' \; # delete all files and folders excluding the `.git` and `_site` folder
  - find _site -mindepth 1 -maxdepth 1 -exec mv '{}' . \; # move all files and folders in the `_site` folder to the root
  - rm -rf _site # delete the site folder

deploy
  provider: pages
  skip_cleanup: true
  github_token: $GH_TOKEN # Environment variable key which stores your GitHub Personal Access Token
  keep_history: false # do a force push
  on:
    branch: master
```
*Credits for this config file goes to [Ayush Gupta](https://ayushgupta.me/blog/2021/10/20/deploy-jekyll-to-github-pages-using-travis-ci.html#create-your-travis-configuration-file)*

Now just open Travis, create a project and follow the default steps. All your commits will get built to `gh-pages` branch and you can setup Github Pages to use that branch for hosting. Here the custom domain/HTTPS features will be configured through Github Pages settings in your repo.

A few extra steps but decent option if you're set on using Github Pages.

## Github Pages: with Github Actions

Now lets take a look at Github's very own CI/CD tool **Github Actions** to attempt the same.

> GitHub Actions makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub.

Github Actions is a newer tool compared to Travis CI but has been gaining a lot of traction lately, and is a good alternative if you just want to stay in Github ecosystem.

As prerequisites:
- You'll have to [generate a Github Personal Access Token](https://github.com/settings/tokens/new) to give repo permissions to your Actions Pipeline.
- Go to your repository settings, and then the Secrets tab, and add the above generated token to a new secret named `GITHUB_TOKEN`.

We'll use [Jekyll Actions](https://github.com/marketplace/actions/jekyll-actions) from the marketplace to set up our action easily. Here's how our `.github/workflows/github-pages.yml` configuration file will look like:

```yml
name: Deploy Jekyll site to GitHub Pages

on:
  push:
    branches:
      - master

jobs:
  github-pages:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: helaili/jekyll-action@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

Aaaand we're done! All your commits will be instantly built and deployed to Github Pages.

I felt the Github Actions flow was more straight forward than the Travis CI flow, pertaining to the fact that you don't have to create an account and pipeline on a new site. But if you're already familiar with using Travis CI or want to get into CI/CD, then Travis can be a good option for you!

## Conclusion

So, in conclusion, we looked at three different methods to setup continous deployment for our Jekyll Site. If you just want to setup something quickly, then Netlify is the best and easiest option. Else, if you're set on using Github Pages and want a lot more control on your deployment pipeline, consider choosing the Travis flow or the Github Actions flow.

Thanks for reading and hope this helped you!

![](https://i.imgur.com/zmxTNdx.png)



