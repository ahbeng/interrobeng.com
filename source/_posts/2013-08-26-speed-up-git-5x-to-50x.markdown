---
layout: post
title: "Speed Up Git (5x to 50x)"
date: 2013-08-25 23:23
comments: true
categories: CS3216
---
**Note:** Results may vary, depending on distance from your Git servers. In my
completely unscientific benchmarks using `time`, after the following steps,
`git pull` went from ~5s, using GitHub, to ~0.1s, using EC2 on AWS Singapore.

### Why?

``` console
$ time git pull
Already up-to-date.

real    0m5.075s
```
5 seconds just to tell you that your Git repository is already up-to-date?
**Unacceptable.**

### Enable SSH Connection Sharing and Persistence

In Singapore, the round-trip time to github.com is ~250ms. Establishing an SSH
connection every time you perform a Git operation costs many round-trips, but
you can keep them around and reuse them with the following lines in
`~/.ssh/config`:

``` aconf ~/.ssh/config
ControlMaster auto
ControlPath /tmp/%r@%h:%p
ControlPersist yes
```

`ControlMaster auto` enables the sharing of multiple SSH sessions over a single
network connection, and auto-creating a master connection if it does not already
exist.

`ControlPath /tmp/%r@%h:%p` specifies the path to the control socket used for
connection sharing. `%r` will be substituted by the remote login username, `%h`
by the target host name and `%p` by the port.

`ControlPersist yes` keeps the master connection open in the background
indefinitely.

With this, `git pull` goes down to ~1s, a 5x improvement. Of course, this speeds
up your other repeated SSH connections too!

### Setup Git Mirror with Automatic Mirroring to GitHub

For sub-second Git remote operations, you're going to need a server with low
latency. (Imagine, when physically working together, you could `git pull` the
very moment someone says "Pushed!".)

However, you can still have all the niceties GitHub provides, by automatically
pushing to GitHub from your server. There are several alternatives for keeping
GitHub updated, like using cron jobs or multiple remotes, but I think using a
post-receive hook is the most elegant solution.

On your server, set up a GitHub
[deploy key](https://help.github.com/articles/managing-deploy-keys#deploy-keys):

``` console
$ ssh-keygen -t rsa -C "your_email@example.com"
$ cat .ssh/id_rsa.pub
```

Go to your target repository's Settings -> Deploy Keys -> Add deploy key and
paste the public key in and submit. Then you can mirror your repository:

``` console
$ git clone --mirror git@github.com:ahbeng/example.git
```

To automatically to mirror to GitHub after you've pushed, set up a
[Git hook](http://git-scm.com/book/en/Customizing-Git-Git-Hooks) at
`hooks/post-receive`:

``` bash hooks/post-receive
#!/bin/bash
nohup git push --mirror &>/dev/null &
```

Your Git client won't disconnect till the script has completed, so a simple
`git push --mirror` would defeat the purpose of setting this up, since you'd be
adding on GitHub's latency to your pushes again. So, we background the process
and keep it running after logging out using
[nohup](http://en.wikipedia.org/wiki/Nohup), and redirect all I/O streams to
`/dev/null` to prevent SSH from hanging on logout.

Make the hook executable:

``` console
$ chmod +x hooks/post-receive
```

Now you can use your new Git mirror locally. Assuming you have added the
server's private key to ssh-agent:

``` console
$ ssh-add ec2.pem
```

You could clone it:

``` console
$ git clone user@hostname:example
```

Or change your existing repository's remote:

``` console
$ git remote set-url origin user@hostname:example
```

### Results

Taken alone, using the Git mirror brings `git pull` down to ~1.2s. When combined
with SSH connection sharing:

``` console
$ time git pull
Already up-to-date.

real    0m0.111s
```

0.1s Git remote operations. Awesome.
