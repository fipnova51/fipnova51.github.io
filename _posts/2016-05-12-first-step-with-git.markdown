---
published: true
title: First step with git -- install and basic commands
layout: post
tags: [git]
categories: [dev]
---
*How does git work? this note is about my first step with using git on Lubuntu. The topics go from installing git to the basic command and maybe branching*

<!--excerpt-->

## Installing git

If `git` is not installed on your system -- in our case Lubuntu -- then let's install it with

    sudo apt-get install git-core

### Minimal git config

there are plenty of configuration parameters at different level:
* system level
* user level
* session level

configuration at user level are saved in file `~/.gitconfig` or `~/.config/git/config`. 

This file is modified with command `git config --global <parameter>` . 

Example: I want to set my name

    git config --global user.name 'Simon Souv'

If I want to display a parameter, execute `git config <parameter>` 

<pre style='color:#000000;background:#ffffff;'>simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#797997; '>$</span> git config user<span style='color:#800000; font-weight:bold; '>.</span>name
Simon Souv
</pre>

### Getting help

there are 3 ways to get some information about a command

~~~
git help <command>
get <command> --help
man git <command>
~~~

## Basic commands

### repository initialization

#### initialize a repository with an existing directory

~~~
cd <your directory>
git init
~~~
your directory will contain a sub-directory called `.git` containing the git repository skeleton. Let's add afew files to initiate an initial commit


