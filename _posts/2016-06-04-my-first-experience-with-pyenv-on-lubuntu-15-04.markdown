---
published: true
title: My first experience with PYENV on LUBUNTU 15.04
layout: post
tags: [ubuntu, pyenv]
categories: [dev, system]
---
*This posts explains how I made `pyenv` works on my virtual Lubuntu 15.04*

<!--excerpt-->

First of all, how to get find out Ubuntu version ? `lsb_release -a`

I mainly followed the info from [here](https://amaral.northwestern.edu/resources/guides/pyenv-tutorial)

Install GIT with `sudo apt-get install git-core`

Execute command:

~~~
cd
git clone git://github.com/yyuu/pyenv.git .pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc
~~~

then 

`curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash`

If the previous command is failing because of certificates, add the following lines in file `.curlrc`

~~~
capath=/etc/ssl/certs
cacert=/etc/ssl/certs/ca-certificates.crt
~~~

If the `curl` command is working, `pyenv` should be installed. You can verify this with

~~~
~$ pyenv global
system
~$ pyenv versions
* system (set by /home/staff/jmoreira/.pyenv/version)
~~~

If you have this output it basically means you have just a system-wide python version

Let's install a PYTHON version in your `home directory` (remember we did a `cd` before initiating any command)

`pyenv install 3.4.0`

This command will probably fail. There's a log file generated under `tmp`with name `python-build.*`

If you look at the log file, `curl` is looking a a certificate that does not exist, let's create a sym link to point to the right one

~~~
sudo su -
mkdir -p /tmp/pki/certs
ln -s /etc/ssl/certs/ca-certificates.crt  /etc/pki/tls/certs/ca-bundle.crt
~~~

Then try again `pyenv install 3.4.0`. This time it should work

Now if you can see 2 different version

~~~
pyenv versions
* system (set by /home/simon/.pyenv/version
3.4.0
~~~

To switch of version globally execute `pyenv global 3.4.0`

You can check then your python version with

~~~
python
>>>import sys
>>>sys.version_info
sys.version_info(major=3, minor=4,micro=0 blah blah blah)
~~~

