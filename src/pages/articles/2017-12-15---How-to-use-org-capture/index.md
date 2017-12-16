---
title: Org-Capture Basics
date: 2017-12-15 09:52:06
layout: post
path: "/posts/org-capture-basics"
category: "Programming"
tags:
- emacs
- orgmode
description: "I go over how org-capture works and how you can implement it to stay more productive. Tl;dr: It's a quick way to document things without ever leaving what your working on."
---
# How to use Org Capture

I love org-mode. It's made my life so much simpler. It allows me to capture quick ideas, todos, or notes without interrupting the flow of what I was doing. Today, I'm trying out org-capture for this very blog, and I'll explain to you how that happens. First, I'll do an ELI5 of org-capture.

## ELI5 Org-Capture

Org-capture is simply a way to capture thoughts, ideas, todos, anything really, very quickly but initiating a mini buffer with specific prompts that you define, to fill out an orgmode file automatically. Instead of having to mess around with whatever boilerplate you can just automate everything away except the content itself.

**Example:** Lets walk through creating a todo on my todolist.

1. I have to navigate to my Todolist file/Todo app
2. Create template however it's made to fall in line with other todos (for me it's a date heading with an `**` at the beginning of the line)
3. Enter in dates via manual typing or a calendar picker
4. Type out description
5. Save and exit

This doesn't sound too bad but, especially if you use a 3rd party app, it's just tedious to do, especially if you're in the middle of working on some code.

Here's the org-capture way

1. Hotkey to open todo capture
2. Type description
3. `C-c c` to save it

So that's 2 less steps, and it also all happens in a mini buffer in Emacs. So you don't lose the context of what your working on. It's really awesome.

## How it Works

There's a couple of things you have to setup in your Emacs config before this will work. 

You need to make sure you have org-mode installed in emacs. 

```emacs-lisp
  (use-package org
  :ensure t)
  
  ;; OR for evil-mode users

  (use-package org-evil
  :ensure t)
```

Then we have to set our templates up to handle this.

**Simple Todo template**
```emacs-lisp
(setq org-capture-templates
      '(("t" "To Do Item" entry (file+headline "~/orgfiles/todo.org" "ToDo")
         "* TODO %?\n%u" :prepend t)))
```

So this uses an Emacs dsl from what I can tell, to initiate the prompts and eventually just write straight to a file of your choosing. 

First you set the list that the variable `org-capture-templates`. Then you declare your org-capture shortcut, in this case `t`, then the Description of your template. Then you simply say to take the file and the section headline as arguments with `file+headline` then type the path to the file you want that template to go to, and the heading you want at the top of the section. Then there's the magic. The result looks something like this:

![](https://i.imgur.com/Enct2q3.png)

A full list of this dsl can be found [here in the Org-mode manual](http://orgmode.org/manual/Template-expansion.html#Template-expansion)

To be honest, I copied this template from [Mike Zamansky](http://cestlaz.github.io/) who has a [fantastic YouTube series](https://youtu.be/49kBWM3RQQ8) on Emacs.

## Current Use

Well I used org-capture to start this blog. I made a new template for blogs, which was indentical to my notes template but just had a different heading and filename. I org-capture'd my idea to write a blog about org-capture. Then later I went into the `blogs.org` file and immediately started writing the blog, using `#+BEGIN_SRC + #+END_SRC` for markdown. This is awesome cause I can see all my ideas and just write in the one big org file, and decide whether or not to post them.

Here's a screenshot of my progres so far.

![](https://i.imgur.com/i6tQ2Lf.png)

This also gives me a really easily searchable list of personal archives.

I have capture templates for Todos, Notes, Code Snippets, and now blogs.

## Additional Resources

- [My Dotfiles](https://github.com/juliusdelta/dotfiles) - Here you can see all my templates and my entire Emacs config.
- [Org Capture to Capture Code Snippets](http://200ok.ch/posts/org-mode-capture-template-for-code-snippets.html)
- [Org-mode Manual](http://orgmode.org/manual/)
