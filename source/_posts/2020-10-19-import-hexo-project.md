---
layout: post
title:  "Importing Hexo Project"
date:   2020-10-19 11:38:54 +0900
categories: blog
---

### Creating the directory.

let's create the directory which is used the root directory of your hexo project.

### Pulling your git repository.

```
    > git init
    > git remote add origin [YOUR GIT REPOSITORY]
    > git pull origin master
```

You just initialize the git in your hexo project and connect with you remote repository. and finally pulling the codes.

### Installing node modules.

You should install the packages for using hexo. I think the two commands would operate well each alone. I just always do these both.

```
    > npm install
    > yarn
```

### Install Hexo module from npm.

```
    > npm install hexo-cli -g
```


### Downloading your own hexo theme

If you use the hexo theme, then download it from somewhere and copy and paste to 'directory/themes/' and You should check the 'config.yml' file name whether it is needed transition or not.

```
    > mv [YOUR THEME] ./directory/themes
    > delete '.example' at 'config.yml.example'
```

### Launching local Hexo server

Let's check your thing is operated well or not.

```
    > hexo server
```

### Installing Git deploy package.

You gotta need to install 'hexo-deployer-git' package to upload your hexo project to remote using git.

```
    > npm install --save hexo-deployer-git
```

### Uploading Hexo

```
    > hexo deploy --generate
```

### Git source upload

```
    > git add .
    > git commit -m "COMMENT"
    > git push -u origin master
```

You should know one thing that the real source code and hexo project are sperated. so you gotta push both two repositories.