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

<pre style='color:#000000;background:#ffffff;'>git add <span style='color:#808030; '>*</span>py
git add README<span style='color:#800000; font-weight:bold; '>.</span>txt
git commit -m <span style='color:#0000e6; '>'initial project version'</span>
<span style='color:#808030; '>[</span><span style='color:#0000e6; '>master (root</span><span style='color:#808030; '>-</span><span style='color:#0000e6; '>commit) 676d2e4</span><span style='color:#808030; '>]</span> initial project version
 <span style='color:#008c00; '>2</span> files changed, <span style='color:#008c00; '>4</span> insertions<span style='color:#800080; '>(</span>+<span style='color:#800080; '>)</span>
 create mode <span style='color:#008c00; '>100644</span> README<span style='color:#800000; font-weight:bold; '>.</span>txt
 create mode <span style='color:#008c00; '>100644</span> script<span style='color:#800000; font-weight:bold; '>.</span>py
</pre>
<br/>

#### Cloning an existing repository

~~~
git clone <url> <new working directory name> #by fedault it will the same as the repository name ot the URL
~~~
example
~~~
<pre style='color:#000000;background:#ffffff;'>simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#40015a; '>/git_workshop</span><span style='color:#797997; '>$</span> git clone https<span style='color:#808030; '>:</span><span style='color:#40015a; '>/</span><span style='color:#40015a; '>/github.com/libgit2/libgit2</span>
Cloning into <span style='color:#0000e6; '>'libgit2'</span><span style='color:#800000; font-weight:bold; '>.</span><span style='color:#800000; font-weight:bold; '>.</span><span style='color:#800000; font-weight:bold; '>.</span>
remote<span style='color:#808030; '>:</span> Counting objects<span style='color:#808030; '>:</span> <span style='color:#008c00; '>69685</span>, <span style='color:#800000; font-weight:bold; '>done</span><span style='color:#800000; font-weight:bold; '>.</span>
remote<span style='color:#808030; '>:</span> Compressing objects<span style='color:#808030; '>:</span> <span style='color:#008c00; '>100</span>% <span style='color:#800080; '>(</span><span style='color:#008c00; '>20108</span><span style='color:#40015a; '>/20108</span><span style='color:#800080; '>)</span>, <span style='color:#800000; font-weight:bold; '>done</span><span style='color:#800000; font-weight:bold; '>.</span>
remote<span style='color:#808030; '>:</span> Total <span style='color:#008c00; '>69685</span> <span style='color:#800080; '>(</span>delta <span style='color:#008c00; '>48291</span><span style='color:#800080; '>)</span>, reused <span style='color:#008c00; '>69630</span> <span style='color:#800080; '>(</span>delta <span style='color:#008c00; '>48236</span><span style='color:#800080; '>)</span>, pack-reused <span style='color:#008c00; '>0</span>
Receiving objects<span style='color:#808030; '>:</span> <span style='color:#008c00; '>100</span>% <span style='color:#800080; '>(</span><span style='color:#008c00; '>69685</span><span style='color:#40015a; '>/69685</span><span style='color:#800080; '>)</span>, <span style='color:#008c00; '>33</span><span style='color:#800000; font-weight:bold; '>.</span><span style='color:#008c00; '>01</span> MiB <span style='color:#e34adc; '>|</span> <span style='color:#008c00; '>1</span><span style='color:#800000; font-weight:bold; '>.</span><span style='color:#008c00; '>20</span> <span style='color:#40015a; '>MiB/s</span>, <span style='color:#800000; font-weight:bold; '>done</span><span style='color:#800000; font-weight:bold; '>.</span>
Resolving deltas<span style='color:#808030; '>:</span> <span style='color:#008c00; '>100</span>% <span style='color:#800080; '>(</span><span style='color:#008c00; '>48291</span><span style='color:#40015a; '>/48291</span><span style='color:#800080; '>)</span>, <span style='color:#800000; font-weight:bold; '>done</span><span style='color:#800000; font-weight:bold; '>.</span>
Checking connectivity<span style='color:#800000; font-weight:bold; '>.</span><span style='color:#800000; font-weight:bold; '>.</span><span style='color:#800000; font-weight:bold; '>.</span> <span style='color:#800000; font-weight:bold; '>done</span><span style='color:#800000; font-weight:bold; '>.</span>
</pre>
<br/>
~~~

### Get the status of your files
<pre style='color:#000000;background:#ffffff;'><span style='color:#bb7977; font-weight:bold; '>cd</span> <span style='color:#e34adc; '>&lt;</span>your_working_directory<span style='color:#e34adc; '>></span>
git status
On branch master
Your branch is up-to-date with <span style='color:#0000e6; '>'origin/master'</span><span style='color:#800000; font-weight:bold; '>.</span>
nothing to commit, working directory clean
</pre>
<br/>

