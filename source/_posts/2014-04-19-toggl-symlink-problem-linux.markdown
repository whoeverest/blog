---
layout: post
title: 'Fixing "Toggl" symlink problem on Linux'
date: 2014-04-19 12:00:00 +0100
comments: true
categories: programming
---

I started using [Toggl](http://toggl.com) recently and upon installing and symlinking it, I came to the following error:

```
:~/programs$ toggl 
terminate called after throwing an instance of 'std::logic_error'
  what():  basic_string::_S_construct null not valid
Aborted (core dumped)
```

Running the program normally with `./TogglDesktop` worked fine, but when using the symlink caused the error.

The problem was (as [Damjan](https://damjan.softver.org.mk/blog/) knew all along) that Toggl expected the path to itself as the first argument in `argv []`. This path was missing / wrong (don't know really) when the binary was symlinked.

To fix, I created a Bash script:

```
#!/bin/sh

~/programs/TogglDesktop/TogglDesktop
```

made is executable:

```
chmod +x run.sh
```

and symlinked it instead of the binary:

```
sudo ln -s ~/programs/Toggl/run.sh /usr/local/bin/toggl
```

I hope it helps someone.