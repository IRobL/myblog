---
layout: post
title:  "Text-Based ToDo Lists with Synchronization"
date:   2019-07-15 12:00:00 -0500
categories: productivity gist
---

Note: this blog was done in a rush, so you'll likely need to go to the readmes for the technologies discussed in order to get things installed 100% proper.  

I've recently become very fond of 3 technologies lately:

- [Basher](https://github.com/basherpm/basher) - A really simple way of packaging bash scripts.
- [gister](https://github.com/weakish/gister/) - A really simple way of synchronizing local changes to a gist with gist.github.com
- [git-crypt](https://github.com/AGWA/git-crypt) - A really simple way of seamlessly encrypting git repositories

## Part 1 - Gister


I want to blog about how great it is to use ~/todo as your todolist.  
I want to tell you how you can install [gister](http://github.com/weakish/gister/) with the simple command...

```
; basher install weakish/gister
```

You'll need to install basher first, it's a package manager for bash scripts.  

My preferred configurations for gister are...

```
echo "
export GISTER_AUTO_COMMIT=true
export GISTER_USE_HTTPS=true
" >> ~/.this_machine
```

From there, you can create a github gist called `todo`, and do...

```
gister sync
```

Now just turn your `~/todo` file into a todo symlink via copying it to your gister location and symlinking to it.

```
mv ~/todo ~/gister/tree/987sdf98798sdf/todo.txt
ln -s ~/gister/tree/987sdf98798sdf/todo.txt ~/todo
```

Now to synchronize changes you've made locally with gist.github.com, do...

```
gister sync
```


## Part 2 - Git-Crypt and Crypto Secured ToDo Lists

Ref: https://coderwall.com/p/kucyaw/protect-secret-data-in-git-repo

So if you don't trust the Internet (and why should you?) just use [git-crypt](https://github.com/AGWA/git-crypt) to encrypt your todo list repo right after you sync it down onto your local (before it has anything private installed yet, of course).  It's like this:

```
sudo apt-get install git-crypt
cd ~/gister/tree/987sdf98798sdf
git-crypt keygen ~/.ssh/crypt-key
git-crypt init ~/.ssh/crypt-key
*.txt filter=git-crypt diff=git-crypt
gister sync
```

Now you're ready to pull that repo down on your other PCs and have them synchronized!  Sweet!
