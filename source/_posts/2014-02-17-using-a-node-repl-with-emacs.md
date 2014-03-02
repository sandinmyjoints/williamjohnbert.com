---
layout: post
title: "Using a Node repl in Emacs with nvm and npm"
date: 2014-02-17 16:37
comments: true
categories: emacs node
---

Running a repl inside Emacs is often convenient for evaluating code, checking
syntax, and myriad other tasks. When I wanted to run a Node REPL, I found that I
needed to do a little set up to get everything working the way I wanted.

My first question was: which Node? With
[nvm](https://github.com/creationix/nvm), I've installed multiple version on my
machine. So I needed a way to specify one to execute.

Another question was: where to run Node? Since [npm](https://www.npmjs.org/)
looks inside `node_modules` directories starting with the current directory and
working up the file system hierarchy, the current working directory is
important. If I want access to the npm modules installed for project A, I need
to start my repl's Node process from `path/to/projectA`.

But that raises another question: what happens when I want to switch to project
B? Do I need to use `process.chdir()` to switch the Node repl's current working
directory to `path/to/projectB`? That's clumsy and annoying.

Here's how I answered these questions:

[nvm.el](https://github.com/rejeep/nvm.el) gives you `nvm-use` to activate a
   version of Node within Emacs. It's basically a nice wrapper around setting
   the enviroment variables `NVM_BIN` and `NVM_PATH` and adding the path to the
   Node version you want to use to your `PATH`. Great!

Except for one problem: `nvm-use` isn't interactive. It's meant to be use
programmatically. So I needed to write a small `do-nvm-use` wrapper that lets me
specify a version and then activate it:

```common-lisp
(require-package 'nvm)

(defun do-nvm-use (version)
  (interactive "sVersion: ")
  (nvm-use version)
  (exec-path-from-shell-copy-env "PATH"))
```

To specify where to run Node, I wrote another small defun, named `run-node`,
that prompts for a directory in which to start Node. Before it does this,
though, it checks whether a program named `node` is in the `exec-path`, and if
not, it runs `do-nvm-use` first. Once we have a Node to execute and a directory
to execute it in, we can make a new `comint` buffer bound to the repl process.

To address the issue of different repls needing to be run for different
projects, `run-node` adds the cwd to the buffer name. Repls for project A and
project B will live in buffers named `*-node-repl-path/to/projectA` and
`*-node-repl-path/to/projectB`, respectively&mdash;making switching to the right
buffer with [ido](http://www.emacswiki.org/emacs/InteractivelyDoThings) trivial.

```common-lisp
(defun run-node (cwd)
  (interactive "DDirectory: ")
  (unless (executable-find "node")
    (call-interactively 'do-nvm-use))
  (let ((default-directory cwd))
        (pop-to-buffer (make-comint (format "node-repl-%s" cwd) "node" nil "--interactive"))))
```

Now to start my Node repls, I just call `run-node` and I'm all set!