what happens if we add a file? `git status` will inform us of an untracked file
<pre style='color:#000000;background:#ffffff;'>simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#40015a; '>/git_workshop/libgit2</span><span style='color:#797997; '>$</span> <span style='color:#bb7977; font-weight:bold; '>echo</span> <span style='color:#0000e6; '>'My project'</span> <span style='color:#e34adc; '>></span> README
simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#40015a; '>/git_workshop/libgit2</span><span style='color:#797997; '>$</span> git status
On branch master
Your branch is up-to-date with <span style='color:#0000e6; '>'origin/master'</span><span style='color:#800000; font-weight:bold; '>.</span>
Untracked files<span style='color:#808030; '>:</span>
  <span style='color:#800080; '>(</span>use <span style='color:#0000e6; '>"git add &lt;file>..."</span> to include <span style='color:#800000; font-weight:bold; '>in</span> what will be committed<span style='color:#800080; '>)</span>

	README

nothing added to commit but untracked files present <span style='color:#800080; '>(</span>use <span style='color:#0000e6; '>"git add"</span> to track<span style='color:#800080; '>)</span>
</pre>
### Tracking new files
any untracked files wont be part of any `commit` snapshot therefore you must `stage` the file (`add` command)
<pre style='color:#000000;background:#ffffff;'>simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#40015a; '>/git_workshop/libgit2</span><span style='color:#797997; '>$</span> git add README
simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#40015a; '>/git_workshop/libgit2</span><span style='color:#797997; '>$</span> git status
On branch master
Your branch is up-to-date with <span style='color:#0000e6; '>'origin/master'</span><span style='color:#800000; font-weight:bold; '>.</span>
Changes to be committed<span style='color:#808030; '>:</span>
  <span style='color:#800080; '>(</span>use <span style='color:#0000e6; '>"git reset HEAD &lt;file>..."</span> to unstage<span style='color:#800080; '>)</span>

	new file<span style='color:#808030; '>:</span>   README
</pre>
<br/>
### Staging modified files
Let's modify an already tracked file and see how `git status` behaves
<pre style='color:#000000;background:#ffffff;'>simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#40015a; '>/git_workshop/libgit2</span><span style='color:#797997; '>$</span> vi CONTRIBUTING<span style='color:#800000; font-weight:bold; '>.</span>md 
simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#40015a; '>/git_workshop/libgit2</span><span style='color:#797997; '>$</span> git status
On branch master
Your branch is up-to-date with <span style='color:#0000e6; '>'origin/master'</span><span style='color:#800000; font-weight:bold; '>.</span>
Changes to be committed<span style='color:#808030; '>:</span>
  <span style='color:#800080; '>(</span>use <span style='color:#0000e6; '>"git reset HEAD &lt;file>..."</span> to unstage<span style='color:#800080; '>)</span>

	new file<span style='color:#808030; '>:</span>   README

Changes not staged <span style='color:#800000; font-weight:bold; '>for</span> commit<span style='color:#808030; '>:</span>
  <span style='color:#800080; '>(</span>use <span style='color:#0000e6; '>"git add &lt;file>..."</span> to update what will be committed<span style='color:#800080; '>)</span>
  <span style='color:#800080; '>(</span>use <span style='color:#0000e6; '>"git checkout -- &lt;file>..."</span> to discard changes <span style='color:#800000; font-weight:bold; '>in</span> working directory<span style='color:#800080; '>)</span>

	modified<span style='color:#808030; '>:</span>   CONTRIBUTING<span style='color:#800000; font-weight:bold; '>.</span>md
</pre>
Let's add file `CONTRIBUTING.md` to the bunch of files for the next commit
<pre style='color:#000000;background:#ffffff;'>simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#40015a; '>/git_workshop/libgit2</span><span style='color:#797997; '>$</span> git add CONTRIBUTING<span style='color:#800000; font-weight:bold; '>.</span>md 
simon@simon-VirtualBox<span style='color:#808030; '>:</span>~<span style='color:#40015a; '>/git_workshop/libgit2</span><span style='color:#797997; '>$</span> git status
On branch master
Your branch is up-to-date with <span style='color:#0000e6; '>'origin/master'</span><span style='color:#800000; font-weight:bold; '>.</span>
Changes to be committed<span style='color:#808030; '>:</span>
  <span style='color:#800080; '>(</span>use <span style='color:#0000e6; '>"git reset HEAD &lt;file>..."</span> to unstage<span style='color:#800080; '>)</span>

	modified<span style='color:#808030; '>:</span>   CONTRIBUTING<span style='color:#800000; font-weight:bold; '>.</span>md
	new file<span style='color:#808030; '>:</span>   README
</pre>
