---
title: TIL - Using Ctags in Emacs
date: 2017-12-04 05:41:47
layout: post
path: "/posts/using-ctags-in-emacs/"
category: "Programming"
tags:
- ctags
- emacs
- dev environment
description: "I discovered Ctags and it changed my life. Setting it up was a little bit of a hassle though. Hopefully this will make it easier."
---

I've been using Emacs consistently for over a year and I love it. Recently a co-worker explained to me why he loved [RubyMine by Jet Brains](https://www.jetbrains.com/ruby/), because he could "step through" to a method definition if he encountered one he didn't know.

Just for reference, I've been grepping to do this in Emacs using `projectile-grep` to search the project. It's been an **OK** experience with that. So I was intruiged on how to do this for Emacs and I discovered how.

# What are Ctags

Ctags are basically an indexing of all method definitions in your entire project. It allows you to jump quickly between files and definitions fast. From what I understand this is a very "IDE-esque" feature. But it's been around long enough for editors like Emacs and Vim. If you want to see what the look like, they're very readable. Just open up the TAGS file that's generated after this tutorial and read through it. It's pretty easy to understand.

# The Process

1. Install Ctags using brew and set an alias to ensure it's called properly. 

```bash
$ brew install ctags
```

2. Set an alias in your .zshrc or .bashrc to ensure you execute the right Ctags program.
```
alias ctags="`brew --prefix`/bin/ctags"
```

3. Make a `~/.ctags` config file to manage Ctags generation globally.
```bash
$ touch ~/.ctags
```

Place these options inside:
```
--recurse=yes
--exclude=.git
--exclude=vendor/*
--exclude=node_modules/*
--exclude=db/* 
--exclude=log/*
```
> **Note: I actually dont have it to exclude the `db` directory in my personal setup. All of these are optional, most are good ideas.**

4. Globally ignore all Ctags files in **all** your projects.
```bash
$ echo "tags" >> ~/.global_ignore
$ git config --global core.excludesfile $HOME/.global_ignore
```

5. Head to the root directory of your project and create the tags.
```bash
$ cd path/to/project
$ ctags -e .
```

> **Note: This will generate Ctags for Emacs specifically. This setup will also work if you're using Vim, you'd simply just remove the `-e` flag from the above script.**

6. In Emacs, head to your project directory and find the table with `M-x visit-tags-table`. This will ask you to locate your tags file for the project. Make sure the directory shown is the directory of your `TAGS` file.

7. It may take a minute, but now you can hit up any function and call `M-x find-tag` and it will jump to that method definition (if there is one). 

# Todos

My next set of todos for this part of my setup, is to figure out how to open the Ctags in a new window split instead of just taking over the whole window everytime. If you figure that out, tweet me what you did!

# Additional Resources

A lot of this info was taken from the [Emacs Wiki](https://emacswiki.org):
- [Build Tags Wiki](https://www.emacswiki.org/emacs/BuildTags)
- [Emacs Tags Wiki](https://www.emacswiki.org/emacs/EmacsTags#tags)
- [Rico Sta. Cruz Blog on Setting up Ctags for Vim](https://ricostacruz.com/til/navigate-code-with-ctags)
