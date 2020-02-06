---
title: "Installation"
date: 2020-02-06T15:26:15Z
draft: false
weight: 10
---

## Download Goproject binary

You will  need to download the latest binary from github. This can be done from a web browswer or the command line with wget.

```
wget https://github.com/jjunqueira/goproject/releases/download/v0.0.1/goproject-darwin
```

## Configure

In order to use the binary you must make it executable and then move it into a folder that is in your $PATH

```
chmod +x goproject-darwin
cp goproject-darwin /usr/local/bin/goproject
```

## First time use

Before you can use goproject you must initialize it. The initialization process will create the required directories and download the default project templates.

```
goproject init
```