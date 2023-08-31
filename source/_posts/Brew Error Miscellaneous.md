---
layout: Brew Error Miscellaneous
title: Brew Error Miscellaneous
date: 2023-07-16 13:10:53
categories:
  - - 折腾记录
tags: 
  - Arm
  - Apple
  - 苹果
  - MacOS
  - Brew
---

## Brew Services Error

```shell
Error: uninitialized constant Homebrew::Service::System
/opt/homebrew/Library/Taps/homebrew/homebrew-services/cmd/services.rb:61:in `services'
/opt/homebrew/Library/Homebrew/brew.rb:94:in `<main>'
```

## Issue
- an unmaintained mirror

## Fix

```shell
git -C "$(brew --repo homebrew/services)" remote set-url origin https://github.com/Homebrew/homebrew-services.git
brew update
```