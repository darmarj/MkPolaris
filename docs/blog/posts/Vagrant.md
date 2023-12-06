---
date: 2022-09-28
authors: [darmarj]
description: >
  Vagrant
categories:
  - Vagrant
---

# Vagrant Plugin

## Background:
Due to the network in China is not good to go on plugin download and upgrade. Hence replace to local source is the better method.

## Workaround:
* [Rubygems](https://rubygems.org/)
* [Rubygems-China](https://gems.ruby-china.com/)
```bash
# Installation
# Download the relevant package from upon website
# Unpack into a dir and cd to the folder
# run following command for Installation
$ ruby setup.rb
```
* If the permission error message warning up, export ENV for tackle down.
```bash
$ export GEM_HOME=$HOME/.gem/ruby/3.1.0/
```

## Rubygems install the package
```bash
$ gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/

$ gem sources -l
https://gems.ruby-china.com

$ gem install vagrant-vbguest
```
