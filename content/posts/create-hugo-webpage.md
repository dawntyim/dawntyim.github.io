---
title: "Create Hugo Webpage"
date: 2023-01-08T22:39:37-08:00
draft: true
author: "Dawn Yim"
summary: ""
tags: ["hugo", "static web", "github.io"]
hideMeta: false
searchHidden: false
ShowBreadCrumbs: false
---
I used Hugo framework to build this github io blog page. There are many pretty themes you can use and customizing and deploying the project only took a few hours. Here's my takeaways on this tiny toy project. 

# Creating Hugo Static Webpage
Starting off your project by following this step in the official webpage. 
https://gohugo.io/getting-started/quick-start/

## Writing Config File

### Base URL
Setting the correct base url is most important thing. Do not skip http or https prefix. 


### Multi-language
Your static web page can support multiple languages and each language can have individual page setup.

The config.toml can be written like
```toml
languageCode = 'en-us'
defaultContentLanguage = 'en'

[languages]
  [languages.ko]
    languagedirection = 'korean'
    title = "Dawn Yim's Blog"
    weight = 2
  [languages.en]
    title = "Dawn Yim's Blog"
    weight = 1
```

The content markdown files need to be set either the default language or other languages. I can write a content in Korean and expose that in Korean version of the page by adding '.ko.' before md extension. The Korean version of `example-post.md` in contetn/posts folder will be `example-post.ko.md`.

### Theme
You need to specify the theme you are using in config.toml file and add a submodule to your project. I'm using PaperMod theme for my project. 

Add theme setting in the config.toml file
```toml
theme = 'PaperMod'
```

And download the submodule using this command in your terminal. `git submodule add <repo_url> <target_folder>`. This will add .gitsubmodules file in your project and will be used when deploying the project. 
```
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

### Adding Features
You can use config file to add features supported by Hugo such as categorization or search. I added archive, tags, and search features. Here's what to add in the config file. 

```toml
[menu]
    [[menu.main]]
    name = "Archive"
    url = "archives"
    weight = 5
    
    [[menu.main]]
    name = "Search"
    url = "search/"
    weight = 10

    [[menu.main]]
    name = "Tags"
    url = "tags/"
    weight = 10
```

## Deploying to Github io
I deployed this project to my github.io page. Github page deployment requires Github Actions run by default. Github Actions is an automated script that will run based on the conditions you set. You can simply use the Github Action template provided by the [official document's deployment section](https://gohugo.io/hosting-and-deployment/hosting-on-github/). 

Create 'gh_pages.yml' (or different name) file in .github/workflows (this can't be changed) folder. This `on` condition means every time there's a push to the listed branches, the action will be run. 
```yml
on:
  push:
    branches:
      - main  # Set a branch that will trigger a deployment
  pull_request:
```
Once the script is run, you can check that in the 'Actions' tab in your repository. This action pushes the changes into 'gh_pages' branch and in the branch you will see output in the public folder of your main branch are pushed to that branch. This one contains the frontend files generated from your content markdown files and the theme. 

To complete the deployment, go to settings -> pages and check 'Deploy from a branch' to deploy from 'gh_pages' branch. You will see `Your site is live at https://dawntyim.github.io/` message on the top of the page. 