---
layout: post
title:  "Configure Node.js"
date:   2022-12-28 06:35:00 +0200
categories: jekyll update
---

## NPM Configuration

Before starting, it's a good idea to configure npm with your personal information. This will be used to populate the package.json file in your future projects.

```
npm config set init-author-name "username" -g
npm config set init-author-url "https://github.com/username" -g
npm config set init-license "MIT" -g
```

## Git Configuration

You'll also want to set up Git with your information and preferred settings.

```bash
git config --global init.defaultBranch main
git config --global user.email "email"
git config --global user.name "username"
```

Once your local project is ready, you can link it with a remote repository as follows:

```bash
git remote add origin https://github.com/username/REPOSITORY.git
git branch -M main
git push -u origin main
```