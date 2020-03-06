---
title: "Installation"
date: 2017-10-17T15:26:15Z
draft: false
weight: 10
---

## Download Goproject binary

You will  need to download the latest binary from github. This can be done from a web browswer or the command line with wget.

```
wget https://github.com/jjunqueira/goproject/releases/download/v0.0.2/goproject-darwin
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

### Dependencies
Some of the templates use [golangci-lint](https://github.com/golangci/golangci-lint). This means that you should probably install it if you intend to use even just the Basic template. Installation is simple and documented [here](https://github.com/golangci/golangci-lint#install). For OSX users you can quickly install it via homebrew:

```
brew install golangci/tap/golangci-lint
brew upgrade golangci/tap/golangci-lint
```
